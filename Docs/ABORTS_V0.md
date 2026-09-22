# Aborts v0 — PDA / Damage / Move

All aborts call **only** `EndFishingSession(Reason)`. Never Clear Timer elsewhere.

## When to arm

In `StartFishingSession` (after cache `PlayerObj` + `StartLocation`):
1. Bind / Enable the three paths below
2. Then `EnterWaitingBite`

In `EndFishingSession`:
1. Clear bite/fight timers
2. Unbind / disable the three paths
3. `FishingState = Idle`

If already Idle → return (idempotent).

## 1. PDA / Inventory (same as Aim — no Assign On PDA Use Started)

`On PDA Use Started` is Obj-override only, not Assign from PlayerObj.

On `BP_FlishableWaters` (Enable Input already for Aim):

1. Enhanced Input **IA_OpenPDA**
   - `InputAction'/Game/_Stalker_2/data/input/InputActions/IA_OpenPDA.IA_OpenPDA'`
2. Enhanced Input **IA_Inventory** (Tab / inventaire — aussi une interruption)
   - `InputAction'/Game/_Stalker_2/data/input/InputActions/IA_Inventory.IA_Inventory'`
3. Both **Started** → if WaitingBite or Fight → `EndFishingSession(AbortPDA)`

## 2. Damage

On `PlayerObj`:
- **Assign On Damage Received** → `OnAbortDamage`
- Same gate: only if WaitingBite or Fight → `EndFishingSession(AbortDamage)`

Optional extras if present: On Receive Bullet / Melee Hit → same abort.

## 3. Move

No “On Move” on Obj. Use distance check:

Variables: `StartLocation` (set at Start), `MaxMoveDist` (float, try **120** first).

While state is WaitingBite or Fight:
- **Set Timer by Event** looping **0.25 s** → `OnMoveCheck`
  - or Event Tick gated by state (heavier)

`OnMoveCheck`:
```
If state not in (WaitingBite, Fight) → return
Dist = Distance(PlayerObj location, StartLocation)
If Dist > MaxMoveDist → EndFishingSession(AbortMove)
```

Clear this timer in `EndFishingSession`.

## Test order

1. Start fishing → open PDA → session ends (Print AbortPDA)
2. Start → take damage → AbortDamage
3. Start → walk away > MaxMoveDist → AbortMove
4. Success path still works (3× Aim → perch)

## Reasons (EFW_FishingEnding or string)

`AbortPDA` | `AbortDamage` | `AbortMove` | (+ existing Success / FailStamina)

