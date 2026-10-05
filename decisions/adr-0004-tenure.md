---
type: Architecture Decision
title: 'ADR-0004: Standardizing Company-Wide Tenure Calculation Rules'
description: Accepted (2024-06)
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/decisions/adr-0004-tenure.md
tags:
- customer-identity
- decisions
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

# ADR-0004: Standardizing Company-Wide Tenure Calculation Rules

## Status

Accepted (2024-06)

## Context

Historically across Tidewell Mutual, multiple departments and downstream systems calculated customer loyalty and tenure independently:
- **Marketing**, **pricing**, and the **contact centre** each calculated "years with us" using disparate business logic.
- Discrepancies occurred when customers renewed late or experienced short coverage gaps between policy terms, leading to inconsistent customer experiences, inaccurate pricing tiers, and mismatched loyalty rewards.

To eliminate diverging rules, `customer-identity` needed to establish a single source of truth for customer tenure.

## Decision

The [[index|customer-identity]] service owns the authoritative, company-wide definition of customer tenure:

1. **Definition**: Tenure represents the number of whole years since the start date of the customer's first policy (`CustomerSince` / `customer_since`).
2. **Gap Tolerance**: A gap of under 90 days between policies does not reset the tenure count. A gap of 90 days or longer starts the count again.
3. **Calculation Rule**: Implemented in Go via `Tenure(since, now time.Time)` in `internal/customers/customer.go`, calculating:
   ```go
   years := now.Year() - since.Year()
   if now.YearDay() < since.YearDay() {
       years--
   }
   if years < 0 {
       return 0
   }
   ```
4. **API Exposure**: Returned as the `tenure_years` integer field on the profile payload via `GET /v1/customers/{id}` (see [[summaries/api-spec]] and [[entities/customer]]).
5. **Single Source of Truth**: Other services across Tidewell Mutual must not compute tenure independently and must consume the calculated `tenure_years` value from `customer-identity`.

## Consequences

- **Centralized Logic**: Detailed calculation behavior and edge cases are documented in [[concepts/tenure-calculation]].
- **Data Model**: The `Customer` model and the `customers` database table record `customer_since` and expose `tenure_years` (see [[entities/customer]]).
- **Downstream Simplicity**: Pricing, marketing, and contact centre systems no longer maintain policy timeline aggregation logic and instead rely strictly on `GET /v1/customers/{id}`.
