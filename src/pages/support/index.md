---
title: Support - Adobe Partner Connection Retail
description: This is the support page of Adobe Partner Connection Retail Program
---

<Superhero slots="heading, text" background="rgb(19, 93, 183)"/>

# Support

For questions about the Adobe Partner Connection Retail integration, contact your Adobe Partner Connection Retail engineering team. They are your primary point of contact for onboarding, configuration, and ongoing operational support.

If you do not have the engineering team contact, reach out to your Adobe Business Development Manager to be connected with the right team.

## Onboarding support

The following items are handled through the engineering team representative during onboarding. Raise these requests before beginning integration testing.

| Request | Who to contact |
|---|---|
| OAuth credentials (`CLIENT_ID`, `CLIENT_SECRET`, scopes) | Adobe Partner Connection Retail engineering team. |
| `offer_id` configuration and campaign mapping | Adobe Partner Connection Retail engineering team |
| IP allowlisting for sandbox environment | Adobe Partner Connection Retail engineering team |
| Sandbox access and test accounts | Adobe Partner Connection Retail engineering team |

## Common issues

### Authentication errors

| Symptom | Likely cause | Resolution |
|---|---|---|
| `401 Unauthorized` | Missing or expired access token | Re-authenticate using `/ims/token/v3` and retry with the new token. |
| `403 Forbidden` | Valid token but wrong scope or missing `X-API-Key` | Verify both `Authorization: Bearer` and `X-API-Key` headers are present. Confirm scopes with your PDO representative. |
| Token not working in sandbox | IP address not allowlisted | Provide your outbound IP address to your PDO representative for allowlisting. |

### Product claim errors

| Error code | Meaning | What to do |
|---|---|---|
| `ALREADY_FULFILLED` (HTTP 409) | Customer has already redeemed this offer | Do not re-issue. The customer has an active subscription. |
| `SUBSCRIPTION_ID_ALREADY_IN_USE` (HTTP 400) | `partner_reference_id` was used with a different `offer_id` | Generate a new `partner_reference_id` for this customer and offer combination. |
| `INVALID_OFFER` (HTTP 400) | `offer_id` is not recognized | Verify the `offer_id` against values provided during onboarding. Contact your PDO representative if the issue persists. |

### Customer redemption issues

If a customer reports they cannot redeem or access their Adobe product, ask them to provide their redemption code, the `rc` value visible in the `experience_url`. This code is the primary reference for Adobe Customer Support to locate and resolve the subscription.

## Raising a support request

When contacting support, include the following to speed up resolution:

- Your `CLIENT_ID` (do not share your `CLIENT_SECRET`)
- The `partner_reference_id` and `offer_id` involved
- The full API response, including any error code and message
- The environment (sandbox or production)
- Timestamp of the request

## Related documents

- [Overview](../guides/index.md)
- [Getting Started](../guides/getting-started/index.md)
- [Adobe Partner Retail APIs](../api/index.md)
