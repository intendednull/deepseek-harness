# intendednull/deepseek-harness: downstream fork

This repository is a downstream fork of
[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness). It carries one
patch on the release nixpkgs builds, `dsh-v0.2.1-rc.3`, for the `ms7c91-nixos` Paseo DeepSeek
profile. The `~/nixos` flake applies this patch to nixpkgs' own `deepseek-harness` recipe
(`forks/deepseek-harness/` in that repo).

It exists because dsh's ACP bridge reports only context occupancy in `usage_update`. Paseo's ACP
adapter reads `PromptResponse.usage` for input, output, and cached token counts, so DSH sessions
currently show no token usage. The patch reports this turn's token-meter totals in that
standard field.

## Upstream contribution policy

dsh's [CONTRIBUTING.md](https://github.com/deepseek-ai/deepseek-harness/blob/master/CONTRIBUTING.md)
on `master` (read 2026-11-01 at `ce7073d51254c26cd63b22a60835b8f948deb447`) says external pull
requests are not accepted at the moment and directs bug reports to GitHub Discussions. It states
no AI-specific policy. Policy key: `unknown`.

Contributing is **off by default**. Nothing is offered upstream unless the operator asks for that
specific change. Agents never draft or file issues, PRs, comments, reviews, or test reports.

## Patch queue

**Base:** `dsh-v0.2.1-rc.3` (`1f81c047fc48fa190a683e469daa699857e9afe8`), the release nixpkgs
ships, so `pnpm-lock.yaml` and the nixpkgs recipe stay valid.

Bottom to top:

1. **`fork: Document the downstream fork`** (`local`): this file, `.claude/rules/fork.md`,
   `.claude/rules/upstream-practice.md` and `.fork/gate`.
2. **`feat(acp): report token usage on prompt responses`** (`unsent`): returns
   `PromptResponse.usage` from the existing `tokenUsage` projection and adds per-turn and
   no-token-meter bridge assertions.

Every commit carries trailers for the `fork` tool in `~/nixos`. They survive rebase and
`format-patch`:

```
Upstream-Status: <state>
Drop-When: <condition>       # optional, free text
```

`Co-Authored-By: DeepSeek-V4.1-Flash-1017 <noreply@deepseek.com>` marks a commit as
agent-written. Fork-authored commits follow dsh's commit style; carried upstream commits keep
their original author and message.

## Branches and tags

| ref | role |
|---|---|
| `nixos` (default branch) | **Integration branch**: an upstream base plus a linear patch queue. It is what `~/nixos` builds. |
| `main` | Mirror of upstream `master`. Never committed to. |
| `patch/<slug>` | Topic branch for one change, cut from `nixos` and fast-forwarded onto it. |

## Guards

- **The `upstream` push URL is `no_push`.** It has no colon or slash, so git treats it as a local
  path and fails without touching the network. Don't "fix" it.
- **`gh repo set-default intendednull/deepseek-harness`** is set in this clone. Every `gh` call
  also names its repo: `gh … -R intendednull/deepseek-harness`.
- **Clone config:** `rerere.enabled` and `rerere.autoupdate` are on, `remote.upstream.tagOpt` is
  `--tags`, and `upstream/HEAD` is set (`git remote set-head upstream --auto`).
- **Push permission:** sessions may push `patch/*` branches and `nixos` (a fast-forward) to
  `origin` without asking. Nothing is ever pushed to an upstream.
- **This fork uses patch-file mode.** There is no source swap, `fork.json` or `pin/*` tag for it.
  Deploying means exporting the patch and committing it in a `~/nixos` worktree; see below.

## Deploying (patch-file mode)

`~/nixos` carries this patch through `forks/deepseek-harness/patches/`, applied to nixpkgs' own
`deepseek-harness` recipe. It does not swap the source or record a pin.

1. Run `.fork/gate`.
2. Export the patch, excluding the local docs commit:
   `git format-patch --zero-commit --no-signature -o <~/nixos worktree>/forks/deepseek-harness/patches nixos~1..nixos`
   (adjust the range to the first patch commit when the queue grows).
3. Commit the patch and the `forks/deepseek-harness/default.nix` wiring in a `~/nixos` worktree on
   a branch. That commit is review-class: no merge to `main`, no `home-manager switch`.
