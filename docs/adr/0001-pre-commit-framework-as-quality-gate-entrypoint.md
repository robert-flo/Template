# pre-commit framework as Quality Gate entrypoint

The repository had two complementary but unintegrated pre-commit halves: a custom Shell Quality Gate (`.git-hooks/pre-commit`) and a `pre-commit` framework config (file hygiene + markdown). We decided the **pre-commit framework is the sole Git Entrypoint** for the Quality Gate; the Shell Quality Gate is integrated *into* that pipeline rather than competing via `core.hooksPath` or a second installed hook.

## Considered options

- **Framework-first (chosen)** — one `pre-commit install`, same tool for local commits and `pre-commit run --all-files` / CI; custom shell logic remains a local hook implementation.
- **hooksPath-first** — full control of the shell script as Git owner, calling the framework from inside; requires remembering `core.hooksPath` on every clone/worktree.
- **Shell-only locally, docs/hygiene in CI** — simpler laptop path, weaker “blocked at commit” guarantee for non-shell domains.
- **Parallel unintegrated systems** — status quo; hooks overwrite or exclude each other.

## Consequences

- Escape hatches, bootstrap, and docs must be described in framework terms (plus any project-specific shell overrides), not as two independent hook install paths.
- The custom shell script remains valuable for project-specific behavior; it is not replaced by the framework, only *invoked under* it.
