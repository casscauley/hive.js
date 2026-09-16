# hive.js backlog

One file per task. Each is written to be picked up cold — what the code looks
like today, what it should look like after, and how to tell it worked.

The `docs/backlog/*.md` files here are the curated end of the pipeline. The other
end is `docs/backlog/notes/` — see [Incoming notes](#incoming-notes) below.

| Task | Size | Depends on |
| --- | --- | --- |
| [Finish the unrest-server migration](unrest-migration.md) | medium (server rename done; SPA catch-all + pruning left) | — |
| [Login = email code + play as guest](email-login.md) | medium | unrest-migration |
| [Save and view replays](replays.md) | medium | — |
| [Animate piece movement](animate-piece-movement.md) | medium | — |
| [Autoplaying game behind the login screen](login-attract-mode.md) | small | replays (for the game list), email-login (for the screen) |
| [AI improvement plan](ai-improvements.md) | large (open-ended) | — |
| [New game settings don't survive a reload](newgame-settings-persistence.md) | small | — |

Done: [restore-csrf](restore-csrf.md) (2026-07-30) — kept for its notes on the
token flow and for the unfixed `join_room`-mutates-on-GET issue.

Suggested order: **unrest-migration → email-login**. **replays** and
**animate-piece-movement** are independent and can go at any time.
**login-attract-mode** is last — it wants a replay list to draw from, the new
login screen to sit behind, and the movement animation to not look broken.
**newgame-settings-persistence** is an isolated bug fix, good for a short
sitting. **ai-improvements** is a research project, not a task; treat its
sections as a menu.

## Deploy reminders for anything here

hive runs **daphne**, not uwsgi, so there is no `touch-reload`. Backend changes
need `systemctl restart hive`, **which drops every open websocket** — restart
when nobody is mid-game. Frontend changes need `cd client && npm run build`
(Vite, output to `dist/`, served by nginx directly). Never `npm run dev` on this
box.

## Incoming notes

`docs/backlog/notes/` is the inbox, written by `unrest_api.backlog` from the
admin `notes` view: `YYYY-MM-DD-slug.md` plus images in `notes/img/`. Nobody
hand-authors those — the library owns the layout and makes the directories on
first write. It arrives with [unrest-migration](unrest-migration.md); until then
the folder won't exist.

A note is raw input, not a task. Either it gets fixed immediately and deleted in
the same commit as the fix, or it gets promoted into a `docs/backlog/<slug>.md` task
file here, added to the table above, and deleted. Notes shouldn't accumulate.

## History of this directory

`docs/` was the GitHub Pages build output — a committed `vue-cli-service build
--dest ../docs`, last built 2021-07-07, deleted by `7ff5045`. Pages may still be
configured to serve from it in repo settings; the 2021 build is reachable in
history either way. **Check Settings → Pages → Source is None.**

The backlog lived at the repo root until 2026-09-14, when the global convention
moved to `docs/backlog/` across all projects. `docs/todos.md` and
`docs/todos/ai.md` were folded in at the same time: `ai.md` became
[ai-improvements.md](ai-improvements.md) verbatim, the "arrows over the pieces"
item was closed as done by `e533857` (the remaining `chalk` machinery in
`game/Board/specials.js` marks pillbug/mosquito targets — a different feature),
the replay item was a duplicate of [replays.md](replays.md), and the settings
bug became
[newgame-settings-persistence.md](newgame-settings-persistence.md).

**`unrest_api.backlog` still defaults to `BASE_DIR.parent / 'backlog'`**, so
`UNREST_BACKLOG_ROOT` in `server/settings/00-base.py` has to point here. That is
set, but the notes endpoint itself only comes alive with
[unrest-migration](unrest-migration.md).
