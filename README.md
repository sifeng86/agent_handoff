# agent_handoff

A git-only handoff template for sequential work across agents (Claude Code, Codex, Antigravity `agy`, ...), accounts, machines, and humans. No CLI to install: just files and git.

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

### With an agent (recommended)

Open any agent (Claude Code, Codex, agy) in your project and say:

> Install agent handoff into this project, following `<path-or-URL of this repo>/INSTALL.md`.

It checks the project, asks what it needs one question at a time (existing workflow, branch, roles, artifacts folder, push), shows a summary, and changes nothing until you confirm. It never overwrites existing files.

### By hand

From this repo's root, with `P=<your project>` and `A=docs/handoff` (the artifacts folder; change it if you use another, see step 6):

1. Check that `$P/.handoff/` does not exist (if it does, see Update). If the project already has an agent workflow (a `.agent-state.md`, relay commits, a handoff folder), decide first whether to migrate its unfinished task into `.handoff/state.md`.
2. Copy the protocol and roles, and create the artifacts folder:
   ```bash
   cp -r template/.handoff "$P/" && mkdir -p "$P/$A" && touch "$P/$A/.gitkeep"
   ```
3. `AGENTS.md`:
   - none yet: `cp template/AGENTS.md "$P/"`
   - already there: copy the lines from `<!-- handoff:begin -->` to `<!-- handoff:end -->` in `template/AGENTS.md` to the end of yours.
4. `CLAUDE.md`:
   - none yet: `cp template/CLAUDE.md "$P/"`
   - already there: add a line `@AGENTS.md` (Claude reads `AGENTS.md` through it; Codex and agy read it natively). Move rules shared by all agents into `AGENTS.md`.
5. Roles: in `$P/.handoff/roles/`, delete the ones you don't use; add one by copying `_template/`.
6. Not using the default branch or `docs/handoff/`? Add a line below the handoff block in `$P/AGENTS.md`, e.g. `- Handoff branch: develop`, `- Handoff artifacts: docs/research/ (replaces docs/handoff/ in PROTOCOL and roles)`.
7. On the handoff branch, commit only these files and push, so every machine gets it:
   ```bash
   git -C "$P" add .handoff "$A" AGENTS.md CLAUDE.md && git -C "$P" commit -m "chore: install agent handoff" -- .handoff "$A" AGENTS.md CLAUDE.md && git -C "$P" push
   ```

Optional: if your agent tool can turn off its automatic memory, consider doing so for the project. The protocol already forbids project content in native memory, but a setting is harder to ignore.

### Update an installed project

With an agent: "Update agent handoff in this project, following `<path-or-URL>/INSTALL.md`." It replaces `PROTOCOL.md` and asks about each other difference.

By hand: only `PROTOCOL.md` is template-owned; everything else in `.handoff/` belongs to the project.

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

To accept finished work, tell any agent "accepted" / "驗收通過": it ticks the `user` item and sets `Status: done`.

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

## Tool environment notes

Agents must be able to commit and push without a prompt they cannot answer:
- **Codex**: the `workspace-write` sandbox makes `.git` read-only; add the repo's `.git` to `sandbox_workspace_write.writable_roots`. On hosts where its `bwrap` sandbox fails, run Codex where it works.
- **agy headless (`-p`)**: it cannot prompt, so commands and writes are denied unless allowed in `permissions.allow` (`~/.gemini/antigravity-cli/settings.json` or `--settings`), e.g. `"command(git)"`, `"write_file(<repo>)"`, plus every command the agent chains or reads with (`cd`, `ls`, `cat`, `find`, `head`, `grep`, ...); one unlisted command fails the whole turn. Interactive sessions just ask.
- **Claude Code headless (`-p`)**: allow the tools it needs with `--allowedTools` (e.g. `Bash(git:*)`).
