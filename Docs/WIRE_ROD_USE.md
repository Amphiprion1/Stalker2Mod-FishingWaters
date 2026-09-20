# Wire FW_FishingRod Use -> fishing session (no water yet)

## Hook (cfg + proven)

`FW_FishingRod` → `FW_FishingRodUseEffect` → `FW_FishingRodShake` →
`BP_FW_FishingRodUse` (LegacyCameraShake) — Print String OK in-game.

## Reuse CraftableZone Actor (preferred)

Do **not** spawn a second fishing Actor.

| BP | Role |
|----|------|
| `BP_Mod_FishableWaters` | ModWorldSubsystem — already spawns/attaches helper |
| `BP_FlishableWaters` | Actor on player — CZ recipes / dismantle **and** fishing FSM |
| `BP_FW_FishingRodUse` | Thin trigger on Utiliser only |

### Wire

1. On `BP_FlishableWaters`: add custom event / function `StartFishingSession` (v0: Print String `FW fishing: session start`).
2. On `BP_FW_FishingRodUse` → Event Receive Play Shake:
   - **Get All Actors Of Class** `BP_FlishableWaters` (Context Sensitive off if needed)
   - **Get** (0) → call `StartFishingSession`
3. Later: inside `StartFishingSession` → Get Player Character → Cast to Obj → Get SP / FSM.
   (Actor is already attached to player — Attach parent / Get Owner may also work.)

Water / spot = last.
