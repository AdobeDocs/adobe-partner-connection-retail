# Adobe Partner Retail APIs

Adobe Partner Connection Retail exposes the following partner-facing API:

| API | What It Does |
|---|---|
| [Product Claim API](#product-claim-api) | Initiates a product claim on behalf of a customer. Returns an `experience_url` that takes the customer into Adobe to sign in, consent, and gain access. |

## Base URLs

| Environment | Base URL |
|---|---|
| Sandbox (Stage) | `https://partners-stage.adobe.io/retail` |
| Production | `https://partners.adobe.io/retail` |

## Authentication

All requests must include the following headers:

```
Authorization: Bearer {ACCESS_TOKEN}
X-API-Key: {CLIENT_ID}
```

See [Getting Started](../getting-started/index.md) for instructions on obtaining an access token.

## Product Claim API

The Product Claim API initiates a product claim for a customer. Adobe resolves the campaign, sources and binds a redemption code, and returns a ready-to-use redemption URL with the code embedded.

|Endpoint|Method
|---|--|
| /v1/workflows| POST |

### Request

**Headers**

| Header | Value |
|---|---|
| `Authorization` | `Bearer {ACCESS_TOKEN}` |
| `X-API-Key` | `{CLIENT_ID}` |
| `Content-Type` | `application/json` |

**Body**

```json
```json
{
  "workflow_type": "CLAIM_PRODUCT",
  "workflow_attributes": [
    {
      "key": "partner_reference_id",
      "value": "{YOUR_UNIQUE_REFERENCE_ID}"
    },
    {
      "key": "offer_id",
      "value": "{OFFER_ID}"
    }
  ]
}
```

The following table provides details of the request parameters:

| Field | Type | Required | Description |
|---|---|---|---|
| `workflow_type` | string | Yes | Always `CLAIM_PRODUCT` for product redemption workflows. |
| `workflow_attributes` | array of objects | Yes | List of attribute objects (key/value) required by the workflow. Must include `partner_reference_id` and `offer_id`. |
| `workflow_attributes[].key` | string | Yes | Attribute name. Required keys: `partner_reference_id`, `offer_id`. |
| `workflow_attributes[].value` | string | Yes | Attribute value. For `partner_reference_id`, provide your unique identifier for the customer and offer combination. For `offer_id`, provide the Adobe offer ID assigned during onboarding. |

> **Important:** `partner_reference_id` must be unique per customer per offer. Re-sending the same `partner_reference_id` with the same `offer_id` is treated as a duplicate (see [Duplicate Handling](#duplicate-handling-and-error-scenarios) below). Sending the same `partner_reference_id` with a *different* `offer_id` returns an error.

### Response

Status: Success (HTTP 200)

Adobe returns an `experience_url` containing the pre-populated redemption code in the `rc` parameter. Redirect the customer to this URL.

**Stage response example:**

```json
{
  "workflow_type": "CLAIM_PRODUCT",
  "experience_url": "https://redeem-stg.adobe.com/express-premium?asm=cs&rc=NHYT-6Y3Y-KMAF-OB6B-HDE5-J6F4&pid={your-partner-reference-id}",
  "short_experience_url": "https://www.stage.adobe.com/go/s-retail?uuid=zxXJczyeP",
  "workflow_attributes": [
    {
      "key": "partner_reference_id",
      "value": "sub_003_test23"
    },
    {
      "key": "offer_id",
      "value": "30006521"
    }
  ]
}
```

**Production response example:**

```json
{
  "workflow_type": "CLAIM_PRODUCT",
  "experience_url": "https://redeem.adobe.com/express-premium?asm=cs&rc=NHYT-6Y3Y-KMAF-OB6B-HDE5-J6F4&pid=indosat&uuid=zxXJczyeP",
  "short_experience_url": "https://www.adobe.com/go/retail?uuid=zxXJczyeP",
  "workflow_attributes": [
    {
      "key": "partner_reference_id",
      "value": "sub_003_test23"
    },
    {
      "key": "offer_id",
      "value": "30006521"
    }
  ]
}
```

| Field | Description |
|---|---|
| `experience_url` | The full Adobe Redemption UI URL. Contains the redemption code in the `rc` query parameter. Redirect the customer here. |
| `short_experience_url` | A shortened URL. Useful for SMS or character-limited delivery channels. |
| `rc` parameter | The redemption code embedded in the `experience_url`. Store this code for business reporting and as the customer reference for Adobe support. |

### Delivering the URL to the Customer

You can deliver the `experience_url` using one of the following modes.

**Mode A: Direct Redirect (HTTP 200)**

Your backend receives the `experience_url` and your frontend loads it directly in the customer's browser.

```
Customer browser → Partner App → [Partner Backend calls API] → Redirect to experience_url
```

**Mode B: Server-Side Redirect (HTTP 302)**

If your integration cannot load the URL client-side, Adobe can respond with an HTTP 302 redirect directly to the Redemption UI. Contact your Adobe PDO representative to enable this mode for your integration.

### Customer Redemption Flow

After the customer is redirected to the `experience_url`, the following steps occur:

- Customer signs in or creates an Adobe account. The redemption code is pre-populated.
- Customer provides consent to share activation details.
- Entitlement is provisioned and the subscription is activated.
- Customer is redirected to the product and is automatically signed in.

**Note:** No callback is sent to your backend after activation. For affinity and OEM programs, the redemption code in the experience_url is the source of truth for tracking.

## Duplicate Handling and Error Scenarios

The API handles duplicate and invalid requests deterministically. Your integration must handle the following scenarios.

### Scenario 1: New Request (Success)

A new `partner_reference_id` + `offer_id` combination. Returns HTTP 200 with `experience_url`.

### Scenario 2: Duplicate Request, Not Yet Redeemed (HTTP 202)

The same `partner_reference_id` + `offer_id` is submitted again, but the customer has not yet redeemed the code.

Adobe returns the same `experience_url`. The operation is  idempotent, no new code is issued.

> Handle this gracefully: retry or re-display the same URL to the customer.

### Scenario 3: Already Fulfilled (HTTP 409)

The redemption code for this `partner_reference_id` + `offer_id` has already been redeemed by the customer.

```json
{
  "code": "ALREADY_FULFILLED",
  "message": "<>"
}
```

Do not re-issue a new code. The customer already has an active subscription.

### Scenario 4: Reference ID Already In Use (HTTP 400)

The `partner_reference_id` was previously used with a **different** `offer_id`.

```json
{
  "code": "SUBSCRIPTION_ID_ALREADY_IN_USE",
  "message": "<>"
}
```

> Each `partner_reference_id` must be unique per customer per offer. Generate a new `partner_reference_id` if the customer is claiming a different product.

### Scenario 5: Invalid Offer (HTTP 400)

The `offer_id` is not recognized or not mapped to your partner configuration.

```json
{
  "code": "INVALID_OFFER",
  "message": "<>"
}
```

Verify the `offer_id` against the values provided during onboarding. Contact your PDO representative if the issue persists.

## Error Response Summary

| HTTP Status | Description | Action |
|---|---|---|
| `200` | Success. New code issued. | Redirect customer to `experience_url`. |
| `202` | Duplicate, not yet redeemed. | Re-use the returned `experience_url`. |
| `400` | `partner_reference_id` used with a different `offer_id`. | Use a new `partner_reference_id`. |
| `400` | `offer_id` not recognized. | Verify `offer_id` from onboarding. |
| `401` | Missing or invalid access token. | Re-authenticate and retry. |
| `403` | Token valid but not authorized for this resource. | Verify `X-API-Key` and partner scope. |
| `409` | Customer already redeemed this offer. | Do not re-issue. |

### Non-Functional Expectations

| Requirement | Target |
|---|---|
| Response time | < 2 seconds (p95), including code sourcing and URL construction |
| Availability | 99.99% uptime |
| Protocol | HTTPS only |
| Token validity | Approximately 24 hours. Cache and reuse until near expiry |

## Related Documents

- [Overview](../index.md)
- [Getting started](../getting-started/index.md)
- [Support](../support/index.md)
