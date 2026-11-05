# agent_handoff

A git-only handoff template for sequential work across agents (Claude, Codex, ...), accounts, machines, and humans. No CLI to install: just files and git.

## Install into a project

First install only. It refuses to run if `.handoff/` already exists, so it never overwrites state, memory, or customized roles.

```bash
P=<project>
if [ -e "$P/.handoff" ]; then echo "already installed: see Update"; else
  cp -r template/.handoff "$P/" && mkdir -p "$P/docs/handoff" && touch "$P/docs/handoff/.gitkeep"
  # AGENTS.md: copy, or append the whole handoff block (pointer + state-relay override).
  if [ -e "$P/AGENTS.md" ]; then
    { echo; sed -n '/<!-- handoff:begin -->/,/<!-- handoff:end -->/p' template/AGENTS.md; } >> "$P/AGENTS.md"
  else cp template/AGENTS.md "$P/"; fi
  # CLAUDE.md: copy, or import AGENTS.md.
  if [ -e "$P/CLAUDE.md" ]; then
    grep -qxF '@AGENTS.md' "$P/CLAUDE.md" || echo '@AGENTS.md' >> "$P/CLAUDE.md"
  else cp template/CLAUDE.md "$P/"; fi
fi
```

If an existing `CLAUDE.md` repeats rules already in `AGENTS.md`, remove the duplicates after importing.

Then edit `.handoff/roles/` to fit the project: add roles by copying `_template/`, remove the ones you don't use.

### Update an installed project

Only `PROTOCOL.md` is template-owned; everything else in `.handoff/` belongs to the project.

```bash
cp template/.handoff/PROTOCOL.md "$P/.handoff/"
diff -ru template/.handoff/roles "$P/.handoff/roles"   # merge role changes by hand
diff template/.handoff/state.md "$P/.handoff/state.md"  # add any new fields by hand
```

## Use

- Start a session: tell the agent its role ("you are implementer"), or just say "resume".
- Pipeline: write it in `.handoff/state.md`, e.g. `planner -> reviewer -> planner -> implementer -> planner -> user`.
- The pipeline is not one-way: every role checks its inputs before starting and pushes back to the previous role instead of guessing. The same question pushed back twice without resolution goes to the user.
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

Without a planner (small fixes), the user's instruction is the spec: it must state scope, expected behavior, and how to verify, or the implementer pushes back to `user`.

| Use case | Pipeline |
|---|---|
| Small fix | `implementer -> user` |
| Small fix with review | `implementer -> reviewer -> user` |
| Standard feature | `planner -> reviewer -> planner -> implementer -> planner -> user` |
| Security-sensitive or high-risk change (code is reviewed too) | `planner -> reviewer -> planner -> implementer -> reviewer -> planner -> user` |
| Analysis / design only | `planner -> reviewer -> planner -> user` |
| Independent check of finished work | `auditor -> user` |
