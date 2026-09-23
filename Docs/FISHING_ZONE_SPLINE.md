# FishableWaters — BP_FW_FishingZone (spline shore) + pick closest

## Actor BP_FW_FishingZone

Components:
- Root (DefaultSceneRoot)
- **Spline** (Spline Component) — Closed Loop = true; place points along the **shoreline**

Variables (Instance Editable):
| Name | Type | Default |
|---|---|---|
| ZoneRow | Name | DefaultAnywhere |
| ShoreBand | Float | 350 |
| FacingDotMin | Float | 0.25 |
| CachedCentroid | Vector | (runtime, World) |

### Construction Script / BeginPlay — cache centroid

1. `Num = Spline.Get Number Of Spline Points`
2. Sum = 0, Count = 0
3. For `i = 0 .. Num-1`:
   - `P = Spline.Get Location at Spline Point (i, World)`
   - Sum += P; Count++
4. `CachedCentroid = Sum / Count` (if Count > 0)

(Closed loop: if last point duplicates first, you can loop `Num-1` — either is fine for centroid.)

### Function `EvaluateForPawn(Pawn) → (Allowed: Bool, Dist: Float)`

1. `PlayerLoc = Pawn.Get Actor Location`
2. `Closest = Spline.Find Location Closest to World Location (PlayerLoc)`  
   (Coordinate Space = World; returns world location on the curve)
3. `Dist = Vector Distance (PlayerLoc, Closest)`
4. If `Dist > ShoreBand` → return (false, Dist)
5. Facing toward water (centroid = interior):
   - `ToWater = Normal (CachedCentroid - PlayerLoc)`  
     (use 2D if you want: zero Z on both before Normal — more stable on slopes)
   - `Fwd = Pawn.Get Actor Forward Vector` (optionally zero Z + re-normalize)
   - `Dot = Dot Product (Fwd, ToWater)`
6. If `Dot < FacingDotMin` → return (false, Dist)
7. Return (true, Dist)

---

## On BP_FlishableWaters — Function `FindBestFishingZone(Pawn) → (Found: Bool, Zone: BP_FW_FishingZone)`

1. `Get All Actors Of Class` → `BP_FW_FishingZone`
2. Locals: `BestZone` (object, none), `BestDist` = 999999
3. ForEach Zone:
   - Call `Zone.EvaluateForPawn(Pawn)` → Allowed, Dist
   - If Allowed **and** Dist < BestDist:
     - BestDist = Dist
     - BestZone = Zone
4. If BestZone is valid → return (true, BestZone)
5. Else → return (false, none)

**Closest among valid only** — a nearer zone that fails facing/band is ignored; the next valid wins.

Optional cull (many zones later): before Evaluate, skip if `Distance(Player, Zone.CachedCentroid) > 5000`.

---

## StartFishingSession (replace DefaultAnywhere hardcode)

After PlayerObj / StartLocation / StartHP cache:

1. `Pawn = Get Player Pawn (0)`
2. `FindBestFishingZone(Pawn)` → Found, Zone
3. If **not** Found:
   - Print "Pas de zone de peche" (or silent)
   - return (do **not** enter WaitingBite)
4. `ActiveZoneRow = Zone.ZoneRow`
5. `ResolveActiveFish` — if false, return
6. Enter WaitingBite as today (WaitMin/Max from ActiveFish)

---

## Placement tips (your pond screenshot)

1. Drop `BP_FW_FishingZone` at the pond.
2. Select Spline → add points **on the waterline** all the way around; enable **Closed Loop**.
3. Keep points in order (don’t cross). Centroid should land roughly in the middle of the water.
4. Set `ZoneRow` to a row in `DT_FW_FishingZones`.
5. Tune `ShoreBand` (start ~350) so standing on the bank passes, standing deep inland fails.
6. Tune `FacingDotMin` (~0.25 ≈ 75° cone) so looking at water works, looking at woods fails.

Debug: after Evaluate, Print Dist + Dot + ZoneRow.

---

## Why centroid (not spline Right vector)

Right/Left of the spline flips if you draw clockwise vs CCW.  
`CachedCentroid - Player` always points toward the water interior for a shore loop around the pond.
