# FishableWaters GameData

Sole cfg home: `Content/GameLite/ModGameData/FishableWaters/`

- `ItemPrototypes/` — fish items (`FWPerch` and fillets) and baits (`FW_Bait_Worm`, `FW_Bait_Meat`)
- `EffectPrototypes/` + `CameraShakePrototypes/` — bait Use shakes (`FW_Effect_BaitWorm` / `FW_Effect_BaitMeat`)
- `QuestPrototypes/` + `QuestNodePrototypes/` — Craftable Zone disassemble only: `FishableWaters_FWPerch_Disassembling_FWPerch_Remove` (removes 1× `FWPerch`)

The weapon rod (`FW_FishingRod_Wep`), hook ammo (`FW_Hook`), mag (`FW_Rod_Mag`), and rod mesh prototypes were removed. Do not add them back.

The fishing quest `FW_FishingSession` (quest, nodes, and the item-container spawn `2F6397BE4CDEBD802D5B4CAE08C80524`) was removed. Do not add it back.

- Localization/ - **stub only** (`Resources/Localization/FW_Feedback_Loc_stub.cfg`): copy SIDs into Content/DataTables/LOC_Mod_FishableWaters (not loaded as GameData)
