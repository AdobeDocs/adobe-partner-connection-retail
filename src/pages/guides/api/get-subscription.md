# Get Subscription API

The Get Subscription API retrieves subscription details using the `partner_reference_id` supplied by the partner. Use this API to check the subscription status and track the outcome of an asynchronous renewal or cancellation.

| Endpoint | Method |
|---|---|
| `/v1/subscriptions/{partner_reference_id}` | GET |

Use the [base URL](index.md#base-urls) for your environment.

## Request

Include the `Authorization` and `X-API-Key` headers described in [Authentication](index.md#authentication).

**Path parameter**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `partner_reference_id` | string | Yes | The partner's unique tracking identifier for the customer's product subscription. |

No request body is required.

## Response

Status: Success (HTTP 200)

```json
{
  "partner_reference_id": "sub_003_test23",
  "offer_id": "30006521",
  "status": "ACTIVE",
  "start_date": "2026-10-01T00:00:00Z",
  "end_date": "2026-11-01T00:00:00Z"
}
```

The values above are illustrative.

| Field | Type | Description |
|---|---|---|
| `partner_reference_id` | string | The partner's tracking identifier for the subscription. |
| `offer_id` | string | The Adobe offer associated with the subscription. |
| `status` | string | The current subscription state: `ACTIVE`, `CANCELLING`, or `CANCELLED`. |
| `start_date` | ISO 8601 DateTime | The start date and time of the subscription term. |
| `end_date` | ISO 8601 DateTime | The end date and time of the subscription term. |

| Status | Meaning |
|---|---|
| `ACTIVE` | The subscription is active. |
| `CANCELLING` | A deferred cancellation has been accepted. The subscription remains available until the end of the current term and then becomes `CANCELLED`. |
| `CANCELLED` | The subscription has been cancelled. |

### Status codes

| HTTP Status | Description | Action |
|---|---|---|
| `200` | Subscription retrieved successfully. | Read the status and term dates. |
| `400` | Invalid `partner_reference_id`. | Check the identifier before retrying. |
| `401` | Missing or invalid authentication. | Obtain a valid access token and retry. |
| `403` | Insufficient permissions. | Verify your partner credentials and permissions. |
| `404` | No subscription found for the identifier. | Verify the `partner_reference_id`. |
| `5xx` | Internal Adobe error. | Retry later. Contact support if the error persists. |

## Related documents

- [API Reference](index.md)
- [Update Subscription API](update-subscription.md)
- [Getting started](../getting-started/index.md)
- [Support](../support/index.md)
