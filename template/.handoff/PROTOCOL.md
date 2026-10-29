# Handoff Protocol

Work in this repo is passed between agents (any provider, any account, any machine) and humans, one at a time. Git is the only shared channel: anything not committed and pushed does not exist for the next agent.

## 1. Precedence
User's latest instruction > `.handoff/roles/<role>/role.md` > repo `AGENTS.md` / this protocol > global defaults.

## 2. Session Start
1. `git pull --ff-only`. If it fails, stop and report; do not force anything.
2. Determine your role: the user says it, or take `Current` in `.handoff/state.md` (same as the latest `Relay-To` trailer). If neither is clear, ask.
3. Read, in order (the `auditor` role defines its own order):
   1. `.handoff/memory.md` — project memory for all roles
   2. `.handoff/roles/<role>/role.md` and `memory.md`
   3. `.handoff/state.md`, then files listed in its `Artifacts`
4. Run `git status -s` and `git log --oneline -5` to confirm the state matches `state.md`.
5. Start from `Next action`. Do not redo anything listed in `Completed`.

## 3. State File (`.handoff/state.md`)
The baton. Overwrite it freely; git history (`git log -p .handoff/state.md`) is the handoff log. Keep it under 30 lines.

- `Status`: `idle` | `in-progress` | `handoff` | `blocked` | `done`
- `Pipeline`: optional role sequence, e.g. `planner -> reviewer -> planner -> implementer -> planner -> user`. Set by the user, or proposed by the first agent and approved by the user. Empty means: when done, hand to `user`.
- `Current`: role that owns the next move.
- `Next action`: the first concrete step for `Current`. Must be actionable without guessing.
- `Last agent`: `<model>@<account-alias>/<host>` of whoever wrote this version.

## 4. Checkpoint
After each completed step (a planned unit of work, not each tool call):
1. Update `state.md` (`Status: in-progress`, `Completed`, `Remaining`, `Next action`).
2. Commit with the related changes, then **push**.

Quota, session, or machine can end at any moment. Unpushed work is lost work.

## 5. Handoff
When your role's work is done:
1. Set `Status: handoff`, `Current: <next role>`, and a precise `Next action`.
2. Commit and push. Only handoff commits carry these trailers, in the final paragraph, before any `Co-Authored-By`:
   ```
   Relay-To: <role | user>
   Relay-Doc: <path the next role should read first>
   ```
3. Next role = next entry in `Pipeline`; `user` when there is none.

Switching account, provider, or machine for the same role is not a handoff: just checkpoint. The next session continues as the same role.

## 6. Push Back, Don't Guess
If your inputs (plan, spec, review findings) are wrong, contradictory, unsupported by the code, or require guessing:
- Stop that part. Write the questions in a dated file under the artifacts directory.
- Hand off with `Relay-To: <role that handed to you>`. Do not rewrite another role's artifact or relax its criteria yourself.
- Continue only work that does not depend on the answer.

## 7. Stop and Hand to `user`
Use `Relay-To: user` and `Status: blocked` when:
- a decision is outside the approved scope or belongs to the user;
- the circuit breaker trips (repeated failures, environment or permission problems);
- a stop condition written in the plan/spec is met.

## 8. Memory
Provider-native memory (Claude Code auto-memory, `~/.claude/projects/*/memory/`, Codex memories, etc.) must **not** hold anything about this project. It is tied to one machine/account and the next agent cannot see it. Repo memory files are the only project memory.

| Content | Location |
|---|---|
| Personal preferences across all projects | user's global `CLAUDE.md` / `AGENTS.md` (not in this repo) |
| Facts every role needs | `.handoff/memory.md` |
| Instructions for one role ("implementer: ...") | `.handoff/roles/<role>/memory.md` |
| Progress of the current task | `.handoff/state.md` |

- When the user asks to remember something: a named role goes to that role's file; otherwise `.handoff/memory.md`; if unsure, ask.
- One item per line: `- YYYY-MM-DD: <fact or instruction>`. Max 40 lines per file.
- Over the limit, or an item is obsolete: consolidate or delete in its own commit.
- Commit and push memory changes immediately.

## 9. Artifacts
Plans, specs, reviews, audits, and questions go under `docs/handoff/` (unless repo `AGENTS.md` says otherwise), named `YYYY-MM-DD-<type>-<topic>.md`. List the ones the next role needs in `state.md` `Artifacts`.

## 10. Roles
Each role is a folder in `.handoff/roles/`. Add one by copying `_template/`; remove one by deleting its folder. Any number of agents may play the same role over time; the role folder, not the session, carries continuity.

## 11. Optional: Wait for Relay Without Spending Tokens
Only for agents that can run a background command and be woken when it exits. Not required: without it, the user starts a new session and says "resume".

```bash
me=<role>; old=$(git rev-parse HEAD)
until git fetch -q && git log --format=%B "$old"..@{u} | grep -q "^Relay-To: $me$" \
  || [ -f .git/HANDOFF_PAUSE ]; do sleep 60; done
```

- Fetching is required: other agents push from other machines.
- Pause: `touch .git/HANDOFF_PAUSE` (local to this clone). Resume: delete it and restart the wait.
- On wake: `git pull --ff-only`, then Session Start.
