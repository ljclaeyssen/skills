# Raider.io API

Third-party API for Mythic+ scores and gear data. **No authentication required** — fully public.

Base URL: `https://raider.io/api/v1`

## Character Profile

`GET /characters/profile`

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `region` | query | Yes | `us`, `eu`, `kr`, `tw` |
| `realm` | query | Yes | Realm slug (e.g. `ysondre`) |
| `name` | query | Yes | Character name, lowercase |
| `fields` | query | No | Comma-separated list of extra data to include |

### Available Fields

| Field | Description |
|-------|-------------|
| `mythic_plus_scores_by_season:current` | Current season M+ scores by role |
| `mythic_plus_scores_by_season:previous` | Previous season M+ scores |
| `gear` | Currently equipped gear with item levels |

### Example

```
GET https://raider.io/api/v1/characters/profile?region=eu&realm=ysondre&name=lonjon&fields=mythic_plus_scores_by_season:current,gear
```

### Response

Base fields (always returned):
- `name`, `race`, `class`, `active_spec_name`, `active_spec_role`
- `gender`, `faction`, `achievement_points`, `honorable_kills`
- `thumbnail_url`, `profile_url`, `profile_banner`
- `region`, `realm`, `last_crawled_at`

#### M+ Scores (`mythic_plus_scores_by_season`)

```json
{
  "season": "season-tww-2",
  "scores": {
    "all": 2875.4,
    "dps": 1500.2,
    "healer": 1375.2,
    "tank": 0,
    "spec_0": 1375.2,
    "spec_1": 0,
    "spec_2": 1500.2,
    "spec_3": 0
  }
}
```

- `all` = best score across all roles
- `dps`, `healer`, `tank` = best score per role
- `spec_0`–`spec_3` = per-specialization scores

#### Gear (`gear`)

```json
{
  "item_level_equipped": 266,
  "item_level_total": 0,
  "items": {
    "head": { "item_id": 123, "item_level": 266, "name": "...", "item_quality": 4, "icon": "...", "enchant": null, "gems": [], "bonuses": [] },
    ...
  }
}
```

Slots: `head`, `neck`, `shoulder`, `back`, `chest`, `shirt`, `tabard`, `wrist`, `hands`, `waist`, `legs`, `feet`, `finger1`, `finger2`, `trinket1`, `trinket2`, `mainhand`, `offhand`.

`item_quality`: 1=Common, 2=Uncommon, 3=Rare, 4=Epic, 5=Legendary.

### Error Response

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Could not find character..."
}
```

Returns HTTP 400 for characters not in Raider.io's database (never crawled or very inactive).

### Rate Limits

No official rate limit documentation. Recommended:
- Max 10 requests/second
- Timeout: 5000ms
- Cache responses (data updates on crawl, not in real-time)

### Raider.io vs Blizzard M+ Data

| | Blizzard API | Raider.io |
|--|-------------|-----------|
| Auth | Bearer token | None |
| Score | `mythic_rating` (Blizzard's formula) | `scores.all` (Raider.io's formula) |
| Party data | Full party with specs, ilvl, races | Not in profile endpoint |
| Run details | Full best runs per season | Not in profile endpoint |
| Update speed | Near real-time | On crawl (can lag) |
| Gear snapshot | Per-slot via Equipment endpoint | Simplified per-slot |

Use Blizzard for detailed M+ run data. Use Raider.io for quick M+ score lookups without auth.
