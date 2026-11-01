# dsh upstream development practice

Read at: `ce7073d51254c26cd63b22a60835b8f948deb447` (2026-11-01)

Read from `upstream/master`, not from the `dsh-v0.2.1-rc.3` base. Sources:
- docs: `CONTRIBUTING.md`, `AGENTS.md`, `packages/AGENTS.md`, `docs/development.md`,
  `docs/testing.md`, `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/*.md`;
- tooling: `package.json` scripts, `pnpm-workspace.yaml`, `.oxlintrc.json`, `lefthook.yml`,
  `.github/workflows/ci.yml` and `ci-master.yml` `run:` steps;
- history: the last 200 non-merge commits on `master`.

Where they disagree, what the tools enforce wins, then the docs, then the history.

## Policy

dsh's `CONTRIBUTING.md` on `master` says external pull requests are not accepted at the moment
and directs bug reports to GitHub Discussions. It states no AI-specific policy. Policy key:
`unknown`.

For agents in this fork:
- Contributing upstream is **off by default** (operator standing rule). Agents draft and file
  nothing for deepseek-ai: no code, tests, PR or issue text, comments or reviews.
- Any upstream submission is **operator-requested only**. The rest of this file exists so fork
  work matches upstream's style and rebases cleanly.
- A fork commit with a `Co-Authored-By: DeepSeek-V4.1-Flash-1017 <noreply@deepseek.com>` line is agent-written.

## Code style

- **TypeScript, ESM, strict.** Packages use `@deepseek-ai/dsh-<name>`; local relative imports use
  `.ts`. Everything compiles under `strict: true`; no new `as unknown` casts.
- **Oxlint** (tooling). `pnpm run lint` is the checked command; the config is `.oxlintrc.json`.
- **Formatting is conventional.** Follow the surrounding file; the repository has no separate
  formatter step in the gate.
- **Comments are local and complete.** Do not restate code, expand unrelated comments, or narrate
  control flow. Public functions and exports carry concise JSDoc with `@param` / `@returns`.
- **Tests describe behavior, not correctness.** Change obsolete behavior with its tests.

## Commit messages

- **Shape:** Conventional Commits, `type(scope): summary`, e.g. `fix(ui): reuse mobile device
  detection for sidebar opener`, `docs(automation): …`, `test(web): …`. 192 of the last 200
  non-merge subjects have a lowercase prefix.
- **Summary:** imperative, no trailing period.
- **Body:** wrapped at 72; says why or lists behaviour changes.
- **Trailers are rare.** No DCO. Fork commits add `Upstream-Status:`, optional `Drop-When:` and
  `Co-Authored-By: DeepSeek-V4.1-Flash-1017 <noreply@deepseek.com>`.

## Branching and history

- `master` carries merge commits from pull requests; topic branches are rebased and squashed.
- Rebase onto upstream, never merge upstream into a topic branch.
- One concern per patch; keep review fixes in the commit they belong to.

## Tests

- Unit: `pnpm run test` (vitest).
- Coverage gate: `pnpm run test:coverage`, per-file 100% on `packages/*/*/src`.
- ACP: `pnpm vitest run packages/acp/acp/tests/bridge.spec.ts` exercises the real bridge harness.
- Focused evidence for a change is expected; the full suite is CI's job, not a default local run.
- A protocol-visible change needs a keyless recorded-session scenario when the assembled
  transcript changes; update the expected output rather than a normalizer.

## Docs

- Update the affected README and JSDoc contracts in the same commit.
- Bilingual docs follow `docs/AGENTS.md`; translation is a separate explicit step.

## Review

- Reviewers check edge cases, fit with the surrounding design, unrelated changes, and test
  evidence. Keep commits small and self-contained.

## Gate

`.fork/gate` mirrors the relevant CI checks on this host:
- `pnpm run lint`
- `pnpm run typecheck`
- the ACP package tests under coverage, limited to `packages/acp/acp/src/**`
- the ACP snapshot replay (`snapshots/acp/acp.snapshot.ts`)
- `pnpm run build:artifacts && pnpm run build:check`
- `pnpm run test:docs`

It assumes dependencies are installed (`pnpm install --frozen-lockfile`); it does not install
them itself. The snapshot replay needs the nixpkgs `expose-internals-profile-resolution.patch`
on this host's Node; the gate fails loudly when that patch is absent. Deployment is patch-file
mode: export the patch into a `~/nixos` worktree, not a source swap or pin.

## Where the fork deviates and why

- **The base is a release** (`dsh-v0.2.1-rc.3`, what nixpkgs ships), not `master`. The practice is
  still read from `master`.
- **The `local` docs commit** (`FORK.md`, `.claude/rules/fork.md`, this file, `.fork/gate`) is
  fork-only and agent-written.
- **Fork commits carry trailers and agent attribution** that upstream does not use. The tools
  read the trailers; the attribution marks the commit as never submittable.
- **Fork-only docs name upstream commits.** `Read at:` and the base hash are required by the fork
  pattern; upstream's `verify-repository-references` rejects commit identifiers in maintained
  files.
- **`python/sdk/uv.lock:113` fails the same check on the unchanged base.** It is a hex segment in
  a pythonhosted URL that matches a commit only with full history; upstream CI is shallow, and
  absent shallow-history objects cannot match.
