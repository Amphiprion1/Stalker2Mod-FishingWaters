# Wire FW_FishingRod Use -> fishing session (no water yet)

## Goal
When player clicks **Utiliser** on `FW_FishingRod`, start the fishing FSM anywhere (water later).

## Hook (official API)
On player `Obj` (Stalker2.Obj):
- Event / override: **On Before Use Item** (`FItemUID Item UID`)
- Also exists: **On Item Use Ended** (PC)

## Where
Prefer `BP_Mod_FishableWaters` (ModWorldSubsystem) if it can bind to the local player Obj on start.
Else Event Graph on a player-owned helper spawned from the Game Feature.

## v0 proof (do this first)
1. Open `BP_Mod_FishableWaters` (or create `BP_FW_FishingSession` owned by mod subsystem).
2. On subsystem init / BeginPlay: get **Current Player** Obj (`Is Current Player`).
3. Bind or implement **On Before Use Item**.
4. From `Item UID`, resolve prototype SID (search ModKit nodes: Item UID -> Prototype SID / Get Item Prototype). Compare to `FW_FishingRod`.
5. If match: **Print String** `FW fishing: Use rod` (or on-screen debug).
6. Package + test Utiliser — you should see the print. No water check.

## Then FSM stub
Same branch calls `StartFishingSession`:
- State WaitingBite (timer 2–5 s for test)
- Fight: N touches, each: `Cost = Get Max SP * 0.08`; if `Get SP < Cost` -> Fail; else apply `FW_TestStaminaNeg25` (or Set SP) and count++
- Success: give `FWPerch` (same give path as console)

Water / spot = last.