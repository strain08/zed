# FreeBSD downstream patch series

Upstream Zed (`zed-industries/zed`) merges FreeBSD fixes but does **not** build
or ship FreeBSD binaries. This branch carries the small delta needed to build a
working FreeBSD tree and applies it on top of upstream release tags.

## Model

`freebsd/patches/*.patch` is the **single source of truth**. The patches are
applied onto a *pristine upstream tag* to produce a buildable tree — they are
never merged into a long-lived branch. Keeping the delta as a re-appliable
series (rather than a merge) means:

- the delta is always exactly the patch files — auditable at a glance;
- conflicts surface per-patch (you see which fix broke) on each new tag;
- when a patch lands upstream, you delete it and the next release is lighter.

`freebsd/upstream-base` records the upstream commit the current series was cut
against.

## The loop, once per upstream release

```sh
# 1. Apply the series onto a new upstream tag (creates a sibling worktree
#    ../zed-freebsd-build/<tag> and a <tag>-freebsd tag):
script/freebsd-patches apply v1.5.3

# 2. Build the worktree on FreeBSD (CI or a jail/VM), publish artifacts.
```

If a patch conflicts on the new tag, `apply` stops in the worktree mid-`am`.
Resolve it there, `git am --continue`, then capture the fixed series back:

```sh
script/freebsd-patches export ../zed-freebsd-build/v1.5.3 --base v1.5.3
git add freebsd/ && git commit -m "freebsd: refresh series for v1.5.3"
```

## Editing or adding a patch

```sh
script/freebsd-patches apply <latest-tag>      # get a worktree with commits
# edit / rebase -i / add commits in that worktree, then:
script/freebsd-patches export <that-worktree> --base <latest-tag>
git add freebsd/ && git commit -m "freebsd: <what changed>"
```

## Commands

| Command | Purpose |
|---|---|
| `script/freebsd-patches list` | Show the series and subjects |
| `script/freebsd-patches check [<tag>]` | Verify the series applies onto an upstream tag (latest stable if omitted) and record it in the top-level README |
| `script/freebsd-patches apply <tag>` | Apply series onto an upstream tag in a worktree |
| `script/freebsd-patches export <ref> [--base REF]` | Regenerate the series from a branch/worktree |
| `script/freebsd-patches status` | Series size, base, active worktrees |

## Checking compatibility

When a new upstream release lands, run:

```sh
script/freebsd-patches check          # latest upstream stable
script/freebsd-patches check v1.9.0   # or a specific tag
```

This re-applies the series onto the tag in a throwaway worktree (building
nothing) purely to prove it still applies. On success it updates the
**Currently verified against upstream** line at the top of the repository
[`README.md`](../README.md) to that tag and today's date; commit that change.

If a patch stops applying, `check` offers to keep the worktree paused mid-`am`
so you can fix it in place: resolve the conflict, `git am --continue`, then
`export` the fixed series (see above). Declining discards the worktree. Either
way the README is left untouched until a clean run records a new version.

## Draining the queue (the long game)

Every patch accepted upstream is one you never re-apply again. Upstream is
receptive to FreeBSD PRs. Priorities:

- `0003-freebsd-fix-wasm-platform-match-temp` is a stopgap (`temp`) — fix it
  properly before it becomes a permanent rebase liability, then upstream it.
- The remaining `freebsd:` patches are small and `cfg`-gated — good PR
  candidates.
