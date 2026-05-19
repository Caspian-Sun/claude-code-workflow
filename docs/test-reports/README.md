# test-reports Directory

> Holds **test report snapshots** produced by the `/test` command — one file per run, immutable.

## Purpose

Every time `/test` runs (generating new tests, regenerating stale tests, or just rerunning regression), it must drop a report here. These reports close the **Traceability Chain**:

```
PRD → @rules → it() → Test Report (here)
```

The report makes it auditable whether each business rule actually has a green test, **without re-running tests**. Useful for code review, weekly handoffs, retrospectives, and tracking rule-coverage trends over time.

## Naming Convention

```
docs/test-reports/<YYYY-MM-DD-HHmm>-<scope>.md
```

- `<YYYY-MM-DD-HHmm>` — local time the run **started** (24-hour clock)
- `<scope>` — kebab-case, reflects what was tested:
  - Single file → base name (e.g. `search-form`)
  - Module / directory → module name (e.g. `features-list`)
  - Project-wide → `all` or `multi`

Examples:
- `2026-05-19-1430-search-form.md`
- `2026-05-19-1635-features-list.md`
- `2026-05-20-0900-all.md`

## Immutability

- **Never edit** an existing report. Each report is a frozen snapshot of one test run.
- If a rerun produces different results, write a **new** file with a new timestamp.
- Trends emerge naturally from the time-sorted file list.

This matches the philosophy of `docs/retrospectives/`: append-only history is more valuable than a "latest" file that loses signal.

## File Contents

Use [_template.md](_template.md) as the skeleton. Required sections:

1. **Metadata** — date, target (`$ARGUMENTS`), executor, vitest version, total duration
2. **Execution Summary** — total / passed / failed / skipped, plus failing-case details
3. **Business Rule Traceability Matrix** — per source file, `@rules` ↔ `it()` ↔ status. Includes an explicit **Rule Coverage** numeric line.
4. **Changes This Round** — added / modified / removed test cases vs the previous report
5. **Open Issues & Recommendations** — failures triaged by category, uncovered rules, follow-ups

## Relationship to Other Directories

| Directory | Role | Mutability |
|-----------|------|------------|
| `docs/prds/` | Source of `@rules` (business rules) | Editable when requirements change |
| `docs/tasks/` | Task breakdown linked from `@task` | Status field updated; otherwise stable |
| `docs/bug-reports/` | Failures **found by humans / E2E AI** in the running app | Append-only |
| `docs/test-reports/` (here) | Failures / coverage **found by `/test` unit & component tests** | Append-only |
| `docs/retrospectives/` | Framework-level audit by `/meta-audit` | Append-only |

**bug-reports vs test-reports**: bug-reports flow into `/fix` (they describe user-visible defects to be patched). test-reports flow into code review / retrospectives (they describe the state of automated coverage). They do not replace each other.
