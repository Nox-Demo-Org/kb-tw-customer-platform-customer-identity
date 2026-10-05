---
type: Concept
title: Customer Profile Events
description: The customer-identity service uses Google Cloud Pub/Sub to asynchronously notify downstream services when customer profile data changes.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-customer-identity/blob/main/concepts/profile-events.md
tags:
- customer-identity
- concepts
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:07Z'
---

# Customer Profile Events

The `customer-identity` service uses Google Cloud Pub/Sub to asynchronously notify downstream services when customer profile data changes. Event publishing is managed within `internal/events`.

## Topic: [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]]

The constant `TopicProfileUpdated` in `internal/events/profile.go` defines the Pub/Sub topic name:

```go
const TopicProfileUpdated = "customer.profile.updated"
```

### Event Payload

When customer profile attributes change, the publisher emits a message containing the modified attributes:

```json
{
  "customer_id": "string",
  "changed": ["email" | "phone" | "address"],
  "updated_at": "string (timestamp)"
}
```

#### Fields
- `customer_id`: Identifier of the updated [[entities/customer|customer]].
- `changed`: List of attributes modified during the update, containing one or more of `"email"`, `"phone"`, or `"address"`.
- `updated_at`: Timestamp indicating when the change occurred.

---

## Consumer Expectations and Architecture

Different downstream systems handle customer profile updates using distinct integration patterns:

### Event Subscribers
- **`policy-admin`**: Subscribes directly to [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]] to synchronize policyholder contact information across active insurance policies.

### On-Demand Consumers
Not all services subscribe to the Pub/Sub topic. Some systems pull customer information synchronously via the REST API (`GET /v1/customers/{id}`) as needed:
- **`billing-service`**: Reads customer profiles on demand via REST (see [[summaries/api-spec]] and [[ap:kb-tw-billing-billing-service/summaries/events-spec#customer-profile-updated|billing-service (customer.profile.updated)]]).
- **`notifications-hub`**: Reads customer profile and contact details on demand via REST.

---

## Related Documentation
- [[entities/customer]] – Customer profile data model and schema
- [[summaries/api-spec]] – REST API contracts including `GET /v1/customers/{id}`
- [[index]] – Service architecture overview
