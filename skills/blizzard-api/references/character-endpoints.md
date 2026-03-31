# Character Endpoints

All character endpoints use namespace `profile-{region}` and share these path parameters:

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `realmSlug` | path | Yes | Realm slug, **lowercase** (e.g. `ysondre`, `area-52`) |
| `characterName` | path | Yes | Character name, **lowercase** (e.g. `lonjon`) |
| `namespace` | query | Yes | `profile-{region}` (e.g. `profile-eu`) |
| `locale` | query | No | e.g. `en_US`, `fr_FR` — without it, names are localized objects |

Base path: `GET https://{region}.api.blizzard.com/profile/wow/character/{realmSlug}/{characterName}`

---

## Character Profile

`GET /profile/wow/character/{realmSlug}/{characterName}`

Returns the character's basic info: name, class, race, level, item levels, guild, last login.

Key response fields:
- `id` — unique character ID
- `name`, `gender`, `faction`, `race`, `character_class`, `active_spec`
- `level`, `experience`, `achievement_points`
- `average_item_level`, `equipped_item_level`
- `last_login_timestamp` — Unix milliseconds
- `guild?` — guild name, id, realm, faction (absent if guildless)
- `active_title?` — currently displayed title with `display_string`
- `covenant_progress?` — Shadowlands only (chosen covenant, renown level)
- `is_remix` — true for MoP Remix characters
- Lazy `BlizzardHref` links to all sub-resources (equipment, encounters, etc.)

---

## Character Profile Status

`GET /profile/wow/character/{realmSlug}/{characterName}/status`

Returns validity and unique ID. Use to detect deleted/transferred characters.

Response: `{ id: number, is_valid: boolean }`

Delete cached character data if: HTTP 404, `is_valid: false`, or `id` changed.

---

## Character Media

`GET /profile/wow/character/{realmSlug}/{characterName}/character-media`

Returns 3 render URLs:
- `avatar` — small portrait
- `inset` — medium portrait
- `main-raw` — full character render

Response: `{ assets: [{ key: 'avatar'|'inset'|'main-raw', value: string }] }`

---

## Character Appearance

`GET /profile/wow/character/{realmSlug}/{characterName}/appearance`

Returns race, class, gender, active spec, faction, and optional `guild_crest` (emblem, border, background with RGBA colors).

---

## Character Equipment

`GET /profile/wow/character/{realmSlug}/{characterName}/equipment`

Returns all equipped items with full detail.

Key fields per item:
- `slot.type` — `HEAD`, `NECK`, `SHOULDER`, `BACK`, `CHEST`, `SHIRT`, `TABARD`, `WRIST`, `HANDS`, `WAIST`, `LEGS`, `FEET`, `FINGER_1`, `FINGER_2`, `TRINKET_1`, `TRINKET_2`, `MAIN_HAND`, `OFF_HAND`
- `quality.type` — `POOR`, `COMMON`, `UNCOMMON`, `RARE`, `EPIC`, `LEGENDARY`, `ARTIFACT`, `HEIRLOOM`
- `name`, `level.value` (item level), `media` (icon ref)
- `stats[]` — array of stat objects with `type`, `value`, `display`
- `enchantments[]` — with `display_string` and `enchantment_id`
- `bonus_list[]` — internal modifier IDs
- `armor?`, `sell_price?`, `requirements?`, `durability?`

---

## Character Statistics

`GET /profile/wow/character/{realmSlug}/{characterName}/statistics`

Returns all combat stats reflecting currently equipped gear.

Key fields:
- `health`, `power`, `power_type`
- Primary stats: `strength`, `agility`, `intellect`, `stamina` (each has `base` and `effective`)
- Secondary ratings: `melee_crit`, `melee_haste`, `mastery`, `lifesteal`, `dodge`, `parry`, `block`, `spell_crit` (each has `value` %, `rating_bonus`, `rating_normalized`)
- `versatility`, `versatility_damage_done_bonus`, `versatility_healing_done_bonus`
- `attack_power`, `spell_power`
- Weapon: `main_hand_damage_min/max`, `main_hand_speed`, `main_hand_dps` (same for off_hand)
- `armor` (base + effective)
- `speed`, `avoidance` (rating only)

---

## Character Specializations

`GET /profile/wow/character/{realmSlug}/{characterName}/specializations`

Returns all specs with talent loadouts.

Key fields:
- `active_specialization` — currently active spec
- `specializations[]` — one per spec, each with:
  - `loadouts[]` — talent builds, with `is_active`, `talent_loadout_code` (importable string)
  - `selected_class_talents[]`, `selected_spec_talents[]`, `selected_hero_talents[]` (TWW 11.0+)
  - Each talent: `id`, `rank`, `tooltip` (spell name, description, cast time)
  - `pvp_talent_slots[]` — PvP-specific talents

---

## Character Professions

`GET /profile/wow/character/{realmSlug}/{characterName}/professions`

Returns primary (up to 2) and secondary (Cooking, Fishing, Archaeology) professions.

Each profession has `tiers[]` — one per expansion, with:
- `skill_points`, `max_skill_points`
- `tier.name`, `tier.id`
- `known_recipes[]` — list of learned recipes (name + id)

---

## Character Achievements

`GET /profile/wow/character/{realmSlug}/{characterName}/achievements`

Returns all achievements with completion timestamps.

Key fields:
- `total_quantity`, `total_points`
- `achievements[]` — each with `id`, `achievement` (name, id), `completed_timestamp`, optional `criteria`

---

## Character Achievement Statistics

`GET /profile/wow/character/{realmSlug}/{characterName}/achievements/statistics`

Returns achievement-related statistics (kill counts, distances walked, etc.).

---

## Character Reputations

`GET /profile/wow/character/{realmSlug}/{characterName}/reputations`

Returns all faction standings.

Each reputation has:
- `faction` — name and id
- `standing` — `raw`, `value`, `max`, `tier` (0=Hated → 7=Exalted), `name` ("Friendly", "Honored", etc.)

Some TWW renown factions use different standing names.

---

## Character Titles

`GET /profile/wow/character/{realmSlug}/{characterName}/titles`

Returns:
- `active_title?` — currently displayed title with `display_string` (e.g. `"{name}, Chronoscholar"`)
- `titles[]` — all earned titles (name + id)

---

## Character Quests

`GET /profile/wow/character/{realmSlug}/{characterName}/quests`

Returns:
- `in_progress[]` — currently active quests (name + id)
- `completed` — href to paginated completed quests list

### Completed Quests

`GET /profile/wow/character/{realmSlug}/{characterName}/quests/completed`

Returns the full list of completed quest IDs.

---

## Character PvP Summary

`GET /profile/wow/character/{realmSlug}/{characterName}/pvp-summary`

Returns:
- `honor_level` — prestige honor level
- `honorable_kills` — lifetime HKs
- `pvp_map_statistics[]` — per-battleground/arena win/loss records

### PvP Bracket Statistics

`GET /profile/wow/character/{realmSlug}/{characterName}/pvp-bracket/{pvpBracket}`

Returns per-bracket rated stats. Returns 404 if the character has no rated games in that bracket.

Bracket values: `2v2`, `3v3`, `rbg`, `shuffle-{classSlug}-{specSlug}` (Solo Shuffle).

---

## Character Encounters

`GET /profile/wow/character/{realmSlug}/{characterName}/encounters`

Entry point — returns hrefs to raids and dungeons sub-resources.

### Raid Encounters

`GET /profile/wow/character/{realmSlug}/{characterName}/encounters/raids`

Hierarchical structure: `expansions[]` → `instances[]` → `modes[]` → `encounters[]`

Each mode has:
- `difficulty.type` — `LFR`, `NORMAL`, `HEROIC`, `MYTHIC`
- `status.type` — `COMPLETE`, `IN_PROGRESS`
- `progress` — `completed_count`, `total_count`, per-boss `encounters[]` with `completed_count` and `last_kill_timestamp`

Includes ALL expansions the character has ever raided.

### Dungeon Encounters

`GET /profile/wow/character/{realmSlug}/{characterName}/encounters/dungeons`

Same hierarchical structure as raids. Includes dungeons and delves.

---

## Mythic Keystone Profile

`GET /profile/wow/character/{realmSlug}/{characterName}/mythic-keystone-profile`

Returns:
- `current_period` — current weekly period reference
- `seasons[]` — all seasons the character has participated in

### Mythic Keystone Season

`GET /profile/wow/character/{realmSlug}/{characterName}/mythic-keystone-profile/season/{seasonId}`

Returns 404 if no M+ runs for that season.

Key fields:
- `mythic_rating.rating` — overall M+ score for the season
- `best_runs[]` — best timed runs, each with:
  - `dungeon` (name, id), `keystone_level`, `duration` (milliseconds)
  - `is_completed_within_time` — true = timed
  - `mythic_rating` — score contribution from this run
  - `members[]` — all 5 party members with `name`, `realm`, `specialization`, `race`, `equipped_item_level`
  - `completed_timestamp` — Unix ms

---

## Character Collections

`GET /profile/wow/character/{realmSlug}/{characterName}/collections`

Entry point — returns hrefs to each collection type: `pets`, `mounts`, `heirlooms`, `toys`, `transmogs`, `decors`.

### Collection Sub-endpoints

`GET /profile/wow/character/{realmSlug}/{characterName}/collections/{type}`

Where `{type}` is: `mounts`, `pets`, `toys`, `heirlooms`, `transmogs`, `decor`.

Returns the list of collected items for that type.

---

## Soulbinds (Legacy)

`GET /profile/wow/character/{realmSlug}/{characterName}/soulbinds`

Shadowlands-specific. Returns soulbind choices and conduits. Still functional but legacy content.

---

## Hunter Pets

`GET /profile/wow/character/{realmSlug}/{characterName}/hunter-pets`

Returns 404 if the character is not a Hunter. Returns the list of tamed pets with names, creature info, and active status.

---

## Character House

`GET /profile/wow/character/{realmSlug}/{characterName}/house/house-{houseNumber}`

Returns 404 if the character has no house. Returns house details including placed furniture and decorations.
