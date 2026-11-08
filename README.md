# splitseat-skill

A Claude Code skill for split-screen co-op on one PC with [SplitSeat](https://github.com/P1tak4s/splitseat), an
audited fork of Couchlink that contacts no outside site unless you turn it on.

## Install

```sh
git clone https://github.com/P1tak4s/splitseat-skill
cp -r splitseat-skill/splitseat ~/.claude/skills/
```

Then ask Claude Code to set up couch co-op for a game; the skill builds SplitSeat from source and writes a local
recipe for it.

License: MIT.
