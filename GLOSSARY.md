# Domain Glossary — Bash Project Template

Domain language for the Bash project template repository — quality gates, development hygiene, and agent workflow vocabulary that are specific to this project.

## Language

### Commit quality

**Quality Gate**:
The full set of automated checks that must pass before a commit is accepted, owned by a single Git entrypoint.
_Avoid_: pre-commit hooks (ambiguous), linters, CI (CI may re-run the same gate but is not the gate itself)

**Shell Quality Gate**:
The shell-specific portion of the Quality Gate: format, lint, and shell quality rules for staged shell files only.
_Avoid_: shell hook, bash lint (when referring to the whole gate), community shell hooks (as the primary implementation), exclusion allowlists for “legacy” shell

**Shell Failure Report**:
A timestamped, AI-oriented artifact produced when the Shell Quality Gate fails lint, with a path printed for humans and agents.
_Avoid_: hidden side-channel reports outside the working tree

**File Hygiene Gate**:
The non-shell portion of the Quality Gate for generic repository hygiene (size, conflict markers, symlinks, structured-file validity, whitespace, EOF).
_Avoid_: generic hooks, basic checks

**Doc Quality Gate**:
The Markdown portion of the Quality Gate for first-party project documentation that Git can stage.
_Avoid_: markdown hook (when referring to the whole doc policy), linting third-party or local agent skills

**Local Agent Tree**:
Developer-local agent skills and related files, ignored by Git and outside product source.
_Avoid_: project docs, tracked skills vendor directory

**Strict Doc Profile**:
The stricter Markdown rule set used by the Doc Quality Gate on commit, with only intentional project-convention exceptions.
_Avoid_: full default markdownlint (implies zero disables), lax profile (not the commit gate)

**Entrypoint**:
The single Git pre-commit hook installation that owns the Quality Gate; no second parallel hook owner.
_Avoid_: hooksPath dual setup, dual pre-commit systems

**Gate Bootstrap**:
The explicit install path that activates the Entrypoint in a clone or worktree and verifies required host tools without installing packages.
_Avoid_: “just run pre-commit install” as the only onboarding story, auto-install of system packages from bootstrap

**Full Gate Bypass**:
Emergency skip of the entire Quality Gate for one commit via Git’s native no-verify path.
_Avoid_: SKIP_HOOKS=1 (retired; previously implied “all hooks” but only affected shell)

**Selective Hook Skip**:
Skipping one or more named hooks inside the Quality Gate while leaving the rest active.
_Avoid_: SKIP_HOOKS (ambiguous), partial bypass
