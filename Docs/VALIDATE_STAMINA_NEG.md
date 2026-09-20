# Validate stamina / spawn (updated)

## Why the first test items did not spawn

1. `PakPlainMod.bat` only rebuilds **OverrideContent** (often empty). Cfg live in **NewContent.pak** (`GameLite/ModGameData` via `DirectoriesToAlwaysStageAsUFS_DLC`).
2. Even so, the previous NewContent.pak **already contained** `FW_TestStaminaNeg/Pos`. So "not in pak" was not the whole story — those SIDs were likely **rejected at load** (custom EffectPrototype with bad `refurl` / negative Stamina), so `XCreateItemInInventoryByID` did nothing.

## Fix deployed (2026-09-20)

Rebuilt NewContent.pak with ladder SIDs (no underscore after FW):

| SID | Purpose |
|---|---|
| `FWTestSpawn` | Spawn only — vanilla `BreadSatiety2` |
| `FWTestStaminaPos` | Use — vanilla `WaterStaminaInstant` (+stamina) |
| `FWTestStaminaPosCustom` | Use — custom `FW_TestStaminaPos25` |
| `FWTestStaminaNeg` | Use — custom `FW_TestStaminaNeg25` (−25%) |

Steam: `~mods\ZZZ_FishableWaters\FishableWatersStalker2-Windows-NewContent.pak` updated (ucas/utoc left as-is). Backup: `*.pak.bak_1643`.

## Your retest

1. Quit game fully, relaunch.
2. Sanity:
   ```
   XCreateItemInInventoryByID FWPerch 0 1 1
   XCreateItemInInventoryByID FWTestSpawn 0 1 5
   ```
3. If Spawn OK:
   ```
   XCreateItemInInventoryByID FWTestStaminaPos 0 1 5
   XCreateItemInInventoryByID FWTestStaminaPosCustom 0 1 5
   XCreateItemInInventoryByID FWTestStaminaNeg 0 1 5
   ```
4. Sprint to ~half bar. Use Pos (vanilla) → should rise. Then PosCustom / Neg.

Report which SIDs appear. That isolates spawn vs effect.

## Cfg-only repack next time

```
RunUAT GSCCookModNewContent ... -CustomConfig=ModCookNewContent -skipcook -skipiostore
```
Then copy staged `FishableWatersStalker2-Windows-NewContent.pak` into `ZZZ_FishableWaters` (keep existing .ucas/.utoc).
`PakPlainMod` alone is not enough for ModGameData cfg.
## Lesson (Gerald 2026-09-20)

- Never leave `*.pak.bak*` / extra paks inside `~mods\ZZZ_FishableWaters` � they break load.
- After package, copy **all 6** New+Override files (pak/ucas/utoc), nothing else.
- `FWTestStaminaPos` + vanilla `WaterStaminaInstant` = spawn OK, stamina rises on use.
- Next: `FWTestStaminaNeg` (custom `-25%` Stamina) + optional `FWTestStaminaPosCustom`.
