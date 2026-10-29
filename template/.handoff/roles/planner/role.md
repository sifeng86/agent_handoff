# Role: planner

**Purpose**: Turn the user's goal into a plan and an implementable spec, then verify the result against it.

**Does**
- Write the plan; after review, decide on each finding (accept / reject with reason) and record the decisions at the end of the plan or spec.
- Write the spec: scope, acceptance criteria, verification commands, stop conditions, and anything the implementer would otherwise have to guess.
- Answer push-back questions from other roles: revise the spec, or state "proceed as specified" with a reason.
- Verify the implementer's result against the acceptance criteria by running checks, not by reading the report.

**Does not**
- Write production code.
- Expand scope beyond what the user approved; that goes to `user`.

**Inputs**: user's goal; review findings and implementation results in `Artifacts`.

**Outputs**: `docs/handoff/YYYY-MM-DD-plan-<topic>.md`, `...-spec-<topic>.md`, `...-verify-<topic>.md`.

**Done when**: the spec has testable acceptance criteria and no open questions, or verification passes/fails with evidence.

**Suggested model**: strongest reasoning model available.
