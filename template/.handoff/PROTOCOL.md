# Handoff Protocol

Work in this repo is passed between agents (any provider, any account, any machine) and humans, one at a time. Git is the only shared channel: anything not committed and pushed does not exist for the next agent.

## 1. Precedence
User's latest instruction > `.handoff/roles/<role>/role.md` > repo `AGENTS.md` / this protocol > global defaults.

This protocol replaces any global state-relay rule: use `.handoff/state.md`, never create `.agent-state.md`. If a legacy `.agent-state.md` exists, migrate its useful content into `.handoff/state.md` and retire it.

**Handoff files are always writable.** Every role, including read-only ones, may update `.handoff/state.md`, write its own output file, update memory files (§8), and commit and push those. A role's read-only restriction applies to everything else.

## 2. Session Start
1. `git pull --ff-only`. If it fails, stop and report; do not force anything.
2. Determine your role: the user says it, or take `Current` in `.handoff/state.md`. If neither is clear, ask.
3. Read, in order (the `auditor` role defines its own order):
   1. `.handoff/memory.md` — project memory for all roles
   2. `.handoff/roles/<role>/role.md` and `memory.md`
   3. `.handoff/state.md`, then files listed in its `Artifacts`
4. Run `git status -s` and `git log --oneline -5` to confirm the state matches `state.md`.
5. Start from `Next action`. Do not redo anything listed in `Completed`.

## 3. State File (`.handoff/state.md`)
The baton and the only source of truth for who moves next. Overwrite it freely; git history (`git log -p .handoff/state.md`) is the handoff log. Keep it under 30 lines.

- `Status`: `idle` | `in-progress` | `handoff` | `blocked` | `done`
- `Pipeline`: optional role sequence, e.g. `planner -> reviewer -> planner -> implementer -> planner -> user`. Set by the user, or proposed by the first agent and approved by the user. Empty means a single step: the role the user names, then `user`.
- `Step`: 1-based position in `Pipeline` that is being worked on or waiting. Disambiguates roles that appear more than once.
- `Current`: role that owns the next move. Equals the role at `Step`, except during a detour (§6).
- `Return-To`: set only during a detour: the `Step` to resume afterwards. Otherwise empty.
- `Next action`: the first concrete step for `Current`. Must be actionable without guessing.
- `Last agent`: `<model>@<account-alias>/<host>` of whoever wrote this version.

## 4. Checkpoint
After each completed step (a planned unit of work, not each tool call):
1. Update `state.md` (`Status: in-progress`, `Completed`, `Remaining`, `Next action`).
2. Commit with the related changes, then **push**.

Quota, session, or machine can end at any moment. Unpushed work is lost work.

Switching account, provider, or machine for the same role is not a handoff: just checkpoint. The next session continues as the same role.

## 5. Handoff
When your role's work is done, update `state.md` per the table, commit, and push.

| Situation | `Status` | `Step` | `Current` |
|---|---|---|---|
| Normal: work done | `handoff` | `Step + 1` | role at new `Step` |
| Reviewer `changes-requested` and the next step is not the author's role | `handoff` | author's step (the latest earlier step with that role) | author's role |
| Reviewer `changes-requested` for the same artifact a third time | `blocked` | unchanged | `user` |
| New `Step` is the final `user` step, or past the end | `done` | last step | `user` |
| Stop condition (§7) | `blocked` | unchanged | `user` |

A reviewer's `approve`, or `changes-requested` when the author's role is next anyway, is a normal handoff: the author handles the findings in its own step.

Handoff commits carry these trailers in the **final paragraph, with no blank line between them and any `Co-Authored-By`**, so `git interpret-trailers --parse` can read them:
```
Relay-To: <role | user>
Relay-Doc: <path the next role should read first>
Co-Authored-By: ...
```
Trailers are a log for humans and tools; `state.md` decides who moves next.

## 6. Push Back, Don't Guess (Detour)
If your inputs (plan, spec, review findings, user instruction) are wrong, contradictory, unsupported by the code, or require guessing:
1. Stop that part. Write the questions in a dated file under the artifacts directory.
2. Set `Status: handoff`, `Return-To: <current Step>`, `Current: <role that handed to you>` (`user` if the user started you). Keep `Step` unchanged.
3. The asked role answers (revising its own artifact if needed), then sets `Current` to the role at `Return-To`, clears `Return-To`, and hands off. A detour never advances `Step`.
4. During a detour, further questions go to `user` with `Status: blocked`; keep `Return-To`. The user answers and names who continues.

Do not rewrite another role's artifact or relax its criteria yourself. Continue only work that does not depend on the answer.

A role the user invokes outside the `Pipeline` (e.g. an ad-hoc `auditor`) is also a detour: set `Return-To` to the current `Step`, and on completion hand to `user`, who decides how to continue.

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

## 11. Optional: Wait for Your Turn Without Spending Tokens
Only for agents that can run a background command and be woken when it exits. Not required: without it, the user starts a new session and says "resume".

```bash
me=<role>
while :; do
  git fetch -q || { echo "handoff-wait: fetch failed" >&2; exit 1; }
  s=$(git show '@{u}:.handoff/state.md') || { echo "handoff-wait: no upstream state" >&2; exit 1; }
  printf '%s\n' "$s" | grep -qx 'Status: handoff' &&
    printf '%s\n' "$s" | grep -qx "Current: $me" && exit 0
  [ -f .git/HANDOFF_PAUSE ] && { echo "handoff-wait: paused" >&2; exit 2; }
  sleep 60
done
```

- It checks the latest upstream `state.md`, not commit history, so a superseded relay never wakes you and an already-pulled one is not missed.
- Exit 0: your turn. `git pull --ff-only`, re-check `Status`/`Current` locally, then Session Start. Exit 1: fix fetch/upstream and report. Exit 2: paused.
- Pause: `touch .git/HANDOFF_PAUSE` (local to this clone). Resume: delete it and restart the wait.
