# OmniPay Commerce Platform - Development Backlog

Here is a backlog of items to complete the ecommerce payment platform. Each item is a task that can be worked on independently.

Child tasks are indented below their parent task and should be completed as part of the parent task.

MVP items will not be labeled.  Non-MVP items will be labeled as (not MVP) or (non-MVP).

Ignore non-MVP items until all MVP items have been completed. 

## Project Setup & Infrastructure
- [ ] Initialize Next.js storefront application
- [ ] Set up NestJS API backend
- [ ] Configure development environment and dependencies
- [ ] Implement Infrastructure as Code (IaC)
- [ ] Set up basic WAF/rate limiting  (not MVP)
- [ ] Configure CAPTCHA on payment intents  (not MVP)
- [ ] Configure staging and sandbox environments  (not MVP)
- [ ] Set up CI/CD pipeline (not MVP)

## UI Design & Branding
- [ ] Create wireframes for storefront and admin dashboard
- [ ] For this task only, you are a senior UX designer and professional frontend engineer.  Design and implement UI. Use Tailwind CSS.

## Core Architecture & Payment Abstraction
- [ ] Design and implement payment abstraction layer
- [ ] Create provider interface for payment processors
- [ ] Implement integration configuration system
- [ ] Build frontend component registry
- [ ] Create abstract payment payload structure
- [ ] Implement provider mapping and adapter pattern
- [ ] Set up in-memory store for demo data

## Products & Catalog
- [ ] Create products.json static feed
- [ ] Implement product display with search/filter/sort
- [ ] Build product catalog browsing functionality
- [ ] Implement cart management (client-side localStorage)
- [ ] Add cart persistence with signed cartId cookie

## Payment Methods - Cards
- [ ] Integrate Stripe payment elements
- [ ] Implement Stripe client-confirm flow
- [ ] Integrate Braintree hosted fields
- [ ] Implement Braintree tokenize flow
- [ ] Add card tokenization and vaulting
- [ ] Implement network tokenization support
- [ ] Add support for Visa/Mastercard/AmEx/Discover

## Payment Methods - ACH
- [ ] Create a mock ACH provider (SusieBank).
- [ ] Integrate Plaid Link for account verification (not MVP)
- [ ] Set up Dwolla integration for ACH transfers (not MVP)
- [ ] Implement Plaid to Dwolla token exchange (not MVP)
- [ ] Add instant account validation (not MVP)
- [ ] Implement micro-deposit fallback (not MVP)
- [ ] Create NACHA-style debit authorization records (not MVP)

## Payment Methods - Digital Wallets
- [ ] Create a mock digital wallet (SusiePay).
- [ ] Integrate Apple Pay on-domain setup (not MVP)
- [ ] Integrate Google Pay on-domain setup (not MVP)
- [ ] Implement PayPal Checkout integration (not MVP)
- [ ] Add PayPal order capture functionality (not MVP)
- [ ] Handle wallet-specific authorization flows

## Payment Methods - Gift Cards
- [ ] Create a mock gift card provider (SusieGift).
- [ ] Design gift card data structure and validation
- [ ] Implement gift card ledger (in-memory for demo)
- [ ] Create gift card lifecycle management (issue, activate, redeem, void)
- [ ] Build gift card form component
- [ ] Implement partial redemption functionality
- [ ] Add gift card balance inquiry
- [ ] Create 16-digit code + PIN system
- [ ] Implement split-tender with gift cards + other methods

## Checkout Flow & User Experience
- [ ] Build guest checkout functionality
- [ ] Implement optional user account creation
- [ ] Create unified checkout UI
- [ ] Add payment method selection interface
- [ ] Implement checkout completion flow
- [ ] Add order confirmation and receipts
- [ ] Create return URL handling

## Order Management System (OMS)
- [ ] Design order state machine (created → awaiting_payment → paid → refunded/voided/failed/disputed)
- [ ] Implement order creation and tracking
- [ ] Build order status management
- [ ] Add order search and filtering
- [ ] Create order export functionality

## Refunds & Disputes
- [ ] Implement full refund functionality
- [ ] Add partial refund support
- [ ] Create manual refund initiation in admin
- [ ] Build dispute intake system (read-only)
- [ ] Add provider-synced dispute handling
- [ ] Implement refund reconciliation

## Admin Dashboard
- [ ] Create admin authentication system
- [ ] Build orders management interface
- [ ] Add payments view and management
- [ ] Implement refunds management panel
- [ ] Create gift cards administration
- [ ] Add search and export functionality
- [ ] Build reconciliation reports interface

## Reconciliation & Reporting
- [ ] Implement daily aggregated payments reports
- [ ] Create cross-provider payment matching
- [ ] Build downloadable CSV export
- [ ] Add order↔payment reconciliation
- [ ] Create financial reporting dashboard

## Fraud & Security
- [ ] Implement provider risk signal integration
- [ ] Add basic velocity checks
- [ ] Create allow/decline hooks for fraud screening
- [ ] Implement tokenization throughout system
- [ ] Add PCI-DSS compliance measures (SAQ-A)
- [ ] Ensure TLS everywhere implementation
- [ ] Implement least-privilege access controls

## Webhooks & Event Processing
- [ ] Create webhook ingestion system
- [ ] Implement idempotency handling
- [ ] Add signature verification for all providers
- [ ] Build replay handling mechanism
- [ ] Create dead-letter queue for failed events
- [ ] Implement webhook retry logic
- [ ] Add event deduplication

## Smart Routing & Orchestration
- [ ] Implement retry logic for soft declines
- [ ] Add 3-D Secure flow handling
- [ ] Create routing rules by BIN/geo/failure code
- [ ] Implement payment orchestration
- [ ] Add intelligent failover mechanisms

## Observability & Monitoring
- [ ] Set up structured logging
- [ ] Implement metrics collection (p95 checkout time, auth rates)
- [ ] Create security and audit trails
- [ ] Add performance monitoring
- [ ] Build operational dashboards
- [ ] Implement alerting system

## API Endpoints
- [ ] `GET /products` - Product catalog endpoint
- [ ] `GET /integrations/config` - Integration configuration
- [ ] `POST /payments/intent` - Payment intent creation
- [ ] `POST /payments/confirm` - Payment confirmation
- [ ] `POST /payments/finalize` - Client-confirm finalization
- [ ] `POST /gift-cards/redeem` - Gift card redemption
- [ ] Webhook endpoints for all providers (`/webhooks/*`)

## Testing & Quality Assurance
- [ ] Create unit tests for payment abstraction layer
- [ ] Add integration tests for each payment provider
- [ ] Implement end-to-end checkout flow tests
- [ ] Create webhook processing tests
- [ ] Add fraud detection testing
- [ ] Build performance testing suite
- [ ] Test split-tender scenarios

## Documentation & Deployment
- [ ] Create API documentation
- [ ] Document payment integration patterns
- [ ] Add deployment guides
- [ ] Create troubleshooting documentation
- [ ] Document security practices

## Optional/Future Enhancements
- [ ] Mock Provider for edge case simulation
- [ ] Price lists and promotion system
- [ ] Coupon codes functionality
- [ ] Multi-merchant theming (white-label)
- [ ] Saved customer payment methods
- [ ] Advanced analytics and reporting

## Post-MVP Features (Future)
- [ ] BNPL integration (Affirm, Klarna, Afterpay)
- [ ] Multi-currency pricing and settlement
- [ ] International payment methods
- [ ] Subscription and recurring billing
- [ ] Physical shipping integration
- [ ] Tax engine integration
- [ ] Inventory management system