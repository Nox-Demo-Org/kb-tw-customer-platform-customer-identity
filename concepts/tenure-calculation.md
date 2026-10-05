---
type: Concept
title: Customer Tenure Calculation
description: In Tidewell Mutual, customer tenure measures the duration of a customer's relationship with the company in whole years.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/concepts/tenure-calculation.md
tags:
- customer-identity
- concepts
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

# Customer Tenure Calculation

In Tidewell Mutual, customer tenure measures the duration of a customer's relationship with the company in whole years. As established in [[decisions/adr-0004-tenure|ADR-0004]], `customer-identity` is the single source of truth for tenure calculation across all services and business domains (including marketing, pricing, and contact centre systems).

## Business Rules

Tenure calculation follows these key rules:
1. **Whole Years**: Tenure is measured as completed whole years since the customer's initial policy start date (`CustomerSince` / `customer_since`).
2. **Policy Gap Tolerance**: Gaps under 90 days between policies do not reset customer tenure.
3. **Reset Threshold**: Gaps of 90 days or longer between active policies reset the tenure count, starting the count over from the new policy date.
4. **Single Source of Truth**: External services must not implement independent tenure calculations and must consume `tenure_years` from `customer-identity`.

## Implementation Details

The tenure calculation logic is implemented in `internal/customers/customer.go` via the `Tenure` function:

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

### Calculation Mechanics
- Calculates the raw difference in calendar years (`now.Year() - since.Year()`).
- Checks if the current day of the year (`now.YearDay()`) has reached or passed the anniversary day (`since.YearDay()`). If not, decrements the year count by 1.
- Clamps the result so that negative values return `0`.

## API and Model Exposure

Calculated tenure is exposed on the [[entities/customer|Customer]] model and returned in the profile API response:

- **Entity Fields** (`internal/customers/customer.go`):
  - `CustomerSince` (`time.Time`, JSON key `customer_since`): The baseline timestamp from which tenure is measured.
  - `TenureYears` (`int`, JSON key `tenure_years`): The evaluated whole-year count.
- **REST Endpoint**:
  - `GET /v1/customers/{id}` returns the customer profile including `customer_since` and `tenure_years`. See [[summaries/api-spec]].

## Downstream Consumption

Consumers such as `customer-portal`, `claims-intake`, and `billing-service` retrieve `tenure_years` directly via `GET /v1/customers/{id}` rather than deriving tenure from historical policy dates.
