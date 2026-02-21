# Global Coding Agent Notes

## Context

- Working directory: `/home/owen/Code` (root that contains all cloned OptiGrid repos).
- At the end of substantial tasks, suggest concise AGENTS.md updates when
  recurring user preferences are inferred.

## Workflow Basics

- After first cloning any repo (for example to `/tmp`), run
  `prek install --install-hooks --overwrite` once to install hooks
  idempotently.
- Never use `--no-verify` when committing; fix hook and lint failures before
  creating commits.

## Python repo coding standards

- Use `uv` for Python tooling (`uv run`, `uv add`); avoid system `python`/`pip`.
- Run `prek run --all-files` before finalizing changes.
- Keep IDE-specific config out of git (`.vscode/`, `.idea/`, etc.).
- Use Numpy-style docstrings for public functions; private helpers generally
  don’t need docstrings.
- Prefer testing through public module functions; avoid direct tests of private
  helpers unless unavoidable.
- Expand public docstrings when return-value semantics or side effects are not
  obvious from names/type hints alone.
- Prefer flat functional structure; avoid nested function definitions unless
  they clearly improve readability.
- Prefer self-documenting code; keep comments minimal and only for non-obvious
  tradeoffs/workarounds.
- If code is unavoidably opaque, unusual, or appears over-complicated due to a
  non-obvious constraint (especially a third-party dependency workaround), add
  a concise comment explaining why it is in that state.
- Avoid superfluous code (unused config, unused return values, dead branches,
  and one-line/single-statement wrapper functions without clear semantic
  value); extra code has ongoing maintenance cost.
- Avoid unnecessary abstractions; prefer simple data structures/functions
  unless a wrapper/class has clear reuse or safety value.
- Minimise explicit conditional branches where practical; each additional
  branch increases test and maintenance burden for branch coverage.
- Do not defensively handle exceptions or error states that invalidate the
  system’s core assumptions (for example required directories/resources
  missing); let these errors bubble up so failures are visible via service
  health and Sentry.
- Keep typing strict and explicit; avoid dynamic/loosely-typed patterns where possible.
- Keep type signatures narrow where practical; minimise union type hints and
  prefer non-nullable types unless nullability is genuinely required by the
  domain.
- Prefer semantically explicit names in local context.
- Prefer stable, searchable constant names for log events.
- Prefer idiomatic naming and implementation patterns of the language/library in use.
- Keep Python module filenames unique across a repo (avoid repeated generic
  names like multiple `utils.py`).
- Never add `__init__.py` files under test directories.
- Mirror `src/` structure under `tests/`, and keep test module naming aligned
  with source modules.
- Prefer higher-value integration/system tests for core flows; maintain strong
  coverage (target around 95%+ where practical).
- Keep PRs small and single-purpose; split unrelated refactors/bugfixes/features.

## Terraform/IaC coding standards

- Prefer explicit module inputs for operationally important settings; avoid
  implicit defaults for critical runtime config (for example service endpoints,
  archive buckets, credential references).
- During `terragrunt apply`/`terraform apply`, record the apply completion
  timestamp and then check affected service log streams to confirm post-deploy
  logs are green.
- Prefer fully data-driven Terraform patterns over hybrid implementations.
- Minimize configuration surface area; remove redundant variables once a
  canonical object exists.
- Favor "default or full replace" configuration behavior over partial
  override/merge logic unless there is a strong requirement.
- Don’t preserve backward compatibility for brand-new, not-yet-adopted features.
- Optimize for the module consumer’s mental model first; behavior should be
  obvious and predictable.
- Design modules under `modules/` for reuse across services; avoid
  service-specific assumptions unless truly required.
- Make defaults explicit and easy to discover via module inputs and documentation.
- For risky infrastructure changes, roll out incrementally with explicit
  checkpoints and validation.
- Remove temporary migration scaffolding (for example `moved` blocks and
  transitional flags) after successful apply/plan verification.

## Copilot code review (non-interactive)

- Prefer non-interactive mode for reliable scripted output:
  - `copilot -p "/review" --allow-all-tools --allow-all-paths --allow-all-urls -s`
- If you want a custom scope, use an explicit prompt:
  - `copilot -p "Review this branch for actionable issues only."`
    `--allow-all-tools --allow-all-paths --allow-all-urls -s`
- For PR-focused checks, review only current branch diff against `main` in the prompt.
- Return only actionable findings; if none, report exactly `No issues found`.
- `copilot /review` can take many minutes (sometimes with little or no
  intermediate output), so allow extra time before assuming it is stalled.

## Codex CLI code review (non-interactive)

- Prefer the built-in non-interactive review command:
  - `codex review --base main "Return actionable findings only; say 'No issues
    found' if none."`
- Review local working tree changes (staged/unstaged/untracked):
  - `codex review --uncommitted "Return actionable findings only; say 'No
    issues found' if none."`
- Review a specific commit:
  - `codex review --commit <sha> "Return actionable findings only; say 'No
    issues found' if none."`
- `codex review` can take many minutes with little intermediate output, so
  allow extra time before assuming it is stalled.
