# AGENTS.md — Bash Project Template

## Table of Contents

1. [Repository Structure & Purpose](#repository-structure--purpose)
2. [Style](#style)
3. [Error Handling](#error-handling)
4. [ShellCheck & Scripting Safety](#shellcheck--scripting-safety)
5. [Verification](#verification)
6. [Git Worktree Workflow (Development)](#git-worktree-workflow-development)
7. [Branching & Release Policy](#branching--release-policy)
8. [Task Planning & Skills Workflow (Matt Pocock Skills)](#task-planning--skills-workflow-matt-pocock-skills)
9. [Task Execution Workflow](#task-execution-workflow)

---

## Agent skills

### Issue tracker

GitHub Issues via `gh` CLI. External PRs are not triaged. See `docs/agents/issue-tracker.md`.
When creating or renaming an issue, prefix its title with the issue number and a
separator: `<number> - <descriptive title>` (for example, `51 - Migrate legacy
Codex CLI task to canonical mise lifecycle`).

### Triage labels

Default vocabulary (needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context repo (`GLOSSARY.md` + `docs/adr/` at repo root). See `docs/agents/domain.md`.

### Pull request history

Recent squashed PR history lives in `.github/pr-history.md` (updated at release /
squash time). When researching past decisions or why a change was made:

1. Check `.github/pr-history.md` first — each entry has the squash commit hash,
   PR number, title, and full body with context.
2. If more detail is needed, use `gh pr view <number>` to read the full review
   thread, or `git log -S <symbol>` to trace code churn.

### Embedded code by the User

This convention applies exclusively when the User explicitly points to new code
or untracked files they just added to the repository. In that case, assume the
implementation is already functional, has been tested in daily use, and is ready
to be integrated. Work should start from it and preserve its design, architecture,
visual language, and recognizable behavior.

- `to-tickets <path>` requests integrating the pointed-out code. Incremental
  improvements are allowed, but the essence of the implementation must be
  preserved and the integration must not become a rewrite.
- `grill-with-docs <path>` requests evaluating a refactoring of the pointed-out
  code via a `grill-me` session. Do not implement it until an explicit agreement
  on scope, decisions, and validation points is reached with the User.
- Apply improvements incrementally and atomically, preserving the functional
  state first.
- Do not turn an integration into a rewrite without explicit authorization.
- Outside this scope, `to-tickets` continues to be the normal expected behavior
  of the repository.

## Repository Structure & Purpose

- `make/` - Modular Makefile system organizing operational targets by domain:
  - `aliases.mk` - Convenient aliases and helper shortcuts.
  - `docker.mk` - Container build and run automation.
  - `git.mk` - Git workflow targets, worktree operations, and repository hygiene.
  - `hooks.mk` - Git hooks management and local hook installation.
  - `quality.mk` - Quality gate targets (formatting, linting, and verification).
  - `release.mk` - Release preparation and versioning integration.
- `tests/` - Automated test suite running behavioral contracts and unit checks for scripts and templates.
- `docs/` - Architecture decision records (`docs/adr/`), domain glossary (`GLOSSARY.md`), and agent operational guidelines (`docs/agents/`: `issue-tracker.md`, `triage-labels.md`, `domain.md`).
- `.git-hooks/` - Local Git hooks, including the pre-commit quality gate (shfmt and shellcheck enforcement).
- `.github/` - GitHub Actions CI/CD workflows, issue templates, and pull request templates.
- `Dockerfile` & `dockerfile.sh` - Reproducible, containerized execution baseline for development and verification.
- `release-please-config.json` & `.release-please-manifest.json` - Automated changelog generation and semantic release management via Conventional Commits.

## Modular Helpers & Script Organization

> [!IMPORTANT]
> **Code Reuse Over Duplication**: Before implementing any custom helper logic (such as git operations, logging, spinners, retries, or environment checks), inspect the existing helper modules within the repository and reuse them instead of writing one-off shell routines.

When creating or modifying shell scripts, maintain clean separation of concerns:

- **Semantic Logging:** Provide clear, structured feedback for informational, warning, and error states rather than raw `echo` statements.
- **Robustness & Retries:** For commands susceptible to transient network or I/O failures, implement structured retry logic.
- **Pure Functions:** Prefer isolated, deterministic helper functions that take input as arguments and return predictable output or exit codes.

## Style

- Two spaces for indentation, no tabs.
- Use bash 5 conditionals: use `[[ ]]` for string/file tests and `(( ))` for numeric tests.
- In `[[ ]]`, don't quote variables, but do quote string literals when comparing values (e.g., `[[ $branch == "dev" ]]`).
  > Note: this applies only inside `[[ ]]`. For command arguments outside `[[ ]]`, see the SC2086 rule in "ShellCheck & Scripting Safety" — there, quoting is always required.
- Prefer `(( ))` over numeric operators inside `[[ ]]` (e.g., `(( count < 50 ))`, not `[[ $count -lt 50 ]]`).
- For strings/paths with spaces, quote them instead of escaping spaces with a backslash (e.g., `"$APP_DIR/Disk Usage.desktop"`, not an unquoted path with escaped spaces).
- Shebangs:
  - Standard bash scripts must use `#!/usr/bin/env bash` consistently.
  - Scripts intentionally designed for POSIX compliance should use `#!/usr/bin/env sh` or `#!/bin/sh`.
- **`local` in functions**: every function-scoped variable must be declared with `local`. When the value comes from a command, declare the empty variable first and assign it on a separate line — never `local var=$(cmd)` on a single line, since it masks the command's exit code (SC2155). Correct example:

  ```bash
  local key_id=""
  key_id=$(gpg --list-secret-keys ... || true)
  ```

- **Naming**:
  - Variables and functions: `snake_case`.
  - Read-only constants (colors, icons, script-level config): `UPPER_SNAKE_CASE` + `readonly`.
  - Functions: prefixed by verb/category based on responsibility, not by source file (e.g., `print_*`, `verify_*`, `setup_*`, `configure_*`, `get_*`, `do_*`).

## Error Handling

- For scripts with strict error handling (`set -e` / `pipefail`), protect pipelines or command substitutions in variable assignments that might return a non-zero exit status (e.g., `grep` queries returning empty results) by appending `|| true` or `|| echo ""` to prevent premature shell termination.

## ShellCheck & Scripting Safety

- **Zero-Warning Policy**: All new or modified shell scripts must pass `shellcheck` with zero warnings or errors before committing. The **Shell Quality Gate** (`shfmt` + `shellcheck` via the pre-commit Entrypoint) blocks the commit if any warnings are found. This rule is non-negotiable.
- **No path exclusion allowlist**: every staged shell file (`*.sh` or shell shebang) is in scope. There is no transitional “legacy skip” list in the gate.
- **Direct Command Checks (SC2181/SC2319)**: Avoid checking `$?` indirectly (e.g., `if [ $? -eq 0 ]`). Check commands directly (e.g., `if my_command; then`) or use success tracking variables (`success=0; my_command || success=1; if (( success == 0 )); then`).
- **Quote Variable Expansions (SC2086)**: Always double-quote variable expansions when they are used as command arguments to prevent word splitting (e.g., `"$var"`), except inside `[[ ]]` where expansion is safe.
- **Built-in Parameter Expansion (SC2001)**: Avoid calling external tools like `sed` or `awk` for simple string replacements on single variables; prefer built-in Bash parameter expansion (e.g., `${var//search/replace}`).
- **Localizing False Positives**: Do not ignore entire files for linter warnings. Use inline `# shellcheck disable=SCxxxx` directives only on the specific lines where a false positive occurs (e.g., AWK variables inside single quotes).

## Verification

### Quality Gate (automatic on commit)

The **Quality Gate** is owned by the **pre-commit framework** as the sole Git **Entrypoint** (do not set a competing `core.hooksPath`). It runs three domains on commit:

| Domain                 | What it does                                                                                    |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| **File Hygiene Gate**  | large files, merge conflict markers, symlinks, structured-file checks, trailing whitespace, EOF |
| **Doc Quality Gate**   | Strict Doc Profile via framework-managed `markdownlint-cli2` (not Docker)                       |
| **Shell Quality Gate** | Local hook `.git-hooks/ravn-shell-quality`: staged shell only, `shfmt` then `shellcheck`        |

Shell details:

- **shfmt**: auto-fixes in place and re-stages. Flags: `shfmt -i 2 -sr -kp -ci -w`.
- **shellcheck**: zero warnings. On failure, writes a **Shell Failure Report** at `logs/shellcheck-report-<timestamp>.log` in the current worktree (path is printed; `logs/` is gitignored).
- Partial-stage shell files are refused when unstaged hunks are visible (direct script runs); under pre-commit, unstaged changes are stashed before hooks so format cannot expand the commit boundary.

### Gate Bootstrap

Configure the local Quality Gate in a cloned repository (verify tools on
`PATH` and install the Entrypoint):

```bash
make repository-bootstrap
```

Required host tools: `pre-commit`, `shfmt`, `shellcheck`. On Arch, for example: `sudo pacman -S pre-commit shfmt shellcheck`.

Use `make hooks-install` when only the local Quality Gate Entrypoint needs
refreshing. Maintainers configuring the canonical GitHub repository run
`make repository-bootstrap CONFIGURE_REMOTE=1` to synchronize changelog
labels and replace default-branch protection. `make git-setup REPO=...` calls
the local bootstrap after creating worktrees when the repo ships `make/hooks.mk`
and `.pre-commit-config.yaml`.

### Escape hatches

| Level                   | Mechanism                | Effect                                                  |
| ----------------------- | ------------------------ | ------------------------------------------------------- |
| **Full Gate Bypass**    | `git commit --no-verify` | Skips the entire Quality Gate for that commit           |
| **Selective Hook Skip** | `SKIP=<hook-id>[,...]`   | Skips only named hooks (e.g. `SKIP=ravn-shell-quality`) |

Do **not** use `SKIP_HOOKS=1` — it is retired and is not part of the contract.

### Manual (full-repo audit)

To audit the repository beyond staged-only commits, e.g. before a release:

```bash
# Full non-mutating quality check (File Hygiene + Doc Quality + Shell Quality + Tests)
make verify

# Or run non-mutating lint checks only:
make lint

# Or run all pre-commit hooks across the repository:
pre-commit run --all-files

# Or apply repository formatting rules:
make format
```

## Git Worktree Workflow (Development)

### Issue completion cleanup

After an issue or pull request is merged successfully into the base branch
from which its worktree was created, the agent must verify that the merge is
confirmed and the issue/PR is closed. The agent then automatically removes the
obsolete local and remote topic branch and removes the local issue worktree to
keep the repository clean.

Strict safety constraints:

- Do not perform cleanup after a failed, partial, or unverified merge.
- Never delete `master`, the base integration branch, or its worktree.
- Report all removed branches and worktrees in the final handoff summary.

To protect the user's active development tree from accidental resets or uncommitted code loss during development, and to maintain strict task isolation:

- **Isolated Development in `~/Work/`**: All active development work must be carried out inside isolated worktrees under `~/Work/<repo>/` (which are created from the bare repository at `~/.local/share/git-bare/<repo>`).
- **No Direct Commits in Base Clones**: Do not perform feature development or commit changes directly inside the base repository worktree.

- **Mandatory branch-baseline preflight**: Before creating a normal feature or chore branch from the base integration branch (e.g., `dev` or `master`), first run `git fetch origin --prune` and verify `git merge-base --is-ancestor origin/master origin/<base>`. A zero exit status means the released history is already present in the base branch. If it fails, update the base branch with `git merge --ff-only origin/master` from a clean base worktree, push it, fetch again, and repeat the ancestry check. This advances the branch reference without creating a direct commit or merge commit. If fast-forward is impossible, do not force it or derive new work: use an isolated synchronization branch and PR to resolve the divergence first. This prevents features from missing released behavior.

- **Automation Utilities**: the worktree helpers must be available in `PATH`.
  - **MANDATORY**: use `git-create-worktree` for general feature/chore branches.
  - **MANDATORY**: use `git-issue-worktree` for GitHub-tracked issues.
  - Do not create worktrees with raw `git worktree` commands or other ad-hoc procedures.
  - > [!IMPORTANT]
    > **MANDATORY**: `git-bare-clone` must always be used to create bare repositories (whether invoked via a `make` target or manually) — never create a bare repo with raw `git` commands. This is a recurring compliance gap: agents have created bare repos manually instead of using this script.
- **Workflow Benefit**: Developing under `~/Work/` ensures complete isolation between tasks, allowing concurrent development branches without untracked artifact contamination or uncommitted work collisions.

## Branching & Release Policy

All changes must be created on temporary topic branches in isolated worktrees. **The following rules are non-negotiable and must be strictly followed by all agents and developers:**

- **Temporary topic branches**: Create every change on a dedicated branch using `git-create-worktree` or `git-issue-worktree`; push it to the remote and open a pull request.
- **`master`**: The integration branch. It must never receive direct commits. Merge all changes into `master` only through pull requests from remote topic branches.
- **Merging**: Before merging, rebase the topic branch onto `origin/master`; do not merge `master` into the topic branch. After rebasing, update the remote branch only with `git push --force-with-lease`.

> [!WARNING]
> **This policy is currently enforced by convention only, not by tooling.** The pre-commit hook (see "Verification") checks formatting/linting, but does **not** currently block direct commits to `master`. Until branch protection is implemented (at the hook level or via the git host), agents and developers must self-enforce this policy manually — treat it as strictly as if it were technically blocked.
>
> [!TIP]
> **GitHub CLI (`gh`)**: always prefer `gh` for GitHub operations (issues, PRs, releases, repo metadata). **Do not use `gh repo sync`** — it can overwrite local changes, discard commits, and bypass the worktree isolation workflow. Use `git fetch` + `git rebase` instead for keeping branches up to date.

## Task Planning & Skills Workflow (Matt Pocock Skills)

This repository mandates [Matt Pocock's engineering skills](https://github.com/mattpocock/skills) for planning and reviewing work. `/setup-matt-pocock-skills` has already been run — the issue tracker, triage label vocabulary, and domain doc layout it configures live at `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`, and `docs/agents/domain.md`.

> [!NOTE]
> Unsure which skill fits a given situation? Run `/ask-matt` — it's the router over all installed skills.

### Task Sizing (before choosing a path)

> This classification is a local convention for this repository — it does not come from the Matt Pocock skills repo itself, which branches on "single-session vs. multi-session" rather than "trivial vs. engineering." It is layered on top of the official flow below, not a replacement for it. Do not confuse this with the `/triage` skill (below), which is a different thing: `/triage` moves _incoming_ issues/PRs through a bug/enhancement state machine — it has nothing to do with sizing a task you're about to start.

1. **Trivial / Administrative Tasks** — simple config changes (e.g., `.gitignore`, env var templates), doc typo fixes, minor dependency bumps.
   - **Fast-Track**: skip straight to `/implement`, then close with `/code-review`.
2. **Engineering Tasks** — anything that alters, adds, or removes business logic, scripts, build targets, libraries, behavioral contracts, or architecture.
   - **Full Pipeline**: execute the 5-step chain below, sequentially, without exceptions.

> [!IMPORTANT]
> **Classify out loud, don't decide silently.** `/grill-with-docs` has `disable-model-invocation: true` by design — Matt Pocock deliberately reserved the decision to start an interview for the human, not the agent. Silently classifying a task as Trivial and jumping straight to `/implement` overrides that design choice. Instead:
>
> 1. Classify the task using the criteria above.
> 2. **State the classification and the resulting path before writing any code** — e.g., _"Classifying this as Trivial — going straight to `/implement`. Say so if you'd rather start with `/grill-with-docs`."_ This gives the user a cheap veto before work begins, not after.
> 3. **When in doubt, default to the Full Pipeline**, not the shortcut — consistent with "Strict Operational Rules" § Anti-Rationalization below. The Fast-Track is for genuinely unambiguous, low-risk edits; if there's any real design or domain ambiguity, that ambiguity is exactly what `/grill-with-docs` exists to resolve.

### The Main Build Chain

```txt
/grill-with-docs → /to-spec → /to-tickets → /implement → /code-review
```

This is the official main flow of the Matt Pocock skills (per `ask-matt`'s routing logic). Each step is **user-invoked only** (the agent does not reach for these on its own) — but per "Strict Operational Rules" below, once a task is classified as an Engineering Task, the agent must drive this chain itself in sequence rather than waiting to be prompted step by step.

| Step | Command            | Purpose                                                                                                                                    | Exit Gate                                           |
| :--: | :----------------- | :----------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------- |
|  1   | `/grill-with-docs` | A relentless interview to sharpen a plan or design, writing resolved terms to `GLOSSARY.md` and hard decisions as ADRs as it goes.          | Design ambiguities resolved; glossary/ADRs updated. |
|  2   | `/to-spec`         | Synthesize the current conversation into a spec (no re-interviewing) and publish it to the issue tracker with the `ready-for-agent` label. | Spec published to the tracker.                      |
|  3   | `/to-tickets`      | Break the spec/conversation into tracer-bullet tickets, each declaring its blocking edges, published to the tracker.                       | Atomic ticket set with blocking edges published.    |
|  4   | `/implement`       | Implement a piece of work from a spec or ticket, driving `/tdd` internally at agreed seams. Runs typechecking and tests regularly.         | Working code, tests passing. No "vibe coding."      |
|  5   | `/code-review`     | Two-axis parallel review of the diff since a fixed point — Standards (repo conventions) and Spec (does it match the ticket/PRD).           | Side-by-side Standards vs. Spec report.             |

> [!IMPORTANT]
> **Standard review framing**: every invocation of `/code-review` in this repository must be framed with this literal instruction:
>
> ```text
> Review this repository as if you are blocking or approving a production PR.
> ```
>
> This framing is mandatory and non-negotiable — it consistently produces a stricter, higher-signal review than a neutral "review this" prompt, and it is what's used everywhere `/code-review` is invoked in this repo (including "Task Execution Workflow" § Phase 4).

<!-- -->

> [!NOTE]
> **Context hygiene** (per `ask-matt`): keep steps 1–3 in one unbroken context window — don't `/compact` or clear context until after `/to-tickets`, so grilling, spec, and tickets build on the same reasoning. Each `/implement` then starts fresh from the ticket. If a session approaches the model's effective reasoning window before `/to-tickets` is done, use `/handoff` rather than pushing on with degraded context.

### Decision Tree

```txt
                              New Request
                                   │
                                   ▼
                    Classify: Trivial or Engineering?
                    (see "Task Sizing" criteria)
                                   │
              ┌────────────────────┴────────────────────┐
              ▼                                          ▼
          TRIVIAL                                   ENGINEERING
              │                                          │
              ▼                                          ▼
   State classification +                     State classification +
   path out loud. Give user                   path out loud. Give user
   a cheap veto before                        a cheap veto before
   starting.                                  starting.
              │                                          │
              │ (no objection)                           │ (no objection)
              ▼                                          ▼
        /implement                              /grill-with-docs
              │                                          │
              │                            ┌─────────────┴─────────────┐
              │                            ▼                           │
              │                   Design ambiguity                     │
              │                   still unresolved?                    │
              │                            │                           │
              │                    YES ────┘                          NO
              │                     │                                  │
              │              Keep grilling                             ▼
              │              (same context                        /to-spec
              │               window — no                              │
              │               /compact yet)                            ▼
              │                                          Multi-session build?
              │                                       (per ask-matt, NOT "how
              │                                        many files touched")
              │                                                        │
              │                                        ┌───────────────┴───────────────┐
              │                                        ▼                               ▼
              │                                       NO                              YES
              │                                        │                               │
              │                                        ▼                               ▼
              │                                  /implement                     /to-tickets
              │                                  (same context                        │
              │                                   window)                    /implement per ticket
              │                                        │                    (fresh context each,
              │                                        │                     via /handoff if needed)
              └────────────────────┬───────────────────┴───────────────────────────────┘
                              ▼
                       /code-review
                          (Standards + Spec)
                                    │
                                    ▼
                       Commit → push → PR into `master`
                    (never direct commits — see
                     "Branching & Release Policy")
```

### On-ramps

- **Incoming bugs/requests not created by you** → `/triage` first (moves them through categorize → verify → grill → agent-ready brief), which then feeds `/implement`. Tickets that `/to-tickets` already produced are already agent-ready — do not re-triage them.
- **A hard, resistant bug** (intermittent, regressed between known-good states) → `/diagnosing-bugs` instead of guessing.

### Strict Operational Rules & Anti-Rationalization

- **No Parallel Paths**: The Matt Pocock skills chain is not a suggestion or one option among several — it is _the_ workflow for this repository. The agent must not invent, improvise, or substitute its own ad-hoc planning process (its own informal "let me think through this" sequence, a custom checklist, a different ordering of steps) in place of `/grill-with-docs → /to-spec → /to-tickets → /implement → /code-review`. If a task is an Engineering Task, the path is already decided — the agent's job is to walk it, not to design an alternative.
- **Internal by Default**: The agent must drive this chain itself, internally, as its own default operating procedure — not as something it only does when explicitly asked to "use the skills" or "follow Matt Pocock's workflow." Treat every applicable task as if the chain were already silently invoked the moment work begins, the same way "Style" or "ShellCheck & Scripting Safety" apply without needing to be requested.
- **Absolute Sequentiality**: Never run `/implement` for an Engineering Task unless a valid spec (`/to-spec`) and broken-down tickets (`/to-tickets`) already exist to back it up.
- **No Skipping Under Pressure**: Time pressure, an urgent tone from the user, a "just do it quickly," or the agent's own confidence that it "already understands the task" are not valid reasons to bypass a step. If a step feels unnecessary, that feeling is itself the signal to check with the user (per "Task Sizing") rather than to quietly skip it.
- **Verification is Non-Negotiable**: Do not claim a task is complete based on intuition. `/implement` and `/code-review` exit gates require deterministic proof (passing tests, successful builds, explicit terminal confirmation) — see "Verification" for the repository's lint and verification commands (`make verify`) that back this up.
- **The Socratic Mandate**: During `/grill-with-docs`, do not be agreeable. Actively find flaws, missing requirements, and architectural conflicts _before_ a single line of production code is written.

### How this fits with the template workflow

`/implement` (step 4) is where the generic Pocock flow meets repository-specific mechanics. "Task Execution Workflow" (below) is the detailed, repo-specific process — worktree isolation, modular development, automated contracts, and containerized verification — that governs _how_ `/implement` and the surrounding steps actually get carried out in this repository. Read the two together: this section decides _which skills to invoke and when_; "Task Execution Workflow" defines _what happens inside each phase_ for this template.

## Task Execution Workflow

> [!IMPORTANT]
> **MANDATORY & NON-NEGOTIABLE**: For Engineering Tasks (see "Task Planning & Skills Workflow" § Task Sizing), these phases describe what happens specifically during and around `/implement`. Whenever a new task or feature is requested, the agent must execute these phases in strict sequence. No phase may be skipped. Each phase cross-references the governing section that defines its rules — do not restate or bypass those rules.

### Phase 1 — Environment & Task Setup

1. **Create an isolated issue worktree** whenever the user activates `/implement` for a GitHub issue. Use `git-issue-worktree` with the issue number, a descriptive slug, the active repository path, and the current base branch (`master`). For chores or tasks without GitHub issues, use `git-create-worktree`.
   - Do not replace `git-issue-worktree` with a raw `git worktree` command or an absolute path to the helper.
   - Record the base branch before implementation begins; that exact branch is the merge target later.
   - Do not implement or commit changes directly in the base worktree.
2. **Determine the best implementation approach** for the requested task. Decide on the proper architectural seam (e.g., helper function, modular script, Makefile target under `make/`, or test contract under `tests/`).

### Phase 2 — Implementation & Engineering Standards

1. **Follow the code and styling contracts**: adhere strictly to Bash 5 standards, strict variable quoting, separation of concerns, and pure functions (see "Style" and "Modular Helpers & Script Organization").
2. **Handle errors defensively**: ensure commands in pipelines or substitutions that may return non-zero exit codes are protected with `|| true` or `|| echo ""` when running under strict shell options (`set -e` / `pipefail`).

### Phase 3 — Testing & Validation

1. **Run lint and hygiene checks** on all touched files: execute `make lint` (or run `shellcheck` and `shfmt -i 2 -sr -kp -ci -d` on modified shell files). Ensure zero warnings or errors.
2. **Run automated behavioral contracts**: execute `make test` to verify that existing and newly added behavioral tests pass in `tests/`.
3. **Run full verification baseline**: execute `make verify` and, if applicable, validate the reproducible container build via `make docker-build`.

### Phase 4 — Merge, Cleanup & Handoff

1. **Run `/code-review`** against the fixed point where the worktree branched off, using the standard review framing (see "Task Planning & Skills Workflow" § The Main Build Chain) — Standards + Spec review. Resolve any Blocker/Major findings before proceeding.
2. **Commit, push, and open a Pull Request**: format commits using Conventional Commits, push the topic branch to the remote, and open a PR targeting `master` (see "Branching & Release Policy"). `master` must never receive direct commits.
3. **Synchronize the base worktree** after the PR merges, then verify the merged result and its clean status.
4. **Clean up automatically once the merge is confirmed**: verify that the issue/PR is closed on GitHub. Automatically remove the obsolete local and remote topic branch, and delete the isolated worktree to prevent cluttering the repository. Never delete `master`, the base branch, or unmerged work.
5. **Give the user manual validation instructions**: explain the exact commands or steps to test the completed change. Then report that the base branch is ready for the next issue. Do not silently start the next issue in the same handoff.
