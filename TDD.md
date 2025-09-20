# SPEC-001-OmniPay Commerce Platform

## Background

### Vision

Enable brands and SMB merchants to run a modern online store with a conversion‑optimized checkout that supports multiple payment methods (cards, ACH, gift cards, wallets now; BNPL and multi‑currency later) while maintaining PCI‑DSS Level 1 posture and providing robust order management and reconciliation.

### Problem & Rationale

* Existing SME e‑commerce stacks often lock merchants to a single processor, resulting in higher fees and lower authorization rates.
* Alternative tender types (ACH, wallets, gift cards) are increasingly expected; lack of them hurts conversion and AOV.
* Merchants need an OMS and admin tooling that reconcile payments across providers without bespoke spreadsheets.
* Compliance and fraud controls are costly to build per‑merchant; a shared, tokenized platform reduces PCI scope and risk.

### Target Users

* DTC brands and digital‑first SMBs seeking reliable storefronts with flexible payments.
* Finance/Ops teams needing clean reconciliation, refunds/partial refunds, and dispute workflows.

### Scope (MVP vs. Later)

**MVP**

* Storefront with catalog (browse, filter, sort), cart, guest/registered checkout.
* Payment methods: cards (Visa/Mastercard/AmEx/Discover), ACH (Plaid/Dwolla), gift cards (incl. partial redemption), wallets (Apple Pay, Google Pay, PayPal).
* Payment processor abstraction (e.g., Stripe, Adyen, Braintree) with retry/orchestration and network tokenization where available.
* OMS with order tracking, receipts, refunds/partial refunds, dispute intake.
* Admin with orders/payments views, reconciliation reports, manual refunds.
* Security: tokenization, vaulting via provider, TLS everywhere, least‑privilege access, fraud screening via 3rd‑party.

**Post‑MVP**

* BNPL (Affirm, Klarna, Afterpay).
* Internationalization: multi‑currency pricing/settlement, local payment methods.

### Success Metrics (leading + lagging)

* ↑ Cart→Checkout conversion.
* ↑ Payment success/authorization rate per payment type and per BIN region.
* ↓ Mean checkout completion time.
* % share of alternative payments (ACH, wallets, gift cards).
* Chargeback and fraud rates within threshold; refund SLA compliance.

### Key Constraints & Assumptions (to confirm)

* **PCI model**: SAQ‑A style via hosted fields/elements; tokens only—no PANs touch our servers even in demo.
* **Demo scope**: Single currency (USD). **No taxes, no shipping, no local law logic**—we only collect the payment due.
* **Catalog**: **Non‑physical demo SKUs** (e.g., digital/license/credits). No inventory or fulfillment flows.
* **Traffic scale (Yr‑1)**: Peak 150 RPS storefront, 25 RPS checkout API; order volume \~50k–250k/month equivalent for performance testing.
* **Operational posture**: 99.9% monthly availability target; RTO 4h, RPO 15m.

---

## Requirements

### MoSCoW Prioritization (Demo constraints applied)

**Must Have**

* Storefront (or "Checkout Playground") presenting non‑physical SKUs with search/filter/sort; guest checkout and optional account creation.
* Payment methods (sandbox): **Cards**, **ACH (Plaid/Dwolla)**, **Gift Cards (internal issuer, partial redemption)**, **Wallets** (Apple Pay, Google Pay, PayPal).
* **Payment Abstraction Layer** supporting at least two processors (e.g., Stripe + Braintree) with a common interface and switchable routing.
* **Tokenization & Hosted Fields/Elements** to keep app in SAQ‑A scope; vaulting at provider.
* **OMS** with states: `created → awaiting_payment → paid → refunded/partially_refunded | voided | failed | disputed`.
* **Refunds** (full/partial) and provider‑synced **dispute intake** (read‑only for demo).
* **Admin Dashboard**: orders, payments, refunds, gift cards; manual refund initiation; search/export.
* **Reconciliation**: daily aggregated payments report across providers, matching orders↔payments; downloadable CSV.
* **Fraud Screening**: use provider risk signals (e.g., Radar/Decisioning) with allow/decline hooks; basic velocity checks.
* **Webhooks** ingestion with idempotency, signature verification, replay handling, and dead‑letter queue.
* **Observability & Audit**: structured logs, metrics (p95 checkout time, auth rate), and security/audit trails.

**Should Have**

* Saved customer payment methods (card & bank tokens); network tokens when supported by provider.
* Smart **retry/orchestration** for soft declines and 3‑D Secure flows; route rules by BIN/geo/failure code.
* ACH account validation via **Plaid Link** (instant) with micro‑deposit fallback; NACHA‑style debit authorization record.
* Gift card lifecycle: issue, activate, redeem/partial redeem, void, balance inquiry; 16‑digit code + PIN.
* Wallets: on‑domain Apple Pay/Google Pay setup; PayPal Checkout with order capture.
* CI/CD pipeline, IaC, staging + sandbox keys management; basic WAF/rate limiting & CAPTCHA on payment intents.

**Could Have**

* Demo **Mock Provider** to simulate edge cases (3DS, timeouts, partial captures) for testing.
* Price lists & promotions (coupon codes) affecting payable amount (no tax calculations).
* Basic multi‑merchant theming (white‑label) for demos.

**Won’t Have (MVP)**

* Physical shipping, inventory, or address validation.
* Tax engine, multi‑currency pricing/settlement, local payment methods outside wallets.
* Subscriptions/recurring billing and invoicing.

## Method

### Integration Configuration → Frontend Component Loader

Each payment integration has a **runtime configuration** that tells the frontend which internal UI component to mount and how to initialize it. The API exposes `/integrations/config` which returns a list of enabled integrations plus any **init secrets** required to render SDK components.

**Example response (demo):**

```json
{
  "integrations": [
    {"id":"stripe_card","enabled":true,"component":"StripePaymentElement","mode":"client_confirm"},
    {"id":"braintree_card","enabled":true,"component":"BraintreeHostedFields","mode":"tokenize"},
    {"id":"paypal_wallet","enabled":true,"component":"PayPalButtons","mode":"approve"},
    {"id":"ach_dwolla_plaid","enabled":true,"component":"PlaidLink","mode":"tokenize"},
    {"id":"giftcard","enabled":true,"component":"GiftCardForm","mode":"code_pin"}
  ],
  "init": {
    "stripe": {"paymentIntentClientSecret": "pi_..._secret_xxx"},
    "braintree": {"clientToken": "sandbox_f252zhq7_hh4cpc39zq4rgjcg"},
    "paypal": {"clientId": "<sandbox-client-id>"},
    "plaid": {"linkToken": "link-sandbox-..."}
  }
}
```

The frontend maintains a **registry** mapping `component` → internal React components (`<StripePaymentElement/>`, `<BraintreeHostedFields/>`, `<PayPalButtons/>`, `<PlaidLink/>`, `<GiftCardForm/>`).

### Abstract Payment Payload (sent to internal API)

All checkouts send the same **abstract payload** to our API, which adapts it to the target provider. Supports both **tokenize-then-server-confirm** and **client-confirm** patterns.

```json
{
  "idempotencyKey": "1b2a1b9e-...",
  "cart": {"id":"c_123","items":[{"sku":"sku_1","qty":1,"price":4999}]},
  "amount": 4999,
  "currency": "USD",
  "payment": {
    "method": "card|wallet|ach|gift_card",
    "provider": "stripe|braintree|paypal|dwolla",
    "mode": "tokenize|client_confirm|approve",
    "processorToken": "pm_xxx | nonce_xxx | PAYPAL_ORDER_ID | plaid_public_token | dwolla_secure_exchange_id | gift:CODE:PIN",
    "intentId": "pi_xxx|bt_txn_id|dwolla_transfer_id (optional)",
    "returnUrl": "https://demo.local/return"
  },
  "metadata": {"email":"user@example.com"}
}
```

**Provider mappings (examples):**

* **Stripe (card, wallets)**

    * *Mode A (client\_confirm – recommended):* API creates PaymentIntent → returns `clientSecret` in `/integrations/config`. Frontend calls Stripe to confirm. Frontend then POSTs `/payments/finalize` with `{intentId, status}`.
    * *Mode B (tokenize):* Frontend creates `payment_method` (`pm_xxx`) → POSTs abstract payload. API calls `confirm` with `payment_method=pm_xxx`.
* **Braintree (card)**: Frontend gets `nonce` from Hosted Fields → send as `processorToken`. API executes `transaction.sale`.
* **PayPal (wallet)**: Frontend obtains `orderId` via JS SDK approve → send as `processorToken`. API captures the order.
* **ACH (Plaid → Dwolla)**: Frontend returns `public_token`. API exchanges for Dwolla secure exchange / funding source and creates a transfer.
* **Gift Card (internal, demo)**: Frontend posts `CODE`+`PIN` → API redeems from in‑memory ledger and optionally routes remainder to another provider in the same call (split tender).

### Stateless Demo Storage

* **Products**: Served from a **static JSON feed** (`/products.json`).
* **Cart**: Kept **client‑side** (localStorage) with a signed `cartId` cookie for SSR/return visits. API receives full cart payload and recalculates amount server‑side from the JSON feed (source of truth).
* **Payments & Webhooks**: No persistent DB. The server holds an **in‑memory store** (Map) for the lifetime of the process to display recent transactions/logs in Admin. Safe to lose on restart.
* **Idempotency**: In‑memory TTL cache keyed by `idempotencyKey` for 24h (demo adjustable). Webhook events are deduped via provider event IDs.

### Updated Architecture (stateless demo)

```plantuml
@startuml
skinparam componentStyle rectangle
package "OmniPay Demo" {
  [Next.js Storefront
(Admin + Checkout)] as Web
  [NestJS API
(Orchestration + Webhooks)] as API
  [In-Memory Store
(maps: carts/intents/logs)] as MEM
  [products.json] as PROD
}

node Providers {
  [Stripe] as Stripe
  [Braintree] as Braintree
  [PayPal] as PayPal
  [Plaid] as Plaid
  [Dwolla] as Dwolla
}

Web --> API : REST JSON (config, intents, finalize)
Web --> PROD : GET /products.json
Web ..> Stripe : Elements/Payment Element
Web ..> Braintree : Hosted Fields
Web ..> PayPal : JS SDK
Web ..> Plaid : Link

API <--> MEM : carts/intents/idempotency/logs
API --> PROD : validate totals
API <--> Stripe : intents, refunds
API <--> Braintree : transactions, refunds
API <--> PayPal : orders/capture, refunds
API <--> Dwolla : transfers
API --> Plaid : token exchange

Stripe --> API : webhook
Braintree --> API : webhook
PayPal --> API : webhook
Dwolla --> API : webhook
@enduml
```

### API Contracts (demo)

* `GET /products` → returns `/products.json` contents (server‑side cached for 60s)
* `GET /integrations/config` → returns enabled integrations + init secrets
* `POST /payments/intent` → body: abstract payload (without `processorToken` if `client_confirm`) → returns provider init values (e.g., Stripe PI `clientSecret` or BT `clientToken`)
* `POST /payments/confirm` → body: abstract payload with `processorToken` (or `intentId` for client‑confirm) → returns `{status, providerRefs}`
* `POST /payments/finalize` → body: `{intentId, status, cartId}` (client‑confirm closeout)
* `POST /gift-cards/redeem` → body: `{code, pin, amount}` → returns `{applied, remaining}` (in‑memory)
* Webhooks: `/webhooks/stripe|braintree|paypal|dwolla` (HMAC/signature verified, enqueue to in‑memory log)

### Frontend Component Registry (simplified)

```ts
// registry.tsx
export type IntegrationId = 'stripe_card'|'braintree_card'|'paypal_wallet'|'ach_dwolla_plaid'|'giftcard';
export const registry: Record<string, React.FC<any>> = {
  StripePaymentElement: StripeCard,
  BraintreeHostedFields: BraintreeCard,
  PayPalButtons: PayPalButton,
  PlaidLink: PlaidButton,
  GiftCardForm: GiftCardComponent
};
```

**Mounting flow**

1. On checkout, call `/integrations/config`.
2. Render components by `component` field; pass any `init` secrets.
3. On submit, build **abstract payload** and POST `/payments/confirm` (or `/payments/finalize` for client‑confirm providers).

### Adapter Mapping (server pseudocode)

```ts
// payments.service.ts
switch (payload.payment.provider) {
  case 'stripe':
    if (payload.payment.mode === 'client_confirm') {
      // already confirmed in browser; just verify and fetch PI
      const pi = await stripe.paymentIntents.retrieve(payload.payment.intentId!);
      return { status: pi.status, providerRefs: { intentId: pi.id, chargeId: pi.latest_charge }};
    } else {
      // tokenize → server confirm
      const pi = await stripe.paymentIntents.create({ amount, currency, payment_method: payload.payment.processorToken, confirmation_method: 'automatic', confirm: true });
      return { status: pi.status, providerRefs: { intentId: pi.id }};
    }
  case 'braintree':
    const result = await bt.transaction.sale({ amount: toDollars(amount), paymentMethodNonce: payload.payment.processorToken, options: { submitForSettlement: true }});
    return { status: result.success ? 'succeeded' : 'failed', providerRefs: { transactionId: result.transaction?.id }};
  case 'paypal':
    const capture = await paypal.orders.capture(payload.payment.processorToken);
    return { status: capture.status, providerRefs: { orderId: capture.id }};
  case 'dwolla':
    const transfer = await dwolla.transfers.create({ source: payload.payment.processorToken, amount, currency });
    return { status: transfer.status || 'pending', providerRefs: { transferId: transfer.id }};
}
```

### Split‑Tender (Gift Card + Another Method)

* API first redeems from the in‑memory **gift card ledger** (atomic operation under a per‑code lock).
* If remainder > 0, it proceeds to the mapped provider using the same abstract payload (with `amount` replaced by remainder). Returns a composite response.

### Demo Gift Card Ledger (in‑memory)

* Seed on boot: `[ {code:"4111-1111-1111-1111", pin:"1234", balance: 5000}, ... ]` (stored as salted hashes in memory with plain demo display in Admin).
* Idempotent redemption via `(code,pin,cartId)` key to avoid double‑apply on retries.

### Security & Compliance Notes (demo)

* Keep SAQ‑A posture: **all PAN collection is via provider components** (hosted fields/elements/JS SDK). We never handle raw card data.
* Verify webhook signatures; apply replay window; store last processed event IDs in memory TTL set.
* Rate limit sensitive endpoints (gift card balance/redeem, intent creation, PayPal order create).

*(Next section will include **Implementation** details: file layout, minimal code scaffolds, and how to wire the components.)*

## Implementation

*(to be completed after Method confirmation)*

*(to be completed after Method confirmation)*

*(to be completed after Method confirmation)*

## Milestones

*(to be completed after Method confirmation)*

## Gathering Results

*(to be completed after Implementation planning)*

## Need Professional Help in Developing Your Architecture?

Please contact me at [sammuti.com](https://sammuti.com) :)
