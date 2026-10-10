# FishableWaters plan

## Ladder (cfg written 2026-10-10, not play-tested)

Six species. Depth is in centimetres and is still a Blueprint trace. Unknown depth must not roll a fish whose `MinDepthCm` is above 0. `EArtifactSoul` is not a spawn gate. Equipped, each Soul adds `ArtifactIncreaseRegenStamina2` (+5 regen). The fight spends stamina per touch and waits `Gap` seconds, so vanilla regen (and carry weight) decide if the next touch is still payable.

Numbers below assume Max SP 100 and idle `RegenSP` 5. Two equipped Souls are treated as +10, which the cfg does not prove. Weight uses `InventorySPDrainCoef` 0.024 and the overweight coef 0.05, but the curve itself is not in the cfg.

Presses stay at least 2.4 s apart. Touch count is a range. Cost is higher so the longer rest does not refill the bar. A fight lasts about 8–25 s of waiting, nine presses at most.

| Row | Loot | Bait | Depth cm | Touches | Gap s | Stam % | Who finishes, full bar, idle |
|---|---|---|---|---|---|---|---|
| FWBleak | FWBleak | FW_Bait_Worm | 20–80 | 2–4 | 2.6–3.2 | 12 | anyone, including slowed regen |
| FWPerch_Common | FWPerch | FW_Bait_Worm | ≥ 40 | 4–6 | 2.6–3.0 | 16 | anyone, including slowed regen |
| FWPerch_Heavy | FWPerchHeavy | FW_Bait_Meat | ≥ 60 | 5–8 | 2.6–3.0 | 20 | light always. Slowed regen still wins 5 touches and fails on the 8th |
| FWPike | FWPike | FW_Bait_Meat | ≥ 80 | 7–9 | 2.8–3.1 | 23 | light, no artifact. The 9-touch fight ends near 5 SP. Slowed regen fails on the 7th |
| FWCatfish | FWCatfish | FW_Bait_Meat | ≥ 120 | 7–9 | 2.8–3.1 | 29 | one equipped Soul. No Soul fails by touch 6 or 7 |
| FWCatfish_Patriarch | FWCatfishPatriarch | FW_Bait_Deep | ≥ 150 | 7–9 | 2.4–2.8 | 40 | two equipped Souls. One Soul fails on the 7th even in the short fight. Regen cut from 15 toward 12 fails the 9-touch fight |

Pier weights when depth and bait pass: Bleak 40, Perch 35, Heavy 20, Pike 20, Catfish 15, Patriarch 10. Do not drop the patriarch for a missing Soul.

Harvest and cook amounts are in `Docs/LADDER_EDITOR.md`. Craftable Zone quest cfg matches those amounts. The binary data tables do not, until Gerald types the rows.

## Still out of cfg

- `ST_FW_FishSpecies` fields, `DT_FW_FishSpecies` rows, `DT_Disassemble`, `DT_CraftRecipes`, `DT_CraftCategories`, `LOC_Mod_FishableWaters`
- `BP_FW_Shake_BaitDeep`
- Depth trace, line-mesh component. No Soul-count gate.
- Do not recreate the rod weapon, the hook, `FW_FishingSession`, or `DT_FW_FishingZones`

## Test matrix (one cook, after the editor list)

1. Use each bait on the pier. Active bait matches the item just used. Deep bait must not register as meat.
2. Shallow water rolls only bleak and common perch. A failed depth trace rolls neither heavy, pike, catfish, nor patriarch.
3. Full stamina, standing still, presses at least 2.4 s apart. Heavy perch: a slowed regen wins a 5-touch fight and loses an 8-touch fight. Pike: slowed regen always loses, a light player with no artifact wins and is nearly empty after 9 touches. Catfish: no Soul loses, one equipped Soul wins. Patriarch: one Soul loses even the 7-touch fight, two equipped Souls win the 9-touch fight only while regen stays near 15.
4. Disassemble each carcass. Yields match the editor table. The knife is not consumed.
5. Craft worm bait (tushkan + salt → 3), meat bait (flesh → 2), deep bait (pseudodog + salt → 1). Swiss knife stays.
6. Cook each meal. Ingredient counts match. Exactly one fuel unit is removed. Frying pans and pots stay.
7. Eat grilled bleak, perch skewer, pike steak, catfish stew, patriarch meal. Regen on the last meal is untested: `Duration` on `RegenStamina` is not in the wiki.
