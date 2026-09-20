# Fishing FSM — clean implementation (BP_FlishableWaters)

Do **not** put the whole session in `StartFishingSession`.
That function only **enters** the machine. Everything else is small handlers + one teardown.

## 1. Data (variables on BP_FlishableWaters)

| Name | Type | Role |
|------|------|------|
| `FishingState` | Byte / Enum | `Idle` `WaitingBite` `Fight` `Ending` |
| `PlayerObj` | Obj | Cached Cast to Obj |
| `StartLocation` | Vector | For move abort |
| `BiteTimerHandle` | Timer Handle | WaitingBite delay |
| `FightWindowHandle` | Timer Handle | Optional Interact window |
| `TouchesDone` | Int | Fight progress |
| `TouchesNeeded` | Int | Default 3 (v0) |
| `StaminaCostPct` | Float | Default 8 |
| `MaxMoveDist` | Float | Default 80–150 (cm) — abort if farther |
| `bListenersBound` | Bool | Idempotent bind/unbind |

Enum tip: My Blueprint → Enumeration `EFW_FishingState` or use Byte + comments.

## 2. Public API (only these cross the boundary)

```
StartFishingSession   ← CameraShake / rod Use calls this
EndFishingSession(Reason)  ← ONLY place that clears timers + unbinds + sets Idle
```

Reasons (string or enum): `Success` `FailStamina` `FailTimeout` `AbortMove` `AbortDamage` `AbortPDA` `AbortBusy` `Cancel`

**Idempotent:** calling `EndFishingSession` when already `Idle` = no-op.

## 3. StartFishingSession (thin)

1. If `FishingState != Idle` → return (ignore double Utiliser)
2. Resolve `PlayerObj` (Get Player Character → Cast to Obj). Fail → return
3. `StartLocation = PlayerObj` actor location
4. `TouchesDone = 0`
5. `BindSessionListeners` (once)
6. `EnterWaitingBite`

Nothing else here. No Delay chain. No Fight logic.

## 4. States

### Idle
No timers. Listeners unbound (or bound but all handlers early-out on Idle).

### WaitingBite — `EnterWaitingBite`
1. `FishingState = WaitingBite`
2. Print / cue optional
3. `Set Timer by Event` → `OnBiteTimer`, time = Random Float in Range (2.0, 5.0), looping = false  
   Store handle in `BiteTimerHandle`
4. **Do not** use a long Delay node in the Event Graph

`OnBiteTimer`:
- If state != WaitingBite → return
- `EnterFight`

### Fight — `EnterFight`
1. Clear bite timer if still valid
2. `FishingState = Fight`
3. Print `bite` / sound
4. Arm Interact (see §6)
5. Optional: start Fight timeout timer (e.g. 45 s) → `EndFishingSession(FailTimeout)`

Each successful Interact (`OnFightTouch`):
1. If state != Fight → return
2. `Cost = Get Max SP(PlayerObj) * (StaminaCostPct / 100)`
3. If `Get SP(PlayerObj) < Cost` → `EndFishingSession(FailStamina)`
4. Else apply drain (`FW_TestStaminaNeg25` / Set SP) → `TouchesDone++`
5. If `TouchesDone >= TouchesNeeded` → give `FWPerch` → `EndFishingSession(Success)`
6. Else reopen Interact window / wait next press

### Ending
Brief optional — or jump straight to Idle inside `EndFishingSession`.

## 5. EndFishingSession(Reason) — single teardown

1. If `FishingState == Idle` → return
2. `FishingState = Ending` (or Idle immediately)
3. Clear Timer by Handle: `BiteTimerHandle`, `FightWindowHandle`, any timeout
4. `UnbindSessionListeners`
5. Disable Interact arming
6. Print Reason (debug)
7. `FishingState = Idle`

Every abort path **only** calls this. Never Clear Timer scattered in 5 places without going through End.

## 6. Listeners (bind on Start, unbind on End)

Bind on **PlayerObj** (official Obj events — verified names in ModEditor):

| Event | Action |
|-------|--------|
| **On Damage Received** | `EndFishingSession(AbortDamage)` |
| **On PDA Use Started** | `EndFishingSession(AbortPDA)` — PDA / menu open |
| **On Stamina Changed** | optional: if Next SP ≤ 0 during Fight → FailStamina |

Also useful if present on Obj: receive bullet / melee hit (same AbortDamage).

**Movement** (no dedicated “On Move”):
- On `Event Tick` **only while** state is WaitingBite or Fight:
  - `Dist = Distance(Player location, StartLocation)`
  - If `Dist > MaxMoveDist` → `EndFishingSession(AbortMove)`
- Or: Timer 0.25 s looping while active (lighter than full Tick)

**Interact presses (Fight):**
- Prefer Enable Input on `BP_FlishableWaters` + Input Action Interact / Use
- Or poll key — avoid inventing private APIs
- Gate: only accept press when `FishingState == Fight`

## 7. Graph layout (avoid spaghetti)

```
[CameraShake] → StartFishingSession
                      │
                      ├─ EnterWaitingBite → Timer → OnBiteTimer → EnterFight
                      │
                      ├─ OnFightTouch (Interact)
                      ├─ OnDamage / OnPDA / TickMove  ──┐
                      │                                 │
                      └──────────────────────────────► EndFishingSession
```

Each bubble = **one function**. Event Graph only wires calls.

## 8. Build order (verify each step)

1. Start + End + state print only (double Utiliser ignored)
2. WaitingBite timer → EnterFight print
3. AbortPDA + AbortDamage
4. AbortMove (distance)
5. Fight touches without stamina
6. Stamina gate + drain
7. Give FWPerch on Success
8. Water / spot **last**

## 9. What stays where

| BP | Job |
|----|-----|
| `BP_FW_FishingRodUse` | Print optional; call `StartFishingSession` on FlishableWaters |
| `BP_FlishableWaters` | Owns FSM + listeners + loot |
| `BP_Mod_FishableWaters` | Spawn/attach only (unchanged) |

## Fight input: Aim (RMB / LT)

### Why you do NOT see On Aim Pressed from Player Obj

On Aim Pressed appears in Blueprint_API_Guide.pdf and as OnAimPressed_BP in the binary.
Same class as **On Before Use Item**: BlueprintImplementableEvent on an **Obj subclass** — not Assign/Bind from a Player Obj reference.
From that pin you only get callables like **Press Aim** (forces aim — useless for listening).

### Reliable capture on BP_FlishableWaters

1. When entering Fight (or Start): **Enable Input** → Player Controller 0
2. Event Graph (empty space, not from Player Obj): add **EnhancedInputAction IA_Aim**
   - Asset: InputAction'/Game/_Stalker_2/data/input/InputActions/Delayable/IA_Aim.IA_Aim'
3. On **Started** (or Triggered): if FishingState == Fight → OnFightTouch
4. On EndFishingSession: **Disable Input** (optional if you leave Enable on)

IA_Aim is the game action for mouse RMB + gamepad LT.
