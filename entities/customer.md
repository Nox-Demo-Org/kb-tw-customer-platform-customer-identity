---
type: Component
title: Customer Entity
description: The Customer entity represents the master customer profile in customer-identity, encapsulating customer identity attributes, contact details, and policy tenure.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/entities/customer.md
tags:
- customer-identity
- entities
sources:
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/internal/customers/customer.go
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/README.md
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/docs/adr/0004-tenure.md
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

<!-- anchor: internal/customers/customer.go:L1-L27 -->
<!-- anchor: README.md:L1-L13 -->
<!-- anchor: docs/adr/0004-tenure.md:L1-L5 -->

# Customer Entity

The `Customer` entity represents the master customer profile in `customer-identity`, encapsulating customer identity attributes, contact details, and policy tenure. It is defined in `internal/customers/customer.go` and is backed by the `customers` database table.

## Data Model

The `Customer` struct is the payload returned by the `GET /v1/customers/{id}` endpoint (see [[summaries/api-spec]]).

```go
type Customer struct {
	ID            string    `json:"id"`
	Name          string    `json:"name"`
	Email         string    `json:"email"`
	Phone         string    `json:"phone"`
	Postcode      string    `json:"postcode"`
	CustomerSince time.Time `json:"customer_since"`
	TenureYears   int       `json:"tenure_years"`
}
```

### Fields

| Field | Type | JSON Key | Description |
| --- | --- | --- | --- |
| `ID` | `string` | `id` | Unique identifier for the customer record. |
| `Name` | `string` | `name` | Full name of the customer. |
| `Email` | `string` | `email` | Customer email address. |
| `Phone` | `string` | `phone` | Customer contact telephone number. |
| `Postcode` | `string` | `postcode` | Customer postal code. |
| `CustomerSince` | `time.Time` | `customer_since` | Timestamp indicating when the customer's first policy started. |
| `TenureYears` | `int` | `tenure_years` | Calculated tenure represented as whole years since `CustomerSince`. |

## Tenure Calculation

Customer tenure is calculated via the `Tenure(since, now time.Time) int` function in `internal/customers/customer.go`.

In accordance with [[decisions/adr-0004-tenure]] (see also [[concepts/tenure-calculation]]):
- Tenure is counted as whole years since `CustomerSince`.
- Gaps under 90 days between policies do not reset the tenure count.
- Gaps of 90 days or longer restart the tenure count.
- Other services must not calculate tenure independently; they rely on the calculated `TenureYears` attribute served by `customer-identity`.

```go
func Tenure(since, now time.Time) int {
	years := now.Year() - since.Year()
	if now.YearDay() < since.YearDay() {
		years--
	}
	if years < 0 {
		return 0
	}
	return years
}
```

## Responsibilities

- **Master Customer Record**: Serves as the authoritative source of customer contact and profile attributes across Tidewell Mutual.
- **Tenure Standardization**: Computes and supplies the standardized `tenure_years` metric to ensure consistent pricing, marketing, and contact centre experiences.
- **Profile Queries**: Exposes profile information over REST via `GET /v1/customers/{id}` for consuming applications such as `claims-intake`, `billing-service`, and `customer-portal`.
- **Change Propagation**: Triggers notifications across downstream services (such as `policy-admin` and [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]]) via the [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]] Pub/Sub event (see [[concepts/profile-events]]).

## Dependencies

- **Database Table**: `customers` table stores the core profile fields and initial start dates (`CustomerSince`).
- **Standard Library**: Relies on Go's standard `time` package for `time.Time` handling and calendar-based tenure evaluation.
- **Consumers**:
  - `GET /v1/customers/{id}`: Consumed by `claims-intake`, `billing-service`, and `customer-portal`.
  - [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]]: Consumed by `policy-admin` and downstream services when profile information changes.
