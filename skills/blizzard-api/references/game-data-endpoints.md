# Game Data Endpoints — Professions

These endpoints use the `static-{region}` namespace (not `profile`). They return game data that changes per patch, not per player action.

---

## Profession Index

`GET /data/wow/profession/index`
Namespace: `static-{region}`

Returns the list of all professions in the game.

Example: `GET https://eu.api.blizzard.com/data/wow/profession/index?namespace=static-eu`

Response: `{ professions: [{ key, name, id }] }`

Common profession IDs:
| Profession | ID |
|-----------|-----|
| Alchemy | 171 |
| Blacksmithing | 164 |
| Enchanting | 333 |
| Engineering | 202 |
| Herbalism | 182 |
| Inscription | 773 |
| Jewelcrafting | 755 |
| Leatherworking | 165 |
| Mining | 186 |
| Skinning | 393 |
| Tailoring | 197 |
| Cooking | 185 |
| Fishing | 356 |

---

## Profession Detail

`GET /data/wow/profession/{professionId}`
Namespace: `static-{region}`

Returns profession info and all skill tiers (one per expansion).

Example: `GET https://eu.api.blizzard.com/data/wow/profession/202?namespace=static-eu`

Key fields:
- `id`, `name`, `description?`
- `type` — PRIMARY / SECONDARY
- `skill_tiers[]` — each with:
  - `key`, `name`, `id`
  - `minimum_skill_level`, `maximum_skill_level`

Use `skill_tiers[].id` with the Skill Tier Detail endpoint to get recipes.

---

## Skill Tier Detail

`GET /data/wow/profession/{professionId}/skill-tier/{skillTierId}`
Namespace: `static-{region}`

Returns all recipes for a specific expansion tier of a profession, grouped by category.

Example: `GET https://eu.api.blizzard.com/data/wow/profession/202/skill-tier/2499?namespace=static-eu`

Key fields:
- `name` — tier name (e.g. "Kul Tiran Engineering / Zandalari Engineering")
- `minimum_skill_level`, `maximum_skill_level`
- `categories[]` — each with:
  - `name` — category name (e.g. "Bombs", "Devices", "Weapons")
  - `recipes[]` — list of recipes (name + id)
