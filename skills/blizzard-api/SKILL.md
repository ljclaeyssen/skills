---
name: blizzard-api
description: "Integrate the Blizzard Battle.net API and Raider.io API into web projects. Use this skill whenever the user wants to fetch World of Warcraft data: character profiles, equipment, mythic+ scores, raid progress, guild rosters, professions, PvP stats, achievements, collections, or any WoW game data. Also triggers for: 'Battle.net API', 'Blizzard OAuth', 'WoW API', 'raider.io', 'mythic+ score', 'guild roster', 'character equipment', 'item level', 'raid progress', 'wow character data', 'Battle.net token', 'namespace profile-eu', or any mention of programmatically accessing World of Warcraft game data."
---

# Blizzard Battle.net & Raider.io API Integration

This skill provides battle-tested patterns for integrating the Blizzard Battle.net API and Raider.io API into web applications. It covers authentication, all Profile API endpoints (character + guild), Game Data endpoints, and SPA integration patterns.

## Quick Start

1. Register an app at [develop.battle.net](https://develop.battle.net) to get `client_id` and `client_secret`
2. Get an OAuth2 token via Client Credentials flow
3. Call endpoints with `Authorization: Bearer {token}` and `?namespace={namespace}`

```bash
# Get token
TOKEN=$(curl -s -X POST "https://oauth.battle.net/token" \
  -d "grant_type=client_credentials" \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" | jq -r '.access_token')

# Fetch a character
curl -s "https://eu.api.blizzard.com/profile/wow/character/ysondre/lonjon?namespace=profile-eu&locale=en_US" \
  -H "Authorization: Bearer $TOKEN"
```

## API Overview

### Base URLs

| API | Base URL |
|-----|----------|
| Blizzard (EU) | `https://eu.api.blizzard.com` |
| Blizzard (US) | `https://us.api.blizzard.com` |
| Blizzard (KR/TW) | `https://{kr\|tw}.api.blizzard.com` |
| Blizzard (CN) | `https://gateway.battlenet.com.cn` |
| OAuth | `https://oauth.battle.net/token` |
| Raider.io | `https://raider.io/api/v1` |

### Namespaces

Every Blizzard API call requires a `namespace` query parameter:

| Namespace | Pattern | Use |
|-----------|---------|-----|
| Profile | `profile-{region}` | Character & guild data (live, changes frequently) |
| Static | `static-{region}` | Game data — items, professions, spells (updated per patch) |
| Dynamic | `dynamic-{region}` | Realms, connected realms, WoW token |

Region codes: `us`, `eu`, `tw`, `kr`.

### Rate Limits

| Limit | Value |
|-------|-------|
| Requests/second | 100 per client |
| Requests/hour | 36,000 per client |
| Token lifetime | ~24h (86,399s) |

### Authentication Modes

| Flow | Use case | Endpoints |
|------|----------|-----------|
| **Client Credentials** | Server-to-server, no user login | All Character & Guild endpoints |
| **Authorization Code** (OAuth) | User must authorize via Battle.net login | Account Profile endpoints (scope `wow.profile`) |

## Endpoint Catalog

### Character Endpoints — `profile-{region}`

Base path: `/profile/wow/character/{realmSlug}/{characterName}`

| Endpoint | Path suffix | Description |
|----------|-------------|-------------|
| Profile | _(none)_ | Name, class, race, level, ilvl, guild, last login |
| Status | `/status` | Validity check (`is_valid`, character ID) |
| Media | `/character-media` | Avatar, inset, main-raw render URLs |
| Appearance | `/appearance` | Race, class, gender, guild crest |
| Equipment | `/equipment` | All equipped items with stats, enchants, quality |
| Statistics | `/statistics` | Combat stats (str, agi, int, crit, haste, mastery...) |
| Specializations | `/specializations` | Talent loadouts, PvP talents, hero talents |
| Professions | `/professions` | Primary/secondary professions with tiers and recipes |
| Achievements | `/achievements` | All achievements with timestamps |
| Achievement Statistics | `/achievements/statistics` | Achievement-related statistics |
| Reputations | `/reputations` | Faction standings (Hated→Exalted) |
| Titles | `/titles` | Earned titles + active title |
| Quests | `/quests` | In-progress quests + link to completed |
| Completed Quests | `/quests/completed` | Full list of completed quest IDs |
| PvP Summary | `/pvp-summary` | Honor level, honorable kills, BG/arena stats |
| PvP Bracket | `/pvp-bracket/{pvpBracket}` | Per-bracket rated PvP stats |
| Encounters (index) | `/encounters` | Entry point for raid/dungeon data |
| Raid Encounters | `/encounters/raids` | Boss kills by expansion/instance/difficulty |
| Dungeon Encounters | `/encounters/dungeons` | Dungeon completion history |
| M+ Profile | `/mythic-keystone-profile` | Current period, list of seasons |
| M+ Season | `/mythic-keystone-profile/season/{seasonId}` | Best runs, M+ rating, party details |
| Collections (index) | `/collections` | Entry point for all collections |
| Collections (sub) | `/collections/{type}` | `mounts`, `pets`, `toys`, `heirlooms`, `transmogs`, `decor` (note: JSON key is `decors` with s, URL path is `decor` without) |
| Soulbinds | `/soulbinds` | Shadowlands soulbind data (legacy) |
| Hunter Pets | `/hunter-pets` | Hunter pet list (404 if not hunter) |
| House | `/house/house-{houseNumber}` | Player housing data |

### Account Endpoints (9) — OAuth required

These require an access token with scope `wow.profile` obtained via **Authorization Code Flow** (user login).

| Endpoint | Path |
|----------|------|
| Account Profile Summary | `/profile/user/wow` |
| Protected Character | `/profile/user/wow/protected-character/{realmId}-{characterId}` |
| Account Collections Index | `/profile/user/wow/collections` |
| Account Mounts | `/profile/user/wow/collections/mounts` |
| Account Pets | `/profile/user/wow/collections/pets` |
| Account Toys | `/profile/user/wow/collections/toys` |
| Account Heirlooms | `/profile/user/wow/collections/heirlooms` |
| Account Transmogs | `/profile/user/wow/collections/transmogs` |
| Account Decor | `/profile/user/wow/collections/decor` |

### Guild Endpoints (4) — `profile-{region}`

Base path: `/data/wow/guild/{realmSlug}/{nameSlug}`

| Endpoint | Path suffix | Description |
|----------|-------------|-------------|
| Profile | _(none)_ | Name, faction, member count, crest, timestamps |
| Roster | `/roster` | Full member list with ranks, classes, races |
| Activity | `/activity` | Recent achievement activity feed |
| Achievements | `/achievements` | Guild achievement progress with criteria |

### Game Data — Professions (3) — `static-{region}`

| Endpoint | Path | Description |
|----------|------|-------------|
| Profession Index | `/data/wow/profession/index` | List of all professions |
| Profession Detail | `/data/wow/profession/{professionId}` | Profession info + skill tiers |
| Skill Tier Detail | `/data/wow/profession/{professionId}/skill-tier/{skillTierId}` | Recipes by category |

### Raider.io (1) — No auth required

| Endpoint | Path | Description |
|----------|------|-------------|
| Character Profile | `/characters/profile?region=&realm=&name=&fields=` | M+ scores, gear snapshot |

## Reference Files

Read these as needed based on the task:

| File | When to read |
|------|-------------|
| `references/authentication.md` | Setting up OAuth, token management, interceptors |
| `references/character-endpoints.md` | Implementing character data features (profile, gear, M+, raids...) |
| `references/guild-endpoints.md` | Implementing guild features (roster, activity, achievements) |
| `references/game-data-endpoints.md` | Fetching static game data (professions, recipes) |
| `references/raider-io.md` | Integrating Raider.io M+ scores and gear data |
| `references/typescript-models.md` | TypeScript interfaces for all API responses |
| `references/spa-integration.md` | Auth service, HTTP interceptor, token caching patterns (Angular/React/Vue) |

## Common Pitfalls

1. **Character names must be lowercase** in URLs. `Lonjon` → `lonjon`.
2. **Guild name slugs use hyphens** for spaces: `Blood Legion` → `blood-legion`.
3. **`namespace` is mandatory** on every call — omitting it returns a 400 error.
4. **Without `locale`, field names are localized objects** (all 12 locales). Pass `locale=en_US` to get simple strings.
5. **Tokens expire after ~24h** — cache them and refresh 60s before expiry. On 401, re-authenticate and retry once.
6. **Some endpoints return 404 legitimately** — hunter-pets for non-hunters, pvp-bracket if no rated games, house if no house built.
7. **Timestamps are Unix milliseconds**, not seconds — divide by 1000 for JS `Date`.
8. **Raider.io is a third-party API** — no auth needed, but recommended max 10 req/s with 5s timeout.
