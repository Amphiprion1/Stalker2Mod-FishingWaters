# FishableWaters — wire DT_FW_* into BP_FlishableWaters

Assets present:
- Content/Structures/ST_FW_FishSpecies, ST_FW_FishSpawnEntry, ST_FW_FishingZone
- Content/DataTables/DT_FW_FishSpecies, DT_FW_FishingZones

## A) Variables on BP_FlishableWaters

| Name | Type | Default |
|---|---|---|
| FishSpeciesDT | Data Table Object Reference → DT_FW_FishSpecies | set in Defaults |
| FishingZonesDT | Data Table Object Reference → DT_FW_FishingZones | set in Defaults |
| ActiveZoneRow | Name | DefaultAnywhere |
| ActiveFishRow | Name | (runtime) |
| ActiveFish | ST_FW_FishSpecies | (runtime) |
| TouchesNeeded | Integer | (runtime) |
| TouchesDone | Integer | 0 |
| ActiveLootSID | String | (runtime, optional cache of LootItemSID) |

Keep existing: FishingState, StartHP, StartLocation, PlayerObj, timers, etc.

## B) Function `ResolveActiveFish` (pure-ish, returns Bool success)

1. `Get Data Table Row` from FishingZonesDT, Row = ActiveZoneRow, Out Row = Zone (ST_FW_FishingZone).
   - Fail → Print "bad zone" → return false.
2. From Zone.Spawns: ForEach → Sum Weight (only if Weight > 0 and Fish.RowName is valid).
   - Sum <= 0 → fail.
3. `Random Float in Range` 0 .. Sum.
4. Second ForEach Spawns: cumulative += Weight; if Roll < cumulative → selected Spawn.Break → Fish.Row Name → set ActiveFishRow; break loop.
5. `Get Data Table Row` FishSpeciesDT, Row = ActiveFishRow → set ActiveFish.
   - Fail → return false.
6. `TouchesNeeded = Random Integer in Range (ActiveFish.TouchesMin, ActiveFish.TouchesMax)` (UE inclusive).
7. `TouchesDone = 0`.
8. `ActiveLootSID = ActiveFish.LootItemSID`.
9. Return true.

## C) StartFishingSession

After Idle checks / cache PlayerObj / StartLocation / StartHP:

1. Call ResolveActiveFish.
2. If false → EndFishingSession(Fail) or abort early (no session).
3. Else → EnterWaitingBite (use ActiveFish wait range).

## D) EnterWaitingBite

Replace hardcoded 2–5 s:

`Duration = Random Float (ActiveFish.WaitMinSec, ActiveFish.WaitMaxSec)`
Set Timer by Event → OnBiteTimer → EnterFight.

## E) OnFightTouch (stamina gate)

Replace hardcoded 8%:

`Cost = Get Max SP * (ActiveFish.StaminaCostPct / 100.0)`
If Get SP < Cost → EndFishingSession(Fail)
Else Set SP (Get SP - Cost); TouchesDone++; if TouchesDone >= TouchesNeeded → Success.

(Optional later: GapMin/Max between touches — skip for v0.)

## F) Success loot

Replace hardcoded `FWPerch`:

`Execute Console Command`:
`XCreateItemInInventoryByID {ActiveFish.LootItemSID} 0 1 1`
(or Append format string with ActiveLootSID)

## G) Defaults checklist

- FishSpeciesDT → DT_FW_FishSpecies
- FishingZonesDT → DT_FW_FishingZones
- ActiveZoneRow → DefaultAnywhere
- DT rows filled (Common/Heavy + zone weights)

## H) Quick PIE test

1. Fire rod → session starts.
2. Print ActiveFishRow + Wait duration + TouchesNeeded once after Resolve.
3. Confirm Common (~70%) vs Heavy (~30%) over several casts.
4. Confirm loot SID matches row.
