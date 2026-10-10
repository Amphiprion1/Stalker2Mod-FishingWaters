# FishableWaters

Fishing for S.T.A.L.K.E.R. 2. You stand on a small pier, use a bait, and press F. The fish you catch can be cut and cooked with Craftable Zone.

This file is for players who want the item ids, and for other mod authors who want to add a fishing spot, a fish, or a bait.

The plugin id is `FishableWaters`. Players must also have **Craftable Zone**. Cutting a fish into meat also needs **Faster Item Info Panel**, because Craftable Zone reads the item name from that panel.

## Words used here

- **SID** — the text id of an item, an effect, or a quest. Example: `FWPerch`. You type this id in configs and data tables. The game does not use the display name.
- **FishId** — the id of a fish *rule* (how long it fights, which bait it wants). It is often the same text as the item SID. It is not always the same.
- **Session** — the one actor that runs fishing. Its asset name is `BP_FW_FishableWaters`. Its actor tag is the same text.
- **Provider** — an actor from another mod. After 2 seconds, the session finds every provider and asks it to register fish.
- **Game Feature** — a Zone Kit mod. Another mod must list `FishableWaters` as a dependency.

## What the player does

1. Craft or find a bait: `FW_Bait_Worm`, `FW_Bait_Meat`, or `FW_Bait_Deep`.
2. Use the bait in the inventory. This equips it. It does not start fishing.
3. Stand on a fishing pier. A hint says you can fish. The key is **F** (gamepad bottom face button).
4. Press F. The session checks the water in front of the aim, the depth, and the bait.
5. Wait for the bite.
6. During the fight, press F again on each bite. Each press spends stamina. If you run out of stamina, the fish is lost.
7. On a catch, the carcass item is added to the inventory.
8. Cut the carcass with Craftable Zone (a knife is required, and it is not consumed). Cook the meat at a fire.

Walking away, taking damage, or opening the PDA stops the session. A failed fight keeps the active bait. A successful catch clears it.

The player stays standing. This is not a vanilla sit or talk action. Doors, chests, and NPCs keep their own F key. The fishing key is added only while you stand on the pier and you are not already looking at another interaction.

## How the mod is built

Three actors do the work.

| Asset | Path | Job |
|---|---|---|
| `BP_Mod_FishableWaters` | `/FishableWaters/BP_Mod_FishableWaters` | Mod World Subsystem. On its first tick it spawns the session. If the session is destroyed, it spawns it again. It has no fishing logic. |
| `BP_FW_FishableWaters` | `/FishableWaters/BP_FW_FishableWaters` | The session. States, bait, fish roll, fight, loot, and toasts. Actor tag: `BP_FW_FishableWaters`. |
| `BP_FW_Pier` | `/FishableWaters/Actors/BP_FW_Pier` | A pier you place in the world. Each copy has its own fish list. |

Do not spawn a second session. Do not put the fishing loop on a weapon, a quest, or a vanilla contextual action.

Zone Kit does not expose **Get World Subsystem** in Blueprints. The subsystem is not something other Blueprints can look up. Other mods talk to the **session actor**, through the interfaces below.

### Startup of the session

On BeginPlay, `BP_FW_FishableWaters` does this:

1. Read every row of `DT_FW_FishSpecies` and register it.
2. Register this mod's Craftable Zone recipes (craft, cook, disassemble).
3. **Wait 2 seconds.**
4. Find every actor that implements `BPI_FishableWaters_Provider`.
5. On each one, call `RegisterFishSpecies` and pass this session as `FW_API`.

There is one scan, 2 seconds after the session starts. A provider actor that appears later is not asked again. Spawn your provider at the start of your own mod, with no extra delay.

### The two interfaces

Both interfaces live in `/FishableWaters/`.

`BPI_FishableWaters_API` is on the session. Other mods **call** these functions:

| Function | Argument | Meaning |
|---|---|---|
| `RegisterFishSpecies` | `FishId` (name), `Fish` (`ST_FW_FishSpecies`) | Add or replace one fish rule. |
| `SetActiveBait` | `BaitSID` (string) | The bait the player just used. The next cast uses this id. |

`BPI_FishableWaters_Provider` is what **your** actor implements. It has one function:

| Function | Argument | Meaning |
|---|---|---|
| `RegisterFishSpecies` | `FW_API` (the API interface) | The session calls this. Inside it, you call the API. |

The two functions share the same name. They are not the same function.

- On **your** actor (Provider): you **receive** `FW_API`.
- On the **session** (API): you **send** the fish struct.

```
Session BeginPlay
  wait 2 seconds
  for each Provider actor:
      Provider.RegisterFishSpecies(FW_API = session)
          session.RegisterFishSpecies(FishId, fish data)
```

Calling `RegisterFishSpecies` twice with the same FishId is safe. The second call replaces the first.

### Pier and fish list

A pier does not contain the fight numbers. It only lists FishIds and weights.

On the pier instance, the variable is `Zone` (`ST_FW_FishingZone`):

- `spawns` — array of `ST_FW_FishSpawnEntry`
  - `FishId` — string. Must match a FishId already registered.
  - `Weight` — number. Higher means more common. `0` is ignored.

When the player walks into the pier, the pier gives this list to the session (`SetFishingZone`) and adds the input context `IMC_FW`. When the player leaves, the pier clears the zone and removes that context.

Only one zone is active. A second pier replaces the list. Do not overlap two piers.

The cast key is `IA_FW_Cast` in `/FishableWaters/Input/`. It is bound only on the session. Do not also bind it on the pier, or one press runs twice.

### One cast

1. State must be Idle, and a pier zone must be active.
2. A line trace along the player's aim looks for an actor whose class name **starts with** `BP_Water_`.
3. A second trace goes down from that hit to the ground. Depth is the distance between the two hits, in centimetres.
4. `ResolveActiveFish` keeps a spawn entry only when all of these are true:
   - `Weight` is greater than 0
   - the FishId exists in the species map
   - `RequiredBaitSID` is empty, or it equals the active bait
   - water depth is greater than or equal to `MinDepth`
5. One entry is picked at random, using the weights.
6. The state becomes WaitingBite for a random time between `WaitMinSec` and `WaitMaxSec`.
7. The state becomes Fight. The player must press F inside `ReactivityWindowInSecond`. Each good press spends `StaminaCostPct` percent of max stamina, then the session waits `GapMinSec`–`GapMaxSec` seconds. Vanilla stamina regen runs during that wait.
8. After enough presses, the session gives one `LootItemSID` and ends with Success.

If the cast is refused, the reason is `NoWater`, `NoFish`, or `NoBait`.

End reasons: `Success`, `FailStamina`, `FailTimeout`, `AbortMove`, `AbortDamage`, `AbortPDA`, `AbortBusy`, `Cancel`.

`EndFishingSession` is the only place that stops timers and returns to Idle.

### Fish rule fields

Struct: `/FishableWaters/Structures/ST_FW_FishSpecies`

| Field | Type | Meaning |
|---|---|---|
| `LootItemSID` | string | Item added to the inventory on a catch. |
| `WaitMinSec`, `WaitMaxSec` | float | Seconds before the bite. |
| `TouchesMin`, `TouchesMax` | int | How many good F presses the fight needs. |
| `GapMinSec`, `GapMaxSec` | float | Seconds between presses. |
| `StaminaCostPct` | float | Percent of **max** stamina spent on each good press. |
| `MinDepth` | float | Minimum water depth in centimetres. `0` means no minimum. |
| `ReactivityWindowInSecond` | float | How long the player has to press F. Default `0.75`. |
| `LineFishMesh` | soft static mesh | Mesh shown on the line. Several fish may share one mesh. |
| `LineFishMeshScale` | float | Scale of that mesh. Default `1`. |
| `RequiredBaitSID` | string | Bait item SID. Empty means any bait is fine. |

The default line mesh is `/Game/_Stalker_2/VFX/Environment/Water/SM_VFX_Fish3_VAT`.

There is no "required artifact" field. An equipped Soul only helps because it raises vanilla stamina regen.

## Stock fish

FishId values below are the row names in `DT_FW_FishSpecies`. The stock pier uses the weight column.

| FishId | Loot item | Bait | Min depth (cm) | Wait (s) | Presses | Gap (s) | Stamina % | Pier weight |
|---|---|---|---|---|---|---|---|---|
| `FWBleak` | `FWBleak` | `FW_Bait_Worm` | 20–80 | 1–3 | 2–4 | 2.6–3.2 | 12 | 40 |
| `FWPerch` | `FWPerch` | `FW_Bait_Worm` | 40+ | 2–5 | 4–6 | 2.6–3.0 | 16 | 35 |
| `FWPerchHeavy` | `FWPerchHeavy` | `FW_Bait_Meat` | 60+ | 3–6 | 5–8 | 2.6–3.0 | 20 | 20 |
| `FWPike` | `FWPike` | `FW_Bait_Meat` | 80+ | 3–7 | 7–9 | 2.8–3.1 | 23 | 20 |
| `FWCatfish` | `FWCatfish` | `FW_Bait_Meat` | 120+ | 4–8 | 7–9 | 2.8–3.1 | 29 | 15 |
| `FWCatfishPatriarch` | `FWCatfishPatriarch` | `FW_Bait_Deep` | 150+ | 6–10 | 7–9 | 2.4–2.8 | 40 | 10 |

`Min depth 40+` means `MinDepth = 40` and no maximum. Bleak also has a maximum of 80 cm.

These fight numbers assume max stamina 100 and idle regen 5. A heavier load slows regen. Equipped Souls raise it. The patriarch is hard because the gap is short and each press costs 40%.

## Important SIDs

Copy these ids as plain text. `ModID` for every FishableWaters Craftable Zone row is `FishableWaters`.

### Baits

Using the item plays a short camera shake. The shake calls `SetActiveBait`. It does not start fishing.

| Item SID | Effect SID | Camera shake SID | Shake Blueprint | EN name | FR name |
|---|---|---|---|---|---|
| `FW_Bait_Worm` | `FW_Effect_BaitWorm` | `FW_CamShake_BaitWorm` | `BP_FW_Shake_BaitWorm` | Bloodworm | Ver de vase |
| `FW_Bait_Meat` | `FW_Effect_BaitMeat` | `FW_CamShake_BaitMeat` | `BP_FW_Shake_BaitMeat` | Mutant flesh | Chair mutante |
| `FW_Bait_Deep` | `FW_Effect_BaitDeep` | `FW_CamShake_BaitDeep` | `BP_FW_Shake_BaitDeep` | Deep bait | Appât des fonds |

All three are consumables (`EItemType::Consumable`), `Usable = true`, `ConsumeOnUse = true`. They inherit `TemplateConsumable`. Using one plays its shake, and that shake calls `SetActiveBait`.

### Carcasses (the catch)

These are not food. Cut them with Craftable Zone. World mesh is the vanilla `DeadFish` mesh.

| Item SID | EN name | FR name | Weight | Grid |
|---|---|---|---|---|
| `FWBleak` | Bleak | Ablette | 0.4 | 1×1 |
| `FWPerch` | Perch | Perche | 2.5 | 2×1 |
| `FWPerchHeavy` | Heavy perch | Grosse perche | 4.5 | 2×1 |
| `FWPike` | Pike | Brochet | 7.0 | 2×1 |
| `FWCatfish` | Catfish | Silure | 14.0 | 2×2 |
| `FWCatfishPatriarch` | Patriarch | Patriarche | 28.0 | 3×2 |
| `FWCatfishTrophy` | Catfish trophy | Trophée de silure | 2.0 | 2×2 |

`FWCatfishTrophy` is the patriarch's head. It is not cooked. It drops with the patriarch steaks.

Item name keys in the localization table are `sid_items_<LocalizationSID>_name` and `sid_items_<LocalizationSID>_description`. For these carcasses, `LocalizationSID` equals the item SID. Example: `sid_items_FWBleak_name`.

### Raw meat and cooked food

| Item SID | EN name | Made from | Notes |
|---|---|---|---|
| `FWBleakFilletRaw` | Raw bleak fillet | 1× `FWBleak` | |
| `FWPerchFilletRaw` | Raw perch fillet | `FWPerch` or `FWPerchHeavy` | Name key is `sid_items_fish_fliet_raw_name` (the LocalizationSID is `fish_fliet_raw`, not `FWPerchFilletRaw`). |
| `FWPikeFilletRaw` | Raw pike fillet | 3× from one `FWPike` | |
| `FWCatfishFilletRaw` | Raw catfish fillet | 4× from one `FWCatfish` | |
| `FWCatfishSteakRaw` | Raw catfish steak | 2× from one patriarch | |
| `FWBleakGrilled` | Grilled bleak | 1× `FWBleakFilletRaw` | |
| `FWPerchFilletCooked` | Perch skewer | 2× `FWPerchFilletRaw` | |
| `FWPikeSteak` | Pike steak | 2× `FWPikeFilletRaw` | |
| `FWCatfishStew` | Catfish stew | 3× `FWCatfishFilletRaw` | Also removes some radiation. |
| `FWPatriarchMeal` | Patriarch meal | 2× `FWCatfishSteakRaw` | Also applies `FW_Effect_MealRegenStamina` (+5% stamina regen, duration 120 seconds). |

Raw fillets use the Lootable Zone meat effects `KpBlinddogMeatSatiety` and `KpBlindDogMeatRadiation`.

### Cut recipes (Craftable Zone disassemble)

Tool: `KpLM_IZHMASH-74-4812`, amount 1, **not** consumed. `ModID` = `FishableWaters`.

`DisassemblingID` is the hover name in every language, joined with `|`. It is not a quest id.

| Item cut | Amount | Results | Hover text (`DisassemblingID`) |
|---|---|---|---|
| `FWBleak` | 1 | 1× `FWBleakFilletRaw` | `Ablette\|Bleak` |
| `FWPerch` | 1 | 2× `FWPerchFilletRaw` | `Perche\|Perch` |
| `FWPerchHeavy` | 1 | 3× `FWPerchFilletRaw` | `Grosse perche\|Heavy perch` |
| `FWPike` | 1 | 3× `FWPikeFilletRaw` | `Brochet\|Pike` |
| `FWCatfish` | 1 | 4× `FWCatfishFilletRaw` | `Silure\|Catfish` |
| `FWCatfishPatriarch` | 1 | 2× `FWCatfishSteakRaw` and 1× `FWCatfishTrophy` | `Patriarche\|Patriarch` |

The quest that **removes** the carcass is:

`FishableWaters_<ItemSID>_Disassembling_<ItemSID>_Remove`

Example: `FishableWaters_FWBleak_Disassembling_FWBleak_Remove`

The quest removes 1 item. The amounts you receive are in `DT_Disassemble`, not in the quest.

### Bait craft recipes

Category id: `FW_Bait`. Category name key: `sid_fw_craft_bait` (EN "Baits", FR "Appâts").

Tool: `KpSwissKnife`, amount 1, **not** consumed.

| Result | Amount | Ingredients | Remove quest |
|---|---|---|---|
| `FW_Bait_Worm` | 3 | 1× `KpTushkanMeat` + 1× `KpSalt` | `FishableWaters_FW_Bait_Worm_Craft_FW_Bait_Worm` |
| `FW_Bait_Meat` | 2 | 1× `KpFleshMeat` | `FishableWaters_FW_Bait_Meat_Craft_FW_Bait_Meat` |
| `FW_Bait_Deep` | 1 | 1× `KpPseudodogMeat` + 1× `KpSalt` | `FishableWaters_FW_Bait_Deep_Craft_FW_Bait_Deep` |

One result SID can have only one ingredient list. A second meat (boar, chimera) needs a different result SID.

### Cook recipes

Each cook row removes the meat **and** 1 fuel. Fuel SIDs: `KpDryFuel`, `KpCharcoal`, `KpKerosene`, `KpGasBaloon`. Pans and pots are not removed.

| Result | Amount | Meat removed | Meat quest |
|---|---|---|---|
| `FWBleakGrilled` | 1 | 1× `FWBleakFilletRaw` | `FishableWaters_FWBleakGrilled_Cook_FWBleakFilletRaw` |
| `FWPerchFilletCooked` | 1 | 2× `FWPerchFilletRaw` | `FishableWaters_FWPerchFilletCooked_Cook_FWPerchFilletRaw` |
| `FWPikeSteak` | 1 | 2× `FWPikeFilletRaw` | `FishableWaters_FWPikeSteak_Cook_FWPikeFilletRaw` |
| `FWCatfishStew` | 1 | 3× `FWCatfishFilletRaw` | `FishableWaters_FWCatfishStew_Cook_FWCatfishFilletRaw` |
| `FWPatriarchMeal` | 1 | 2× `FWCatfishSteakRaw` | `FishableWaters_FWPatriarchMeal_Cook_FWCatfishSteakRaw` |

Fuel quest name:

`FishableWaters_<ResultSID>_Cook_<FuelSID>`

Example: `FishableWaters_FWPikeSteak_Cook_KpCharcoal`

### Other SIDs you can ignore

`FWTestSpawn`, `FWTestStaminaPos`, `FWTestStaminaNeg`, `FWTestStaminaPosCustom`, `FW_TestStaminaNeg25`, and `FW_TestStaminaPos25` are test items. Do not use them in a public mod.

There is no fishing-rod weapon. Do not look for `FW_FishingRod_Wep`, `FW_Hook`, or the quest `FW_FishingSession`. Those were removed.

## Add content from another mod

Your mod is its own Game Feature. It does not edit FishableWaters files. Players install both mods.

In your `.uplugin`, add the dependency (this is the same step as Craftable Zone → Edit Plugin → Dependencies):

```json
"Plugins": [
  {
    "Name": "FishableWaters",
    "Enabled": true
  }
]
```

If you also add craft or cook recipes, depend on `CraftableZone` too, and use **your** ModID on your own rows. Do not write rows into FishableWaters data tables. A pak cannot merge rows into someone else's data table.

Your provider must be an **Actor** that exists in the world. The session uses **Get All Actors with Interface**. Put `BPI_FishableWaters_Provider` on the actor your Mod World Subsystem spawns in BeginPlay. Spawn it immediately. The session asks only once, 2 seconds after its own BeginPlay.

### A new fishing spot

Use this when an existing fish should also be catchable in a new place. No new item is required.

1. Depend on `FishableWaters`.
2. Place `BP_FW_Pier` (`/FishableWaters/Actors/BP_FW_Pier`) in your level.
3. On **that instance**, edit `Zone` → `spawns`.
4. For each fish, set `FishId` to a stock FishId (`FWBleak`, `FWPerch`, …) or to a FishId your provider registers. Set `Weight` above 0.
5. Package your mod.

The pier mesh and the overlap are already on the Blueprint. You only fill the list.

Example: a pond that only has bleak and perch.

| FishId | Weight |
|---|---|
| `FWBleak` | 60 |
| `FWPerch` | 40 |

The player still needs the matching bait, and the water in front of them must be deep enough for that fish.

### A new fish

A new fish is three things: an item, a rule, and at least one pier entry.

1. Create the carcass item in **your** `ItemPrototypes` cfg. Pick a unique SID, for example `MY_Carp`. Do not reuse `FWPerch`.
2. On your provider actor, implement `RegisterFishSpecies` (the Provider function).
3. When it runs, call the API:

   - `RegisterFishSpecies`
   - FishId: `MY_Carp` (this is the rule id)
   - Fish → `LootItemSID`: `MY_Carp`
   - Fill wait, presses, gap, `StaminaCostPct`, `MinDepth`, `RequiredBaitSID`, mesh, and scale.

4. Place a `BP_FW_Pier`, or add a spawn on a pier **your** mod owns:

   - `FishId` = `MY_Carp`
   - `Weight` = for example `25`

You cannot add a row to the stock pier from another pak. The stock pier belongs to FishableWaters. Place your own pier, even if it stands next to the same water.

`RequiredBaitSID` must be a real item SID. Stock values are `FW_Bait_Worm`, `FW_Bait_Meat`, and `FW_Bait_Deep`. An empty `RequiredBaitSID` means the fish accepts any active bait.

To let the player cut and cook the new fish, do that in **Craftable Zone**, not in FishableWaters:

- Your own disassemble row (`ModID` = your mod, `ItemNeededSID` = your carcass).
- A remove quest named `<YourModID>_<ItemSID>_Disassembling_<ItemSID>_Remove`.
- Your actor also implements `BPI_CZ_Provider` and registers that row the same way Craftable Zone's example mod does.

FishableWaters will not see those recipes. Craftable Zone will.

### A new bait

A bait is an item the player uses. The use sets the active bait. It must not start a fishing session.

1. Create a consumable in your `ItemPrototypes`. Unique SID, for example `MY_Bait_Berry`. Copy the shape of `FW_Bait_Worm`: `TemplateConsumable`, `Usable = true`, `ConsumeOnUse = true`, one entry in `EffectPrototypeSIDs`.
2. Create an effect that inherits `ConcussionCameraShake`. Set `CameraShakePrototypeSID` to your shake id. Keep `ValueMin` and `ValueMax` at `1`. A value of `0` can be skipped.
3. Create a camera-shake prototype. `CameraShakePath` points at your Blueprint.
4. Duplicate `BP_FW_Shake_BaitWorm`. On **Receive Play Shake**, keep **Get All Actors with Tag**. The tag is `BP_FW_FishableWaters`. Cast the first actor to `BP_FW_FishableWaters`, and change the string passed to `SetActiveBait` to `MY_Bait_Berry`. Do not call `StartFishingSession`.

Then a fish rule can set `RequiredBaitSID` to `MY_Bait_Berry`. Using the item is enough.

`BP_FW_Shake_BaitWorm`, `BP_FW_Shake_BaitMeat`, and `BP_FW_Shake_BaitDeep` already search for the tag `BP_FW_FishableWaters`. Copy one of them and change only the bait SID.

Give the item a name or the inventory shows the raw SID. Keys:

- `sid_items_MY_Bait_Berry_name`
- `sid_items_MY_Bait_Berry_short_name`
- `sid_items_MY_Bait_Berry_description`

`LocalizationSID` on the item is `MY_Bait_Berry` (do not put the `sid_items_` prefix there). Put the keys in your own localization data table.

To let the player craft the bait, add a Craftable Zone recipe in **your** mod and a remove quest:

`<YourModID>_MY_Bait_Berry_Craft_MY_Bait_Berry`

## Rules that keep mods compatible

- Depend on `FishableWaters`. Do not copy its Blueprints into your pak.
- FishableWaters does not depend on your mod. Do not make it import your assets.
- One session actor. Your mod only implements the Provider and places piers.
- Register fish inside the provider call. Do not edit `DT_FW_FishSpecies`. A bait does not go through the provider.
- A new place is a new `BP_FW_Pier` instance with its own `Zone.spawns`.
- A bait use only calls `SetActiveBait`.
- Do not override `IMC_Exploration`, `IMC_PlayerCA`, or the player animation Blueprint. The fishing key is the separate context `IMC_FW`, added only on the pier.
- Do not add the removed rod weapon, the hook, or the quest `FW_FishingSession`.

## Files this mod loads

Configs that the game loads live in:

`Content/GameLite/ModGameData/FishableWaters/`

| Folder | What it contains |
|---|---|
| `ItemPrototypes/` | Baits, carcasses, meat, meals |
| `EffectPrototypes/` | Bait shakes, patriarch regen |
| `CameraShakePrototypes/` | Links an effect to a shake Blueprint |
| `QuestPrototypes/` and `QuestNodePrototypes/` | Craftable Zone remove quests |

`Resources/Localization/FW_Feedback_Loc_stub.cfg` is a copy list for humans. The game does not load it. Real texts are in `Content/DataTables/LOC_Mod_FishableWaters`.
