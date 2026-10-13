# Repository standard

The single source of truth for how repositories owned by yangflow are set up. Every yangflow repository is
public and open source, so there is one part plus a checklist.

Repositories are self-contained: they do not call reusable workflows, scripts or pinned revisions from
`yangflow/.github`. They only inherit its community health files and templates through GitHub's account
defaults.

## Contents

- [Root files](#root-files)
- [.github/](#github)
- [docs/ and design/](#docs-and-design)
- [Labels](#labels)
- [Releases](#releases)
- [Repository settings](#repository-settings)
- [Repository checklist](#repository-checklist)
- [Starting a new project](#starting-a-new-project)

## Root files

- `README.md`: `# Name`, a one-line description, badges (CI, license; a repository with releases also shows the
  latest release), then the sections Features (optional), Quick start, Usage, Development (exact commands),
  Contributing (link) and License. English. A Chinese README may exist as `README.zh-CN.md`, with an
  `English | 中文` switch line at the top of both.
- `LICENSE`: MIT, `Copyright (c) 2026 yangflow`, by default. Apache-2.0 is allowed when it is chosen at creation
  (currently `hoplet`); such a repository also keeps a `NOTICE` file. Do not change an existing repository's
  license without the maintainer's decision.
- `THIRD_PARTY.md`: only when the repository ships third-party code or licenses in its build artifacts (for
  example a MyGo app bundling `resources/ThirdPartyLicenses/`).
- `CHANGELOG.md`: Keep a Changelog style with an `## [Unreleased]` section. Repositories that use release-please
  let it write the entries. No `CHANGELOG.zh.md`.
- `CONTRIBUTING.md`: per repository, short, English. It states: issues and pull requests may be written in
  English or Simplified Chinese; open an issue before large changes; development setup; the exact test and lint
  commands; Conventional Commits for commit messages and pull request titles
  (`feat|fix|docs|refactor|perf|test|build|ci|chore|revert(scope)!: subject`); branches named
  `<type>/<issue>-<slug>`; squash merge by the maintainer; no DCO or sign-off. A repository without its own file
  inherits the account default.
- `CODE_OF_CONDUCT.md`, `SECURITY.md`, `GOVERNANCE.md` and `SUPPORT.md`: not per repository. The first two are
  inherited from `yangflow/.github`; governance and support are covered by `CONTRIBUTING.md`. A repository may
  keep its own `SECURITY.md` only if it has repository-specific scope (for example a self-hosted server with a
  trust boundary); reporting must then go through `https://github.com/yangflow/<repo>/security/advisories/new`
  (no e-mail).
- `AGENTS.md`: a self-contained agent guide (what it is, layout, commands, conventions, red lines). It does not
  reference `yangflow/.github`, milestones, tracking issues, sign-off or pinned SHAs. It has an "Issues and pull
  requests" section that spells out the issue forms and the pull request template (headings and labels), because
  agents create issues and pull requests with `gh` and the command line skips the forms and the template. Red lines always include:
  credentials never go into pull requests, issues, logs or chat; releases, signing and anything sent to an
  external service need the maintainer's confirmation; no blind `git add -A` of unknown files.
- `CLAUDE.md`: one line, `@AGENTS.md`. No `GEMINI.md` and no `.github/copilot-instructions.md`.
- `.editorconfig`: utf-8, lf, final newline, trimmed trailing whitespace; 2 spaces by default, tabs for Go and
  Makefiles, 4 spaces for Python and Swift.
- Toolchain pins where the ecosystem has them: `.go-version`, `.node-version`.

## .github/

- `CODEOWNERS`: `* @yangflow`
- `dependabot.yml`: `github-actions` monthly, plus the repository's own ecosystem (gomod, npm, pip, docker)
  weekly when present.
- `workflows/ci.yml`: self-contained; runs on `pull_request` and on push to `main`;
  `permissions: contents: read`; concurrency with cancel-in-progress for pull requests; actions pinned to major
  tags (for example `actions/checkout@v5`).
- `workflows/pr-title.yml`: `pull_request_target` with types `opened`, `edited`, `synchronize`, `reopened`;
  `permissions: pull-requests: read`; no checkout; the title (`${{ github.event.pull_request.title }}`) is passed
  through an environment variable and validated against the Conventional Commits pattern in bash; skipped for
  `dependabot[bot]`.
- Other workflows only when the repository needs them (`package.yml` for unsigned development builds,
  `release.yml` for releases). No `dco.yml`.
- Issue and pull request templates: none per repository; they are inherited from `yangflow/.github`
  (bug, feature, documentation; a short pull request template). Blank issues are disabled there.
- Not present: any `uses: yangflow/.github/...` caller, document-check configuration, repository-type markers or
  template version markers in documents.

## docs/ and design/

Keep only documents that describe the project to its users and contributors: product spec and scope,
architecture, ADRs (`docs/adr/`), design notes, testing notes. A plain roadmap may stay as `docs/roadmap.md`.
Do not keep internal process material (milestones, process ledgers, roadmaps with internal dependencies),
template version markers (HTML comments naming a template and version), or links into private repositories.
Remove `.claude/`, `.codex/` and similar agent directories unless they are part of what the project offers; in
that case say so in the README.

Apps with a UI keep `design/` (mockups, tokens, icon sources and the scripts that export them).
`design/README.md` is a short self-contained description: what is there, how to preview it, how to regenerate
icons. No status ledger, status words or sign-off states. `AGENTS.md` keeps one design rule: "A UI change starts
from the mockup in design/; the maintainer confirms the visual direction before UI code is written."
Screenshots for a pull request go into the pull request body, not into the repository.

## Labels

`scripts/sync-labels.sh <owner/repo>` in `yangflow/.github` applies the labels in `labels.yml` (bug, enhancement,
documentation, question, good first issue, help wanted, dependencies, duplicate, invalid, wontfix). A repository
may add its own scope labels (for example `ui`, `accessibility`, `scope:web`); the sync never creates or removes
them. Labels that duplicate a Conventional Commits type (`feat`, `fix`, `docs` and so on) or encode a process
state (`needs-decision`, `needs-triage`, `accepted`) are not used.

## Releases

- Repositories with releases use release-please (`googleapis/release-please-action` at its current major tag,
  `release-please-config.json` and `.release-please-manifest.json`, a `VERSION` file when useful). Tags are
  `vX.Y.Z`. The workflow has `contents: write` and `pull-requests: write` and uses `GITHUB_TOKEN`, or a GitHub
  App token when release pull requests must trigger CI.
- Signing, notarization and update feeds are set up per repository and documented in `docs/releasing.md`. Until
  they exist, a repository may only publish unsigned development builds as workflow artifacts.
- Other repositories (no release yet, tooling) have no release automation; `CHANGELOG.md` `[Unreleased]` is
  maintained by hand.

## Repository settings

Set on GitHub by the yangflow account (they are not stored in files):

- Squash merge only; the default commit message is the pull request title; delete head branches on merge.
- A ruleset on `main`: pull request required; the CI and PR title checks required; no force pushes.
- Private vulnerability reporting enabled.
- Discussions off unless the repository's `CONTRIBUTING.md` points to them.

## Repository checklist

Check each item when creating or auditing a repository.

- [ ] Name, one-line description and topics set; public
- [ ] `LICENSE` is MIT (or Apache-2.0 with `NOTICE`, chosen at creation)
- [ ] `README.md` has the required sections and badges; `README.zh-CN.md` optional with the language switch
- [ ] `CONTRIBUTING.md` lists the exact test and lint commands, Conventional Commits and the language rule
- [ ] `AGENTS.md` is self-contained with the red lines and the "Issues and pull requests" section; `CLAUDE.md` is the single line `@AGENTS.md`; no
      `GEMINI.md`, no `.github/copilot-instructions.md`
- [ ] `CHANGELOG.md` has `## [Unreleased]`; no `CHANGELOG.zh.md`
- [ ] `.editorconfig` present
- [ ] No per-repository `CODE_OF_CONDUCT.md`, `GOVERNANCE.md`, `SUPPORT.md`, issue or pull request templates;
      no `SECURITY.md` unless repository-specific
- [ ] `.github/CODEOWNERS` is `* @yangflow`
- [ ] `.github/dependabot.yml`: github-actions monthly plus the repository's ecosystem weekly
- [ ] `.github/workflows/ci.yml` is self-contained, with `contents: read` and actions pinned to major tags
- [ ] `.github/workflows/pr-title.yml` checks Conventional Commits; no `dco.yml`
- [ ] No `uses: yangflow/.github/...`, no template version markers, no links into private repositories
- [ ] Labels synced: `scripts/sync-labels.sh yangflow/<repo>` from `yangflow/.github`
- [ ] Repository settings applied (see [Repository settings](#repository-settings))
- [ ] Releases: release-please configured, or none (see [Releases](#releases))

## Starting a new project

There are no template repositories. Start from the existing repository with the same stack and copy only the
setup, not the product:

- `idset` or `oriel`: a pure Go desktop app with MyGo (`.go-version`, `scripts/check.sh`, `ci.yml`,
  `package.yml`).
- `hoplet`: a TypeScript and Go service (pnpm, Biome, Vitest, Docker image, release-please).

```bash
gh repo create yangflow/<name> --public --license mit
```

Copy `.editorconfig`, `.gitignore`, `.github/CODEOWNERS`, `.github/dependabot.yml`, `.github/workflows/ci.yml`
and `.github/workflows/pr-title.yml` from that repository, write `README.md`, `CONTRIBUTING.md`, `AGENTS.md`,
`CLAUDE.md` and `CHANGELOG.md` for the new project, then go through the
[Repository checklist](#repository-checklist).
