# FishableWaters plan

## Ladder (cfg written 2026-10-10, not play-tested)

Six species. Depth is in centimetres and is still a Blueprint trace. Unknown depth must not roll a fish whose `MinDepthCm` is above 0. The patriarch also needs two `EArtifactSoul` in the inventory (possession, not equipped).

| Row | Loot | Bait | Depth cm | Fight |
|---|---|---|---|---|
| FWBleak | FWBleak | FW_Bait_Worm | 20–80 | wait 1–3, touches 2–3, stam 5% |
| FWPerch_Common | FWPerch | FW_Bait_Worm | ≥ 40 | wait 2–5, touches 3–5, stam 8% |
| FWPerch_Heavy | FWPerchHeavy | FW_Bait_Meat | ≥ 60 | wait 3–6, touches 6–8, stam 10% |
| FWPike | FWPike | FW_Bait_Meat | ≥ 80 | wait 3–7, touches 7–10, stam 12% |
| FWCatfish | FWCatfish | FW_Bait_Meat | ≥ 120 | wait 4–8, touches 9–12, stam 14% |
| FWCatfish_Patriarch | FWCatfishPatriarch | FW_Bait_Deep | ≥ 150 | wait 6–10, touches 12–16, stam 16% |

Pier weights when the gates pass: Bleak 40, Perch 35, Heavy 20, Pike 20, Catfish 15, Patriarch 10. Drop a fish that fails depth, bait, or the Soul count, then roll on what remains.

Harvest and cook amounts are in `Docs/LADDER_EDITOR.md`. Craftable Zone quest cfg matches those amounts. The binary data tables do not, until Gerald types the rows.

## Still out of cfg

- `ST_FW_FishSpecies` fields, `DT_FW_FishSpecies` rows, `DT_Disassemble`, `DT_CraftRecipes`, `DT_CraftCategories`, `LOC_Mod_FishableWaters`
- `BP_FW_Shake_BaitDeep`
- Depth trace, Soul count, line-mesh component
- Do not recreate the rod weapon, the hook, `FW_FishingSession`, or `DT_FW_FishingZones`

## Test matrix (one cook, after the editor list)

1. Use each bait on the pier. Active bait matches the item just used. Deep bait must not register as meat.
2. Shallow water rolls only bleak and common perch. A failed depth trace rolls neither heavy, pike, catfish, nor patriarch.
3. Patriarch does not roll with one Soul, and does roll with two Souls, the deep bait, and depth ≥ 150.
4. Disassemble each carcass. Yields match the editor table. The knife is not consumed.
5. Craft worm bait (tushkan + salt → 3), meat bait (flesh → 2), deep bait (pseudodog + salt → 1). Swiss knife stays.
6. Cook each meal. Ingredient counts match. Exactly one fuel unit is removed. Frying pans and pots stay.
7. Eat grilled bleak, perch skewer, pike steak, catfish stew, patriarch meal. Regen on the last meal is untested: `Duration` on `RegenStamina` is not in the wiki.
