# New game settings don't survive a reload

**Size: small.** One component, one storage key.

## Why

`NewGame.vue` looks like it persists your last game setup, but it only ever
reads. This is a live bug, not a feature request — the code to restore is there
and the code to save never was.

## What the code looks like today

`client/src/components/NewGame.vue`:

- `const LS_KEY = 'NEW_GAME_RULES'` (line 104) is read exactly once, at line 222:
  `rules = cleanRules(ls.get(LS_KEY) || { variants: {}, pieces: { ...pieces.VANILLA } })`
- Nothing anywhere calls `ls.set(LS_KEY, ...)`. `git grep 'LS_KEY'` returns those
  two lines and nothing else, so the fallback branch is the only one that ever
  runs.
- The AI settings are separate component state with no storage story at all:
  `ai_enabled: false`, `ai_difficulty: 'medium'`, `ai_color: 2` (lines 195-197),
  driven by `UnrestSchemaForm` through `onAIChange`.
- `selected_preset_slug: 'classic_hive'` (line 194) likewise resets every time.

Note for anyone who read the old `docs/todos.md`: it called the constant
`LS_AI_KEY`. There is no such identifier — it is `LS_KEY`, and it covers rules,
not AI.

## What it should look like after

- Rules, the selected preset, and the three AI fields all round-trip through
  `local-storage-json` (already imported as `ls`).
- One key or two is a judgement call: `NEW_GAME_RULES` already exists and is
  read, so the cheapest fix is to write it on change and widen what it holds.
  Keep the `cleanRules()` call on the way in — stored rules from an older piece
  set must not be trusted.
- Saving on change beats saving on submit; the menu is where people fiddle and
  then reload.

## How to tell it worked

1. Open the new game menu, pick a non-default preset, enable AI, change the
   difficulty and colour.
2. Reload the page.
3. Every one of those four is still set. `localStorage.NEW_GAME_RULES` in the
   console shows them.
4. Clear the key, reload, and confirm you land on `classic_hive` with AI off —
   the fallback branch still has to work.
