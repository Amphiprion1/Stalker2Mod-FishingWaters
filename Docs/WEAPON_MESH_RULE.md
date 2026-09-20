# Weapon mesh rule (verified)

`MeshPrototypeSID` for a held weapon **must** be a **SkeletalMesh** (e.g. `UDP_Skeletal`).

Setting it to static `SM_gen_fishing_rod_01_a` → no FP hands, Fire/ShootCameraShake dead.

| Field | Value |
|-------|--------|
| `MeshPrototypeSID` | `UDP_Skeletal` (FP hold / fire) |
| `MeshInWorldPrototypeSID` | `FW_FishingRod_Mesh` (ground / world prop) |
| Icon | `T_FW_FishingRod` |

Later: custom skeletal rod or attach static mesh to weapon socket — not a one-line cfg swap.
