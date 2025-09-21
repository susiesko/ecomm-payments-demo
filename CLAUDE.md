# Claude Code Development Guide

This file contains helpful information for Claude Code to efficiently work on the OmniPay Commerce Platform project.

## Project Overview

This is an e-commerce payment demo platform that supports multiple payment methods (cards, ACH, wallets, gift cards) with a focus on PCI compliance and payment orchestration.

## Development Commands

### Project Setup
```bash
# Frontend (Next.js)
cd frontend && npm install
npm run dev

# Backend (NestJS)
cd backend && npm install
npm run start:dev

# Full stack development
npm run dev
```

### Testing
```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run e2e tests
npm run test:e2e
```

### Code Quality
```bash
# Lint code
npm run lint

# Type check
npm run typecheck

# Format code
npm run format
```

### Build & Deploy
```bash
# Build for production
npm run build

# Start production server
npm run start
```

## Project Structure

```
/
├── frontend/          # Next.js storefront and admin
├── backend/           # NestJS API server
├── shared/            # Shared types and utilities
├── docs/              # Documentation
├── TDD.md             # Technical design document
├── BACKLOG.md         # Development backlog
└── CLAUDE.md          # This file
```

## Key Architectural Patterns

### Payment Abstraction Layer
- All payment providers implement a common interface
- Abstract payload format for all payment methods
- Provider adapters handle specific integration details

### Security & Compliance
- SAQ-A PCI compliance (no PAN data on servers)
- Tokenization via provider hosted fields/elements
- TLS everywhere, least-privilege access

### Demo Constraints
- In-memory storage (no persistent database)
- Static product catalog from JSON
- Client-side cart management

## Payment Provider Integrations

### Supported Providers
- **Stripe**: Cards, wallets (client-confirm and tokenize modes)
- **Braintree**: Cards (tokenize mode)
- **PayPal**: Wallet payments
- **Plaid + Dwolla**: ACH transfers
- **Internal**: Gift cards

### Mock Providers (MVP)
- **SusieBank**: Mock ACH provider
- **SusiePay**: Mock digital wallet
- **SusieGift**: Mock gift card provider

## Common Development Tasks

### Adding a New Payment Method
1. Create provider adapter in `backend/src/providers/`
2. Add frontend component in `frontend/src/components/payment/`
3. Register component in frontend registry
4. Update integration config endpoint
5. Add webhook handler if needed

### Adding New API Endpoints
1. Create controller in `backend/src/controllers/`
2. Add service logic in `backend/src/services/`
3. Define DTOs in `backend/src/dto/`
4. Add route documentation

### Frontend Development
- Uses Next.js 14+ with App Router
- Tailwind CSS for styling
- TypeScript throughout
- Client-side cart management

### Backend Development
- NestJS with TypeScript
- In-memory data store
- Provider abstraction pattern
- Webhook handling with signature verification

## Important Files to Know

### Configuration
- `frontend/src/config/` - Frontend configuration
- `backend/src/config/` - Backend configuration
- `products.json` - Static product catalog

### Core Components
- `frontend/src/components/payment/` - Payment method components
- `backend/src/providers/` - Payment provider adapters
- `backend/src/services/payment.service.ts` - Core payment logic

### Types & Interfaces
- `shared/types/` - Shared TypeScript definitions
- API contracts and payment schemas

## Development Best Practices

### Code Style
- Follow existing TypeScript conventions
- Use ESLint and Prettier configurations
- Maintain consistent naming patterns

### Security
- Never log sensitive payment data
- Validate all inputs server-side
- Use provider SDKs for PCI compliance

### Testing
- Unit tests for business logic
- Integration tests for payment flows
- Mock external provider calls

## Debugging & Troubleshooting

### Common Issues
- Provider sandbox vs production keys
- Webhook signature verification
- CORS configuration for payment elements

### Logging
- Structured JSON logs
- Payment flow tracking with correlation IDs
- Error handling with provider-specific details

## Environment Variables

```bash
# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Braintree
BRAINTREE_MERCHANT_ID=...
BRAINTREE_PUBLIC_KEY=...
BRAINTREE_PRIVATE_KEY=...

# PayPal
PAYPAL_CLIENT_ID=...
PAYPAL_CLIENT_SECRET=...

# Plaid
PLAID_CLIENT_ID=...
PLAID_SECRET=...

# Dwolla
DWOLLA_KEY=...
DWOLLA_SECRET=...
```

## MVP Priority

Focus on MVP items first (unlabeled items in BACKLOG.md):
1. Core payment abstraction
2. Basic storefront with cart
3. Stripe card payments
4. Mock providers (SusieBank, SusiePay, SusieGift)
5. Basic admin dashboard
6. Order management

Non-MVP items are marked as "(not MVP)" in the backlog and should be completed after MVP is functional.

## Need Help?

Refer to:
- `TDD.md` for detailed technical specifications
- `BACKLOG.md` for current development tasks
- Provider documentation for integration details
- This file for development workflows