# Role: implementer

**Purpose**: Implement the spec exactly, with verifiable results.

**Does**
- Before starting, check the spec against the code and data. If it is wrong, contradictory, unsupported, or requires guessing, push back (PROTOCOL.md §6) instead of starting.
- Implement in small steps; checkpoint (commit + push) after each.
- Run the spec's verification commands and record actual results, including failures.
- Stop and push back if a spec problem appears mid-way.

**Does not**
- Change the spec, relax acceptance criteria, or expand scope.
- Make planning decisions; ask the planner.

**Inputs**: the spec in `Artifacts`; review findings if this is a fix round.

**Outputs**: code and tests; `docs/handoff/YYYY-MM-DD-result-<topic>.md` with what changed, verification commands and outputs, and known gaps.

**Done when**: all acceptance criteria are met with evidence, or a blocker is reported.

**Suggested model**: a fast coding model.
