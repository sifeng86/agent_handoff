# agent_handoff

A git-only handoff template for sequential work across agents (Claude, Codex, ...), accounts, machines, and humans. No CLI to install: just files and git.

## How it works

- **One active agent at a time** (plus its own subagents). The work passes like a baton; git is the only channel.
- **`.handoff/state.md` is the task card**: what the task is, the route (`Pipeline` checklist), decisions, progress, and `Relay-To`: whose turn it is.
- **Agents judge where to hand off**: by default the next unchecked `Pipeline` item; back to whoever owns a problem; to the user for anything they can't resolve.
- **Ask, don't guess**: any conflict, ambiguity, or uncertainty goes to the responsible role, or to the user when it's unclear who that is.
- **The user is above the pipeline**: you can give the baton to any role at any time.
- **Roles are folders** (`.handoff/roles/<role>/`) with their own instructions and memory, so any account, provider, or machine can play a role and inherit what it learned.

Rules: [`template/.handoff/PROTOCOL.md`](template/.handoff/PROTOCOL.md). Full example: [`examples/jwt-walkthrough/`](examples/jwt-walkthrough/).

## Prerequisites

- The project is a git repo with a remote, and the branch you work on tracks it (`git pull` works).
- Every machine and account you use can push to that remote.
- All handoffs happen on one branch (the default branch unless the project's `AGENTS.md` names another).

## Install into a project

First install only. It checks every target before writing and stops at the first failure, so it never overwrites state, memory, or customized roles. It refuses to run if `.handoff/` already exists.

```bash
P=<project>; T=template   # run from this repo's root
(
  fail() { echo "install stopped: $*" >&2; exit 1; }
  # Check every target before writing anything.
  { [ -e "$P/.handoff" ] || [ -L "$P/.handoff" ]; } && fail "$P/.handoff already exists (see Update)"
  for d in "$P/docs" "$P/docs/handoff"; do
    { [ -e "$d" ] || [ -L "$d" ]; } && [ ! -d "$d" ] && fail "$d exists but is not a directory"
  done
  cp -r "$T/.handoff" "$P/" || fail "copying .handoff; remove $P/.handoff and rerun"
  { mkdir -p "$P/docs/handoff" && touch "$P/docs/handoff/.gitkeep"; } || fail "creating docs/handoff; remove $P/.handoff and rerun"
  # AGENTS.md: copy, or append the whole handoff block (hard rules).
  if [ -e "$P/AGENTS.md" ]; then
    { echo; sed -n '/<!-- handoff:begin -->/,/<!-- handoff:end -->/p' "$T/AGENTS.md"; } >> "$P/AGENTS.md" || fail "appending to AGENTS.md"
  else cp "$T/AGENTS.md" "$P/" || fail "copying AGENTS.md"; fi
  # CLAUDE.md: copy, or import AGENTS.md on its own line (skipped when either file is a symlink, to avoid a self-import).
  if [ -L "$P/AGENTS.md" ] || [ -L "$P/CLAUDE.md" ]; then echo "symlinked AGENTS.md/CLAUDE.md: check by hand that Claude sees the handoff block"
  elif [ -e "$P/CLAUDE.md" ]; then
    grep -qxF '@AGENTS.md' "$P/CLAUDE.md" || printf '\n@AGENTS.md\n' >> "$P/CLAUDE.md" || fail "appending to CLAUDE.md"
  else cp "$T/CLAUDE.md" "$P/" || fail "copying CLAUDE.md"; fi
  grep -q 'handoff:begin' "$P/AGENTS.md" || fail "handoff block missing from AGENTS.md"
  echo "installed: review, then commit and push (below)"
)
```

Then:
- If the project had a `CLAUDE.md` but no `AGENTS.md`, move rules shared by all agents into `AGENTS.md` (Codex reads only `AGENTS.md`), and remove duplicates.
- Edit `.handoff/roles/` to fit the project: add roles by copying `_template/`, remove the ones you don't use.
- Commit and push, so every machine gets it:
  ```bash
  git -C "$P" add .handoff docs/handoff AGENTS.md CLAUDE.md && git -C "$P" commit -m "chore: install agent handoff" && git -C "$P" push
  ```
- Optional: if your agent tool has a setting to turn off its automatic memory, consider turning it off for the project. The protocol already forbids project content in native memory, but a setting is harder to ignore.

### Update an installed project

Only `PROTOCOL.md` is template-owned; everything else in `.handoff/` belongs to the project.

```bash
cp template/.handoff/PROTOCOL.md "$P/.handoff/"
diff -ru template/.handoff/roles "$P/.handoff/roles"   # merge role changes by hand
diff template/.handoff/state.md "$P/.handoff/state.md"  # add or remove fields by hand
diff <(sed -n '/handoff:begin/,/handoff:end/p' template/AGENTS.md) <(sed -n '/handoff:begin/,/handoff:end/p' "$P/AGENTS.md")  # refresh the block by hand
```

## Use

| You want to | Say | Mode |
|---|---|---|
| Ask a question or discuss | anything | conversation: writes nothing |
| Start a task | "start task: ..., pipeline: ..." | task |
| Continue whoever's turn it is | "resume" / "接手" | task |
| Give the baton to a specific role | "you are planner, ..." | task |
| Make a role remember something | "implementer, remember: ..." | writes only that role's `memory.md` |

When an agent hands a problem to you (`Relay-To: user`), decide and tell any role directly, usually the planner. It records your decision in `Decisions` and takes over.

## Default roles

| Role | Job |
|---|---|
| `planner` | Plan, spec, verify results; answers other roles' questions |
| `reviewer` | In-loop review against spec/plan; changes go back to the author |
| `implementer` | Implement the spec; checks it first and asks instead of guessing |
| `auditor` | Independent third-party audit; report goes to the user. When it takes over the task it holds the baton; a one-off audit only delivers a report |

## Common pipelines

Write the route as a checklist in `.handoff/state.md`, one item per step:

```
Pipeline:
- [ ] planner: plan
- [ ] reviewer: review plan
- [ ] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
```

| Use case | Steps |
|---|---|
| Small fix | implementer → user |
| Small fix with review | implementer → reviewer → user |
| Standard feature | planner → reviewer → planner → implementer → planner → user |
| Security-sensitive or high-risk change (code is reviewed too) | planner → reviewer → planner → implementer → reviewer → planner → user |
| Analysis / design only | planner → reviewer → planner → user |
| Independent check of finished work | auditor → user |

Without a planner (small fixes), your instruction is the spec: state scope, expected behavior, and how to verify, or the implementer will ask.
