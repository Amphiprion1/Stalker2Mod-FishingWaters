# FishableWaters — DataTables (species + zones)

## Why two tables

- **DT_FW_FishSpecies** = what a fish *is* (wait time, fight stamina/touches, loot).
- **DT_FW_FishingZones** = which fish can appear *here*, each with a **Weight**.
- Spot BP only stores a **zone RowName** (or a soft ref to one zone row). No duplicated fish stats on the spot.

Weighted pick: `Sum(Weight)`, roll `R in [0, Sum)`, walk cumulative until hit. Ignore `Weight <= 0`.

---

## 1) Struct `ST_FW_FishSpecies` (User Defined Struct)

Create in ModEditor under `Content/DataTables/` (or `Content/Structs/`).

| Variable | Type | Notes |
|---|---|---|
| DisplayName | Text | Optional UI / debug |
| LootItemSID | String | e.g. `FWPerch` — passed to `XCreateItemInInventoryByID` |
| WaitMinSec | Float | WaitingBite timer min |
| WaitMaxSec | Float | WaitingBite timer max (`>= WaitMinSec`) |
| TouchesMin | Integer | Fight Aim presses min |
| TouchesMax | Integer | Fight Aim presses max |
| GapMinSec | Float | Optional delay between touches (v0 can ignore) |
| GapMaxSec | Float | Optional |
| StaminaCostPct | Float | % of **Max SP** per successful touch (same as Fight v0) |

Row Name = species id (`FWPerch_Common`, `FWPerch_Heavy`, …). Do **not** duplicate Row Name in a String field.

---

## 2) Struct `ST_FW_FishSpawnEntry` (nested)

| Variable | Type | Notes |
|---|---|---|
| Fish | Data Table Row Handle | Table = `DT_FW_FishSpecies`, Row = species Row Name |
| Weight | Float | Relative chance (`>= 0`) |

In the struct editor: add `Fish`, set pin type to **Data Table Row Handle**, then in the DataTable asset set Row Type to `ST_FW_FishSpecies` so the handle is constrained (or leave unconstrained and always pick the fish DT).

---

## 3) Struct `ST_FW_FishingZone`

| Variable | Type | Notes |
|---|---|---|
| DisplayName | Text | Optional |
| Spawns | Array of `ST_FW_FishSpawnEntry` | At least one entry with Weight > 0 |

Row Name = zone id (`DefaultAnywhere`, `LesserZone_Pond_01`, …).

---

## 4) DataTables

| Asset | Row struct | Path |
|---|---|---|
| `DT_FW_FishSpecies` | `ST_FW_FishSpecies` | `Content/DataTables/DT_FW_FishSpecies` |
| `DT_FW_FishingZones` | `ST_FW_FishingZone` | `Content/DataTables/DT_FW_FishingZones` |

### Seed rows (match `species_table_v0.txt`)

**DT_FW_FishSpecies**

| Row Name | LootItemSID | WaitMin | WaitMax | TouchesMin | TouchesMax | GapMin | GapMax | StaminaCostPct |
|---|---|---|---|---|---|---|---|---|
| FWPerch_Common | FWPerch | 2.0 | 5.0 | 3 | 5 | 1.2 | 2.5 | 8 |
| FWPerch_Heavy | FWPerch | 3.0 | 6.0 | 6 | 9 | 1.0 | 2.0 | 12 |

**DT_FW_FishingZones**

| Row Name | Spawns |
|---|---|
| DefaultAnywhere | FWPerch_Common @ 70, FWPerch_Heavy @ 30 |
| (later spots) | … |

`DefaultAnywhere` is what `BP_FlishableWaters` uses until water spots exist.

---

## 5) ModEditor click-path

1. Content Browser → `Mods/FishableWaters/Content/DataTables` (create folder if needed).
2. Right-click → Blueprint → **Structure** → `ST_FW_FishSpecies` → add variables above → Compile/Save.
3. Structure → `ST_FW_FishSpawnEntry` → Fish (Data Table Row Handle) + Weight (Float).
4. Structure → `ST_FW_FishingZone` → DisplayName + Spawns (Array of ST_FW_FishSpawnEntry).
5. Right-click → Miscellaneous → **Data Table** → pick `ST_FW_FishSpecies` → name `DT_FW_FishSpecies` → Add rows from seed table.
6. Data Table → pick `ST_FW_FishingZone` → `DT_FW_FishingZones` → row `DefaultAnywhere` with two Spawns.
7. On `BP_FlishableWaters`: variable `FishSpeciesDT` (Data Table soft/object ref) = `DT_FW_FishSpecies`; `FishingZonesDT` = `DT_FW_FishingZones`; `ActiveZoneRow` (Name) = `DefaultAnywhere`.

Do **not** wire weighted pick into the FSM yet unless you want it now — assets alone unlock the water step.

---

## 6) Later BP helpers (not required for this prep)

- `ResolveZoneSpawns(ZoneRow)` → Array of entries from `DT_FW_FishingZones`.
- `PickWeightedFish(Entries)` → Row Handle / Row Name.
- `GetFishRow(RowName)` → `ST_FW_FishSpecies` → drive Wait timer + Fight stamina/touches + loot SID.

Replace hardcoded Wait 2–5 s / 3 touches / 8% / `FWPerch` with the resolved row.

---

## Files kept in sync

- Draft text: `Content/GameLite/ModGameData/FishableWaters/Fishing/species_table_v0.txt`
- Zones draft: `.../Fishing/zones_table_v0.txt`
- This doc: `Docs/DATATABLES_V0.md`
