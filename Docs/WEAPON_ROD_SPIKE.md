# Weapon-rod spike (canonical path)

Decision 2026-09-20: abandon inventory-Use close; rod = **Weapon**, bait = **Ammo** (mag 1).

Official guide: https://cdn.stalker2.com/Mod.io/S2_HoC-Zone_Kit_Custom_Weapon_Modding_Guide.pdf  
Clone base: **GunUDP_HG** (simplest pistol).

## Control map

| Input | Meaning |
|-------|---------|
| Equip rod (weapon slot) | Leaves inventory naturally |
| **Fire** | Cast / start session (spend 1 bait) |
| **Aim** (`IA_Aim`) | Férer (Fight touch) — already proven |
| Reload | Load 1 bait into mag |

## Fire → FSM hook (do NOT invent On Shot bind)

Same class as Aim/BeforeUse: `On Shot` / `On Fire Pressed` are likely Obj overrides, not Assign from PlayerObj.

**Use proven path:** `ShootCameraShakePrototypeSID` on GeneralWeaponSetup → our `FW_FishingRodShake` / `BP_FW_FishingRodUse` → `StartFishingSession`.

Cast = one shot = CameraShake = session start. Mag 1 ⇒ one cast per reload.

## Prototype chain (minimal)

Under `ModGameData/FishableWaters/` (AddConfigsPath already points here):

| Folder / file | SID (proposed) | Inherit / copy from |
|---------------|----------------|---------------------|
| `ItemPrototypes/` | `FW_FishingRod_Wep` | `GunUDP_HG` via BaseGame WeaponPrototypes |
| `ItemPrototypes/` | `FW_Bait` | `TemplateAmmo` / A045-like |
| `WeaponData/WeaponGeneralSetupPrototypes/` | `FW_FishingRod_Wep` | `GunUDP_HG` setup; **MaxAmmo=1**; ShootCameraShake=`FW_FishingRodShake`; mesh parts → rod |
| `WeaponData/.../PlayerWeaponAttributes/` | `FW_FishingRod_Wep_Player` | UDP player attrs; later swap AnimBP if needed |
| `WeaponData/.../PlayerWeaponSettings/` | damage ≈ 0 | so cast doesn’t kill |
| Mag attach | keep UDP MagDefault **or** custom mag capacity 1 | verify Mag capacity field on attach |

Old GuitarUsable `FW_FishingRod` → rename/keep as `FW_FishingRod_Legacy` for a while (don’t delete until weapon casts).

## Build order (verify each)

1. Clone UDP → `FW_FishingRod_Wep` spawns + equips (vanilla meshes OK first)
2. `MaxAmmo=1` + bait ammo SID — Fire consumes 1
3. Point `ShootCameraShakePrototypeSID` at existing fishing shake → Print / StartFishingSession
4. Swap mesh to fishing rod mesh
5. Damage 0 / no projectile harm
6. Wire Aim férer (already on FlishableWaters)
7. Water last

## Risks

- Mag attach capacity may override MaxAmmo — test both
- Custom caliber enum may not exist — prefer existing caliber + unique ammo SID only our weapon lists
- AnimBP_udp_fp will look like a pistol until we find/adapt a pose — acceptable for spike
- ModGameData WeaponData path must be picked up by AddConfigsPath (same as Item/Effect) — if not, mirror official GameData layout under mod Content

## Out of scope for spike

Full FP fishing anim, water volume, species DT polish.
