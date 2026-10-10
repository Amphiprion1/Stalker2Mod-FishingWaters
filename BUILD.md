# FishableWaters build notes

## 2026-10-10 ladder cfg

Text only. No Package Mod. No Blueprint graph edits. No `.uasset` data tables edited.

Added:

- `Content/GameLite/ModGameData/FishableWaters/ItemPrototypes/FishableWaters_LadderItems.cfg` — carcasses, raw fillets, meals, trophy. Vanilla `DeadFish` / `Bread` meshes. Shared perch icons.
- `FW_Bait_Deep` in `FishableWaters_ConsumablePrototypes.cfg`
- `FW_Effect_BaitDeep`, `FW_Effect_MealRegenStamina` in `FishableWaters_EffectPrototypes.cfg`
- `FW_CamShake_BaitDeep` pointing at `BP_FW_Shake_BaitDeep` (asset not created yet)
- 33 Craftable Zone quest pairs (prototype + nodes). The existing `FishableWaters_FWPerch_Disassembling_FWPerch_Remove` quest was left as-is.
- `Resources/Localization/FW_Feedback_Loc_stub.cfg` strings. This file is not loaded.
- `Resources/ItemGeneratorPrototypes/FishableWaters_ItemGeneratorPrototypes.cfg` adds `FW_Bait_Deep` x2. This file is not loaded until it is copied into ModGameData.
- `Resources/Fishing/species_table_v0.txt`, `Docs/LADDER_EDITOR.md`, `Docs/DATATABLES_V0.md`, `PLAN.md`

Changed on the existing cooked perch item only: `LocalizationSID = FWPerchFilletCooked`, effects `SausageSatiety2` + `BreadHealing1`. Its Lootable Zone skewer mesh is unchanged. `RemoveDrunkness10` and `BreadSatiety2` are no longer on that item.

Quest naming, copied from `CraftableZone_Example1` (ModID string there is `CraftableZoneExample`):

- Disassemble remove: `FishableWaters_{Item}_Disassembling_{Item}_Remove` removes 1 carcass.
- Craft: `FishableWaters_{Result}_Craft_{Result}` removes every ingredient in order.
- Cook ingredients: `FishableWaters_{Result}_Cook_{FirstIngredient}` removes that ingredient with the recipe count.
- Cook fuel: `FishableWaters_{Result}_Cook_{FuelSID}` removes 1. Fuels: `KpDryFuel`, `KpCharcoal`, `KpKerosene`, `KpGasBaloon`.

Not generated:

- Cook-tool quests. The example removes the tool item. A frying pan must not be deleted. If a cook log names a missing `*_Cook_{ToolSID}` quest, add that one quest with the count CZ expects.
- A second craft path for boar or chimera. The quest SID is the result SID, so one ingredient list per result. Flesh and pseudodog are the lists in cfg. Swap the SID in the quest and in the data table together if you want the other meat.
- A LootingItems plugin dependency. The plugin `Name` inside that pak was not read. `CraftableZone` is already in `FishableWaters.uplugin`.
- Rod, hook, `FW_FishingSession`, spawn `2F6397BE4CDEBD802D5B4CAE08C80524`, `DT_FW_FishingZones`.
