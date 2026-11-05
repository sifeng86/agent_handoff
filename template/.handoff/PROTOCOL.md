# Handoff Protocol

Work in this repo is passed between agents (any provider, any account, any machine) and humans. Git is the only shared channel: anything not committed and pushed does not exist for the next agent.

## 0. Ground Rules
- **One active agent at a time**, plus its own subagents. Everyone else is idle. Before giving the work to another role, the user stops the current one.
- **Ask, don't guess.** Any conflict, ambiguity, or uncertainty (in inputs, in the code, in git such as a merge conflict, or about what to do next): stop that part and ask the role responsible for it. If you don't know who that is, ask `user`. Never guess, force, or silently pick an option.
- **One branch.** All handoffs happen on the default branch, unless the repo `AGENTS.md` names another.

## 1. Precedence
User's latest instruction > `.handoff/roles/<role>/role.md` > repo `AGENTS.md` / this protocol > global defaults.

This protocol replaces any global state-relay rule: use `.handoff/state.md`, never create `.agent-state.md`. If a legacy `.agent-state.md` exists, migrate its useful content into `.handoff/state.md` and retire it.

In task mode, every role, including read-only ones, may update `state.md`, write its own output and memory files (§7), and commit and push them. A role's read-only restriction applies to everything else.

## 2. Modes
- **Conversation mode** (default): questions, discussion, unrelated topics. Read files as needed, but write nothing. Exceptions, only when the user explicitly asks: "remember ..." goes to a memory file (§7); "change file X" changes that file. Neither touches `state.md`. If something is worth recording, offer to record it.
- **Task mode**: the user assigns a role ("you are implementer"), says "resume" / "接手", or opens a task ("start task X"). Follow §3.

## 3. Session Start (task mode)
1. `git pull --ff-only`. If it fails, ask (§0).
2. Your role: the one the user names; otherwise `Relay-To` in `.handoff/state.md`. If that is `user` or unclear, ask.
3. Read, in order (the `auditor` role defines its own order):
   1. `.handoff/memory.md` — project memory for all roles
   2. `.handoff/roles/<role>/role.md` and `memory.md`
   3. `.handoff/state.md`, the files in its `Artifacts`, and `git log --oneline -10` for context
4. `git status -s`. Uncommitted changes (e.g. a session cut off by quota): inspect the diff, then continue it or commit it as `wip`; never discard it. If `state.md` disagrees with the code and git log, trust the code and the log, correct `state.md`, and note it in `Decisions`; if they can't be reconciled, ask.
5. **Check your inputs** (plan, spec, findings, user instruction): clear, consistent with each other and with the code, and enough to finish without guessing? If not, ask (§0) instead of starting.
6. Start from `Next action`. Do not redo anything in `Completed`.

**The user assigns you directly** (even if `Relay-To` names someone else or `user`, and whether or not your role is in the `Pipeline`): decide from the user's words whether you are to **take over the task** or only **give a one-off report or opinion**; if unclear, ask. Taking over: the user's word wins. Add the instruction to `Decisions` (`- YYYY-MM-DD user: ...`), set `Relay-To` to yourself, and add or edit `Pipeline` items if the route changed. One-off report: see §6, last paragraph.

**A new task while one is unfinished** (`Status` is not `idle` or `done`): ask the user whether to park the current card (move it to `docs/handoff/YYYY-MM-DD-parked-<topic>.md`) or defer the new task.

## 4. State File (`.handoff/state.md`)
The task card. Overwrite it freely; git history (`git log -p .handoff/state.md`) is the handoff log. Keep it short (about 30 lines); put long content in artifacts.

- `Status`: `idle` | `in-progress` | `handoff` | `blocked` | `done`
- `Relay-To`: the role that moves next, or `user`. The only authority on whose turn it is.
- `Task`: one line.
- `Pipeline`: advisory route as a checklist, one item per step (`- [x] planner: plan`, `- [ ] reviewer: review plan`). The next unchecked item is the default next step. Set by the user, or proposed by the first agent and approved by the user. Empty means: the role the user names, then `user`.
- `Next action`: the first concrete step for `Relay-To`. Must be actionable without guessing.
- `Completed`: what is done.
- `Decisions`: one line each; rewrite a superseded one instead of appending a contradiction. Record user instructions and repeated questions here.
- `Artifacts`: files the next role needs.

## 5. Checkpoint
After each completed step (a planned unit of work, not each tool call): update `state.md` (`Status: in-progress`, `Completed`, `Next action`), commit with the related changes, then **push**. Never stop in task mode without pushing.

If a push is rejected: `git pull --rebase`, then push again. A conflict is a question (§0). Never force-push or rewrite pushed history.

Switching account, provider, or machine for the same role is not a handoff: just checkpoint. The next session continues as the same role.

## 6. Handoff
When your part is done, or you need an answer from someone else:
1. Tick your checklist item if it is done. Choose who moves next:
   - by default, the next unchecked item;
   - if an artifact has a problem, the role responsible for it (e.g. a reviewer's `changes-requested` usually goes to the author);
   - `user` when the decision belongs to the user (scope, direction, trade-offs, permissions), when you can't tell who is responsible, when a stop condition in the plan/spec is met, or when the same issue comes back a second time.
2. Update `state.md`: `Relay-To`, a precise `Next action`, and `Status` (`handoff`; `blocked` when waiting on the user; `done` with `Relay-To: user` when only the final `user` item is left).
3. Commit and push. The commit body says why you chose that next role. Optionally add `Relay-To: <role>` as a trailer, in the final paragraph with no blank line before `Co-Authored-By`; it is a log only.

**Asking is a handoff.** Write the question in a dated file under the artifacts directory and hand off to the role you ask. That role answers (revising its own artifact if needed) and hands back. Before handing off, finish the work that does not depend on the answer. Do not rewrite another role's artifact or relax its criteria yourself.

**Only the baton holder changes `Relay-To`**: the role it names, or a role the user just assigned. A role the user asks only for a one-off report or opinion (e.g. an ad-hoc audit) does not hold the baton and does not touch `state.md`: it commits and pushes only its own output and reports to the user.

## 7. Memory
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

## 8. Artifacts
Plans, specs, reviews, audits, and questions go under `docs/handoff/` (unless repo `AGENTS.md` says otherwise), named `YYYY-MM-DD-<type>-<topic>.md`. List the ones the next role needs in `Artifacts`.

## 9. Roles
Each role is a folder in `.handoff/roles/`. Add one by copying `_template/`; remove one by deleting its folder. Any number of agents may play the same role over time; the role folder, not the session, carries continuity.

## 10. Optional: Wait for Your Turn Without Spending Tokens
Only for agents that can run a background command and be woken when it exits. Not required: without it, the user starts a new session and says "resume".

```bash
me=<role>
while :; do
  git fetch -q || { echo "handoff-wait: fetch failed" >&2; exit 1; }
  s=$(git show '@{u}:.handoff/state.md') || { echo "handoff-wait: no upstream state" >&2; exit 1; }
  [ -f .git/HANDOFF_PAUSE ] && { echo "handoff-wait: paused" >&2; exit 2; }
  printf '%s\n' "$s" | grep -qx 'Status: handoff' &&
    printf '%s\n' "$s" | grep -qx "Relay-To: $me" && exit 0
  sleep 60
done
```

- It reads the latest upstream `state.md`, so a superseded handoff never wakes you and an already-pulled one is not missed.
- Exit 0: your turn. `git pull --ff-only`, then Session Start. Exit 1: ask (§0). Exit 2: paused.
- Pause: `touch .git/HANDOFF_PAUSE` (local to this clone). Resume: delete it and restart the wait.
