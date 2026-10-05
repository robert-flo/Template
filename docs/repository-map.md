# Repository map

This concise map covers every versioned file except `tests/`, whose files are
executable verification contracts rather than template building blocks. Update
this map whenever an in-scope file is added, removed, or changes purpose.

## Repository root

| File | Purpose and reason |
| --- | --- |
| `.directory` | Sets the repository folder icon so graphical file managers identify the project. |
| `.dockerignore` | Excludes unnecessary files from Docker build contexts to keep validation images small. |
| `.editorconfig` | Establishes shared editor defaults so contributors produce compatible text files. |
| `.gitignore` | Prevents local outputs, logs, and user-specific artifacts from becoming history. |
| `.markdownlint.yaml` | Defines Markdown quality rules, including Release Please changelog compatibility. |
| `.pre-commit-config.yaml` | Declares the single local quality-gate entrypoint used before commits. |
| `.release-please-manifest.json` | Stores the current Release Please package version for automated releases. |
| `AGENTS.md` | Gives contributors and coding agents the repository workflow, safety, and quality rules. |
| `CHANGELOG.md` | Records released user-visible changes and is maintained by Release Please. |
| `COMMIT_MESSAGE_GUIDELINES.md` | Defines the Conventional Commit and Gitmoji conventions that feed release automation. |
| `CONTRIBUTING.md` | Explains how to contribute safely, consistently, and through pull requests. |
| `Dockerfile` | Provides a clean container environment for template verification. |
| `GLOSSARY.md` | Records shared domain vocabulary and architectural context for ongoing work. |
| `LICENSE` | States the legal terms under which the template can be used and redistributed. |
| `Makefile` | Composes the focused `make/` modules into the public command interface. |
| `README.md` | Introduces the template and routes users to setup, commands, and policies. |
| `RELEASE_POLICY.md` | Documents the automated release lifecycle and maintainer responsibilities. |
| `dockerfile.sh` | Provides the executable Hello World payload verified by the Docker contract. |
| `release-please-config.json` | Configures Release Please changelogs, release assets, and Gitmoji commit titles. |
| `version.txt` | Supplies the template version consumed by its release and verification contracts. |

## Contributor tooling and editor support

| File | Purpose and reason |
| --- | --- |
| `.commandcode/taste/taste.md` | Lets CommandCode learn repository-specific command preferences for future assistance. |
| `.git-hooks/ravn-shell-quality` | Formats and ShellChecks staged shell files as part of the local quality gate. |
| `.vscode/extensions.json` | Recommends editor extensions so contributors receive the intended language support. |
| `.vscode/settings.json` | Shares VS Code workspace settings that preserve repository formatting behavior. |
| `.vscode/shellcheck.sh` | Adapts ShellCheck invocation for the VS Code task environment. |
| `.vscode/tasks.json` | Defines VS Code tasks for common repository checks and commands. |

## GitHub collaboration and automation

| File | Purpose and reason |
| --- | --- |
| `.github/PULL_REQUEST_TEMPLATE.md` | Prompts pull-request authors for scope, validation, and release-impact information. |
| `.github/ISSUE_TEMPLATE/bug_report.yml` | Collects reproducible bug reports with the context needed to triage them. |
| `.github/ISSUE_TEMPLATE/custom.yml` | Provides a flexible issue form for work outside a specialized template. |
| `.github/ISSUE_TEMPLATE/documentation_update.yml` | Collects documentation requests with the affected audience and material. |
| `.github/ISSUE_TEMPLATE/feature_request.yml` | Collects feature proposals with their problem, outcome, and constraints. |
| `.github/workflows/ci.yml` | Runs the repository verification suite on GitHub before changes are integrated. |
| `.github/workflows/lock.yml` | Locks inactive conversations to keep issue and pull-request discussions manageable. |
| `.github/workflows/release-please.yml` | Lets Release Please prepare releases and publish verified release assets. |

## Make command modules

| File | Purpose and reason |
| --- | --- |
| `make/aliases.mk` | Supplies short aliases that make the public Make interface faster to use interactively. |
| `make/docker.mk` | Defines Docker build and validation commands so clean-environment checks are repeatable. |
| `make/git.mk` | Defines guided Git commands with the repository's user-facing visual language. |
| `make/hooks.mk` | Installs and checks the local quality-gate hooks required before commits. |
| `make/quality.mk` | Exposes deterministic formatting, linting, and verification commands for contributors and CI. |
| `make/release.mk` | Exposes release diagnostics and GitHub setup commands without handling secrets directly. |

## Architecture and agent references

| File | Purpose and reason |
| --- | --- |
| `.agents/agents/PM-rf-Template.md` | Defines the Antigravity PM agent for this repository (ADR 0001/0002 in antigravity-fleet). |
| `.agents/agents/RV-rf-Template.md` | Defines the Antigravity reviewer agent for this repository. |
| `.agents/agents/WK-rf-Template.md` | Defines the Antigravity worker agent for this repository. |
| `docs/adr/0001-pre-commit-framework-as-quality-gate-entrypoint.md` | Records why pre-commit is the sole local quality-gate entrypoint. |
| `docs/adr/0002-quality-gate-escape-hatches.md` | Defines the permitted, auditable ways to bypass quality checks when necessary. |
| `docs/adr/0003-doc-quality-gate-without-docker.md` | Records the decision to lint documentation locally without a Docker dependency. |
| `docs/adr/0004-strict-doc-quality-on-commit-project-scope.md` | Establishes strict documentation checks across the committed project scope. |
| `docs/adr/0005-single-markdownlint-config-file.md` | Records why Markdown linting uses one shared configuration file. |
| `docs/adr/0006-quality-gate-bootstrap.md` | Defines how clones install the quality gate and verify required host tools. |
| `docs/adr/0007-shell-failure-reports-in-worktree-logs.md` | Records why shell-quality failures write diagnostic reports inside each worktree. |
| `docs/adr/0008-no-shell-path-exclusions.md` | Establishes that every staged shell file is subject to the ShellCheck gate. |
| `docs/adr/0009-repository-bootstrap-scope.md` | Defines the safe boundary between local bootstrap and canonical remote configuration. |
| `docs/adr/0010-release-please-managed-lifecycle.md` | Records the Release Please ownership model for versions, changelogs, and releases. |
| `docs/agents/domain.md` | Explains where shared domain vocabulary and architectural decisions are maintained. |
| `docs/agents/issue-tracker.md` | Documents GitHub issue workflow and the metadata agents must maintain. |
| `docs/agents/triage-labels.md` | Defines the common label vocabulary used to classify and route repository work. |
| `docs/make/README.md` | Documents every public Make command, its purpose, and representative use cases. |
| `docs/repository-map.md` | Maps the template's non-test files so new users can understand its construction. |
