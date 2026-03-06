# Rails 8 + Stripe Subscriptions: Golden Spec

A proven pattern for integrating Stripe subscriptions into a Rails 8 SaaS application with webhooks, billing portal, and Discord notifications.

## Architecture Overview

```
Plans page → Stripe Checkout (hosted) → Webhook activates subscription → Billing portal for management
```

- **Stripe Checkout** handles payment collection (no custom forms, PCI compliance handled by Stripe)
- **Webhooks** are the source of truth for subscription state (not the success redirect)
- **Billing Portal** lets customers manage payment methods and cancellation (hosted by Stripe)
- **No Stripe.js or Elements** needed for basic subscription billing

## Stripe Account Setup

### Sandbox vs Live

Stripe now uses "Sandboxes" instead of the old test/live toggle. Key differences from the old model:

- **Price IDs are different** between sandbox and live — they share the first few characters but the full IDs are different. Don't assume they're the same.
- **API keys** are still prefixed `pk_test_`/`sk_test_` for sandbox and `pk_live_`/`sk_live_` for live
- **Webhook signing secrets** (`whsec_...`) are always different per endpoint

### Product + Price Setup

Create the product in both sandbox and live:

1. **Product catalog** → **+ Add product**
2. Name, description, recurring pricing ($X/month)
3. Copy the **Price ID** (`price_...`) — this goes in your env vars

**Important:** You need separate Price IDs for sandbox and live environments. Do not reuse them.

### Webhook Endpoints

Create one endpoint per environment:

| Environment | Endpoint URL | Mode |
|-------------|-------------|------|
| Staging | `https://staging.example.com/webhooks/stripe` | Sandbox |
| Production | `https://example.com/webhooks/stripe` | Live |
| Local dev | Stripe CLI forwards to `localhost:3000` | Sandbox |

**Events to subscribe to:**
- `checkout.session.completed` — activate subscription after payment
- `invoice.paid` — sync invoice records, update subscription period
- `invoice.payment_failed` — mark subscription past_due
- `customer.subscription.updated` — sync status changes (cancellation scheduled, etc.)
- `customer.subscription.deleted` — cancel subscription locally

### Customer Portal

Configure in **Settings** → **Billing** → **Customer portal**:

- **Payment methods**: Enable
- **Cancel subscriptions**: Enable, "At end of billing period" recommended
- **Switch plans**: Disable (if single plan)
- **Redirect link**: Set to your app URL (fallback only — code sets `return_url` explicitly)

Portal settings are shared across sandbox and live.

### Tax Registration

Stripe may prompt to "Add a tax registration" — this is for Stripe Tax (automated sales tax/VAT). **Skip for launch.** It's optional and can be added later when you need tax compliance.

## Secrets Management

### Four Keys Per Environment

| Key | Source | Sandbox prefix | Live prefix |
|-----|--------|---------------|-------------|
| `STRIPE_PUBLISHABLE_KEY` | Developers → API keys | `pk_test_` | `pk_live_` |
| `STRIPE_SECRET_KEY` | Developers → API keys (Reveal) | `sk_test_` | `sk_live_` |
| `STRIPE_WEBHOOK_SECRET` | Developers → Webhooks → endpoint → Signing secret | `whsec_` | `whsec_` |
| `STRIPE_PRICE_ID` | Product catalog → Price ID | `price_` | `price_` |

### AWS Secrets Manager Storage

Store as JSON in a single secret per environment:

```bash
aws secretsmanager create-secret \
  --region us-east-2 \
  --name myapp-staging-stripe \
  --secret-string '{
    "publishable_key": "pk_test_...",
    "secret_key": "sk_test_...",
    "webhook_secret": "whsec_...",
    "price_id": "price_..."
  }'
```

**Update a single key** without rewriting everything:

```bash
SECRET=$(aws secretsmanager get-secret-value --region us-east-2 \
  --secret-id myapp-staging-stripe \
  --query SecretString --output text \
  | jq --arg val "price_NEW_VALUE" '.price_id = $val')

aws secretsmanager put-secret-value --region us-east-2 \
  --secret-id myapp-staging-stripe \
  --secret-string "$SECRET"
```

### Kamal Secrets File

```bash
# .kamal/secrets.staging
STRIPE_PUBLISHABLE_KEY=$(aws secretsmanager get-secret-value --region us-east-2 --secret-id myapp-staging-stripe --query SecretString --output text | jq -r '.publishable_key')
STRIPE_SECRET_KEY=$(aws secretsmanager get-secret-value --region us-east-2 --secret-id myapp-staging-stripe --query SecretString --output text | jq -r '.secret_key')
STRIPE_WEBHOOK_SECRET=$(aws secretsmanager get-secret-value --region us-east-2 --secret-id myapp-staging-stripe --query SecretString --output text | jq -r '.webhook_secret')
STRIPE_PRICE_ID=$(aws secretsmanager get-secret-value --region us-east-2 --secret-id myapp-staging-stripe --query SecretString --output text | jq -r '.price_id')
```

## Rails Implementation

### Gem

```ruby
gem "stripe"
```

No special Rails integration gem needed — just the `stripe` gem directly.

### Initializer

```ruby
# config/initializers/stripe.rb
Rails.configuration.stripe = {
  publishable_key: ENV['STRIPE_PUBLISHABLE_KEY'],
  secret_key: ENV['STRIPE_SECRET_KEY'],
  webhook_secret: ENV['STRIPE_WEBHOOK_SECRET']
}

Stripe.api_key = Rails.configuration.stripe[:secret_key]

# Validate in deployed environments (skip during Docker asset precompile)
if (Rails.env.production? || Rails.env.staging?) && ENV["SECRET_KEY_BASE_DUMMY"].blank?
  %w[STRIPE_PUBLISHABLE_KEY STRIPE_SECRET_KEY STRIPE_WEBHOOK_SECRET].each do |key|
    raise "Missing required environment variable: #{key}" if ENV[key].blank?
  end
end
```

### User Model: stripe_customer_id

```ruby
# Migration
add_column :users, :stripe_customer_id, :string
add_index :users, :stripe_customer_id, unique: true
```

```ruby
# app/models/user.rb
def find_or_create_stripe_customer
  if stripe_customer_id.present?
    Stripe::Customer.retrieve(stripe_customer_id)
  else
    customer = Stripe::Customer.create(email: email, metadata: { user_id: id })
    update!(stripe_customer_id: customer.id)
    customer
  end
end
```

### Checkout Flow

Use `Stripe::Customer` (not `customer_email`) so the customer is reusable:

```ruby
def create_stripe_checkout_session
  customer = current_user.find_or_create_stripe_customer
  session = Stripe::Checkout::Session.create(
    payment_method_types: ['card'],
    line_items: [{ price: @plan.stripe_price_id, quantity: 1 }],
    mode: 'subscription',
    success_url: success_subscriptions_url,
    cancel_url: cancel_subscriptions_url,
    customer: customer.id,
    metadata: { plan_id: @plan.id, user_id: current_user.id }
  )
  redirect_to session.url, allow_other_host: true, status: 303
end
```

**Important:** The plans page should POST directly to the checkout action (via `button_to`), not link to an intermediate `new` page. With a single plan there's nothing to configure.

Keep the success redirect simple. Do not trust a `session_id` query parameter to decide which local user or subscription to update. If you choose to include `{CHECKOUT_SESSION_ID}` for display or debugging, treat it as read-only and verify ownership before showing anything sensitive.

**Critical — Turbo + cross-origin redirects:** `button_to` submits via Turbo (fetch) by default. Fetch requests cannot follow cross-origin redirects to `checkout.stripe.com` due to CORS. You **must** add `data: { turbo: false }` to any button that redirects to Stripe:

```erb
<%= button_to "Get Started", subscriptions_path(plan_id: @plan.id),
    method: :post, data: { turbo: false },
    class: "btn btn-primary" %>
```

This applies to checkout buttons AND billing portal buttons — any `button_to` that redirects to a Stripe-hosted page.

### Success Redirect

The success redirect is **UX only** — the webhook is the source of truth. Do not activate or modify subscriptions from the success action:

```ruby
def success
  redirect_to dashboard_path, notice: "Checkout complete. We're confirming your subscription now."
end
```

If you want a richer success page, render a pending state and poll your local subscription record until the webhook marks it active. Never trust `params[:session_id]` alone for authorization or record lookup.

**Never use hardcoded period dates** like `Time.now + 30.days` when syncing a subscription. In the webhook handler, retrieve the actual subscription from Stripe for real `current_period_start`/`current_period_end`.

### Webhook Controller

```ruby
class StripeWebhooksController < ApplicationController
  skip_before_action :verify_authenticity_token
  skip_before_action :staging_http_auth  # if applicable
  skip_before_action :require_subscription
  skip_before_action :set_current_series  # skip any user-scoped filters

  def create
    payload = request.body.read
    sig_header = request.env["HTTP_STRIPE_SIGNATURE"]
    event = Stripe::Webhook.construct_event(payload, sig_header, webhook_secret)

    case event.type
    when "checkout.session.completed" then handle_checkout(event.data.object)
    when "invoice.paid"               then handle_invoice_paid(event.data.object)
    when "invoice.payment_failed"     then handle_payment_failed(event.data.object)
    when "customer.subscription.updated" then handle_sub_updated(event.data.object)
    when "customer.subscription.deleted" then handle_sub_deleted(event.data.object)
    end

    head :ok  # Always return 200, even for unhandled events
  rescue JSON::ParserError, Stripe::SignatureVerificationError
    head :bad_request
  end
end
```

**Key points:**
- Skip ALL before_action filters — webhooks have no session, no CSRF token, no authenticated user
- Always return `head :ok` for handled AND unhandled events — Stripe retries on non-2xx
- Find users via `stripe_customer_id` or `metadata.user_id`, never session
- The webhook should be the only code path that activates, updates, or cancels subscriptions

### Billing Portal

```ruby
def billing_portal
  portal_session = Stripe::BillingPortal::Session.create(
    customer: current_user.stripe_customer_id,
    return_url: settings_billing_url
  )
  redirect_to portal_session.url, allow_other_host: true, status: 303
end
```

Route as a collection POST on subscriptions. The portal handles payment method updates and cancellation — no need to build those UI flows yourself.

### Invoice Sync

```ruby
class SyncInvoicesJob < ApplicationJob
  def perform(user)
    return unless user.stripe_customer_id.present?

    Stripe::Invoice.list(customer: user.stripe_customer_id, limit: 100)
      .auto_paging_each do |stripe_invoice|
        invoice = user.invoices.find_or_initialize_by(stripe_invoice_id: stripe_invoice.id)
        invoice.update!(
          amount_paid: stripe_invoice.amount_paid,
          currency: stripe_invoice.currency,
          status: stripe_invoice.status,
          invoice_pdf_url: stripe_invoice.invoice_pdf,
          hosted_invoice_url: stripe_invoice.hosted_invoice_url,
          invoice_number: stripe_invoice.number,
          paid_at: stripe_invoice.status_transitions.paid_at ?
            Time.at(stripe_invoice.status_transitions.paid_at) : nil
        )
      end
  end
end
```

### Seed Data

The Plan record stores the `stripe_price_id`. Use an env var so it works across environments:

```ruby
Plan.find_or_initialize_by(name: 'Standard').update!(
  price_cents: 3900,
  stripe_price_id: ENV["STRIPE_PRICE_ID"] || "price_standard_monthly",
  active: true
)
```

**After every deploy where the price ID changes**, re-run seeds: `bin/rails db:seed`

## Route Configuration

```ruby
# Webhook — before any authenticated routes
post '/webhooks/stripe', to: 'stripe_webhooks#create'

# Subscriptions
resources :subscriptions, only: [:new, :create, :destroy] do
  collection do
    get :success
    get :cancel
    post :billing_portal
  end
end
```

## Local Development

Use the **Stripe CLI** to forward webhooks to localhost:

```bash
stripe listen --forward-to localhost:3000/webhooks/stripe
```

This prints a webhook signing secret (`whsec_...`) — use it as your local `STRIPE_WEBHOOK_SECRET`. It's different from the dashboard endpoint secret.

## Testing Webhooks

Generate valid signatures in specs:

```ruby
def generate_signature(payload)
  timestamp = Time.now.to_i
  signed_payload = "#{timestamp}.#{payload}"
  signature = OpenSSL::HMAC.hexdigest("SHA256", webhook_secret, signed_payload)
  "t=#{timestamp},v1=#{signature}"
end
```

Stub Stripe API calls (don't hit real Stripe in tests):

```ruby
allow(Stripe::Subscription).to receive(:retrieve).and_return(
  double(id: "sub_test", current_period_start: 1.day.ago.to_i, current_period_end: 30.days.from_now.to_i)
)
```

## Common Gotchas

| Gotcha | Fix |
|--------|-----|
| **Sandbox vs live Price IDs look similar** | They share a prefix but are different IDs. Always copy the full ID from each environment separately. |
| **Success redirect is fragile** | User can close the tab before redirect. Webhook is the source of truth — success redirect should be UX only. |
| **Hardcoded period dates** | Never use `Time.now + 30.days`. Always get real dates from `Stripe::Subscription.retrieve`. |
| **Webhook signature fails in staging** | Make sure the webhook endpoint skips HTTP basic auth (`skip_before_action :staging_http_auth`). |
| **`customer_email` vs `customer`** | Use `customer: customer.id` (not `customer_email:`) so the Stripe customer is reusable for billing portal and invoice sync. |
| **Trusting `session_id` from the URL** | Never use a success-page `session_id` to decide which local records to update. Subscription state should be written from the verified webhook only. |
| **Webhook returns non-200** | Stripe retries with exponential backoff. Always return `head :ok` even for unhandled events. |
| **Missing `stripe_customer_id`** | Persist it when you create the Stripe customer, and reconcile it in the verified webhook handler if needed. |
| **Turbo blocks Stripe redirect** | `button_to` uses Turbo fetch by default, which can't follow cross-origin redirects (CORS). Add `data: { turbo: false }` to all buttons that redirect to Stripe (checkout, billing portal). |
| **`current_period_start/end` missing** | Stripe API 2025-03-31 moved these from the subscription object to subscription items. Use `stripe_sub.items.data[0].current_period_start` instead. When using `Stripe::Subscription.retrieve`, pass `expand: ["items"]`. Webhook payloads include items automatically. |

## Post-Deploy Checklist

1. Create Stripe product + price in the correct mode (sandbox or live)
2. Register webhook endpoint with the 5 events
3. Store 4 keys in AWS Secrets Manager
4. Deploy the app
5. Run `db:seed` to set the Price ID on the Plan record
6. Configure Customer Portal settings (cancellation policy, branding)
7. Test end-to-end: signup → checkout → webhook delivery → subscription active → billing portal
