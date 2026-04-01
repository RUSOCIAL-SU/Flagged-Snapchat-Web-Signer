# Attest Flagged API

**Base URL:** `https://flagged-snapweb-attest.rusocial.io`

## `POST /token`

Generates a base64-encoded Snapchat web attestation token.

### Request

```json
{
  "username": "BvqMOcgHyY",
  "proxy": "http://user:pass@host:port"
}
```

| Field      | Type     | Required | Description                                           |
|------------|----------|----------|-------------------------------------------------------|
| `username` | `string` | Yes      | Snapchat username to embed in the token (1-64 chars). |
| `proxy`    | `string` | Yes      | Proxy URL for the bootstrap request.                  |

### Response `200`

```json
{
  "token_b64": "AQAB...",
  "username": "BvqMOcgHyY"
}
```

| Field       | Type     | Description                       |
|-------------|----------|-----------------------------------|
| `token_b64` | `string` | Base64-encoded attestation token. |
| `username`  | `string` | Username embedded in the token.   |

### Errors

| Status | Description                                |
|--------|--------------------------------------------|
| `422`  | Invalid or missing request body fields.    |
| `500`  | Token construction failed.                 |
| `502`  | Bootstrap request to Snapchat failed.      |

---

To buy the **unflagged API**, join our Discord: [discord.gg/rusocial](https://discord.gg/rusocial)
