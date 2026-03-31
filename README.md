# Agent Skills

Reusable skills for AI coding agents (Claude Code, Cursor, Copilot, and [40+ others](https://agentskills.io)).

## Install

```bash
# All skills
npx skills add ljclaeyssen/skills

# A specific skill
npx skills add ljclaeyssen/skills --skill blizzard-api
npx skills add ljclaeyssen/skills --skill wowhead-tooltips
```

## Available Skills

| Skill | What it does | Triggers on |
|-------|-------------|-------------|
| [blizzard-api](skills/blizzard-api) | Integrate the Blizzard Battle.net & Raider.io APIs into web projects. Covers OAuth authentication, 34+ endpoints (character profiles, equipment, M+ scores, raid progress, guild rosters, professions, PvP, collections...), TypeScript models, and SPA integration patterns for Angular, React, and Vue. | `Battle.net API`, `WoW API`, `Blizzard OAuth`, `raider.io`, `mythic+ score`, `guild roster`, `character equipment`, `wow character data`, `item level`, `raid progress` |
| [wowhead-tooltips](skills/wowhead-tooltips) | Integrate Wowhead's tooltip system into web apps. Covers the power script setup, injected DOM structure, CSS overrides, icon sizing, quality color classes, and SPA refresh patterns (MutationObserver-based) for Angular, React, and Vue. | `wowhead tooltip`, `wow item icon`, `data-wowhead`, `wowhead script`, `whTooltips`, `iconizeLinks`, `item tooltip` |

## License

MIT
