---
okf_version: '0.2'
title: customer-identity
description: customer-identity is the central authentication and customer profile service for Tidewell Mutual.
generated:
  at: '2026-10-05T12:42:07Z'
---

# customer-identity

`customer-identity` is the central authentication and customer profile service for Tidewell Mutual. Owned by the **Customer Platform** team, it provides sign-in authentication, JWT token issuance and key rotation, customer master profile data, and tenure calculation.

### Main Components
- **Authentication & Token Service (`internal/auth`)**: Validates credentials and issues 15-minute access tokens and 30-day refresh tokens signed with ES256, exposing a JWKS endpoint for local token verification by downstream services.
- **Customer Profile Domain (`internal/customers`)**: Manages customer record attributes (`id`, `name`, `email`, `phone`, `postcode`, `customer_since`, `tenure_years`) and standardizes company-wide customer tenure calculations.
- **Event Publisher (`internal/events`)**: Publishes [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]] events over Google Cloud Pub/Sub whenever core customer attributes change.
- **HTTP Service Entrypoint (`cmd/identity`)**: Configures routing for auth and customer profile REST endpoints.

### Key Architectural Decisions
- **Unified Tenure Definition (ADR-0004)**: Centralized tenure logic where gaps under 90 days between policies do not reset customer tenure, preventing discrepancies between marketing, pricing, and contact centre systems.
- **Decentralized Token Verification**: Signs tokens with ES256 using rotated Secret Manager keys and exposes public keys via `/.well-known/jwks.json`, allowing other services to verify access tokens locally without round-trips.

<!-- okf:contents -->

## Contents

- [Concepts and flows](/concepts/index.md) — 3 pages. Flows, lifecycles and cross-cutting mechanisms.
- [Architecture decisions](/decisions/index.md) — 1 page. One ADR per architecture decision the code or documents make evident.
- [Components and data models](/entities/index.md) — 2 pages. One page per significant component and core data model.
- [Interfaces and references](/summaries/index.md) — 1 page. API, event and module references for the application.
- [Change log](/log.md) — every generation and sync, newest first.

<!-- /okf:contents -->
