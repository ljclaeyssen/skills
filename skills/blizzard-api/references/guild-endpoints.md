# Guild Endpoints

All guild endpoints use namespace `profile-{region}`.

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `realmSlug` | path | Yes | Realm slug, lowercase (e.g. `ysondre`) |
| `nameSlug` | path | Yes | Guild name slug, lowercase, **hyphens for spaces** (e.g. `blood-legion`) |
| `namespace` | query | Yes | `profile-{region}` (e.g. `profile-eu`) |
| `locale` | query | No | e.g. `en_US`, `fr_FR` |

Base path: `GET https://{region}.api.blizzard.com/data/wow/guild/{realmSlug}/{nameSlug}`

---

## Guild Profile

`GET /data/wow/guild/{realmSlug}/{nameSlug}`

Returns basic guild info.

Key fields:
- `id`, `name`, `faction` (ALLIANCE/HORDE)
- `achievement_points`, `member_count`
- `realm` — name, id, slug
- `created_timestamp` — Unix ms
- `crest` — guild emblem, border, background (each with id, media ref, RGBA color)
- Lazy hrefs to `roster`, `achievements`, `activity`

---

## Guild Roster

`GET /data/wow/guild/{realmSlug}/{nameSlug}/roster`

Returns the full member list.

Each member has:
- `rank` — 0 = Guild Master, 1–9 = custom ranks
- `character`:
  - `name`, `id`, `level`
  - `realm` (id + slug, no name)
  - `playable_class` — `BlizzardKeyedRef` (id only, no name — resolve via static data)
  - `playable_race` — `BlizzardKeyedRef` (id only)
  - `faction.type` — `HORDE` or `ALLIANCE`

The roster does NOT include character names for very low-level alts that haven't logged in recently.

---

## Guild Activity

`GET /data/wow/guild/{realmSlug}/{nameSlug}/activity`

Returns the recent activity feed. Currently returns primarily `CHARACTER_ACHIEVEMENT` events.

Each activity entry:
- `activity.type` — e.g. `"CHARACTER_ACHIEVEMENT"`
- `timestamp` — Unix ms
- `character_achievement`:
  - `character` — name, id, realm
  - `achievement` — name (localized), id

Strings in this endpoint use `BlizzardLocalizedString` (all 12 locales as an object, unless `locale` param is set).

---

## Guild Achievements

`GET /data/wow/guild/{realmSlug}/{nameSlug}/achievements`

Returns guild achievement progress.

Key fields:
- `total_quantity`, `total_points`
- `achievements[]` — each with:
  - `achievement` — name, id
  - `completed_timestamp?` — present only if completed
  - `criteria`:
    - `id`, `amount?`, `is_completed`
    - `child_criteria[]` — sub-objectives with their own `amount` and `is_completed`
