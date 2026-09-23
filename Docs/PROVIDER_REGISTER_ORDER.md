# FishableWaters — provider register order (CZ-style)

You already mirrored CraftableZone: `BPI_FishableWaters_API` + `BPI_FishableWaters_Provider`.

## Do not trust one scan alone

There is **no** BP event that means “every Game Feature actor in the load order has already spawned.”  
`BeginPlay` on your ModWorldSubsystem can run **before** or **after** another mod’s MWSS.

So use the same robustness CZ needs in practice: **push + pull**, idempotent registers.

## Recommended

### 1) Patch mods PUSH (main path)

Content mod (like `CraftableZone_Example1`):
- `.uplugin` → `"Plugins": [ { "Name": "FishableWaters", "Enabled": true } ]`  
  (ensures your plugin/assets exist; does **not** alone guarantee actor BeginPlay order)
- Their `MWSS_*` implements `BPI_FishableWaters_Provider`
- On **their** `BeginPlay`:
  1. Find API: `Get All Actors With Interface` → `BPI_FishableWaters_API` (or Get Actor of Class `BP_Mod_FishableWaters`)
  2. If found → call `RegisterFishSpecies` / `RegisterZoneSpawn` (or one `ProvideToAPI(API)` event on the Provider interface that does all of it)
  3. If not found → **Retry**: Set Timer 0.1–0.25 s, max 10–20 tries, then give up + Print

This answers “how do they know my subsystem exists?”: they search for the API; if too early, they retry. Your actor is spawned from `WorldSubsystemData` when FishableWaters GF is active — dependency + retry is enough.

### 2) Framework PULL (catch early providers)

On `BP_Mod_FishableWaters` **BeginPlay**:
1. Seed defaults from your DTs into the registry
2. `Get All Actors With Interface` → `BPI_FishableWaters_Provider`
3. For each → call Provider event e.g. `OnFishableWatersCollect(API self)` so they push data now
4. Optional: **one** deferred rescan (Timer 0.0 next tick, or Delay 0.2 once) for providers that spawned the same frame after you

Do **not** rescan every tick forever.

### 3) Idempotent API

`RegisterFishSpecies` / `RegisterZoneSpawn` must overwrite or merge safely if called twice (BeginPlay + retry + pull).

## What NOT to use as the only hook

| Hook | Why insufficient alone |
|---|---|
| Only FW `BeginPlay` scan | Late GF providers miss the scan |
| Only Provider `BeginPlay` without retry | FW MWSS may not exist yet that frame |
| Game Feature Activate on the Feature asset | Fine for init, still not ordered vs other mods’ actors |
| Hoping load order alone | Fragile across users’ `~mods` names |

## Clone of Example1 layout

| CraftableZone | FishableWaters |
|---|---|
| `BPI_CZ_API` on CZ MWSS | `BPI_FishableWaters_API` on `BP_Mod_FishableWaters` |
| `BPI_CZ_Provider` on example MWSS | `BPI_FishableWaters_Provider` on patch MWSS |
| Example `.uplugin` depends on CraftableZone | Patch `.uplugin` depends on FishableWaters |
| Example registers recipes/disassemble | Patch registers fish + zone keys (`SM_water_plane_…`) |

## Minimal FW BeginPlay sketch

1. Load DT → Register* (your defaults)
2. ForEach Provider → `OnFishableWatersCollect(self as API)`
3. Set Timer 0.2 s → same ForEach once more (optional safety)

## Minimal Provider BeginPlay sketch

1. Find API (interface)
2. If valid → Provide (Register*)
3. Else → retry timer

That is the safe answer: **no single magic event** — dependency + push with retry + one pull on the framework.
