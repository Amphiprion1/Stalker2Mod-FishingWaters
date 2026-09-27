# Rod rebase onto Gauss Scar (GunGauss_Scar_SP)

Date: 2026-09-27. Replaces the RPG base (Docs/ROD_RPG_BASE.md, now superseded).

## Why

The RPG base (MaxAmmo = 1) made the player reload a hook after every cast.
Gerald wants a big magazine so reloads are rare. Bait is used up separately (FW_Bait_Worm / FW_Bait_Meat consumables).

The Gauss Scar has a vanilla default magazine with 99 rounds: `GunGauss_Scar_MagHuge`.
It fires one shot at a time, spawns no shells, and has no belt / ammo box mesh.
99 also matches the hook stack size (FW_Hook MaxStackCount = 99).

Rule kept from the UDP test: the reload animation only works with the **exact vanilla mag SID**.
A custom mag SID (like the old FW_FishingRod_Mag) gave no hand animation and a long wait.
So the capacity is 99, not 100. Do not make a custom mag to get 100.

## What changed (short)

| File | Change |
|------|--------|
| ItemPrototypes/FishableWaters_WeaponPrototypes.cfg | refkey `GunGauss_Scar_SP`; `NPCWeaponAttributes = Scar_GunGauss_SP_NPC`; `MeshPrototypeSID = Gauss_Skeletal`. Weight, cost, grid 2x1, icons, world mesh `FW_FishingRod_Mesh` unchanged. |
| WeaponGeneralSetupPrototypes/FishableWaters_WeaponGeneralSetup.cfg | refkey `GunGauss_Scar_SP`; `MaxAmmo = 99`; `MinJamChance`/`MaxJamChance = 0`; `FireTypes {bskipref} [0] SemiAutomatic` + `DefaultFireType = SemiAutomatic`; ReloadTypes override removed (inherits Full + Tactical); `WeaponReloadTimePerAttachment [0] = GunGauss_Scar_MagHuge` (all multipliers 1.0); `CompatibleAttachments {bskipref}` = only the mag on `jnt_clip_base`; `PreinstalledAttachmentsItemPrototypeSIDs {bskipref}` = only the mag, hidden; `LastShootIdle = false`; `DisplayBP = Blueprint''`. Kept: `FireEventOneShot = AkAudioEvent''`, `bSpawnShell = false`, `FW_FishingRodShake`, `A045`, dead NPC ammo 0, `AmmoTypeProjectiles {bskipref} [0] Default -> empty`, and Gerald's `WeaponStaticMeshParts` block (SM_FW_FishingRod on jnt_offset) byte for byte. |
| CharacterWeaponSettingsPrototypes/FishableWaters_PlayerWeaponSettings.cfg | refkey `GunGauss_Scar_SP_Player_WS`; zero damage kept; added `CoverPiercing = 0.0` (Gauss has 10). |
| WeaponAttributesPrototypes/FishableWaters_PlayerWeaponAttributes.cfg | refkey `GunGauss_Scar_SP_Player` (this brings the Gauss Scar AnimBP `AnimBP_gauss_scar_fp`); added `DisplayBP = Blueprint''`. |

Not changed: AmmoPrototypes (FW_Hook), camera shakes, MeshPrototypes, AttachPrototypes, all Blueprints / .uasset.

Why `{bskipref}` on CompatibleAttachments / Preinstalled: vanilla Scar merges these lists with its parent GunGauss_SP,
so without it the X4 Gauss_Scope, Gauss_DefaultMuz and laser would be preinstalled on the rod.

`DisplayBP` (Gauss ammo screen) exists in vanilla in both GeneralSetup (GunGauss_SP) and PlayerWeaponAttributes (GunGauss_SP_Player), so it is cleared in both.

Caliber note: magazines have no caliber field. Caliber lives only on the weapon (`AmmoCaliber`) and on the ammo (`Caliber`). So the vanilla Gauss mag works with A045 hooks.

## Do NOT touch

- Never write a cfg entry for `GunGauss_Scar_MagHuge` or its parent `GunGauss_MagIncreased` (no MaxAmmo change, no FittingWeaponsSIDs, no mesh change). That would change the vanilla Gauss Scar too, or break the reload anim.
- Never edit anything under `Stalker2/Content/GameLite` (vanilla, read-only).
- Keep the mag SID in WeaponReloadTimePerAttachment, CompatibleAttachments and Preinstalled exactly `GunGauss_Scar_MagHuge`.
- Keep projectile `empty` (never `PGA`, the Gauss slug).

## Known risks / things to check

1. Rod position: SM_FW_FishingRod was lined up against the RPG (TEST actor uses SM_gl_rpg7). The Gauss skeleton and hand pose differ, so the rod may sit offset or rotated. Re-align using the Gauss mesh (SM_sr_gauss) if needed.
2. The Gauss battery (mag mesh) will show at `jnt_clip_base` near the rod, and the reload anim swaps it by hand.
3. The Gauss behaves like a bolt-action weapon (`BoltActionState = ReadyToShoot`). It may play a cycle / lever anim after every cast. Test toggle if it looks bad: add `BoltActionState = EBoltActionWeaponState::NoBoltAction` to the general setup.
4. `FireInterval` is inherited: 2.4 s between casts. Add `FireInterval = 1.5` if it feels too slow.
5. The mag inherits Cost 19000 / Weight 0.4 from GunGauss_MagIncreased. Check the rod's trader price and weight.
6. Gauss sounds that live inside animations (charge / reload) cannot be muted from cfg. Shot sound is muted.
7. Gauss muzzle flash (ParticlesBasedOnHeating in PlayerWeaponAttributes) is still inherited, like the RPG flash before. Neutralise later if visible (do not use an empty {bskipref} struct, see ConsumablePrototypes note).
8. Vanilla .45 ammo may still load (A045), as before.
9. Rods spawned before this change may not get the new mag. Spawn a new rod for testing.

## In-game test (after ModEditor reload / PIE)

```
XCreateItemInInventoryByID FW_FishingRod_Wep 0 1 1
XCreateItemInInventoryByID FW_Hook 0 99 1
XCreateItemInInventoryByID FW_Bait_Worm 0 5 1
```

Check:

1. Equip rod: rod mesh visible in hands; no Gauss scope, no ammo screen.
2. Reload: loads up to 99 hooks with a normal (~3 s) animated reload, not a long silent wait.
3. Fire: one cast per click (hold does not auto-fire), hook count -1, fishing session starts (FW_FishingRodShake / BP), no Gauss shot sound, no damage.
4. Note: any cycle anim after the cast, battery mesh position, rod offset, muzzle flash.

## Rollback

1. Copy files from `Docs/backup_rpg_rod_2026-09-27/*.cfg.txt`.
2. Rename each `Something.cfg.txt` -> `Something.cfg` and put it back over the file with the same name:
   - FishableWaters_WeaponPrototypes.cfg -> Content/GameLite/ModGameData/FishableWaters/ItemPrototypes/
   - FishableWaters_WeaponGeneralSetup.cfg -> .../WeaponData/WeaponGeneralSetupPrototypes/
   - FishableWaters_PlayerWeaponSettings.cfg -> .../WeaponData/CharacterWeaponSettingsPrototypes/
   - FishableWaters_PlayerWeaponAttributes.cfg -> .../WeaponData/WeaponAttributesPrototypes/
3. Restart ModEditor / PIE.