# Audit Response: agent_handoff

Responding to: docs/handoff/2026-11-04-audit-agent-handoff.md (baseline 7630c8c)
Decided by: claude-opus-5-5 (project lead, per user)

| Finding | Decision | Resolution |
|---|---|---|
| F1 read-only roles vs state | accepted | PROTOCOL §1: handoff files are always writable; reviewer/auditor role.md reference it. |
| F2 reviewer routing vs Pipeline | accepted | PROTOCOL §5 transition table: approve → next step; changes-requested → author's step unless author is next; third round → user. |
| F3 wait script uses history | accepted | PROTOCOL §11 checks upstream `state.md` Status/Current; exits 1 on fetch/upstream error, 2 on pause. |
| F4 reinstall overwrites | accepted | README install refuses when `.handoff/` exists; separate Update section (only PROTOCOL.md is template-owned). |
| F5 existing-project install misses override | accepted | Override moved into PROTOCOL §1; AGENTS.md block delimited by `handoff:begin/end` and appended whole. |
| F6 no cursor for repeated roles / push-back | accepted | `Step` and `Return-To` fields; PROTOCOL §6 defines detours (push-back, ad-hoc roles) that never advance `Step`. |
| F7 small fix has no spec | accepted | implementer role.md and README: user instruction is the spec if it states scope, behavior, verification; else push back to user. |
| F8 trailers not parseable | accepted | Examples fixed; PROTOCOL §5 requires no blank line before Co-Authored-By. |
| Obs: done transition | accepted | Covered by §5 table. |
| Obs: example inconsistencies | accepted | `/api/health` unified; spec §7 references fixed. |

## Verification (isolated repos in scratchpad)
- `git interpret-trailers --parse`: 9/9 example relay commits yield `Relay-To`.
- Wait script: superseded relay does not wake reviewer; already-pulled handoff wakes implementer; pause → exit 2; fetch failure → exit 1.
- Install: fresh target OK; existing AGENTS.md/CLAUDE.md get the full block and `@AGENTS.md`; rerun refuses and keeps `Status: in-progress`.

## Not verified
- Real cross-provider/account handoff (still the next step).
