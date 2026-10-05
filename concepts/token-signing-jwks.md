---
type: Concept
title: ES256 Token Signing and JWKS Verification
description: customer-identity implements a decentralized token issuance and verification architecture.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/concepts/token-signing-jwks.md
tags:
- customer-identity
- concepts
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

# ES256 Token Signing and JWKS Verification

`customer-identity` implements a decentralized token issuance and verification architecture. Access tokens issued by the service are cryptographically signed using asymmetric cryptography, allowing downstream services across Tidewell Mutual to verify tokens locally without making synchronous validation requests back to `customer-identity`.

## Signing Algorithm and Key Management

Token signing is handled in `internal/auth/token.go`:

* **Signing Algorithm**: Tokens are signed using **ES256** (ECDSA using the P-256 curve and SHA-256).
* **Key Storage**: Cryptographic private keys are retrieved from **Secret Manager**.
* **Key Rotation**: Signing keys stored in Secret Manager are rotated every **30 days**.

## Token Lifecycles

Defined in `internal/auth/token.go`, the authentication lifecycle uses two token tiers:

| Token Type | Constant | Duration | Description |
| :--- | :--- | :--- | :--- |
| **Access Token** | `AccessTTL` | `15 * time.Minute` (15 minutes) | Short-lived ES256-signed JWT used for authenticating API requests across Tidewell services. |
| **Refresh Token** | `RefreshTTL` | `30 * 24 * time.Hour` (30 days) | Long-lived credential used to obtain new access tokens. |

For detailed token data structures and models, see [[entities/auth-token]].

## Decentralized Verification via JWKS

To eliminate runtime coupling and network round-trips for every authenticated request:

1. `customer-identity` exposes its public keys at `GET /.well-known/jwks.json` (see [[summaries/api-spec]]).
2. Downstream services fetch and cache the JSON Web Key Set (JWKS) from this endpoint.
3. Downstream services locally verify the ES256 signature and expiration (`exp`) of inbound access tokens.
4. When keys are rotated every 30 days, downstream consumers refresh their cached key sets based on the key identifiers (`kid`) present in incoming token headers.

## Token Issuance

Users authenticate via `POST /v1/auth/token` by submitting credentials mapped to the `TokenRequest` struct:

```go
type TokenRequest struct {
	Email    string `json:"email"`
	Password string `json:"password"`
}
```

Upon successful authentication, the service signs and returns the access token alongside the refresh token.

## Related Topics

* [[entities/auth-token]] — Token request structures and cryptographic token lifecycles.
* [[summaries/api-spec]] — API specification for `POST /v1/auth/token` and `GET /.well-known/jwks.json`.
* [[index]] — Overview of the `customer-identity` service architecture.
