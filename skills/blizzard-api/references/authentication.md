# Authentication

## Client Credentials Flow (server-to-server)

Used for all Character, Guild, and Game Data endpoints. No user interaction required.

### Token Request

```
POST https://oauth.battle.net/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}
```

Alternative with Basic auth header:
```
POST https://oauth.battle.net/token
Authorization: Basic {base64(clientId:clientSecret)}
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

### Token Response

```typescript
interface BlizzardOAuthToken {
  access_token: string;
  token_type: string;    // "bearer"
  expires_in: number;    // typically 86399 (~24h)
  scope?: string;
}
```

### Using the Token

All API calls require:
```
Authorization: Bearer {access_token}
```

Plus these query parameters:

| Param | Required | Description |
|-------|----------|-------------|
| `namespace` | Yes | Scopes the data: `profile-{region}`, `static-{region}`, or `dynamic-{region}` |
| `locale` | No | Response language: `en_US`, `fr_FR`, `de_DE`, `es_ES`, `it_IT`, `pt_BR`, `ru_RU`, `ko_KR`, `zh_TW`, `zh_CN` |

### Token Lifecycle

- Valid ~24h (86,399 seconds)
- Cache the token and refresh 60 seconds before expiry
- On HTTP 401: re-authenticate immediately, then retry the failed request once
- Never request a new token per API call — this wastes rate limit budget

### cURL Example

```bash
# Get token
TOKEN=$(curl -s -X POST "https://oauth.battle.net/token" \
  -d "grant_type=client_credentials" \
  -d "client_id=$BLIZZARD_CLIENT_ID" \
  -d "client_secret=$BLIZZARD_CLIENT_SECRET" | jq -r '.access_token')

# Use token
curl -s "https://eu.api.blizzard.com/profile/wow/character/ysondre/lonjon" \
  -G \
  --data-urlencode "namespace=profile-eu" \
  --data-urlencode "locale=en_US" \
  -H "Authorization: Bearer $TOKEN"
```

## Authorization Code Flow (OAuth — user login)

Required only for Account Profile endpoints (scope `wow.profile`). The user must authorize access to their Battle.net account.

### Steps

1. **Redirect** the user to:
   ```
   https://oauth.battle.net/authorize?client_id={CLIENT_ID}&scope=wow.profile&redirect_uri={REDIRECT_URI}&response_type=code
   ```

2. **Receive** the `code` at your redirect URI after user login

3. **Exchange** the code for a token:
   ```
   POST https://oauth.battle.net/token
   Content-Type: application/x-www-form-urlencoded

   grant_type=authorization_code&code={CODE}&redirect_uri={REDIRECT_URI}&client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}
   ```

The resulting token has the `wow.profile` scope and can access Account Profile endpoints.

## Namespaces

| Namespace | Pattern | Content | Freshness |
|-----------|---------|---------|-----------|
| Profile | `profile-{region}` | Character & guild data | Live, changes per login/action |
| Static | `static-{region}` | Game data (items, professions, spells) | Updated per game patch |
| Dynamic | `dynamic-{region}` | Realms, connected realms, WoW token | Real-time |

Region codes: `us`, `eu`, `tw`, `kr`.

Namespace can be passed as a query param (`?namespace=profile-eu`) or header (`Battlenet-Namespace: profile-eu`). Query param is more common and recommended.

## Rate Limits

| Limit | Value |
|-------|-------|
| Requests per second | 100 per client |
| Requests per hour | 36,000 per client |

Rate limit headers are not returned by the API — track usage client-side if needed.
