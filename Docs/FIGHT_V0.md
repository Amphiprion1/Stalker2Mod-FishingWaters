# Fight v0 — build now (weapon Fire path)

Fire already → `BP_FW_FishingRodUse` → `StartFishingSession` on `BP_FlishableWaters`.
Water still deferred.

## Already done
- Enums `EFW_FishingState` / `EFW_FishingEnding` exist
- Start + Get SP proven
- Aim = `IA_Aim` Enhanced Input on FlishableWaters

## Wire Fight (in order)

### A — WaitingBite → Fight
`EnterWaitingBite`: state=WaitingBite, `Set Timer by Event` → `OnBiteTimer` (random 2–5 s).
`OnBiteTimer`: if WaitingBite → `EnterFight` (Print `bite`).

### B — OnFightTouch (IA_Aim Started, only if Fight)
```
Cost = Get Max SP(PlayerObj) * (StaminaCostPct / 100.0)   // default 8%
If Get SP < Cost → EndFishingSession(FailStamina)
Else:
  Set SP(PlayerObj, Get SP - Cost)   // or apply FW_TestStaminaNeg25
  TouchesDone++
  If TouchesDone >= TouchesNeeded (3):
    **Execute Console Command** (v0 — no CreateItem node found in context):
      XCreateItemInInventoryByID FWPerch 0 1 1
    EndFishingSession(Success)
```

### C — EndFishingSession(Reason)
Clear timers, Disable Input (optional), state=Idle. Idempotent if already Idle.

### D — Aborts (bind when Start, unbind in End)
- On Damage Received → AbortDamage
- On PDA Use Started → AbortPDA
- Distance vs StartLocation > MaxMoveDist → AbortMove

## Official nodes (Obj / inventory)
| Need | Node |
|------|------|
| Read stamina | **Get SP** / **Get Max SP** |
| Drain | **Set SP** (prefer) or effect `FW_TestStaminaNeg25` |
| Loot | **CreateItemInInventory** if listed, else **Execute Console Command** `XCreateItemInInventoryByID FWPerch 0 1 1` |
| Aim | **EnhancedInputAction IA_Aim** (path in FISHING_FSM.md) |

## Test
1. Fire rod (1 bait) → wait 2–5 s → Print bite
2. Aim ×3 with stamina → FWPerch in inventory
3. Aim with empty stamina → FailStamina
4. Open PDA mid-fight → abort

Water / spot = last.

