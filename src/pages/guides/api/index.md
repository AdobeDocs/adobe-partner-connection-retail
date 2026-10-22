# Adobe Partner Retail APIs

Adobe Partner Connection Retail exposes the following partner-facing APIs:

| API | Endpoint | Method | What It Does |
|---|---|---|---|
| [Product Claim API](product-claim.md) | `/v1/workflows` | POST | Initiates a product claim on behalf of a customer. Returns an `experience_url` that takes the customer into Adobe to sign in, consent, and gain access. |
| [Get Subscription API](get-subscription.md) | `/v1/subscriptions/{partner_reference_id}` | GET | Retrieves the current status, offer, and term dates for a subscription. |
| [Update Subscription API](update-subscription.md) | `/v2/subscriptions/{partner_reference_id}` | POST | Renews or cancels a subscription through a single API with two operation types. |

## Base URLs

| Environment | Base URL |
|---|---|
| Sandbox (Stage) | `https://partners-stage.adobe.io/retail` |
| Production | `https://partners.adobe.io/retail` |

## Authentication

All requests must include the following headers:

```text
Authorization: Bearer {ACCESS_TOKEN}
X-API-Key: {CLIENT_ID}
```

See [Getting Started](../getting-started/index.md) for instructions on obtaining an access token.

## Related Documents

- [Overview](../index.md)
- [Getting started](../getting-started/index.md)
- [Support](../support/index.md)
