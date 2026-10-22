# Update Subscription API

The Update Subscription API supports renewal and cancellation through one API. Set `type` to `RENEW` or `CANCEL` to select the operation. Both variants, including cancellation policies and handling modes, are available in sandbox.

| Endpoint | Method |
|---|---|
| `/v2/subscriptions/{partner_reference_id}` | POST |

Use the [base URL](index.md#base-urls) for your environment.

Both operations are asynchronous. HTTP `202 Accepted` confirms that Adobe accepted the request, not that processing has completed. Store the returned `transaction_id` for tracking. Poll the [Get Subscription API](get-subscription.md) to check the resulting status and term dates. If your integration already uses optional Notify callbacks, those notifications can also track completion.

## Common request parameters

**Headers**

| Header | Value |
|---|---|
| `Authorization` | `Bearer {ACCESS_TOKEN}` |
| `X-API-Key` | `{CLIENT_ID}` |
| `Content-Type` | `application/json` |

**Path parameter**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `partner_reference_id` | string | Yes | The partner's unique tracking identifier for the customer's product subscription. |

## Renew a subscription

Use `RENEW` to renew an active subscription.

**Body**

```json
{
  "type": "RENEW",
  "offer_id": "{OFFER_ID}"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | Set to `RENEW`. |
| `offer_id` | string | Yes | The Adobe offer ID for the renewal. |

### Renewal window and term dates

A renewal is allowed only from seven days before the current term ends (T-7) through five days after it ends (T+5), where T is the current term's `end_date`. The subscription must be `ACTIVE`.

The renewed term starts at the current term's end date, regardless of when the request is submitted within this window. This keeps consecutive terms contiguous, including renewals submitted during the five-day grace period.

If the subscription is not renewed, it remains active during the five-day grace period after the term ends. Adobe automatically cancels the subscription after the five-day grace period ends. If your integration receives the optional cancellation notification, this automatic cancellation is identified by `initiated_by: SYSTEM`.

### Renewal response

Status: Accepted (HTTP 202)

```json
{
  "type": "RENEW",
  "transaction_id": "8d997812-6fa1-4b42-8859-50cdd581bd61"
}
```

| Field | Type | Description |
|---|---|---|
| `type` | string | `RENEW`. |
| `transaction_id` | string (UUID) | Identifier for the accepted asynchronous operation. |

### Renewal response codes

| HTTP Status | Code or condition | Action |
|---|---|---|
| `202` | Renewal accepted. | Store the `transaction_id` and track completion. |
| `400` | `SUBSCRIPTION_NOT_ACTIVE` | Renewal requires an `ACTIVE` subscription. Check its current status. |
| `400` | `OFFER_ID_REQUIRED` | Supply `offer_id`. |
| `400` | `RENEWAL_WINDOW_VIOLATION` | Submit renewal within the T-7 through T+5 window. |
| `400` | Invalid or missing `type`. | Set `type` to `RENEW`. |
| `401` | Missing or invalid authentication. | Obtain a valid access token and retry. |
| `403` | Insufficient permissions. | Verify your partner credentials and permissions. |
| `404` | Subscription not found. | Verify the `partner_reference_id`. |
| `409` | `TRANSITION_ALREADY_IN_PROGRESS` | Track the in-progress transition before submitting another update. |
| `5xx` | Internal Adobe error. | Check subscription state before retrying an update. Contact support if the error persists. |

## Cancel a subscription

Use `CANCEL` to cancel a subscription. Adobe applies the default cancellation policy unless your partner integration has cancellation-mode overrides enabled.

**Body**

```json
{
  "type": "CANCEL",
  "reason_code": "USER_CANCELLED"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | Set to `CANCEL`. |
| `reason_code` | string | Yes | `USER_CANCELLED` for a user-initiated cancellation, or `NOT_ELIGIBLE` for a partner-initiated cancellation because the customer is no longer eligible or is no longer with the vendor. |
| `mode` | string | No | `IMMEDIATE` or `DEFERRED`. Honored only when Adobe has enabled cancellation-mode overrides for your partner integration; otherwise ignored and the default policy applies. |

### Cancellation handling modes

| Mode | Behavior |
|---|---|
| `IMMEDIATE` | Cancels the subscription immediately. |
| `DEFERRED` | Keeps the subscription available until `end_date`, with no further renewal. Its status is `CANCELLING` until the term ends and then changes to `CANCELLED`. |

Partners with overrides enabled can include `mode` in the same cancellation request:

```json
{
  "type": "CANCEL",
  "reason_code": "USER_CANCELLED",
  "mode": "DEFERRED"
}
```

### Default cancellation policy

When `mode` is omitted, or your integration does not have overrides enabled, Adobe applies the following policy.

| Subscription condition | Cancellation mode |
|---|---|
| Free offer. | `IMMEDIATE` |
| Paid offer within 14 days of the first paid purchase. | `IMMEDIATE` |
| Past the term end, within the five-day grace period. | `IMMEDIATE` |
| Paid offer more than 14 days after the first paid purchase, but still within the current term. | `DEFERRED`, effective at `end_date`, with no further renewal. |

The 14-day policy is measured from the **first paid purchase**, not from each renewal.

### Cancellation response

Status: Accepted (HTTP 202)

```json
{
  "type": "CANCEL",
  "transaction_id": "b7af314c-0678-4cd1-9157-dbe42486942e"
}
```

| Field | Type | Description |
|---|---|---|
| `type` | string | `CANCEL`. |
| `transaction_id` | string (UUID) | Identifier for the accepted asynchronous operation. |

Canceling a subscription that is already `CANCELLED` is idempotent and returns the original `transaction_id`. A subscription that is already `CANCELLING` instead returns HTTP `409` with `ALREADY_CANCELLING`.

### Cancellation response codes

| HTTP Status | Code or condition | Action |
|---|---|---|
| `202` | Cancellation accepted. | Store the `transaction_id` and track completion. |
| `400` | Invalid or missing `type`, or invalid `mode`. | Set `type` to `CANCEL`. If supplied, `mode` must be `IMMEDIATE` or `DEFERRED`. |
| `401` | Missing or invalid authentication. | Obtain a valid access token and retry. |
| `403` | Insufficient permissions. | Verify your partner credentials and permissions. |
| `404` | Subscription not found. | Verify the `partner_reference_id`. |
| `409` | `ALREADY_CANCELLING` | Track the existing cancellation rather than submitting another request. |
| `5xx` | Internal Adobe error. | Check subscription state before retrying an update. Contact support if the error persists. |

## Related documents

- [API Reference](index.md)
- [Get Subscription API](get-subscription.md)
- [Getting started](../getting-started/index.md)
- [Support](../support/index.md)
