# AI interaction guidelines

## Communication

- Be concise and direct.
- Explain non-obvious design trade-offs briefly (useful for senior-interview practice).
- Ask before large refactors, new infrastructure, or pulling in deferred tech.
- Do not add features outside [`project-overview.md`](project-overview.md) / the active feature spec.
- Never delete files or chapter trees without clarification.

## Source of truth

1. [`project-overview.md`](project-overview.md) — stack, phases, deferred/out of scope  
2. [`coding-standards.md`](coding-standards.md) — how to write code in the modern project  
3. [`current-feature.md`](current-feature.md) — what is in progress now  
4. `features/`, `fixes/`, `research/` — on-demand detail  

Root [`AGENTS.md`](../AGENTS.md) summarizes repo layout and book chapter map.

## Workflow (feature / phase / fix)

1. **Document** — Capture goals in `current-feature.md` (and a `features/*.md` or `fixes/*.md` spec if non-trivial).
2. **Scope folder** — Modern root once it exists; otherwise the named `ChapterNN` only.
3. **Implement** — Minimal changes that match the overview + standards.
4. **Verify** — Build/tests in that folder (`./gradlew test` / `gradlew.bat test`); Compose checks when relevant.
5. **Iterate** — Fix failures before expanding scope.
6. **Commit** — Only when the owner asks; focused commits; no secret files.
7. **Close out** — Mark `current-feature.md` completed and append History.

## Branching (when using git branches)

- Prefer `feature/<name>` or `fix/<name>` for modern work.
- Do not rewrite all book chapters on one branch unless explicitly requested.

## When stuck

- After 2–3 failed attempts, stop, explain evidence, and ask.
- Do not randomly swap infrastructure (DB, broker, gateway) to “try something else.”

## Code changes

- Smallest change that completes the current feature.
- No drive-by refactors or unrelated docs.
- Preserve book chapter samples; put modern design in the modern root + `context/`.
