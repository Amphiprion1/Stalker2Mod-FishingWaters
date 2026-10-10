# AGENTS.md - FishableWaters (S.T.A.L.K.E.R. 2 mod)

Instructions for any coding agent (Grok Bot, Grok Build, Cursor, Claude, ...) working on this mod.

## Knowledge base - read first
The shared Stalker 2 modding wiki lives outside this repo:
`C:\Users\Forza-PC\Documents\Wikis\stalker2-modding\`

1. Read `index.md` there, then `topics\agent-workflow.md` and `topics\pitfalls.md`.
2. Before touching a subsystem, read its topic page (e.g. `topics\cfg-gamedata.md`, `topics\game-features.md`,
   `topics\blueprints.md`, `topics\save-load.md`, `topics\packaging-pak.md`, `topics\testing-debugging.md`).
3. Official guide text is in `_sources\official\text\`; cite it rather than guessing.
4. Never invent engine/API facts. If the wiki and official guides are silent, say so and propose a test.

## Project
- Mod folder: `C:\STALKER2ZoneKit\Stalker2\Mods\FishableWaters`
- Zone Kit install: `C:\STALKER2ZoneKit` (local only, no cloud Unreal Engine).
- No C++ mod compile path in Zone Kit: use cfg/GameData patches + Game Feature + minimal Blueprints.

## Working rules
- Human (Gerald) does Zone Kit GUI steps (create/select mod, Package Mod, paste BP text, compile/save, play-test).
  Give exact UI names and one action per step.
- Agent does research, cfg patches, BP paste text, docs, log/crash reading.
- One change per cook/test; roll back at the first regression.
- Override as little as possible; patch cfg, never copy whole vanilla files.
- Keep `PLAN.md` (plan + test matrix) and `BUILD.md` (reproducible edits) in this folder up to date.
- Mod-specific facts go in this folder; general Stalker 2 modding facts go in the wiki with a source.
