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

- **Workflows:**
    - Security workflows
        - [`scorecard.yaml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/scorecard.yaml) — OSSF Scorecard, repo-local (not inherited). Triggers: push to `main`, weekly cron (Mon 15:15 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-platform/security/code-scanning) + `scorecard-sarif` artifact.
        - [`devskim.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/devskim.yml) — inherits `devskim.yaml` from `.github`. Triggers: weekly cron (Sun 00:30 UTC), `workflow_dispatch`. Results: [Security → Code scanning](https://github.com/drasi-project/drasi-platform/security/code-scanning).
    - PR management workflows
        - [`pr-assignment-check.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/pr-assignment-check.yml) — repo-local (not inherited). Trigger: `pull_request_target` (opened/reopened/edited). Enforces linked-issue + author assignment on non-maintainer PRs; applies `needs-issue` and may close unassigned PRs.
        - [`pr-first-approval-label.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/pr-first-approval-label.yml) + [`pr-first-approval-label-run.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/pr-first-approval-label-run.yml) — repo-local two-stage workflow. Stage 1 on `pull_request_review`, Stage 2 on `workflow_run` (write token); manages `need-2nd-review` label.
    - Agentic workflows
        - PR reviewers orchestrator: [`pr-all-reviewers.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/pr-all-reviewers.yml) (trigger: `review:all` label or `workflow_dispatch`)
        - Per-aspect reviewers: `pr-correctness/design/docs/prior-art/security/testing-reviewer` (`.md` + `.lock.yml` pairs)
        - Issue researcher: [`drasi-issue-researcher.md`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/drasi-issue-researcher.md) + `.lock.yml` (trigger: `needs-research` label on issues)
    - Build/lint/test workflows
        - [`build-test.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/build-test.yml) — repo-local (not inherited). Trigger: PRs to `main`/`feature/*`/`release/*`; builds components/CLI and runs the e2e test.
        - [`lint.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/lint.yml) — inherits `rust-lint.yaml` from `.github`. Trigger: PRs to `main`.
            - Rust quality checks via `make lint-check` (clippy + fmt across Rust projects)
            - Repository-wide typo checks
    - Release/publishing workflows
        - [`draft-release.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/draft-release.yml) — repo-local. Trigger: `workflow_dispatch` (inputs: `tag`, `image_prefix`). Builds/publishes images, runs validation, drafts release assets.
        - [`image-validation.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/image-validation.yml) — repo-local reusable workflow (`workflow_call` + `workflow_dispatch`) for multi-arch image validation and pull tests.
        - [`vsce.yaml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/vsce.yaml) — repo-local. Trigger: `workflow_dispatch` (input: `version`). Publishes VS Code extension package using `VSCE_TOKEN`.
- **Labels:** [27 labels](https://github.com/drasi-project/drasi-platform/labels)
    - Reviewer/PR flow: `need-2nd-review`, `needs-2nd-review`, `needs-issue`, `do not merge`, `automerge-patch-candidate`, `automerge-minor-candidate`
    - Issue triage: `bug`, `enhancement`, `documentation`, `question`, `duplicate`, `invalid`, `wontfix`, `good first issue`, `help wanted`, `triaged`, `mentorship`
    - Automation/dependency/language: `automated`, `automation`, `dependencies`, `github_actions`, `go`, `java`, `javascript`, `python`, `rust`
    - Agentic/program: `needs-research`
- **Documents & templates:**
    - In-repo community health files (not inherited): `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `.github/pull_request_template.md`, issue templates
    - In-repo additional docs: `README.md`, `LICENSE` (Apache-2.0)
- **CODEOWNERS:** `@drasi-project/maintainers-platform` ([`.github/CODEOWNERS`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/CODEOWNERS))
- **Agents:** `.github/agents/issue-investigator.agent.md`
- **Ruleset:** [Settings → Rules](https://github.com/drasi-project/drasi-platform/settings/rules) — active, targets `main`, no bypass
    - Require PR before merging: 2 approvals, dismiss stale approvals on new commits, require review from `@drasi-project/maintainers-platform`, require Code Owner review, require approval of most recent push
    - Require status checks to pass: `e2e-tests` ([`build-test.yml`](https://github.com/drasi-project/drasi-platform/blob/drasi-platform-workflow-improvement/.github/workflows/build-test.yml))
    - Block force pushes

#### TODO

- Merge [drasi-project/drasi-platform#432](https://github.com/drasi-project/drasi-platform/pull/432) so the workflow/agent updates documented above are active on `main`.

### `learning`

- **Current state:** no consistency migration has been applied yet; repo still uses learning-specific workflows and labeling.
- **Workflows (current):**
    - Tutorial lifecycle workflows: [`build-tutorial-images.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/build-tutorial-images.yml), [`manual-build-images.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/manual-build-images.yml), [`release.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/release.yml)
    - Tutorial quality/evaluation workflows: [`tutorial-evaluation.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/tutorial-evaluation.yml), [`tutorial-evaluation-scheduled.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/tutorial-evaluation-scheduled.yml)
    - Automation workflow: [`automerge.yml`](https://github.com/drasi-project/learning/blob/main/.github/workflows/automerge.yml)
    - Not currently using centralized reusable workflows (`rust-unit-test`, `rust-lint`, `cargo-audit`, `devskim`) from `.github`.
- **Labels (current):** [16 labels](https://github.com/drasi-project/learning/labels), including `tutorial-failure`, `automated`, `automerge-*`, and standard issue labels.
- **Documents & templates (current):**
    - Inherited from `.github`: `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `pull_request_template.md`
    - In-repo: `README.md`, `LICENSE`, `.github/CODEOWNERS`
- **CODEOWNERS (current):** `@drasi-project/maintainers-learning` ([`.github/CODEOWNERS`](https://github.com/drasi-project/learning/blob/main/.github/CODEOWNERS))
- **Agents (current):** no `.github/agents/` directory detected.
- **Ruleset (current):** _TBD_ (capture from [Settings → Rules](https://github.com/drasi-project/learning/settings/rules)).

#### Proposed next steps

- Normalize labels toward org baseline while retaining `tutorial-failure` and other learning-specific labels.
- Document current traditional branch protection settings for `main`, then migrate to a repository ruleset aligned with org standards.

### `docs`

- **Workflows (current):**
    - [`website.yaml`](https://github.com/drasi-project/docs/blob/main/.github/workflows/website.yaml) — builds Hugo site on PRs/pushes and deploys GitHub Pages on `main` pushes.
    - [`spellcheck.yaml`](https://github.com/drasi-project/docs/blob/main/.github/workflows/spellcheck.yaml) — spellcheck on pushes/PRs using `.github/config/.pyspelling.yml`.
    - [`test.yaml`](https://github.com/drasi-project/docs/blob/main/.github/workflows/test.yaml) — manual docs validation flow tied to a `drasi-platform` release version input.
    - Not using centralized reusable Rust/security workflows from `.github` (`rust-unit-test`, `rust-lint`, `cargo-audit`, `devskim`), which is expected for this repo type.
- **Labels (current):** [9 labels](https://github.com/drasi-project/docs/labels), currently the core issue triage set (`bug`, `documentation`, `enhancement`, `question`, `duplicate`, `invalid`, `wontfix`, `good first issue`, `help wanted`).
- **Documents & templates (current):**
    - In-repo: `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `readme.md`, `.github/CODEOWNERS`
    - Inherited from `.github`: `SECURITY.md`, `pull_request_template.md`
    - Local issue templates: none detected (`.github/ISSUE_TEMPLATE/` not present)
- **CODEOWNERS (current):** `@drasi-project/maintainers-docs` ([`.github/CODEOWNERS`](https://github.com/drasi-project/docs/blob/main/.github/CODEOWNERS))
- **Agents (current):** no `.github/agents/` directory detected.
- **Permissions (current):** _TBD_ (capture write/admin teams from repo settings).
- **Ruleset (current):** _TBD_ (capture from [Settings → Rules](https://github.com/drasi-project/docs/settings/rules)).
- **Repo-specific:** Hugo/Docsy site pipeline and spellcheck dictionaries in `.github/config/` (`.pyspelling.yml`, `en-custom.txt`, `en-drasi.txt`).

#### Proposed next steps

- Document current traditional branch protection settings for `main`, then migrate to a repository ruleset aligned with org standards.
- Research tools, mechanisms, and practices for upgrading Docsy versions.