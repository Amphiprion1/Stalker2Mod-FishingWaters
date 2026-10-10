# FishableWaters GameData

Sole cfg home: `Content/GameLite/ModGameData/FishableWaters/`

- `ItemPrototypes/` — fish, fillets, meals (`FishableWaters_LadderItems.cfg`) and baits (`FW_Bait_Worm`, `FW_Bait_Meat`, `FW_Bait_Deep`)
- `EffectPrototypes/` + `CameraShakePrototypes/` — bait Use shakes, plus `FW_Effect_MealRegenStamina` on the patriarch meal
- `QuestPrototypes/` + `QuestNodePrototypes/` — Craftable Zone only. Kept: `FishableWaters_FWPerch_Disassembling_FWPerch_Remove` (removes 1× `FWPerch`). Added: disassemble removes for the new carcasses, three bait craft quests, five cook-ingredient quests, and one cook quest per meal for `KpDryFuel`, `KpCharcoal`, `KpKerosene`, `KpGasBaloon` (count 1). No cook-tool quests. Numbers and DT rows: `Docs/LADDER_EDITOR.md`.

The weapon rod (`FW_FishingRod_Wep`), hook ammo (`FW_Hook`), mag (`FW_Rod_Mag`), and rod mesh prototypes were removed. Do not add them back.

The fishing quest `FW_FishingSession` (quest, nodes, and the item-container spawn `2F6397BE4CDEBD802D5B4CAE08C80524`) was removed. Do not add it back.

- Localization/ - **stub only** (`Resources/Localization/FW_Feedback_Loc_stub.cfg`): copy SIDs into Content/DataTables/LOC_Mod_FishableWaters (not loaded as GameData)
