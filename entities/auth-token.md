---
type: Component
title: Auth Token
description: The auth entity definitions in internal/auth/token.go model authentication credential payloads, token lifespans, and key management configurations for the customer-identity authentication service.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/entities/auth-token.md
tags:
- customer-identity
- entities
sources:
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/internal/auth/token.go
- resource: https://github.com/Nox-Demo-Org/customer-identity/blob/HEAD/cmd/identity/main.go
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

<!-- anchor: internal/auth/token.go:L1-L16 -->
<!-- anchor: cmd/identity/main.go:L1-L6 -->

# Auth Token

The `auth` entity definitions in `internal/auth/token.go` model authentication credential payloads, token lifespans, and key management configurations for the `customer-identity` authentication service.

## Data Model

### TokenRequest

`TokenRequest` represents the JSON request payload submitted to the `POST /v1/auth/token` endpoint (handled in `cmd/identity/main.go`) to exchange user credentials for JWT access and refresh tokens.

| Field | Type | JSON Key | Description |
|---|---|---|---|
| `Email` | `string` | `email` | Customer or user email address |
| `Password` | `string` | `password` | User account password |

```go
type TokenRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
}
```

## Token Lifecycles & Constants

`internal/auth/token.go` defines the following lifecycles for issued authentication tokens:

*   **`AccessTTL` (`15 * time.Minute`)**: Lifespan for short-lived access tokens used to authorize requests across Tidewell services.
*   **`RefreshTTL` (`30 * 24 * time.Hour`)**: Lifespan (30 days) for refresh tokens used to obtain new access tokens without requiring re-authentication.

## Cryptographic Keys and Verification

*   **Signing Algorithm**: Access tokens are signed using `ES256`.
*   **Key Storage and Rotation**: Cryptographic signing keys are retrieved from Secret Manager and rotated every 30 days.
*   **JWKS Endpoint**: Corresponding public keys are exposed via the `GET /.well-known/jwks.json` endpoint. Downstream services use this endpoint to fetch public keys and perform local, stateless JWT validation without round-trip network calls to `customer-identity`. See [[concepts/token-signing-jwks]] and [[summaries/api-spec]].

## Responsibilities

*   Define the payload structure (`TokenRequest`) for incoming credential-exchange authentication requests.
*   Establish expiration bounds (`AccessTTL` and `RefreshTTL`) for issued access and refresh tokens.
*   Specify the cryptographic signing algorithm (`ES256`), key rotation frequency (30 days), and public key exposure mechanism (`/.well-known/jwks.json`).

## Dependencies

*   **HTTP Router (`cmd/identity/main.go`)**: Binds `TokenRequest` processing to the `POST /v1/auth/token` route.
*   **Secret Manager**: Provides secure storage and rotation for the private `ES256` keys used to sign access tokens.
*   **Downstream Consumers**: Rely on `/.well-known/jwks.json` to obtain public keys for token validation.
*   **Related Documentation**:
    *   [[concepts/token-signing-jwks]]: Deep dive on ES256 key rotation and JWKS verification.
    *   [[summaries/api-spec]]: HTTP route specifications and token exchange endpoints.
    *   [[entities/customer]]: Customer master data entity.
