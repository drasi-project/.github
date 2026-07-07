# Drasi Repository Consistency Standards

Standard settings that apply to every repo in the [`drasi-project`](https://github.com/drasi-project) org, plus a per-repo snapshot of what is configured today.

## Org-wide settings

| Area | Standard | Managed in |
|---|---|---|
| Branch protection | Org-level rulesets (required reviews, required status checks, no force push) | [Org settings → Rulesets](https://github.com/organizations/drasi-project/settings/rules/14440581) |
| Secrets | Org-level: `COPILOT_GITHUB_TOKEN`, `ISSUE_UPDATE_TOKEN`, `DRASI_REVIEWER_APP_ID`, `DRASI_REVIEWER_APP_PRIVATE_KEY` | Org settings → Variables/Secrets |


---

### `.github`

Stores shared assets used by other repositories.

- **Workflows:** reusable workflow definitions/templates in `workflow-templates/`, plus agentic workflow sources in `agentic-workflows/`.
- **Community docs:** org-default community and contribution files (`CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, `AI_POLICY.md`, `MENTORSHIP.md`, `LICENSE`), issue forms in `ISSUE_TEMPLATE/`, and `pull_request_template.md`.

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
    - Repo-specific workflows
        - [`release-plz.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/release-plz.yml) — repo-local. Triggers: push to `main`, `workflow_dispatch` (with `dry_run` / `force_publish` inputs). On regular commits opens/updates a release PR with version bumps + changelog; on a release-PR merge publishes crates to crates.io and creates git tags. Uses the `drasi-core-release` environment and `CARGO_REGISTRY_TOKEN`.
        - [`publish-crate.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/publish-crate.yml) — repo-local manual fallback. Trigger: `workflow_dispatch` (input: `package`). Runs `cargo publish -p <package>` for recovering from a failed `release-plz` run.
        - [`publish-plugins.yml`](https://github.com/drasi-project/drasi-core/blob/main/.github/workflows/publish-plugins.yml) — repo-local. Trigger: `workflow_dispatch` (inputs: `pre_release`, `registry`, `dry_run`). Cross-platform matrix build of plugin `cdylib`s and publish as OCI artifacts to GHCR (signed with cosign); follow-up job sets each package to public.
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
    - Repo-specific workflows
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

- **Workflows:**
    - Security workflows
        - [`scorecard.yaml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/scorecard.yaml) — OSSF Scorecard, repo-local (not inherited). Triggers: push to `main`, weekly cron (Mon 15:15 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-platform/security/code-scanning) + `scorecard-sarif` artifact.
        - [`devskim.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/devskim.yml) — inherits `devskim.yaml` from `.github`. Triggers: push to `main`, PRs to `main`, weekly cron (Sun 00:30 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-platform/security/code-scanning).
    - PR management workflows
        - [`pr-assignment-check.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/pr-assignment-check.yml) — repo-local (not inherited). Trigger: `pull_request_target` (opened/reopened/edited). Enforces linked-issue + author assignment on non-maintainer PRs; applies `needs-issue` and may close unassigned PRs.
        - [`pr-first-approval-label.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/pr-first-approval-label.yml) + [`pr-first-approval-label-run.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/pr-first-approval-label-run.yml) — repo-local two-stage workflow. Stage 1 on `pull_request_review`, Stage 2 on `workflow_run` (write token); manages `need-2nd-review` label.
    - Agentic Workflows
        - PR reviewers orchestrator: [`pr-all-reviewers.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/pr-all-reviewers.yml) (trigger: `review:all` label or `workflow_dispatch`)
        - Per-aspect reviewers: `pr-correctness/design/docs/prior-art/security/testing-reviewer` (`.md` + `.lock.yml` pairs)
        - Issue researcher: [`drasi-issue-researcher.md`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/drasi-issue-researcher.md) + `.lock.yml` (trigger: `needs-research` label on issues)
    - Build/lint/test workflows
        - [`build-test.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/build-test.yml) — repo-local (not inherited). Triggers: pushes to `main`/`release/*`/tags `v*`; PRs to `main`/`feature/*`/`release/*`; builds components/CLI and runs the e2e test.
        - [`lint.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/lint.yml) — inherits `rust-lint.yaml` from `.github`. Triggers: pushes and PRs. Runs Rust quality checks via `make lint-check` (clippy + fmt) and repository-wide typo checks.
    - Repo-specific workflows
        - [`draft-release.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/draft-release.yml) — repo-local. Trigger: `workflow_dispatch` (inputs: `tag`, `image_prefix`). Builds/publishes images, runs validation, drafts release assets.
        - [`image-validation.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/image-validation.yml) — repo-local reusable workflow (`workflow_call` + `workflow_dispatch`) for multi-arch image validation and pull tests.
        - [`vsce.yaml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/vsce.yaml) — repo-local. Trigger: `workflow_dispatch` (input: `version`). Publishes VS Code extension package using `VSCE_TOKEN`.
        - [`automerge.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/automerge.yml) — repo-local. Trigger: weekly cron (Wed 12:00 PM PT), `workflow_dispatch`. Automatically merges Renovate PRs labeled `automerge-patch-candidate` or `automerge-minor-candidate` after a waiting period.
- **Labels:** [33 labels](https://github.com/drasi-project/drasi-platform/labels)
    - Reviewer labels: `review:all`, `review:correctness`, `review:design`, `review:docs`, `review:security`, `review:testing`
    - PR workflow: `need-2nd-review`, `needs-2nd-review`, `needs-issue`, `do not merge`, `automerge-patch-candidate`, `automerge-minor-candidate`
    - Issue triage: `bug`, `enhancement`, `documentation`, `question`, `duplicate`, `invalid`, `wontfix`, `good first issue`, `help wanted`, `triaged`, `mentorship`
    - Automation/dependency/language: `automated`, `automation`, `dependencies`, `github_actions`, `go`, `java`, `javascript`, `python`, `rust`
    - Agentic / program: `needs-research`
- **Documents & templates:**
    - Inherited from `.github` (community health files):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `SECURITY.md`
        - `pull_request_template.md`
    - In-repo (override or repo-specific):
        - `README.md`
        - `LICENSE` (Apache-2.0)
        - `.github/CODEOWNERS`
- **CODEOWNERS:** `@drasi-project/maintainers-platform` ([`.github/CODEOWNERS`](https://github.com/drasi-project/drasi-platform/blob/main/.github/CODEOWNERS))
- **Agents:** `.github/agents/issue-investigator.agent.md`
- **Ruleset:** [Settings → Rules](https://github.com/drasi-project/drasi-platform/settings/rules) — active, targets `main`, no bypass
    - Require PR before merging: 2 approvals, dismiss stale approvals on new commits, require review from `@drasi-project/maintainers-platform`, require Code Owner review, require approval of most recent push
    - Require status checks to pass: `e2e-tests` ([`build-test.yml`](https://github.com/drasi-project/drasi-platform/blob/main/.github/workflows/build-test.yml))
    - Block force pushes

### `learning`

- **Workflows:**
    - Security workflows
        - [`scorecard.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/scorecard.yml) — OSSF Scorecard, repo-local (not inherited). Triggers: push to `main`, weekly cron (Mon 15:15 UTC), `workflow_dispatch`.
        - [`devskim.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/devskim.yml) — inherits `devskim.yaml` from `.github`. Triggers: weekly cron (Sun 00:30 UTC), `workflow_dispatch`.
    - Tutorial lifecycle workflows
        - [`build-tutorial-images.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/build-tutorial-images.yml), [`manual-build-images.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/manual-build-images.yml), [`release.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/release.yml)
    - Tutorial quality/evaluation workflows
        - [`tutorial-evaluation.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/tutorial-evaluation.yml), [`tutorial-evaluation-scheduled.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/tutorial-evaluation-scheduled.yml)
- **Labels:** [16 labels](https://github.com/drasi-project/learning/labels)
    - Issue triage: `bug`, `documentation`, `duplicate`, `enhancement`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`
    - Automation/dependency/language: `automated`, `automerge-patch-candidate`, `automerge-minor-candidate`, `dependencies`, `javascript`, `python`
    - Repo-specific: `tutorial-failure`
- **Documents & templates:**
    - Inherited from `.github` (community health files):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `SECURITY.md`
        - `pull_request_template.md`
    - In-repo (override or repo-specific):
        - `README.md`
        - `LICENSE`
        - `.github/CODEOWNERS`
- **CODEOWNERS:** `@drasi-project/maintainers-learning` ([`.github/CODEOWNERS`](https://github.com/drasi-project/learning/blob/main/.github/CODEOWNERS))
- **Agents:** no `.github/agents/` directory detected.
- **Ruleset:** _TBD_ (capture from [Settings → Rules](https://github.com/drasi-project/learning/settings/rules)).

#### TODO

- Document current traditional branch protection settings for `main`, then migrate to a repository ruleset aligned with org standards.

### `learning-drasi-server`

- **Workflows:**
    - Repo-specific workflows
        - [`release-tutorials.yml`](https://github.com/drasi-project/learning-drasi-server/blob/main/.github/workflows/release-tutorials.yml) — repo-local. Trigger: `workflow_dispatch` (input: `tag`). Packages the `tutorials/` directory into a zip archive and creates or updates a GitHub Release with the bundle as a downloadable asset.
- **Labels:** [9 labels](https://github.com/drasi-project/learning-drasi-server/labels)
    - Issue triage: `bug`, `documentation`, `duplicate`, `enhancement`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`
- **Documents & templates:**
    - Inherited from `.github` (community health files):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `SECURITY.md`
        - `pull_request_template.md`
    - In-repo (override or repo-specific):
        - `README.md`
        - `LICENSE` (Apache-2.0)
        - `.github/CODEOWNERS`
- **CODEOWNERS:** `@drasi-project/maintainers-learning-drasi-server` ([`.github/CODEOWNERS`](https://github.com/drasi-project/learning-drasi-server/blob/main/.github/CODEOWNERS))
- **Agents:** no `.github/agents/` directory detected.
- **Ruleset:** [`learning-drasi-server-main`](https://github.com/drasi-project/learning-drasi-server/settings/rules) — active, targets `main`, bypass: `ruokun-niu` (always allow)
    - Require PR before merging: 2 approvals, require Code Owner review, require approval of most recent push
    - Block force pushes

### `test-infra`

- **Workflows:**
    - Build/lint/test workflows
        - [`build.yml`](https://github.com/drasi-project/test-infra/blob/main/.github/workflows/build.yml) — repo-local. Triggers: pushes to `main`/`release/*`/tags `v*`; PRs to `main`/`feature/*`/`feature-lib`/`release/*`; `workflow_dispatch`. Builds the E2E test framework components (proxy/reactivator/test-service).
        - [`lint.yml`](https://github.com/drasi-project/test-infra/blob/main/.github/workflows/lint.yml) — repo-local. Triggers: pushes (`*`) and PRs to `main`/`feature-lib`. Runs Rust lint checks via `make lint-check` in `e2e-test-framework`.
    - Repo-specific workflows
        - [`draft-release.yml`](https://github.com/drasi-project/test-infra/blob/main/.github/workflows/draft-release.yml) — repo-local. Trigger: `workflow_dispatch` (inputs: `tag`, `image_prefix`). Builds and publishes multi-arch test-infra component images to GHCR and creates manifest lists.
- **Labels:** [9 labels](https://github.com/drasi-project/test-infra/labels)
    - Issue triage: `bug`, `documentation`, `duplicate`, `enhancement`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`
- **Documents & templates:**
    - Inherited from `.github` (community health files):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `SECURITY.md`
        - `pull_request_template.md`
    - In-repo (override or repo-specific):
        - `README.md`
        - `LICENSE`
        - `.github/CODEOWNERS`
- **CODEOWNERS:** `@drasi-project/drasi-engineering-team` ([`.github/CODEOWNERS`](https://github.com/drasi-project/test-infra/blob/main/.github/CODEOWNERS))
- **Agents:** no `.github/agents/` directory detected.
- **Ruleset:** [Settings → Rules](https://github.com/drasi-project/test-infra/settings/rules) — active, targets default branch, no bypass
    - Require PR before merging: 1 approval, dismiss stale approvals on new commits, require Code Owner review, require approval of most recent push
    - Block force pushes

#### TODO

- Add security workflows (`scorecard`, `devskim`) aligned with org standards.
- Expand labels toward org baseline.
- Capture and document active branch ruleset details for `main`.

---

### `docs`

- **Workflows:**
    - Repo-specific workflows
        - [`website.yaml`](https://github.com/drasi-project/docs/blob/main/.github/workflows/website.yaml) — builds Hugo site on PRs/pushes and deploys GitHub Pages on `main` pushes.
        - [`spellcheck.yaml`](https://github.com/drasi-project/docs/blob/main/.github/workflows/spellcheck.yaml) — spellcheck on pushes/PRs using `.github/config/.pyspelling.yml`.
        - [`test.yaml`](https://github.com/drasi-project/docs/blob/main/.github/workflows/test.yaml) — manual docs validation flow tied to a `drasi-platform` release version input (`workflow_dispatch` only).
- **Labels:** [9 labels](https://github.com/drasi-project/docs/labels)
    - Issue triage: `bug`, `documentation`, `duplicate`, `enhancement`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`
- **Documents & templates:**
    - In-repo (override or repo-specific):
        - `CODE_OF_CONDUCT.md`
        - `CONTRIBUTING.md`
        - `readme.md`
        - `.github/CODEOWNERS`
    - Inherited from `.github`:
        - `SECURITY.md`
        - `pull_request_template.md`
- **CODEOWNERS:** `@drasi-project/maintainers-docs` ([`.github/CODEOWNERS`](https://github.com/drasi-project/docs/blob/main/.github/CODEOWNERS))
- **Agents:** no `.github/agents/` directory detected.
- **Ruleset:** [Settings → Rules](https://github.com/drasi-project/docs/settings/rules) — active, targets `main`, no bypass
    - Require PR before merging: 2 approvals, dismiss stale approvals on new commits, require review from `@drasi-project/maintainers-docs`, require Code Owner review, require approval of most recent push
    - Block force pushes
- **Repo-specific:** Hugo/Docsy site pipeline and spellcheck dictionaries in `.github/config/` (`.pyspelling.yml`, `en-custom.txt`, `en-drasi.txt`).

#### TODO

- Merge [drasi-project/docs#249](https://github.com/drasi-project/docs/pull/249) to integrate the `learning-drasi-server` Hugo module as a content source.
- Research tools, mechanisms, and practices for upgrading Docsy versions.