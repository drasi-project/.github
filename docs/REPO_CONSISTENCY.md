# Drasi Repository Consistency Standards

Standard settings that apply to every repo in the [`drasi-project`](https://github.com/drasi-project) org, plus a per-repo snapshot of what is configured today.

## Org-wide standards

| Area | Standard | Managed in |
|---|---|---|
| Branch protection | Org-level rulesets (required reviews, required status checks, no force push) | [Org settings → Rulesets](https://github.com/organizations/drasi-project/settings/rules/14440581) |
| CODEOWNERS | Required in every repo; not inherited | Each repo (`.github/CODEOWNERS`) |
| CI / workflows | Reusable workflows + starter templates | [`drasi-project/.github`](https://github.com/drasi-project/.github) |
| Labels | Standard label set applied across repos | Individual Repos |
| Issue/PR templates | Inherited from `.github` | [`drasi-project/.github`](https://github.com/drasi-project/.github) |
| Community docs | `CODE_OF_CONDUCT`, `CONTRIBUTING`, `SECURITY`, `SUPPORT`, `AI_POLICY`, `MENTORSHIP`, `LICENSE` inherited from `.github` | [`drasi-project/.github`](https://github.com/drasi-project/.github) |
| Agentic workflows | Source `.md` in `.github` | Per-repo: `.github/workflows/` |
| Secrets | Org-level: `COPILOT_GITHUB_TOKEN`, `ISSUE_UPDATE_TOKEN`, `DRASI_REVIEWER_APP_ID`, `DRASI_REVIEWER_APP_PRIVATE_KEY` | Org settings → Variables/Secrets |


---

## Per-repo configuration

For each repo we track:

- **Workflows** — CI workflows the repo calls or hosts
- **Labels** — repo-specific labels beyond the org default set
- **Templates** — issue/PR templates in the repo (otherwise inherited)
- **CODEOWNERS** — team(s) that own the repo
- **Agents** — `.agent.md` files in the repo (otherwise inherited)
- **Permissions** — write/admin teams beyond org defaults
- **Repo-specific** — anything that legitimately diverges from the standard

`inherited` = comes from `.github`, nothing in-repo required. `_TBD_` = not yet audited.

---

### `.github`

Source of truth for all org-level defaults.

- **Workflows:** hosts all reusable workflows (`rust-unit-test`, `rust-lint`, `cargo-audit`, `devskim`) and agentic workflow sources
- **Labels:** _TBD_
- **Templates:** owns the org-default issue forms (`bug`, `feature`, `engineering`, `config`) and `pull_request_template.md`
- **CODEOWNERS:** _TBD_
- **Agents:** `agentic-workflows.agent.md`
- **Permissions:** _TBD_
- **Repo-specific:** `profile/README.md` (org profile), `CHECKLIST.md`, `workflow-templates/`

### `drasi-core`

- **Workflows:** 
    - Security workflows 
        - [`scorecard.yaml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/scorecard.yaml) — OSSF Scorecard, repo-local (not inherited). Triggers: push to `main`, weekly cron (Mon 15:15 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-core/security/code-scanning) + `scorecard-sarif` artifact.
        - [`devskim.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/devskim.yml) — inherits `devskim.yaml` from `.github`. Triggers: weekly cron (Sun 00:30 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-core/security/code-scanning).
        - [`cargo-audit.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/cargo-audit.yml) — inherits `cargo-audit.yaml` from `.github`. Triggers: PRs to `main`, weekly cron (Mon 01:30 UTC). Results: workflow run logs in [Actions](https://github.com/drasi-project/drasi-core/actions/workflows/cargo-audit.yml).
    - PR management workflows
        - [`pr-assignment-check.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/pr-assignment-check.yml) — repo-local (not inherited). Trigger: `pull_request_target` (opened/reopened/edited). Enforces that non-maintainer PRs link an issue and that the author is assigned; applies `needs-issue` / `needs-assignment` labels.
        - [`pr-first-approval-label.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/pr-first-approval-label.yml) + [`pr-first-approval-label-run.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/pr-first-approval-label-run.yml) — repo-local two-stage workflow. Stage 1 triggers on `pull_request_review` (approval) and uploads the PR number as an artifact; Stage 2 triggers on `workflow_run` in the base-repo context (write token) and adds/removes the `need-2nd-review` label based on approval count. Two-stage because `pull_request_review` only grants a read-only token for fork PRs.
    - Agentic Workflows
        - PR reviewers
        - Issue researcher
    - Rust Workflows
        - [`test.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/test.yml) — inherits `rust-unit-test.yaml` from `.github`. Triggers: PRs to `main`/`feature/*`/`feature-lib`/`release/*`, and `workflow_dispatch`.
        - [`ci-lint.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/ci-lint.yml) — inherits `rust-lint.yaml` from `.github`. Triggers: PRs to `main`/`feature/*`/`feature-lib`/`release/*`.
        - [`coverage.yaml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/coverage.yaml) — repo-local (not inherited); uploads to [Coveralls](https://coveralls.io/github/drasi-project/drasi-core). Triggers: push to `main`, PRs to `main`.
        - [`test-ffi.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/test-ffi.yml) — repo-local (not inherited); cross-platform FFI tests. Triggers: PRs to `main`/`feature/*`/`release/*`, and `workflow_dispatch`.
    - Repo specific workflows
        - [`release-plz.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/release-plz.yml) — repo-local. Triggers: push to `main`, `workflow_dispatch` (with `dry_run` / `force_publish` inputs). On regular commits opens/updates a release PR with version bumps + changelog; on a release-PR merge publishes crates to crates.io and creates git tags. Uses the `drasi-core-release` environment and `CARGO_REGISTRY_TOKEN`.
        - [`publish-crate.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/publish-crate.yml) — repo-local manual fallback. Trigger: `workflow_dispatch` (input: `package`). Runs `cargo publish -p <package>` for recovering from a failed `release-plz` run.
        - [`publish-plugins.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/publish-plugins.yml) — repo-local. Trigger: `workflow_dispatch` (inputs: `pre_release`, `registry`, `dry_run`). Cross-platform matrix build of plugin `cdylib`s and publish as OCI artifacts to GHCR (signed with cosign); follow-up job sets each package to public.
    - Organization
- **Labels:** [25 labels](https://github.com/drasi-project/drasi-core/labels)
    - Reviewer labels: `review:all`, `review:correctness`, `review:design`, `review:docs`, `review:prior-art`, `review:security`, `review:testing`
    - PR workflow: `need-2nd-review`, `need-changes`, `needs-issue`, `do not merge`, `stale`, `release`
    - Issue triage: `bug`, `enhancement`, `documentation`, `question`, `duplicate`, `invalid`, `wontfix`, `good first issue`, `help wanted`
    - Agentic / program: `agentic-workflows`, `needs-research`
    - Additional: `mentorship`
- **Documents & templates:**
    - Inherited from `.github` (community health files):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `SECURITY.md`
        - `SUPPORT.md`
        - `AI_POLICY.md`
        - `MENTORSHIP.md`
        - Issue forms in `ISSUE_TEMPLATE/`: `bug.yaml`, `feature.yaml`, `engineering.yaml`, `config.yaml`
    - In-repo (override or repo-specific):
        - `README.md`
        - `LICENSE` (Apache-2.0)
        - `.github/pull_request_template.md`
        - `.github/CODEOWNERS`
- **CODEOWNERS:** `@drasi-project/maintainers-core` ([`.github/CODEOWNERS`](https://github.com/drasi-project/drasi-core/blob/main/.github/CODEOWNERS))
- **Ruleset:** [`drasi-core-main`](https://github.com/drasi-project/drasi-core/settings/rules) — active, targets `main`, no bypass
    - Require PR before merging: 2 approvals, dismiss stale approvals on new commits, require review from `@drasi-project/maintainers-core`, require Code Owner review, require approval of most recent push
    - Require status checks to pass: `test / Rust Unit Tests` ([`test.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/test.yml))
    - Block force pushes

#### TODO

- Confirm whether `.github/.codecov.yml` is still in use (coverage uploads go to Coveralls, not Codecov) — remove if vestigial.

### `drasi-server`

- **Workflows:**
    - Security workflows
        - [`scorecard.yaml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/scorecard.yaml) — OSSF Scorecard, repo-local (not inherited). Triggers: push to `main`, weekly cron (Mon 15:15 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-server/security/code-scanning) + `scorecard-sarif` artifact.
        - [`devskim.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/devskim.yml) — inherits `devskim.yaml` from `.github`. Triggers: weekly cron (Sun 00:30 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-server/security/code-scanning).
        - [`cargo-audit.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/cargo-audit.yml) — inherits `cargo-audit.yaml` from `.github`. Triggers: PRs to `main`, weekly cron (Mon 01:30 UTC). Results: workflow run logs in [Actions](https://github.com/drasi-project/drasi-server/actions/workflows/cargo-audit.yml).
    - PR management workflows
        - [`pr-assignment-check.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/pr-assignment-check.yml) — repo-local (not inherited). Trigger: `pull_request_target` (opened/reopened/edited). Enforces linked-issue + author assignment on non-maintainer PRs; applies `needs-issue` / `needs-assignment` labels.
        - [`pr-first-approval-label.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/pr-first-approval-label.yml) + [`pr-first-approval-label-run.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/pr-first-approval-label-run.yml) — repo-local two-stage workflow. Stage 1 on `pull_request_review`, Stage 2 on `workflow_run` (write token); manages `need-2nd-review` label.
    - Agentic Workflows
        - PR reviewers (`pr-all-reviewers.yml` + 6 per-aspect reviewer pairs)
        - Issue researcher (`trigger-issue-research.yml`)
        - YAML snippet validator (`validate-yaml-snippets.md/.lock.yml`)
    - Rust Workflows
        - [`test.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/test.yml) — inherits `rust-unit-test.yaml` from `.github`. Trigger: PRs to `main`.
        - [`lint.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/lint.yml) — inherits `rust-lint.yaml` from `.github`; uses `make clippy` / `make fmt-check`. Trigger: PRs to `main`.
    - Repo specific workflows
        - [`release.yaml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/release.yaml) — repo-local. Triggers: `workflow_dispatch` (inputs: `tag`, `image_prefix`, `dry_run`), weekly cron (Mon 08:00 UTC, dry-run). Cross-platform matrix build of `drasi-server` + `drasi-sse-cli` binaries (Linux glibc/musl × x86_64/arm64, macOS, Windows MSVC), multi-arch Docker images to GHCR, and a versioned GitHub Release with artifacts.
        - [`docker-build-check.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/docker-build-check.yml) — repo-local. Trigger: PRs to `main`. Builds the Docker image for `linux/amd64` and `linux/arm64` (no push) as a PR sanity check.
        - [`integration-test-getting-started.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/integration-test-getting-started.yml) — repo-local. Triggers: PRs + pushes to `main`/`feature-lib`. Runs the getting-started integration test against a PostgreSQL service.
        - [`copilot-setup-steps.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/copilot-setup-steps.yml) — repo-local. Defines environment setup steps for the GitHub Copilot coding agent (not a normal CI workflow).
- **Labels:** [19 labels](https://github.com/drasi-project/drasi-server/labels)
    - Reviewer labels: `review:all`, `review:correctness`, `review:design`, `review:docs`, `review:prior-art`, `review:security`, `review:testing`
    - PR workflow: `need-2nd-review`
    - Issue triage: `bug`, `enhancement`, `documentation`, `question`, `duplicate`, `invalid`, `wontfix`, `good first issue`, `help wanted`
    - Agentic / program: `agentic-workflows`, `needs-research`
- **Documents & templates:**
    - Inherited from `.github` (community health files):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `SECURITY.md`
        - `SUPPORT.md`
        - `AI_POLICY.md`
        - `MENTORSHIP.md`
        - `pull_request_template.md`
        - Issue forms in `ISSUE_TEMPLATE/`: `bug.yaml`, `feature.yaml`, `engineering.yaml`, `config.yaml`
    - In-repo (override or repo-specific):
        - `README.md`
        - `LICENSE` (Apache-2.0)
        - `.github/CODEOWNERS`
- **CODEOWNERS:** `@drasi-project/maintainers-server` ([`.github/CODEOWNERS`](https://github.com/drasi-project/drasi-server/blob/main/.github/CODEOWNERS))
- **Ruleset:** [Settings → Rules](https://github.com/drasi-project/drasi-server/settings/rules) — active, targets `main`, no bypass
    - Require PR before merging: 2 approvals, dismiss stale approvals on new commits, require review from `@drasi-project/maintainers-server`, require Code Owner review, require approval of most recent push
    - Require status checks to pass: `test / Rust Unit Tests` ([`test.yml`](https://github.com/drasi-project/drasi-server/blob/main/.github/workflows/test.yml))
    - Block force pushes

#### TODO

- Remove `.github/workflows/copilot-setup-steps.yml` (not needed for this repo).

### `drasi-platform`

- **Workflows:** _TBD_
- **Labels:** _TBD_
- **Templates:** inherited
- **CODEOWNERS:** _TBD_
- **Agents:** inherited
- **Permissions:** _TBD_
- **Repo-specific:** _TBD_

### `drasi-learning`

- **Workflows:** _TBD_
- **Labels:** _TBD_
- **Templates:** inherited
- **CODEOWNERS:** _TBD_
- **Agents:** inherited
- **Permissions:** _TBD_
- **Repo-specific:** _TBD_

### `docs` (design-docs)

- **Workflows:** _TBD_ (docs build / link check?)
- **Labels:** _TBD_
- **Templates:** inherited
- **CODEOWNERS:** _TBD_
- **Agents:** inherited
- **Permissions:** _TBD_
- **Repo-specific:** _TBD_
