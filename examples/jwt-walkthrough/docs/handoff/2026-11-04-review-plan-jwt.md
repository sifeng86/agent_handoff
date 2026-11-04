# Review: plan-jwt

Reviewer: gpt-6-astra@codex/host-2
Reviewed: docs/handoff/2026-11-04-plan-jwt.md @ <commit sha>

## Findings
| # | Severity | Finding | Evidence |
|---|---|---|---|
| R1 | major | Token TTL is not defined; implementer would have to guess. | plan §3 |
| R2 | major | 401 response body format not specified; existing API uses `{error: {code, message}}`. | src/api/errors.ts:12 |
| R3 | minor | "Protected routes" not listed. | plan §2 |

## Verdict
changes-requested
