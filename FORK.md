# Fork notes (herdr-core)

This file lives only on the `fork` branch of lunelson/herdr-core. Never commit it to `master` or a PR branch, and leave it out when cherry-picking patches for upstream. Read it with `AGENTS.md`: it adds to those rules and never replaces them. Machine-specific notes (what is installed where, and when) go in the gitignored `AGENTS.local.md`.

## What the fork is

A machine running the fork has a core (`~/.local/share/herdr-core/current` → `active/Herdr Core.app`, linked as `~/.local/bin/hc` and `herdr-core`) built from this checkout's `fork` branch, not an official release. It's installed with `just install` (`scripts/install-local.sh`), so it's signed ad hoc (`Signature=adhoc`, no team ID), not with Herdr's Developer ID. It reports the same version as the upstream release it is based on.

It's paired with a fork build of the app from the sibling checkout `../herdr-app` (branch `fork`, installed at `/Applications/Herdr.app`, with no official app copy while on the fork). See `../herdr-app/FORK.md`, which has the shared routine for both. Some patches need both halves.

## Branches

- `fork`: `upstream/master` plus the patches below and this file, each its own commit. It tracks `origin/fork` on lunelson/herdr-core; it is rebased onto `upstream/master` by naming that explicitly. Only local builds run from it. Never open a PR from it.
- PR branches (e.g. `fix/display-palette`): cut from `upstream/master`, carrying one patch each. Upstream work happens on branches like these, never on `fork`.
- `master`: tracks upstream; keep it untouched.

Remotes: `origin` is lunelson/herdr-core, `upstream` is herdrdev/herdr-core. Every local branch is pushed to `origin` and tracks it, PR branches too, whether or not their PR is open (`git push -u origin <branch>`).

## Patches carried on `fork`

| Patch | Upstream | Status |
|---|---|---|
| `fix: answer palette queries with the viewer's theme colors (#827)` and `fix: keep the palette when a viewer's display facts change (#827)`: display facts accept `palette`, so programs asking for ANSI colors (OSC 4) get the theme's colors, not libghostty's defaults | issue [herdrdev/herdr-app#827](https://github.com/herdrdev/herdr-app/issues/827); PR branch `fix/display-palette` | diagnosis posted on #827 2026-11-13, waiting on Can; PR not opened yet |

Update this table when a patch is added, lands upstream, or is dropped.

## Building

Builds need Zig 0.16 (`crates/ghostty-sys` builds libghostty-vt from `vendor/ghostty`), installed globally through mise (`~/.config/mise/config.toml`: `zig@0.16.0`). Inside hc terminals the `just` mise shim may find no version, so call the script directly or use the full path to `just`.

## Updating `fork` when upstream moves

1. `git fetch upstream`
2. `git switch fork && git rebase upstream/master`
3. If a carried patch has landed upstream, it was squash-merged, so it won't match: the rebase stops with conflicts or empty commits. Drop the carried commit (`git rebase --skip`, or `git rebase -i` and delete it) and update the table above.
4. Run the narrowest tests the conflicts touched (`cargo test --locked -p herdr-core --test <file>`).
5. Reinstall: `sh scripts/install-local.sh`. This restarts the user's real core. Running terminals are kept, but ask the user first. Check: `hc --version` reports the new upstream version and `codesign -dv "$(readlink -f ~/.local/bin/hc)"` says `Signature=adhoc`.
6. Then update the app fork, which carries the other half of the patch (`../herdr-app/FORK.md`: core first, then app, and the checks for both).
7. `git push --force-with-lease origin fork`. After a rebase, `fork` and `origin/fork` have diverged on purpose: don't `git pull`, which would merge the old history back in.

Also rebase PR branches on `upstream/master` and push them (`git push --force-with-lease origin <branch>`).

## Going back to the official core

Once upstream releases the fix (a version newer than the fork build reports), `hc update` installs the official, Developer ID-signed bundle and restarts the core. While the versions are equal, `hc update` reports the local build as current and doesn't replace it. Do the app's return steps at the same time (`../herdr-app/FORK.md`). Then delete that machine's `AGENTS.local.md` or mark it retired; once no machine uses the fork, delete the `fork` branch.

## Caveats

- Because the signature is ad hoc, macOS treats this as a different program from the official core. Programs in terminals may have to ask again for privacy access (microphone, camera, automation), both on switching to the fork and after going back.
- An official Herdr app replaces this core: the first start after it installs one of its own updates runs `hc update`. That's why no official app copy is kept while on the fork (`../herdr-app/FORK.md`); don't install it until going back.
