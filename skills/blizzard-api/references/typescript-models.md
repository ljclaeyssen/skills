# TypeScript Models

Complete TypeScript interfaces for all Blizzard Battle.net API responses.

## Table of Contents

1. [Common Types](#common-types)
2. [Character Profile](#character-profile)
3. [Character Media](#character-media)
4. [Character Appearance](#character-appearance)
5. [Character Collections](#character-collections)
6. [Character Equipment](#character-equipment)
7. [Character Statistics](#character-statistics)
8. [Mythic Keystone](#mythic-keystone)
9. [Encounters (Raids & Dungeons)](#encounters)
10. [Specializations](#specializations)
11. [Professions](#professions)
12. [Achievements](#achievements)
13. [Reputations](#reputations)
14. [Titles](#titles)
15. [Quests](#quests)
16. [PvP](#pvp)
17. [Guild](#guild)
18. [Game Data — Professions](#game-data--professions)
19. [Raider.io](#raiderio)

---

## Common Types

Shared primitives used across all Blizzard API responses.

```typescript
interface BlizzardHref {
  href: string;
}

interface BlizzardLinks {
  _links: { self: BlizzardHref };
}

interface BlizzardKeyedRef {
  key: BlizzardHref;
  id: number;
}

interface BlizzardNamedRef extends BlizzardKeyedRef {
  name: string;
}

interface BlizzardTypedValue {
  type: string;
  name: string;
}

interface BlizzardRealmRef extends BlizzardNamedRef {
  slug: string;
}

interface BlizzardRealmShort {
  key: BlizzardHref;
  id: number;
  slug: string;
}

interface BlizzardCharacterRef {
  key: BlizzardHref;
  name: string;
  id: number;
  realm: BlizzardRealmRef;
}

interface BlizzardColor {
  r: number; g: number; b: number; a: number;
}

interface BlizzardDisplayValue {
  display_string: string;
  color: BlizzardColor;
}

interface BlizzardLocalizedString {
  en_US: string; es_MX: string; pt_BR: string; de_DE: string;
  en_GB: string; es_ES: string; fr_FR: string; it_IT: string;
  ru_RU: string; ko_KR: string; zh_TW: string; zh_CN: string;
}

interface BlizzardLocalizedTypedValue {
  type: string;
  name: BlizzardLocalizedString;
}
```

---

## Character Profile

```typescript
interface BlizzardCharacterProfile extends BlizzardLinks {
  id: number;
  name: string;
  gender: BlizzardTypedValue;
  faction: BlizzardTypedValue;
  race: BlizzardNamedRef;
  character_class: BlizzardNamedRef;
  active_spec: BlizzardNamedRef;
  realm: BlizzardRealmRef;
  guild?: BlizzardCharacterGuild;
  level: number;
  experience: number;
  achievement_points: number;
  last_login_timestamp: number;       // Unix ms
  average_item_level: number;
  equipped_item_level: number;
  active_title?: BlizzardCharacterTitle;
  covenant_progress?: BlizzardCovenantProgress; // Shadowlands only
  is_remix: boolean;
  // Lazy hrefs to sub-resources
  achievements: BlizzardHref;
  titles: BlizzardHref;
  pvp_summary: BlizzardHref;
  encounters: BlizzardHref;
  media: BlizzardHref;
  specializations: BlizzardHref;
  statistics: BlizzardHref;
  mythic_keystone_profile: BlizzardHref;
  equipment: BlizzardHref;
  appearance: BlizzardHref;
  collections: BlizzardHref;
  reputations: BlizzardHref;
  quests: BlizzardHref;
  achievements_statistics: BlizzardHref;
  professions: BlizzardHref;
  name_search: string;
}

interface BlizzardCharacterGuild {
  key: BlizzardHref;
  name: string;
  id: number;
  realm: BlizzardRealmRef;
  faction: BlizzardTypedValue;
}

interface BlizzardCharacterTitle {
  key: BlizzardHref;
  name: string;
  id: number;
  display_string: string;
}

interface BlizzardCovenantProgress {
  chosen_covenant: BlizzardNamedRef;
  renown_level: number;
  soulbinds: BlizzardHref;
}
```

---

## Character Profile Status

```typescript
interface BlizzardCharacterStatus extends BlizzardLinks {
  id: number;
  is_valid: boolean;
}
```

---

## Character Media

```typescript
interface BlizzardCharacterMedia extends BlizzardLinks {
  character: BlizzardCharacterRef;
  assets: BlizzardMediaAsset[];
}

interface BlizzardMediaAsset {
  key: 'avatar' | 'inset' | 'main-raw';
  value: string; // URL to render.worldofwarcraft.com
}
```

---

## Character Appearance

```typescript
interface BlizzardCharacterAppearance extends BlizzardLinks {
  character: BlizzardCharacterRef;
  playable_race: BlizzardNamedRef;
  playable_class: BlizzardNamedRef;
  active_spec: BlizzardNamedRef;
  gender: BlizzardTypedValue;
  faction: BlizzardTypedValue;
  guild_crest?: BlizzardGuildCrest;
}

interface BlizzardGuildCrest {
  emblem: { id: number; media: BlizzardKeyedRef; color: { id: number; rgba: BlizzardColor } };
  border: { id: number; media: BlizzardKeyedRef; color: { id: number; rgba: BlizzardColor } };
  background: { color: { id: number; rgba: BlizzardColor } };
}
```

---

## Character Collections

```typescript
interface BlizzardCharacterCollections extends BlizzardLinks {
  character: BlizzardCharacterRef;
  pets: BlizzardHref;
  mounts: BlizzardHref;
  heirlooms: BlizzardHref;
  toys: BlizzardHref;
  transmogs: BlizzardHref;
  decors: BlizzardHref;
}
```

---

## Character Equipment

```typescript
interface BlizzardCharacterEquipment extends BlizzardLinks {
  character: BlizzardCharacterRef;
  equipped_items: BlizzardEquippedItem[];
}

interface BlizzardEquippedItem {
  item: BlizzardKeyedRef;
  slot: BlizzardTypedValue;
  quantity: number;
  context?: number;
  bonus_list?: number[];
  quality: BlizzardTypedValue;
  name: string;
  media: BlizzardKeyedRef;
  item_class: BlizzardNamedRef;
  item_subclass: BlizzardNamedRef;
  inventory_type: BlizzardTypedValue;
  binding?: BlizzardTypedValue;
  armor?: { value: number; display: BlizzardDisplayValue };
  stats?: BlizzardItemStat[];
  enchantments?: BlizzardEnchantment[];
  sell_price?: {
    value: number;
    display_strings: { header: string; gold: string; silver: string; copper: string };
  };
  requirements?: { level?: { value: number; display_string: string } };
  level?: { value: number; display_string: string };
  durability?: { value: number; display_string: string };
}

interface BlizzardEnchantment {
  display_string: string;
  enchantment_id: number;
  enchantment_slot: { id: number; type: string };
}

interface BlizzardItemStat {
  type: BlizzardTypedValue;
  value: number;
  is_equip_bonus?: boolean;
  display: BlizzardDisplayValue;
}
```

---

## Character Statistics

```typescript
interface BlizzardCharacterStatistics extends BlizzardLinks {
  health: number;
  power: number;
  power_type: BlizzardNamedRef;
  strength: BlizzardStatValue;
  agility: BlizzardStatValue;
  intellect: BlizzardStatValue;
  stamina: BlizzardStatValue;
  armor: BlizzardStatValue;
  melee_crit: BlizzardRatingStatValue;
  melee_haste: BlizzardRatingStatValue;
  mastery: BlizzardRatingStatValue;
  lifesteal: BlizzardRatingStatValue;
  versatility: number;
  versatility_damage_done_bonus: number;
  versatility_healing_done_bonus: number;
  versatility_damage_taken_bonus: number;
  dodge: BlizzardRatingStatValue;
  parry: BlizzardRatingStatValue;
  block: BlizzardRatingStatValue;
  spell_crit: BlizzardRatingStatValue;
  spell_power: number;
  attack_power: number;
  speed: BlizzardRatingValue;
  avoidance: BlizzardRatingValue;
  bonus_armor: number;
  main_hand_damage_min: number;
  main_hand_damage_max: number;
  main_hand_speed: number;
  main_hand_dps: number;
  off_hand_damage_min: number;
  off_hand_damage_max: number;
  off_hand_speed: number;
  off_hand_dps: number;
  mana_regen: number;
  mana_regen_combat: number;
  spell_penetration: number;
}

interface BlizzardStatValue {
  base: number;
  effective: number;
}

interface BlizzardRatingValue {
  rating_bonus: number;
  rating_normalized: number;
}

interface BlizzardRatingStatValue extends BlizzardRatingValue {
  value: number; // percentage
}
```

---

## Mythic Keystone

```typescript
interface BlizzardMythicKeystoneProfile extends BlizzardLinks {
  current_period: { period: BlizzardKeyedRef };
  seasons: BlizzardKeyedRef[];
  character: BlizzardCharacterRef;
}

interface BlizzardMythicKeystoneSeason extends BlizzardLinks {
  season: BlizzardKeyedRef;
  best_runs: BlizzardMythicKeystoneRun[];
  character: BlizzardCharacterRef;
  mythic_rating: BlizzardMythicRating;
}

interface BlizzardMythicKeystoneRun {
  completed_timestamp: number;
  duration: number;               // milliseconds
  keystone_level: number;
  dungeon: BlizzardNamedRef;
  is_completed_within_time: boolean;
  mythic_rating: BlizzardMythicRating;
  members: BlizzardMythicKeystoneMember[];
}

interface BlizzardMythicRating {
  color: BlizzardColor;
  rating: number;
}

interface BlizzardMythicKeystoneMember {
  character: { name: string; id: number; realm: BlizzardRealmShort };
  specialization: BlizzardNamedRef;
  race: BlizzardNamedRef;
  equipped_item_level: number;
}
```

---

## Encounters

```typescript
// Raids
interface BlizzardRaidEncounters extends BlizzardLinks {
  character: BlizzardCharacterRef;
  expansions: BlizzardRaidExpansion[];
}

interface BlizzardRaidExpansion {
  expansion: BlizzardNamedRef;
  instances: BlizzardRaidInstance[];
}

interface BlizzardRaidInstance {
  instance: BlizzardNamedRef;
  modes: BlizzardRaidMode[];
}

interface BlizzardRaidMode {
  difficulty: BlizzardTypedValue;  // LFR | NORMAL | HEROIC | MYTHIC
  status: BlizzardTypedValue;     // COMPLETE | IN_PROGRESS
  progress: {
    completed_count: number;
    total_count: number;
    encounters: BlizzardRaidEncounter[];
  };
}

interface BlizzardRaidEncounter {
  encounter: BlizzardNamedRef;
  completed_count: number;
  last_kill_timestamp: number; // Unix ms
}

// Dungeons — same structure
interface BlizzardDungeonEncounters extends BlizzardLinks {
  expansions: BlizzardDungeonExpansion[];
}

interface BlizzardDungeonExpansion {
  expansion: BlizzardNamedRef;
  instances: BlizzardDungeonInstance[];
}

interface BlizzardDungeonInstance {
  instance: BlizzardNamedRef;
  modes: BlizzardDungeonMode[];
}

interface BlizzardDungeonMode {
  difficulty: BlizzardTypedValue;
  status: BlizzardTypedValue;
  progress: {
    completed_count: number;
    total_count: number;
    encounters: BlizzardRaidEncounter[]; // same shape
  };
}
```

---

## Specializations

```typescript
interface BlizzardCharacterSpecializations extends BlizzardLinks {
  specializations: BlizzardSpecialization[];
  active_specialization: BlizzardNamedRef;
  character: BlizzardCharacterRef;
}

interface BlizzardSpecialization {
  specialization: BlizzardNamedRef;
  glyphs?: BlizzardNamedRef[];
  pvp_talent_slots?: BlizzardPvpTalentSlot[];
  loadouts: BlizzardTalentLoadout[];
}

interface BlizzardPvpTalentSlot {
  selected: {
    talent: BlizzardNamedRef;
    spell_tooltip: { spell: BlizzardNamedRef; description: string; cast_time: string };
  };
  slot_number: number;
}

interface BlizzardTalentLoadout {
  is_active: boolean;
  talent_loadout_code: string; // importable string
  selected_class_talents: BlizzardSelectedTalent[];
  selected_spec_talents: BlizzardSelectedTalent[];
  selected_hero_talents?: BlizzardSelectedTalent[]; // TWW 11.0+
}

interface BlizzardSelectedTalent {
  id: number;
  rank: number;
  tooltip?: {
    talent: BlizzardNamedRef;
    spell_tooltip: { spell: BlizzardNamedRef; description: string; cast_time: string };
  };
}
```

---

## Professions

```typescript
interface BlizzardCharacterProfessions extends BlizzardLinks {
  character: BlizzardCharacterRef;
  primaries: BlizzardProfessionEntry[];
  secondaries?: BlizzardProfessionEntry[];
}

interface BlizzardProfessionEntry {
  profession: BlizzardNamedRef;
  tiers: BlizzardProfessionTier[];
}

interface BlizzardProfessionTier {
  skill_points: number;
  max_skill_points: number;
  tier: { name: string; id: number };
  known_recipes?: BlizzardNamedRef[];
}
```

---

## Achievements

```typescript
interface BlizzardCharacterAchievements extends BlizzardLinks {
  total_quantity: number;
  total_points: number;
  achievements: BlizzardAchievementEntry[];
}

interface BlizzardAchievementEntry {
  id: number;
  achievement: BlizzardNamedRef;
  criteria?: { id: number; is_completed: boolean };
  completed_timestamp: number;
}
```

---

## Reputations

```typescript
interface BlizzardReputations extends BlizzardLinks {
  character: BlizzardCharacterRef;
  reputations: BlizzardReputation[];
}

interface BlizzardReputation {
  faction: BlizzardNamedRef;
  standing: {
    raw: number;
    value: number;
    max: number;
    tier: number;  // 0=Hated → 7=Exalted
    name: string;
  };
}
```

---

## Titles

```typescript
interface BlizzardCharacterTitles extends BlizzardLinks {
  character: BlizzardCharacterRef;
  active_title?: BlizzardCharacterTitle;
  titles: BlizzardNamedRef[];
}
```

---

## Quests

```typescript
interface BlizzardCharacterQuests extends BlizzardLinks {
  character: BlizzardCharacterRef;
  in_progress: BlizzardNamedRef[];
  completed?: BlizzardHref;
}
```

---

## PvP

```typescript
interface BlizzardPvpSummary extends BlizzardLinks {
  honor_level: number;
  pvp_map_statistics: BlizzardPvpMapStat[];
  honorable_kills?: number;
  character: BlizzardCharacterRef;
}

interface BlizzardPvpMapStat {
  world_map: { name: string; id: number };
  match_statistics: { played: number; won: number; lost: number };
}
```

## PvP Bracket

```typescript
interface BlizzardPvpBracketStatistics extends BlizzardLinks {
  character: BlizzardCharacterRef;
  faction: BlizzardTypedValue;
  bracket: { id: number; type: string }; // e.g. "2v2", "3v3", "BATTLEGROUNDS", "SHUFFLE"
  rating: number;
  season: BlizzardKeyedRef;
  tier: BlizzardKeyedRef;
  season_match_statistics: { played: number; won: number; lost: number };
  weekly_match_statistics: { played: number; won: number; lost: number };
}
```

---

## Guild

```typescript
interface BlizzardGuildProfile extends BlizzardLinks {
  id: number;
  name: string;
  faction: BlizzardTypedValue;
  achievement_points: number;
  member_count: number;
  realm: BlizzardRealmRef;
  crest: BlizzardGuildCrest;
  created_timestamp: number;
  roster: BlizzardHref;
  achievements: BlizzardHref;
  activity: BlizzardHref;
  name_search: string;
}

interface BlizzardGuildRoster extends BlizzardLinks {
  guild: BlizzardGuildRef;
  members: BlizzardGuildMember[];
}

interface BlizzardGuildRef {
  key: BlizzardHref;
  name: string;
  id: number;
  realm: BlizzardRealmRef;
  faction: BlizzardTypedValue;
}

interface BlizzardGuildMember {
  character: BlizzardGuildCharacter;
  rank: number; // 0 = GM
}

interface BlizzardGuildCharacter {
  key: BlizzardHref;
  name: string;
  id: number;
  realm: BlizzardRealmShort;
  level: number;
  playable_class: BlizzardKeyedRef; // id only
  playable_race: BlizzardKeyedRef;  // id only
  faction: { type: string };
}

interface BlizzardGuildActivity extends BlizzardLinks {
  guild: BlizzardGuildActivityGuildRef;
  activities: BlizzardGuildActivityEntry[];
}

interface BlizzardGuildActivityGuildRef {
  key: BlizzardHref;
  name: string;
  id: number;
  realm: BlizzardLocalizedRealmRef;
  faction: BlizzardLocalizedTypedValue;
}

interface BlizzardLocalizedRealmRef {
  key: BlizzardHref;
  name: BlizzardLocalizedString;
  id: number;
  slug: string;
}

interface BlizzardGuildActivityEntry {
  character_achievement: BlizzardCharacterAchievementActivity;
  activity: { type: string };
  timestamp: number;
}

interface BlizzardCharacterAchievementActivity {
  character: { key: BlizzardHref; name: string; id: number; realm: BlizzardLocalizedRealmRef };
  achievement: { key: BlizzardHref; name: BlizzardLocalizedString; id: number };
}

interface BlizzardGuildAchievements extends BlizzardLinks {
  guild: BlizzardGuildActivityGuildRef;
  total_quantity: number;
  total_points: number;
  achievements: BlizzardGuildAchievementEntry[];
}

interface BlizzardGuildAchievementEntry {
  id: number;
  achievement: { key: BlizzardHref; name: string; id: number };
  criteria: BlizzardGuildAchievementCriteria;
  completed_timestamp?: number;
}

interface BlizzardGuildAchievementCriteria {
  id: number;
  amount?: number;
  is_completed: boolean;
  child_criteria?: { id: number; amount: number; is_completed: boolean }[];
}
```

---

## Game Data — Professions

```typescript
interface BlizzardProfessionIndex {
  _links: { self: BlizzardHref };
  professions: BlizzardNamedRef[];
}

interface BlizzardProfessionDetail {
  _links: { self: BlizzardHref };
  id: number;
  name: string;
  description?: string;
  type: BlizzardTypedValue;
  media: BlizzardKeyedRef;
  skill_tiers: Array<{
    key: BlizzardHref;
    name: string;
    id: number;
    minimum_skill_level: number;
    maximum_skill_level: number;
  }>;
}

interface BlizzardSkillTierDetail {
  _links: { self: BlizzardHref };
  id: number;
  name: string;
  minimum_skill_level: number;
  maximum_skill_level: number;
  categories: Array<{
    name: string;
    recipes: BlizzardNamedRef[];
  }>;
}
```

---

## Raider.io

```typescript
interface RaiderIoCharacterProfile {
  name: string;
  race: string;
  class: string;
  active_spec_name: string;
  active_spec_role: string;
  gender: string;
  faction: string;
  achievement_points: number;
  honorable_kills: number;
  thumbnail_url: string;
  region: string;
  realm: string;
  last_crawled_at: string;
  profile_url: string;
  profile_banner: string;
  mythic_plus_scores_by_season?: MythicPlusScoresBySeason[];
  gear?: CharacterGear;
}

interface MythicPlusScoresBySeason {
  season: string;
  scores: {
    all: number; dps: number; healer: number; tank: number;
    spec_0: number; spec_1: number; spec_2: number; spec_3: number;
  };
}

interface CharacterGear {
  updated_at: string;
  item_level_equipped: number;
  item_level_total: number;
  artifact_traits: number;
  corruption?: { added: number; resisted: number; total: number };
  items: {
    head?: GearItem; neck?: GearItem; shoulder?: GearItem; back?: GearItem;
    chest?: GearItem; shirt?: GearItem; tabard?: GearItem; wrist?: GearItem;
    hands?: GearItem; waist?: GearItem; legs?: GearItem; feet?: GearItem;
    finger1?: GearItem; finger2?: GearItem; trinket1?: GearItem; trinket2?: GearItem;
    mainhand?: GearItem; offhand?: GearItem;
  };
}

interface GearItem {
  item_id: number;
  item_level: number;
  enchant?: number;
  icon: string;
  name: string;
  item_quality: number; // 1=Common, 2=Uncommon, 3=Rare, 4=Epic, 5=Legendary
  is_legendary: boolean;
  is_azerite: boolean;
  azerite_powers?: number[];
  corruption?: { added: number; resisted: number; total: number };
  domination_shards?: number[];
  gems?: number[];
  bonuses?: number[];
  tier?: string;
}

interface RaiderIoApiError {
  error: string;
  message: string;
  statusCode: number;
}
```
