# LightsOut — working agreement

A single-file ESO addon (`LightsOut.lua`, no build step, no test suite). ESO loads
the `.lua` file directly, so a syntax error or a wrong API call is only visible in
game. Every change is user-facing risk.

## The known-good baseline

`v2.2.0-baseline` is an annotated tag on the last version confirmed working in
game. Never move it, delete it or force-recreate it.

    git rev-parse v2.2.0-baseline                  # the exact working commit
    git show v2.2.0-baseline:LightsOut.lua | shasum -a 256

If `main` ever drifts into a broken state, the tag is what you restore from.

## Everything goes through a pull request

`main` is treated as protected: never commit to it, never merge locally into it,
never force-push it, never rewrite published history.

1. Start from an up-to-date `main`:

       git switch main
       git pull --ff-only

2. Branch for the change, named by intent:

       git switch -c fix/populate-cancel-timeout
       git switch -c feat/target-hud

3. Make the change. Keep one behaviour per branch, and keep commits small.

4. Sanity-check the Lua before pushing — ESO reports a load error as one red
   line in chat, and a broken addon may not load at all:

       luac -p LightsOut.lua

   No Lua toolchain (`luac`, `lua`, `luajit`) was installed on the development
   machine when this was written, so that command will not run as-is. Until one
   is installed, confirm the file parses in an editor with a Lua parser and load
   it once in game, and say in the PR that it was checked that way.

5. Push the branch (never `main`) and open a PR against `main`:

       git push -u origin fix/populate-cancel-timeout

6. The PR description states what was verified in game and what was not.
   A change nobody has loaded in game is not a finished change — say so plainly
   in the PR rather than implying it was tested.

## Rolling back

**One bad change (preferred).** Revert on its own branch, so the rollback is
reviewable like any other change:

    git switch main && git pull --ff-only
    git switch -c revert/<slug>
    git revert <sha>              # merges: git revert -m 1 <merge-sha>
    git push -u origin revert/<slug>

**The whole file back to the working version.**

    git switch -c fix/restore-baseline
    git checkout v2.2.0-baseline -- LightsOut.lua
    git commit -m "Restore LightsOut.lua to v2.2.0-baseline"

**Emergency, in game.** ESO loads the addon from its own `AddOns/LightsOut`
folder, which is outside this repository. To un-break a live session right now,
copy the baseline `LightsOut.lua` (plus `LightsOut.txt`) into that folder and run
`/reloadui`; then land the real fix through a PR. Do not "fix forward" in the
AddOns folder — that copy is not version controlled.

## Versioning

The manifest `## Version:` line in `LightsOut.txt` and `LightsOut.version` in the
Lua must match. Bump both in the same PR that ships the change, and never bump
the version in a PR that has not been loaded in game.
