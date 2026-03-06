# Rails 8 + AWS + Kamal + Terraform: Golden Spec

A proven pattern for deploying Rails 8 applications on AWS using Kamal for container orchestration and Terraform for infrastructure provisioning.

## Architecture Overview

```
Cloudflare DNS → EC2 (Kamal Proxy + Let's Encrypt SSL) → Docker (Thruster + Puma) → Aurora PostgreSQL
```

- **Single EC2 instance** running Docker, managed by Kamal
- **Terraform** provisions durable AWS resources (EC2, Aurora, ECR, S3, IAM, SES, security groups)
- **Kamal** handles container orchestration (build, push, deploy over SSH, optionally tunneled through SSM, zero-downtime)
- **AWS SSM Session Manager** handles operator access without exposing SSH publicly
- **No ALB, no ECS, no NAT Gateway, no Kubernetes**

## Why This Pattern

| | Kamal + EC2 (this pattern) | ECS Fargate |
|---|---|---|
| Deploy speed | ~60-80 seconds | 5-10 minutes |
| Monthly cost | ~$50-75 | ~$69+ (ALB $16, NAT $32) |
| Config complexity | ~50 lines deploy.yml | Hundreds of lines of Terraform |
| Debugging | `kamal console`, `kamal logs` | CloudWatch, no shell |
| Rails alignment | Ships with Rails 8, built by 37signals | Generic AWS service |

Rails 8's philosophy is "No PaaS Required" — Kamal + Solid Queue + Solid Cache means you don't need managed container orchestration for most apps.

## Minimum Instance Sizes

**These are hard-won lessons — do not use smaller instances.**

| Resource | Minimum | Why |
|----------|---------|-----|
| **EC2** | **t3.small** (2 vCPU, 2 GB RAM) | t3.micro (1 GB) will OOM with Rails + Docker + Solid Queue. We tested this extensively — the container gets OOM-killed, sometimes silently. |
| **Aurora Serverless v2** | **0.5 ACU minimum** | Setting min to 0 causes 10-15 second cold starts. 0.5 ACU keeps the instance warm (~$43/mo). |
| **RDS (budget option)** | **db.t4g.micro** | Works for small apps. db.t4g.small recommended if you have Solid Queue + Cache + Cable all active. |
| **EBS root volume** | **20 GB gp3** | Docker images + system packages fill up fast. No separate EBS volume needed — use S3 for file storage. |

## Directory Structure

```
project/
  config/
    deploy.yml                # Base Kamal config (shared: service, registry, builder, aliases)
    deploy.staging.yml        # Staging overrides (hosts, env vars, image name)
    deploy.production.yml     # Production overrides (hosts, env vars, image name)
  .kamal/
    deploy.env                # Instance IDs + ECR registry (gitignored — contains real values)
    deploy.env.template       # Template for deploy.env (committed — placeholder values)
    secrets-common            # ECR login command (shared across environments)
    secrets.staging           # Staging secrets fetched from AWS Secrets Manager
    secrets.production        # Production secrets fetched from AWS Secrets Manager
    hooks/
      pre-build               # Build Tailwind CSS locally (avoids QEMU issues)
  bin/
    deploy                    # Deployment wrapper script (builds CSS, logs into ECR, runs kamal)
    setup-deploy              # First-time setup script (kamal setup instead of kamal deploy)
  terraform/
    modules/app/
      main.tf                 # Provider requirements, locals (name_prefix, common_tags)
      variables.tf            # All inputs with validation
      outputs.tf              # EC2 instance ID/public IP, Aurora endpoint, ECR URL, DNS records, SSM helper commands
      data.tf                 # AWS data sources (default VPC, Ubuntu AMI, AZs, account ID)
      ec2.tf                  # EC2 instance, Elastic IP, user data script
      aurora.tf               # Aurora Serverless v2 cluster + instance + DB credentials
      ecr.tf                  # ECR repository + lifecycle policy (keep 10 images)
      s3.tf                   # Uploads bucket (Active Storage) + backups bucket (Glacier lifecycle)
      security.tf             # Security groups: web (80/443 only) and database (5432 from web only)
      iam.tf                  # Instance profile with policies for S3, ECR, Secrets, SSM, SES, CloudWatch
      secrets.tf              # SSM parameters (non-secret config) + Secrets Manager (Rails master key)
      ses.tf                  # SES domain identity + DKIM + MAIL FROM (staging only, account-wide)
      logging.tf              # CloudWatch log groups (rails, proxy) with retention by environment
    staging/
      main.tf                 # S3 backend config + module call with staging variable passthrough
      terraform.tfvars        # Staging values (t3.small, 0.5 ACU, 7-day backups, staging domain)
      .gitignore
    production/
      main.tf                 # S3 backend config + module call with production variable passthrough
      terraform.tfvars        # Production values (t3.small, higher ACU max, 14-day backups, prod domain)
      .gitignore
```

## Key Design Patterns

### No-Static-Credentials Pattern

No long-lived AWS access keys are stored in the app or committed to the repo. The EC2 instance profile handles server-side AWS API access, and operators use IAM-authenticated SSM sessions instead of public SSH access:

1. Terraform creates an IAM instance profile with least-privilege policies plus SSM permissions for the instance
2. EC2 user data runs on boot, using the instance profile to fetch config from SSM + Secrets Manager
3. Config is rendered to `/etc/{app}/env` as a root-owned `0600` file
4. A periodic refresh job updates the rendered config to handle secret rotation
5. Kamal reads deploy-time secrets from `.kamal/secrets.{env}` files which call `aws secretsmanager`

This is a practical single-host pattern, not a magic security boundary. EBS encryption protects the underlying volume at rest, but privileged users on the instance can still read the rendered env file. Limit SSM access, keep IAM scopes tight, and fetch secrets at runtime instead if your threat model requires stronger isolation.

### Kamal Environment Configs

Kamal supports destination-specific overrides via naming convention:

```
config/deploy.yml            → Base config (shared settings)
config/deploy.staging.yml    → Staging hosts, env vars, image name
config/deploy.production.yml → Production hosts, env vars, image name
```

Deploy with: `kamal deploy -d staging` or `kamal deploy -d production`

### SSM Session Manager for Admin Access

Use Session Manager for shell access and database tunneling instead of opening inbound port 22.

```bash
# Verify the instance is managed by SSM
aws ssm describe-instance-information

# Open a shell on the EC2 host
aws ssm start-session --target <instance-id>
```

Use the shell session for host-level debugging, inspecting Docker, or emergency maintenance. No public SSH ingress is required.

To reach Aurora without exposing port 5432, forward the database connection through the EC2 instance:

```bash
aws ssm start-session \
  --target <instance-id> \
  --document-name AWS-StartPortForwardingSessionToRemoteHost \
  --parameters host="<aurora-endpoint>",portNumber="5432",localPortNumber="5432"

psql -h 127.0.0.1 -p 5432 -U <app>_staging -d <app>_staging
```

Use this for initial database setup, ad-hoc queries, and maintenance tasks without opening database ingress to the internet.

### Kamal Over SSM

Kamal still uses SSH under the hood. If you want `kamal deploy` and `kamal app exec` without exposing port 22 publicly, route SSH through SSM and use the EC2 instance ID as the host:

```sshconfig
Host i-*
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
  ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters portNumber=%p"
  StrictHostKeyChecking accept-new
```

With that in place, set your Kamal hosts to the instance IDs from Terraform outputs instead of public IPs. You still need an SSH key for Kamal because the SSH daemon authenticates the session after SSM opens the transport, but you do not need a public security-group rule for port 22.

### Solid Queue in Puma

For single-server deployments, run background jobs inside the web process:

```yaml
env:
  clear:
    SOLID_QUEUE_IN_PUMA: true
```

Only split to a separate worker when you scale past one server.

### Job Monitoring with Mission Control

`mission_control-jobs` provides a web dashboard for Solid Queue (the Sidekiq Web equivalent). It shows queues, failed jobs, recurring schedules, and lets you retry/discard.

```ruby
# Gemfile
gem "mission_control-jobs"

# config/initializers/mission_control.rb
MissionControl::Jobs.http_basic_auth_enabled = false  # Use app-level auth instead

# config/routes.rb — restrict to admin users via warden/Devise
constraints ->(req) { req.env["warden"].user&.admin? } do
  mount MissionControl::Jobs::Engine, at: "/jobs"
end
```

Dashboard available at `/jobs` for admin users. No additional infrastructure needed — it reads directly from the Solid Queue database.

### Multi-Database for Rails Solid Libraries

Rails 8's Solid Queue, Solid Cache, and Solid Cable each need their own database. All run on the same Aurora instance:

```yaml
# config/database.yml
production:
  primary:
    database: myapp_production
  cache:
    database: myapp_production_cache
    migrations_paths: db/cache_migrate
  queue:
    database: myapp_production_queue
    migrations_paths: db/queue_migrate
  cable:
    database: myapp_production_cable
    migrations_paths: db/cable_migrate
```

**Important:** Terraform only creates the primary database. Create cache/queue/cable manually after apply.

### SES Email Delivery

The `:ses` delivery method requires two gems as of `aws-sdk-rails` v5:

```ruby
# Gemfile
gem "aws-sdk-rails"
gem "aws-actionmailer-ses", "~> 1"
```

```ruby
# config/environments/production.rb (and staging.rb)
config.action_mailer.delivery_method = :ses unless ENV["SECRET_KEY_BASE_DUMMY"]
```

The `SECRET_KEY_BASE_DUMMY` guard prevents the delivery method from being set during Docker asset precompilation (where AWS credentials aren't available).

### bin/kamal_exec Wrapper

Running `kamal app exec` outside of `bin/deploy` fails because host env vars (`STAGING_DEPLOY_HOST`, `APP_HOST`, etc.) aren't set. A `bin/kamal_exec` wrapper solves this once SSH is routed through SSM as shown above:

```bash
#!/usr/bin/env bash
set -e

DESTINATION=${1:-}
if [[ "$DESTINATION" != "staging" && "$DESTINATION" != "production" ]]; then
  echo "Usage: bin/kamal_exec [staging|production] [kamal args...]"
  exit 1
fi
shift

source .kamal/deploy.env

if [[ "$DESTINATION" == "staging" ]]; then
  export APP_HOST="staging.example.com"
else
  export APP_HOST="example.com"
fi

kamal "$@" -d "$DESTINATION"
```

Usage:
```bash
bin/kamal_exec staging app exec "bin/rails db:migrate"
bin/kamal_exec staging app exec "bin/rails db:seed"
bin/kamal_exec staging app exec --interactive --reuse "bin/rails console"
bin/kamal_exec production app logs -f
```

### Tailwind Pre-Build Hook

ARM Mac (Apple Silicon) + amd64 Docker builds = broken Tailwind CSS under QEMU. Three things are required:

1. **`.kamal/hooks/pre-build`**: Runs `bin/rails tailwindcss:build` locally before Docker build
2. **`.gitignore`**: Must track `app/assets/builds/tailwind.css` — Kamal's docker-container builder only sends git-tracked files to the build context
3. **`.dockerignore`**: Must NOT exclude `app/assets/builds/` — the pre-built CSS needs to be in the Docker context
4. **`Dockerfile`**: Detect pre-built CSS and use `assets:precompile_skip_tailwind` to avoid re-running Tailwind under QEMU
5. **`lib/tasks/tailwindcss.rake`** (or `assets.rake`): Define `assets:precompile_skip_tailwind` task that clears and no-ops the `tailwindcss:build` rake task before running `assets:precompile`

**Critical**: The pre-built `tailwind.css` must be committed to git before deploying. Kamal won't see uncommitted files.

### Database Backups: Use Aurora's Built-In Backups

Aurora Serverless v2 provides automated daily snapshots with point-in-time recovery (PITR) down to the second. This is significantly better than a custom `pg_dump` job:

- **Point-in-time recovery**: Restore to any second within the retention window, not just a daily snapshot
- **Zero application code**: No job to maintain, no `pg_dump` version mismatches, no S3 upload logic
- **Cross-AZ replication**: Built into Aurora by default
- **Configurable retention**: 7 days (staging) / 14 days (production) via `aurora_backup_retention` variable

**Do not build a custom `DatabaseBackupJob` using `pg_dump`.** Common failure modes:
- `pg_dump` client version in Docker image doesn't match Aurora PostgreSQL server version (exit code 1)
- `postgresql-client` from Debian repos lags behind Aurora's default engine version
- Error output gets swallowed when piping to gzip

If you need off-AWS disaster recovery (e.g., AWS account compromise), use Aurora's native export-to-S3 feature or AWS Backup with cross-account/cross-region copy rules — not application-level `pg_dump`.

### Let's Encrypt via Kamal Proxy

No ALB or ACM certificate needed. Kamal Proxy provisions and renews Let's Encrypt certificates automatically. Requires ports 80 + 443 open and DNS pointing to EC2 before first deploy.

## Terraform Conventions

| Setting | Value |
|---------|-------|
| State backend bucket | `{org}-terraform-state` |
| Lock table | `{org}-terraform-locks` |
| State key pattern | `{project}/{environment}/terraform.tfstate` |
| Region | `{aws-region}` |
| Naming | `{project}-{environment}-{resource}` |
| Provider default_tags | Project, Environment, ManagedBy |
| Feature flags | Boolean variables with `count` |

## Staging vs Production

| Setting | Staging | Production |
|---------|---------|------------|
| EC2 instance | t3.small | t3.small (minimum viable) |
| Aurora min/max ACU | 0.5 / 2 | 0.5 / 4 |
| Aurora backup retention | 7 days | 14 days |
| Aurora deletion protection | No | Yes |
| Aurora final snapshot | Skipped | Required |
| CloudWatch log retention | 7 days | 30 days |
| S3 backup lifecycle | Glacier 30d, expire 365d | Same |
| SES domain setup | Yes (account-wide) | No (reuses staging's) |
| Deploy | `bin/deploy staging` | `bin/deploy production` |

## Cost Estimate

### With Aurora Serverless v2

| Resource | Monthly |
|----------|--------:|
| EC2 t3.small | ~$17 |
| Aurora Serverless v2 (0.5-2 ACU) | ~$43 |
| EBS (20 GB root) | ~$2 |
| Elastic IP | ~$4 |
| S3 (uploads + backups) | ~$2 |
| Secrets Manager (4 secrets) | ~$2 |
| ECR + CloudWatch | ~$2 |
| **Total per environment** | **~$72** |

### With Standard RDS (Budget Option)

| Resource | Monthly |
|----------|--------:|
| EC2 t3.small | ~$17 |
| RDS db.t4g.micro | ~$13 |
| Everything else | ~$12 |
| **Total per environment** | **~$42** |

## Setup Checklist

### Prerequisites

- AWS CLI configured
- Session Manager plugin installed
- Terraform >= 1.0
- Kamal gem installed (`gem install kamal`)
- SSH key pair generated (required for Kamal over SSM; not required for direct SSM shell access)
- Domain in Cloudflare (or Route 53)

### Step-by-Step

1. **Provision infrastructure:**
   ```bash
   cd terraform/staging
   # Edit terraform.tfvars with your environment values and SSH public key if using Kamal over SSM
   terraform init && terraform plan && terraform apply
   ```

2. **Store Rails master key:**
   ```bash
   aws secretsmanager put-secret-value --region <aws-region> \
     --secret-id <app>-staging-rails-master-key \
     --secret-string file://config/master.key
   ```

3. **Create additional databases:**
   ```bash
   # Terminal 1: keep this port-forward session open
   aws ssm start-session \
     --target <instance-id> \
     --document-name AWS-StartPortForwardingSessionToRemoteHost \
     --parameters host="<aurora-endpoint>",portNumber="5432",localPortNumber="5432"

   # Terminal 2
   psql -h 127.0.0.1 -p 5432 -U <app>_staging -d <app>_staging
   CREATE DATABASE <app>_staging_cache;
   CREATE DATABASE <app>_staging_queue;
   CREATE DATABASE <app>_staging_cable;
   ```

4. **Configure Kamal:**
   ```bash
   cp .kamal/deploy.env.template .kamal/deploy.env
   # Fill in ECR_REGISTRY and STAGING_DEPLOY_HOST from terraform output
   # Use the EC2 instance ID for STAGING_DEPLOY_HOST when routing SSH through SSM
   ```

5. **Configure local SSH over SSM:** Add the `Host i-*` entry from the "Kamal Over SSM" section to `~/.ssh/config`

6. **Set up DNS:** Create A record pointing domain → Elastic IP

7. **First deploy:**
   ```bash
   bin/setup-deploy staging
   ```

8. **Verify:** `curl -I https://staging.example.com/up` → 200

9. **Repeat for production** with `terraform/production` and `-d production`

## Common Gotchas

| Gotcha | Fix |
|--------|-----|
| **t3.micro OOM** | Use t3.small minimum. 1 GB RAM is not enough for Rails + Docker + Solid Queue. |
| **Docker bypasses UFW** | Don't rely on UFW. Use AWS security groups as your firewall. |
| **Kamal health check fails** | Whitelist `/up` in Rack Attack config. |
| **Tailwind empty CSS** | Use pre-build hook on ARM Macs building for amd64. |
| **Aurora cold start** | Keep min ACU at 0.5 (not 0) to avoid 10-15s delays. |
| **Missing cache/queue/cable DBs** | Create manually after terraform apply — only primary is auto-created. |
| **SES sandbox mode** | Request production SES access in AWS console before sending to unverified addresses. |
| **`aws-sdk-rails` v5 + SES** | v5 removed the built-in `:ses` delivery method. You need the separate `aws-actionmailer-ses` gem (~> 1) alongside `aws-sdk-rails`. Without it you get `RuntimeError: Invalid delivery method :ses`. |
| **Kamal can't connect with port 22 closed** | Route SSH through SSM with the `AWS-StartSSHSession` ProxyCommand and use the instance ID as the Kamal host. |
| **Non-secret env vars in Kamal secrets** | Kamal errors if a secret var isn't defined in `.kamal/secrets*`. Values that aren't actually sensitive (S3 bucket names, notification emails) should go in `clear:` not `secret:` in the deploy config. |
| **Custom pg_dump backup jobs** | Don't build them. Aurora's automated backups with PITR are superior. Custom jobs fail silently due to pg_dump version mismatches between Docker image and Aurora. |
| **Secrets rendered to disk on the host** | EBS encryption helps at rest, but privileged users can still read `/etc/{app}/env`. Restrict SSM access and use runtime secret retrieval if you need stronger guarantees. |
