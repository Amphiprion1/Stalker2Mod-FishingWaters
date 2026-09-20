# Weapon rod — test after restage NewContent

## Spawn
```
XCreateItemInInventoryByID FW_FishingRod_Wep 0 1 1
XCreateItemInInventoryByID FW_Bait 0 10 1
```

## Expect
1. Equip rod in pistol slot (UDP clone → pistol slot)
2. Reload loads 1 bait
3. Fire → CameraShake BP → StartFishingSession (same as old Use)
4. Aim → férer (existing IA_Aim)

## If WeaponData missing in ModKit
Move/copy `WeaponData` tree under mod `Content/GameLite/GameData/WeaponData/` (official guide layout) and retest.

## Legacy
`FW_FishingRod` (GuitarUsable) still in ItemPrototypes for fallback.
