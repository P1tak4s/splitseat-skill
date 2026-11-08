---
name: splitseat
description: Play a co-op game, a co-op mod or any online game split-screen on one PC with SplitSeat, our audited Couchlink build (one copy per player, each with its own screen part, controller or keyboard and mouse, optionally its own Steam account), and make a co-op mod work with it through a local recipe. Uses the cleaned fork github.com/P1tak4s/splitseat (branch svarus), never the upstream release zip.
---

# Couch co-op on one PC with SplitSeat

Couchlink (MIT, upstream `cranebeaker/couchlink`) runs one copy of a game per player on one PC, puts each copy on its
own monitor or part of one, and gives each copy only its own controller or keyboard and mouse. The copies meet through
the game's own multiplayer (LAN, a co-op mod's server, or Steam). Up to 8 players.

SplitSeat is our name for the fork. We do not run upstream binaries. We build https://github.com/P1tak4s/splitseat, branch `svarus`
(local clone: `~/dev/couchlink-audit/couchlink`). The security review is `AUDIT.md` in that repo; read it before
changing anything security-relevant.

## What the svarus branch changes

Upstream contacts outside sites on its own. Our build does not, unless an environment variable turns a feature on:

| Feature | Upstream | svarus | Turn on with |
| --- | --- | --- | --- |
| Game catalog + votes | `coopanything.app/data/recomps.js`, `/api/stats` | off | `COUCHLINK_CATALOG_URL` |
| Live recipes | signed by the upstream author's key, auto-refresh | off, no built-in key | `COUCHLINK_RECIPES_URL` + `COUCHLINK_RECIPES_PUBKEY` (our key only) |
| Self-update | GitHub releases of upstream, unsigned | off | `COUCHLINK_RELEASES_URL` |

Never set the recipe key to the upstream key `ffe01f75...`: a recipe can download files into the game folder and start
programs, so whoever holds that key can run code on this PC.

## Build (Windows)

Needs Visual Studio 2022 Build Tools (C++ workload), CMake, Rust (MSVC target) and Flutter.

```powershell
git clone https://github.com/P1tak4s/splitseat; cd splitseat
powershell -File hook/build.ps1            # couchlink_hook.dll x64 + x86
cd core; cargo build --release -p couchlinkd
cd ../app; flutter build windows --release
cd ..; python -I tools/release/package.py --version 0.0.0-svarus
```

Before trusting a new upstream version: `git remote add upstream https://github.com/cranebeaker/couchlink` (once), `git fetch upstream`, read the diff of `core/crates/couchlinkd/src/{catalog,update,hub}.rs`,
`session/remote.rs`, `vscreen_setup.rs`, `couchlink-steam/src/{accounts,goldberg}.rs` and every new URL
(`git diff svarus upstream/main | grep -E "https?://"`), then rebase `svarus`.

## Playing

1. Start the app from our build. **Library**: installed games with a recipe are under "On this PC"; other installed
   Steam games are under "Try it". Without the catalog, games you do not own are not listed.
2. **Players**: press A on a controller or Enter on a keyboard to take a seat. A keyboard and a mouse together are one player.
3. **Steam**: one account is enough for LAN or mod-server games. For games that only connect through Steam lobbies,
   a seat can get its own Steam account: this creates a standard Windows user with a random password kept in
   Credential Manager. Remove it from the seat card when done (deletes the user and the password).
4. **Screens**: drag players onto monitors or split one screen. "Virtual screens" installs a display driver and adds its
   signing certificate to the machine's Trusted Publishers. Only enable it if a game needs it; ask the person first.
5. Press Play. When the session ends Couchlink restores the game folder from its journal.

Expect antivirus warnings: the hook injects `couchlink_hook.dll` into each game copy (CreateRemoteThread + LoadLibrary).
That is how per-copy input works, not malware, but only allow it for our own build.

## Adding a game or a co-op mod (local recipes only)

Recipes are JSON files in `%LOCALAPPDATA%\Couchlink\recipes\` (ours win over built-ins by id). Examples in
`core/recipes/`, full format in `docs/launch.md`, SDK guide in `skills/couchlink-sdk/SKILL.md` of the fork.

- A recipe says how to start the game, where saves live, which single-instance locks to lift, the Steam mode, and a
  server to start first. Templates `$SEAT`, `$PLAYER`, `$WIDTH`, `$HEIGHT`, `$X`, `$Y` give each copy its own arguments.
  Paths must stay inside the game folder, registry edits are HKCU only, and every `fetch` needs a `sha256`.
- Check it: `cd core && cargo run -q -p couchlink-core --example check_recipe -- my-game.json`
- Do not publish recipes to the upstream Discord or site; keep them in the fork under `core/recipes/`.
- A Couchlink-aware co-op mod reads the seat and player name from the command line or environment; seat 0 hosts and
  every other seat joins `127.0.0.1` on the mod's port. No input tricks needed then.

## Testing without people

Follow the `coop-qa` skill. For Couchlink: the recipe passes `check_recipe`, a session is started from a script against
the local service (`start_session`, then wait for `running` in the `session` events; protocol in `docs/protocol.md`,
the port and token are in `%LOCALAPPDATA%\Couchlink\daemon.json`), and the mod's own bots confirm every copy joined.
