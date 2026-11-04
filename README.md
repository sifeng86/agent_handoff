# agent_handoff

A git-only handoff template for sequential work across agents (Claude, Codex, ...), accounts, machines, and humans. No CLI to install: just files and git.

## Install into a project

```bash
cp -r template/.handoff template/docs <project>/
# If <project> has no AGENTS.md / CLAUDE.md:
cp template/AGENTS.md template/CLAUDE.md <project>/
# Otherwise append the pointer line from template/AGENTS.md to the existing AGENTS.md,
# and make sure CLAUDE.md contains "@AGENTS.md" (or the same pointer line).
```

Then edit `.handoff/roles/` to fit the project: add roles by copying `_template/`, remove the ones you don't use.

## Use

- Start a session: tell the agent its role ("you are implementer"), or just say "resume".
- Pipeline: write it in `.handoff/state.md`, e.g. `planner -> reviewer -> planner -> implementer -> planner -> user`.
- Remember something for a role: "implementer, remember: ...". It goes to `.handoff/roles/implementer/memory.md`, not to any provider's native memory.
- Rules: `template/.handoff/PROTOCOL.md`.
- Full example: `examples/jwt-walkthrough/` (every handoff step, commit messages, and sample artifacts).

## Default roles

| Role | Job |
|---|---|
| `planner` | Plan, spec, verify results |
| `reviewer` | In-loop review against spec/plan; findings back to the author |
| `implementer` | Implement the spec; push back instead of guessing |
| `auditor` | Independent third-party audit; report goes straight to the user |

## Common pipelines

Pick per task in `.handoff/state.md`. Roles not listed are simply skipped.

| Use case | Pipeline |
|---|---|
| Small fix | `implementer -> user` |
| Small fix with review | `implementer -> reviewer -> user` |
| Standard feature | `planner -> reviewer -> planner -> implementer -> planner -> user` |
| Security-sensitive or high-risk change (code is reviewed too) | `planner -> reviewer -> planner -> implementer -> reviewer -> planner -> user` |
| Analysis / design only | `planner -> reviewer -> planner -> user` |
| Independent check of finished work | `auditor -> user` |
