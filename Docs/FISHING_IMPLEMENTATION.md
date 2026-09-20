# FishableWaters — Fishing implementation (detailed steps)

Design: no mini-game. Catch = **n–p Interact presses**, spaced **x–y seconds**, each costing **z % stamina**. Rarer fish = more presses / higher cost → needs strong stamina regen (artifacts). Overload slows regen → drop gear → stay exposed.

Vanilla rod mesh (no MeshPrototype SID in base GameData — we add one):
`StaticMesh'/Game/_Stalker_2/props/general/SM_gen_fishing_rod_01_a.SM_gen_fishing_rod_01_a'`
→ ModGameData SID: **`FW_FishingRod_Mesh`**

---

## Phase 0 — Preconditions (already partly done)

1. FishableWaters Game Feature Active + `AddConfigsPath` → `ModGameData/FishableWaters/`.
2. CraftableZone + Faster Item Info Panel in load order (disassemble already works).
3. `FWPerch` item + disassemble → 10× `FWPerchFilletRaw` OK.
4. Keep **cfg-first**: species params in DT/cfg; BP only orchestrates.

---

## Phase 1 — Data: rod item + species table

### 1.1 MeshPrototype (cfg — started)

File: `ModGameData/FishableWaters/MeshPrototypes/FishableWaters_MeshPrototypes.cfg`  
Contains `FW_FishingRod_Mesh` → rod StaticMesh.

### 1.2 Item `FW_FishingRod` (cfg)

In `ItemPrototypes/FishableWaters_ItemPrototypes.cfg` add (ModEditor loc later):

- `SID = FW_FishingRod`
- `LocalizationSID = FW_FishingRod`
- `Type = EItemType::Other` (or whatever matches “tool” you prefer after a quick vanilla compare)
- `Usable = true` **or** false if fishing starts only from water interact (prefer water interact + “must have rod in inventory”)
- `MeshPrototypeSID = FW_FishingRod_Mesh`
- `MeshInWorldPrototypeSID = FW_FishingRod_Mesh`
- Weight ~1.0, grid 1×3 or 2×1, stack 1
- Icon: screenshot/import later (same workflow as `T_FW_Perch`)

LOC keys: `sid_items_FW_FishingRod_name` / `_description` (EN|FR).

### 1.3 DataTable `DT_FishingSpecies` (ModEditor)

Create struct `ST_FishingSpecies` (UserDefinedStruct) with fields:

| Field | Type | Example Perch |
|---|---|---|
| SpeciesSID | String | FWPerch_Common |
| LootItemSID | String | FWPerch |
| TouchesMin | Int | 3 |
| TouchesMax | Int | 5 |
| GapMinSec | Float | 1.2 |
| GapMaxSec | Float | 2.5 |
| StaminaCostPct | Float | 8 |
| BiteChance | Float | 0.55 |
| WeightClass | String / Enum | Common |

Create `Content/DataTables/DT_FishingSpecies`, fill rows from `ModGameData/.../Fishing/species_table_v0.txt`.

Register DT from `BP_FlishableWaters` (same pattern as Disassemble: fill array → your fishing subsystem).

### 1.4 Optional water volume tags

Decide how a spot knows which species pool to use:

- **A (simple):** one global pool per water BP (array of SpeciesSID weights).
- **B (better):** `DT_FishingWaters` row per water GUID / tag (`LesserZone_Pond_01` → species list).

Start with **A**.

---

## Phase 2 — World: fishing spot (ModEditor)

1. Place a **trigger / overlap volume** on a water surface (test map WP region, not bare TestLevel).
2. BP actor `BP_FW_FishingSpot`:
   - On overlap: show interact prompt if player has `FW_FishingRod` in inventory.
   - On Interact: call fishing session on ModWorldSubsystem / `BP_FlishableWaters`.
3. Optional: attach or spawn rod mesh in hand / nearby (cosmetic). Prefer **inventory check only** for v0 — mesh in world as prop is enough.

---

## Phase 3 — Session logic (BP — core)

State machine on `BP_FW_FishingSession` (or component on spot):

```
Idle → WaitingBite → Fight → Success | Fail | Abort
```

### 3.1 WaitingBite

- Require: rod in inventory, player inside spot, not dead.
- Timer random 5–20 s (cfg later).
- Roll `BiteChance` for weighted random species from spot pool.
- On bite: sound cue + enter **Fight**.

### 3.2 Fight (the “touches” loop)

1. Roll `TouchesNeeded = RandomInt(TouchesMin, TouchesMax)`.
2. `TouchesDone = 0`.
3. Loop until done or fail:
   - Wait `RandomFloat(GapMinSec, GapMaxSec)`.
   - Open a short **Interact window** (e.g. 0.8–1.5 s) — same key as Use/Interact, **no** UI mini-game widget if possible (sound + existing interact).
   - On press in window:
     - Try spend `StaminaCostPct` of **max** stamina (or current — pick one and stick to it; prefer % of max).
     - If stamina too low → **Fail** (fish escapes).
     - Else `TouchesDone++`.
   - On miss window → **Fail** (or allow 1 miss — design choice; v0 = fail).
4. If `TouchesDone >= TouchesNeeded` → **Success**.

### 3.3 Success / Fail / Abort

- **Success:** `AddItem(LootItemSID, 1)` (same path you use for cheats / CZ give). Prefer official inventory API used elsewhere in ModKit examples.
- **Fail:** message/sound only.
- **Abort** if: leave volume, take damage (optional), open inventory overload drop mid-fight (allowed — that’s the point), die.

### 3.4 Stamina API (critical verify step)

Before coding drain:

1. In ZoneKit / CraftableZone / ModKit docs / Example BPs, search how stamina is read/written.
2. **Do not invent** a private `UStalker…` call. Prefer:
   - documented ModKit node, or
   - apply a short EffectPrototype that burns stamina (if vanilla has one), or
   - temporary workaround: consume a tiny “effort” item (last resort).
3. Write the chosen method in this doc under “Verified stamina path” once found.

Artifacts / overload need **no special code**: vanilla regen already reacts to artifacts + weight.

---

## Phase 4 — Wire into FishableWaters mod BP

1. `BP_Mod_FishableWaters` (ModWorldSubsystem) holds / spawns fishing helper.
2. `BP_FlishableWaters` (CZ provider) can also own fishing DT load — keep CZ registration intact.
3. Game Feature stays Registered; fishing spot actors live in level or spawned from subsystem.

---

## Phase 5 — Feel / balance pass

| Goal | Lever |
|---|---|
| Casual food | Perch_Common: 3–5 touches, 8% cost |
| Gear check | Perch_Heavy: 6–9 touches, 12% |
| Artifact gate | Rare: 10–14 touches, 18%+ |
| Fear | Fight lasts 20–60 s while underweight |

Test with: empty stamina, full stamina, overload, endurance artifact on.

---

## Phase 6 — Polish (after proto works)

1. Icon for rod.
2. Bite / splash / struggle sounds (Wwise events if available — verify names, don’t invent).
3. Optional: break rod on fail (durability) — later.
4. Multiple waters + `DT_FishingWaters`.
5. Bait item that biases `BiteChance` / species weights.

---

## Suggested build order (checklist)

- [ ] Confirm `FW_FishingRod_Mesh` loads (PIE drop rod mesh).
- [ ] Add `FW_FishingRod` item + loc + give via console.
- [ ] Create `ST_FishingSpecies` + `DT_FishingSpecies` + 2 rows (Common/Heavy).
- [ ] Place `BP_FW_FishingSpot` on a WP water.
- [ ] Implement WaitingBite → Fight **without** stamina (presses only) — prove loop.
- [ ] Add stamina cost once API verified.
- [ ] Grant `FWPerch` on success; test disassemble still works.
- [ ] Balance numbers; write them back into `species_table_v0.txt` / DT.

---

## Console helpers

```
XCreateItemInInventoryByID FW_FishingRod 0 1 1
XCreateItemInInventoryByID FWPerch 0 1 1
```

---

## Out of scope for v0

- Full fishing mini-game UI / tension bar.
- Networked co-op edge cases.
- New fish meshes beyond `DeadFish` / existing icons.
- Private engine stamina hacks.

When Phase 3.4 stamina path is verified, implement BP session next.

---

## Verified from vanilla GameLite (READ-ONLY) — 2026-09-20

Source root (never edit): `C:\STALKER2ZoneKit\Stalker2\Content\GameLite\GameData`

### Mesh
- **`DeadFish`** exists in `MeshPrototypes.cfg` → `SM_VFX_FishDead` / `MI_VFX_FishDead` (matches FWPerch).
- **No** `SM_gen_fishing_rod` / `FishingRod` SID anywhere under GameLite → keep mod mesh `FW_FishingRod_Mesh`.

### Stamina-related effects (public cfg patterns)
| Type | Role | Example SID |
|---|---|---|
| `EEffectType::Stamina` | Instant ±% stamina (all vanilla samples are **restore**, Positive) | `WaterStaminaInstant` (38%) |
| `EEffectType::RegenStamina` | Regen multiplier | `EnergeticStamina`, `IncreaseRegenStamina*` |
| `EEffectType::MaxStamina` | Max pool | (artefacts) |
| `EEffectType::SPDrain` | Multiplies cost of Walk/Run/Sprint (not a flat spend on Interact) | `VodkaStaminaPenalty` |

**Hypothesis for fishing touch cost (to verify in PIE):**
mod EffectPrototype e.g. `FW_FishingTouchStamina` with:
- `Type = EEffectType::Stamina`
- `ValueMin/Max = -8%` (or species table %)
- `Positive = EBeneficial::Negative`

If negative `Stamina` is ignored by the game, fall back to ModKit BP stamina node (search ZoneKit/CraftableZone examples) — still no private API inventing.


---

## Verified stamina (2026-09-20)

### Drain via effect (works in-game)
- Custom effect inheriting `WaterStaminaInstant` via  
  `{refurl=@BaseGame/EffectPrototypes.cfg;refkey=WaterStaminaInstant}`  
  with `ValueMin/Max = -N%`, `Positive = EBeneficial::Negative`.
- Test item `FWTestStaminaNeg` → consume → bar drops. Do **not** use bare `{refkey=[0]}` in mod EffectPrototypes (looks up local file → ModKit missing-field spam).

### Read current stamina (official Blueprint API — Stalker2.Obj)
Source: https://cdn.stalker2.com/guides/Blueprint_API_Guide.pdf

| Node | Returns |
|---|---|
| **Get SP** | `float` current stamina |
| **Get Max SP** | `float` max stamina |
| **On Stamina Changed** | event `Prev SPValue`, `Next SPValue` |
| Set SP / Set Max SP | exist — prefer effect for drain; use Set only if needed |

### Fight touch gate (lose fish if not enough)

On each successful Interact window press:

1. `PlayerObj` = current player `Obj` (same path you use elsewhere / Is Current Player).
2. `Cost = Get Max SP() * (StaminaCostPct / 100.0)`  
   (species table: Common 8, Heavy 12, …).
3. **If `Get SP() < Cost` → Fail** (fish escapes). Do **not** apply drain.
4. Else apply drain:
   - preferred: apply effect SID for that cost (or one generic −% effect scaled — if apply-by-SID isn’t exposed, use `Set SP` to `Get SP() - Cost` as fallback),
   - then `TouchesDone++`.

Regen / artifacts / overload need no extra code: `Get SP` / regen already reflect them.

Optional: bind **On Stamina Changed** during Fight to abort if SP hits 0 mid-window.
