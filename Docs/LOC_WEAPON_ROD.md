# Localization — weapon rod + bait (ModEditor)

Open `Content/DataTables/LOC_Mod_FishableWaters` (DataTable).

Game keys from `LocalizationSID`:
- `sid_items_{SID}_name`
- `sid_items_{SID}_description`
- Ammo HUD may also use `sid_item_{SID}_short_name` or similar — if HUD still shows a raw key, add that row too.

## Rows to add

| Key | EN | FR |
|-----|----|----|
| `sid_items_FW_FishingRod_Wep_name` | Fishing rod | Canne à pêche |
| `sid_items_FW_FishingRod_Wep_description` | Cast with Fire (needs bait). Aim to set the hook. | Lancer avec Tir (appât requis). Viser pour férer. |
| `sid_items_FW_Bait_name` | Bait | Appât |
| `sid_items_FW_Bait_description` | Mag size 1. Reload into the fishing rod. | Chargeur 1. Recharger dans la canne. |
| `sid_item_FW_Bait_short_name` | Bait | Appât |
| `sid_item_fit_bait_short_name` | Bait | Appât |

(If the HUD key differs, copy the exact key from the HUD and add that row.)

Cfg already has `LocalizationSID = FW_FishingRod_Wep` / `FW_Bait`.

## Legacy

GuitarUsable `FW_FishingRod` removed from ItemPrototypes — use `FW_FishingRod_Wep` only.
