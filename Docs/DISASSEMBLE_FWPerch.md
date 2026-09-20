# FWPerch disassemble (Craftable Zone — Notion « 6. Disassembly System »)

Source: https://app.notion.com/p/6-Disassembly-System-3ce9b3bade3c811ea99dceb9fa0a1f8c

## Why battery has right-click Disassemble and FWPerch does not

Craftable Zone adds **Disassemble** on right-click. Lookup path:

1. Hover item → **Faster Item Info Panel** saves the **raw localized display name**
2. Right-click Disassemble → CZ matches that name against recipe field **`DisassemblingID`**
3. `DisassemblingID` is **not** a quest SID — it is all language names of the item, separated by `|`

We previously put `FishableWaters_FWPerch_Disassembling_FWPerch_Remove` in `DisassemblingID` → **never matches** → no working Disassemble for Perch.

Battery works because Lootable/Craftable Zone already registered its multi-language names there.

**Required dependency:** Faster Item Info Panel (you already have it in `~mods`).

## Fix the DT_Disassemble row (ModEditor)

| Field | Set to |
|---|---|
| Row Name / DisassembleRecipeName | `FWPerch` (same) |
| ModID | `FishableWaters` (same everywhere in this mod) |
| **DisassemblingID** | Exact hover names, e.g. `Perch\|Perche` (EN\|FR\|…). Must match Faster Item Info Panel text for that item |
| ItemNeededSID | `FWPerch` |
| ItemAmount | `1` |
| UsesGenerator | `false` |
| Gives1Item | `true` |
| Result1SID | `FWPerchFilletRaw` |
| Result1Amount | `10` |
| ToolNeededSID | **Not** `Knife`. Use a Lootable Zone tool, e.g. `KpMultiToolUsage` (Notion example). Tools often need **Use** once in inventory to become % stacks |
| ToolAmount | e.g. `1` or `2` |
| RemovesTool | `true` (or `false` + never-removed dummy tool trick from Notion) |

Quick test: set `DisassemblingID` temporarily to whatever the Info Panel shows when hovering FWPerch (often the SID `FWPerch` if no loc yet).

## Localization

Cfg now has `LocalizationSID = sid_items_FWPerch_name`. Add EN/FR strings in `LOC_Mod_FishableWaters`, then put **those exact strings** into `DisassemblingID` joined by `|`.

## Quests

Notion’s current Disassembly System example does **not** use quest SIDs for matching. The Remove quest we added is optional/legacy vs Example1; the critical fix is `DisassemblingID` + tool SID + Faster Item Info Panel.

## LocalizationSID (popup name/description)

Game builds keys as `sid_items_{LocalizationSID}_name` and `sid_items_{LocalizationSID}_description`.

- Set `LocalizationSID = FWPerch` (NOT `sid_items_FWPerch_name` — that doubles the prefix).
- In LOC_Mod keep SIDs: `sid_items_FWPerch_name`, `sid_items_FWPerch_description`.
- DisassemblingID stays the **resolved** display text: `Perch|Perche`.

## Item remove quest

Craftable Zone starts:
`FishableWaters_FWPerch_Disassembling_FWPerch_Remove`

Requires DT field `ModID = FishableWaters` (must match quest prefix). ItemRemove node: 1× `FWPerch`.

## Item remove quest

Craftable Zone starts:
`FishableWaters_FWPerch_Disassembling_FWPerch_Remove`

Requires DT field `ModID = FishableWaters` (must match quest prefix). ItemRemove node: 1× `FWPerch`.

