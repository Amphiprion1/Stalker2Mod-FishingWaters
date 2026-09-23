# FishableWaters — patchable config + water traces (no placed zone actor)

## Verdict

| Approach | Good for you | Other modders can patch? |
|---|---|---|
| DataTable only (`DT_FW_*`) | Yes (easy BP) | **No** — another pak cannot merge rows into your DT without replacing the asset |
| Custom `Fishing/*.cfg` under ModGameData | Feels native | **No** — ModKit only merges **known** GameData folders (ItemPrototypes, Effects, …). Plain fishing txt/cfg is ignored |
| ItemPrototypes for loot fish | Yes | **Yes** for items only (SID/stats of `FWPerch`), not wait/touches/weights |
| **Runtime registry on `BP_Mod_FishableWaters`** | Yes | **Yes** — other Game Features call Register* at Activate |

**Keep DT as your default seed** (or fill registry from DT once at startup).  
**Source of truth at runtime = maps on the Mod subsystem**, so patch mods append/override.

Same idea as ActorStatus PIR: framework owns the API; content mods register data.

---

## Target model

### A) Detect water (your idea — prefer this over spline/box)

On cast / `StartFishingSession`:

1. **Line trace** from camera (or pawn eyes) along look direction, length ~800–2000 uu (tune).
2. Prefer **Multi Line Trace By Channel** (Visibility) or a few short traces if needed.
3. Scan hits for an actor whose **class name** starts with `BP_Water_` → `WaterHit`, `WaterActor`.
4. Continue / second downward or same-direction hit for class/name containing `LandscapeStreamingProxy` (or Landscape) → `GroundHit`.
5. If no `WaterHit` → refuse cast (not aiming at water).
6. **Depth** ≈ distance(`WaterHit.ImpactPoint`, `GroundHit.ImpactPoint`)  
   (or `|Water.Z - Ground.Z|` if both hits exist). If no ground hit, depth = unknown → use zone defaults only.

Facing water is free: if the look-trace hits water, you are already oriented toward it.

### B) Zone key = water identity (no FishingZone actor)

Stable key options (pick one, document it):

1. **Best:** Actor Tag on the water BP, e.g. `FW_Zone=LesserPond_01`  
   (via your `ActorPatches` if you can tag vanilla water planes; patch mods can add tags too).
2. **OK:** Sanitized actor name / class name (`BP_Water_Something`).
3. **Fallback:** key `DefaultWater` if hit `BP_Water_*` but no mapping.

Lookup: `ZoneKey → spawn list (FishId + Weight)` in the registry.

Optional filters per fish: `MinDepth`, `MaxDepth` — skip entries when depth out of range before weighted pick.

### C) Registry API (patch surface)

On `BP_Mod_FishableWaters` (ModWorldSubsystem), Instance Editable maps/arrays + functions:

**Fish**
- `RegisterFishSpecies(FishId: Name, Data: ST_FW_FishSpecies)` — add/overwrite
- `GetFishSpecies(FishId) → ST_FW_FishSpecies`

**Zones**
- `RegisterZoneSpawn(ZoneKey: Name, FishId: Name, Weight: Float)` — add one weighted entry (merge)
- `ClearZone(ZoneKey)` / `ReplaceZone(ZoneKey, Spawns[])` — for full overrides
- `GetZoneSpawns(ZoneKey) → Array of ST_FW_FishSpawnEntry` (or your runtime struct with FishId+Weight)

**Boot**
- BeginPlay / Game Feature Activate: foreach row in `DT_FW_FishSpecies` + `DT_FW_FishingZones` → Register*  
  Map DT zone row names to water keys (`DefaultAnywhere` → fallback when ZoneKey unknown).

**Other modder flow**
1. Their Game Feature depends on / loads after FishableWaters.
2. On Activate: get `BP_Mod_FishableWaters` (Get Actor / subsystem accessor you already use).
3. `RegisterFishSpecies("FWPike_Rare", …)`
4. `RegisterZoneSpawn("BP_Water_LesserPond", "FWPike_Rare", 15)`  
   or `RegisterZoneSpawn("LesserPond_01", …)` if using tags.

Document in `Docs/MODDER_API.md` + example patch mod stub later.

---

## What to change in your current FSM

1. Drop requirement for placed `BP_FW_FishingZone` / spline for v1 (keep spline doc as optional art tool if you still want it).
2. `StartFishingSession`:
   - Trace → WaterActor + Depth
   - `ZoneKey = ResolveZoneKey(WaterActor)`
   - Spawns = registry.GetZoneSpawns(ZoneKey) or DefaultAnywhere
   - Filter by depth if fields exist
   - Weighted pick → fill `ActiveFish` (same as today)
3. Keep `ResolveActiveFish` but feed it **registry** instead of only `Get Data Table Row`.

DT stays useful as **authoring defaults inside your mod**, not as the extension point.

---

## Why not “only cfg”

`ItemPrototypes` remain the right place for **loot items** (`FWPerch`, future pike item).  
Wait / touches / zone weights are **fishing rules**, not vanilla GameData types — without a custom prototype class (C++), ModKit will not merge a `FishingZones.cfg`. Registry + DT seed is the BP-safe patch path.

---

## Minimal implementation order

1. Prove traces in PIE: Print class names of hits (confirm `BP_Water_*` + `LandscapeStreamingProxy`).
2. Gate cast on water hit; Print actor name/tags + depth.
3. Move DT load into Register* on subsystem boot.
4. Switch ResolveActiveFish to registry + ZoneKey from water.
5. Write MODDER_API.md; optional tiny example patch GF later.
