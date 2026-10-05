---
type: Interface Reference
title: API Specification
description: This document details the REST endpoints and Pub/Sub event interfaces exposed by customer-identity.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/summaries/api-spec.md
tags:
- customer-identity
- summaries
sources:
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/cmd/identity/main.go
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/internal/auth/token.go
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/internal/customers/customer.go
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

<!-- anchor: cmd/identity/main.go:L1-L6 -->
<!-- anchor: internal/auth/token.go:L1-L16 -->
<!-- anchor: internal/customers/customer.go:L1-L27 -->

# API Specification

This document details the REST endpoints and Pub/Sub event interfaces exposed by `customer-identity`.

---

## REST Endpoints

### 1. Exchange Credentials for Tokens
- **Route**: `POST /v1/auth/token`
- **Primary Consumer**: `customer-portal`
- **Related Documentation**: [[entities/auth-token]], [[concepts/token-signing-jwks]]

Exchanges an email and password for an access token and a refresh token. Tokens are signed with **ES256** using asymmetric keys from Google Cloud Secret Manager that rotate every 30 days.

#### Request Body (`auth.TokenRequest`)
| Field | Type | Description |
|---|---|---|
| `email` | `string` | Customer login email address |
| `password` | `string` | Customer plain-text password |

#### Token Lifetimes
- **Access Token TTL (`AccessTTL`)**: 15 minutes (`15 * time.Minute`)
- **Refresh Token TTL (`RefreshTTL`)**: 30 days (`30 * 24 * time.Hour`)

---

### 2. Retrieve Customer Profile
- **Route**: `GET /v1/customers/{id}`
- **Primary Consumers**: `claims-intake`, `billing-service`, `customer-portal`
- **Database Table**: `customers`
- **Related Documentation**: [[entities/customer]], [[concepts/tenure-calculation]], [[decisions/adr-0004-tenure]]

Returns the master customer profile including calculated tenure based on policy history.

#### Path Parameters
| Parameter | Type | Description |
|---|---|---|
| `id` | `string` | Unique customer identifier |

#### Response Body (`customers.Customer`)
```json
{
  "id": "cust_12345",
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "phone": "+447700900077",
  "postcode": "SW1A 1AA",
  "customer_since": "2019-04-12T00:00:00Z",
  "tenure_years": 5
}
```

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Unique customer identifier |
| `name` | `string` | Full name of the customer |
| `email` | `string` | Customer email address |
| `phone` | `string` | Contact phone number |
| `postcode` | `string` | Postal code |
| `customer_since` | `string` (RFC 3339 timestamp) | Initial policy inception date |
| `tenure_years` | `integer` | Whole years of tenure since `customer_since`, calculated per [[decisions/adr-0004-tenure]] |

---

### 3. JSON Web Key Set (JWKS)
- **Route**: `GET /.well-known/jwks.json`
- **Related Documentation**: [[concepts/token-signing-jwks]]

Exposes the active public keys used to sign access tokens with ES256. Downstream services use this endpoint to verify JWTs locally without making network calls back to `customer-identity`.

---

## Event Topics

### [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]]
- **Topic Name**: [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]] (`TopicProfileUpdated`)
- **Transport**: Google Cloud Pub/Sub
- **Active Consumers**: `policy-admin` (see also [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]])
- **Related Documentation**: [[concepts/profile-events]], [[entities/customer]]

Emitted when a customer's core profile attributes change. Note that `billing-service` and `notifications-hub` read customer profiles on demand via REST rather than subscribing directly to this topic.

#### Message Payload
```json
{
  "customer_id": "cust_12345",
  "changed": ["email", "phone"],
  "updated_at": "2024-03-15T10:30:00Z"
}
```

| Field | Type | Allowed Values / Description |
|---|---|---|
| `customer_id` | `string` | Unique customer ID |
| `changed` | `array[string]` | List of modified fields: `"email"`, `"phone"`, `"address"` |
| `updated_at` | `string` (RFC 3339) | Timestamp when the change occurred |
