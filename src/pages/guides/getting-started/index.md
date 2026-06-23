# Getting started

Before calling any Adobe Partner Retail API, your backend must obtain an OAuth 2.0 access token from Adobe Identity Management Services (IMS). This token authenticates your server-to-server requests.

## Prerequisites

Before you begin, ensure you have the following  details from Adobe partner onboarding:

| Item | Description |
|---|---|
| `CLIENT_ID` | The unique client identifier for your application. |
| `CLIENT_SECRET` | The secret credential for your application. Keep this secure and never expose it on the client side. |
| `SCOPES` | The OAuth scopes granted to your integration, provided during onboarding. |
| `offer_id` | One or more offer IDs mapped to your campaigns, provided by your Adobe Partner Delivery and Operations contact. |
| Outbound IP addresses | Required for sandbox access only. See [IP allowlisting](#ip-allowlisting-sandbox-only) below. |

If you do not have these credentials, contact your Adobe Partner Delivery and Operations representative to initiate partner onboarding.

## IP allowlisting (sandbox only)

To access the Adobe sandbox or stage Redemption UI URL, also known as `experience_url`, your outbound IP addresses must be allowlisted before integration testing can begin.

**What to do:**

- Identify your backend outbound IP addresses.
- Share these IP addresses with your Adobe Partner Delivery and Operations representative during onboarding.
- Adobe will configure sandbox access to allow requests from the provided IP addresses.

**Note:** The production environment is not affected. IP allowlisting is required only for the sandbox or stage environment. Production is publicly accessible and does not require allowlisting.

## Step 1: obtain an access token

Your backend must call the Adobe IMS token endpoint using the `client_credentials` grant type.

**Endpoint**

| Environment | Token URL |
|---|---|
| Sandbox | `https://ims-na1-stg1.adobelogin.com/ims/token/v3` |
| Production | `https://ims-na1.adobelogin.com/ims/token/v3` |

**Request**

```bash
curl -X POST 'https://ims-na1.adobelogin.com/ims/token/v3' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}&grant_type=client_credentials&scope={SCOPES}'
```

**Response**

```json
{
    "access_token": "{ACCESS_TOKEN}",
    "token_type": "bearer",
    "expires_in": 86399
}
```

The following table provides details of the response parameters:

| Field | Description |
|---|---|
| `access_token` | Bearer token to include in subsequent API calls. |
| `token_type` | Always `bearer`. |
| `expires_in` | Token validity in seconds ( approximately 24 hours). |

## Step 2: use the token in API requests

Include the access token as a `Bearer` token in the `Authorization` header for every API request:

```
Authorization: Bearer {ACCESS_TOKEN}
X-API-Key: {CLIENT_ID}
```

Both headers are required. Requests that are missing either header will be rejected with `401 Unauthorized` or `403 Forbidden`.

## Token lifecycle

- Tokens are valid for approximately 24 hours (`expires_in: 86399` seconds).
- Cache the token and request a new one only when it is close to expiry or has expired.
- Token requests are partner scoped. A token issued for your credentials authorizes access only to your own subscriptions and campaigns. Cross-partner access is rejected.

## Security best practices

- Do not expose `CLIENT_SECRET` on the client side. Token generation must happen server-to-server.
- Make all API calls over HTTPS.
- Store credentials securely, for example, in a secrets manager, not in source code.
- Do not log redemption codes from the `experience_url` in plain text.

## Next steps

After obtaining an access token, proceed to call the [Product Claim API](../api/index.md) to initiate a product claim.

- [Adobe Partner Retail APIs](../api/index.md)
