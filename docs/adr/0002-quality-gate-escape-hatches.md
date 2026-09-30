# Quality Gate escape hatches

Under a single pre-commit Entrypoint, a custom `SKIP_HOOKS=1` env var that only the Shell Quality Gate script honored would silently leave File Hygiene and Doc Quality Gates running — agents and humans would think hooks were off when they were not.

We adopt **two explicit levels**, both standard:

1. **Full Gate Bypass** — `git commit --no-verify` skips the entire Quality Gate.
2. **Selective Hook Skip** — `SKIP=<hook-id>` skips only named hooks (e.g. shell only).

`SKIP_HOOKS=1` is retired from the documented contract (and should be removed from the shell script / AGENTS.md when implemented) so it cannot lie.

## Considered options

- **Two standard levels (chosen)** — no custom wrapper; honest semantics; small doc/agent migration.
- **Keep `SKIP_HOOKS=1` = shell only** — least code change; high confusion risk.
- **Custom wrapper so `SKIP_HOOKS=1` skips everything** — preserves old muscle memory; reintroduces a non-framework Entrypoint path.
- **Framework-only docs, drop any project-specific story** — same as chosen, but without naming Full vs Selective in the domain language.
