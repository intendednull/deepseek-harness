# Fork rules: intendednull/deepseek-harness

This is a downstream fork of deepseek-ai/deepseek-harness. Read
[`FORK.md`](../../FORK.md) for the queue, branches, guards and procedures, and
[`upstream-practice.md`](upstream-practice.md) for dsh's development practice.

Rules for agents:

- **Upstream contributions are off by default.** Never draft or file code, PRs, issues, comments,
  reviews, or test reports for deepseek-ai/deepseek-harness. dsh does not accept external pull
  requests; the default is to keep carrying the patch.
- **Never push to deepseek-ai, and never point `gh` at it** beyond a read you really need. Every
  `gh` call names its repo: `-R intendednull/deepseek-harness`. The `upstream` push URL `no_push`
  is deliberate; do not "fix" it.
- **Pushing to `origin`, and only `origin`, is pre-authorised:** `patch/*` branches and `nixos`
  (a fast-forward). Nothing is ever pushed to an upstream.
- **Never commit to `main`.** It mirrors upstream `master`.
- **Do patch work on `patch/<slug>` off `nixos`.** One concern per patch, in dsh's style, with the
  `Upstream-Status:` and optional `Drop-When:` trailers and a
  `Co-Authored-By: DeepSeek-V4.1-Flash-1017 <noreply@deepseek.com>` line.
- **Run `.fork/gate` before landing on `nixos` and before each patch export.**
- **This fork uses patch-file mode.** Deploying means exporting the patch and committing it in a
  `~/nixos` worktree on a branch; see `FORK.md`. It is not a source swap or pin.
