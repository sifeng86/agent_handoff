# Install agent handoff (instructions for the agent)

The user asked you to install or update agent handoff in their project. Work like an install wizard: check first, ask the user one question at a time, and change nothing until the user confirms the final summary.

- **TEMPLATE**: this repo. If the user gave a URL, `git clone --depth 1 <url>` into a temporary directory outside the project. Copy from `TEMPLATE/template/`.
- **PROJECT**: the repo you are working in, unless the user names another.

## Rules
- Never overwrite or delete an existing file. Only create files, or append to `AGENTS.md` / `CLAUDE.md`. Exceptions: `.handoff/PROTOCOL.md` during an update, and edits the user approved in the Step 3 summary.
- One question per message. Give your recommended answer and a one-line reason; "ok" accepts it. Skip questions that do not apply.
- Talk to the user in their language. Files you write stay in English, unless the project's existing docs use another language.
- If anything is unclear or unexpected, ask; do not guess.
- Commit only the files you installed; never stage the user's other changes. Push only if the user said yes.

## Step 1. Check (read-only)
Collect, without writing anything:
1. Git: `git status -sb`, current and default branch, `git remote -v`, whether the branch tracks an upstream, already-staged changes (`git diff --cached --name-only`), and that `git config user.name` / `user.email` are set. Not a git repo: stop. No remote or upstream, staged changes, or no identity: ask the user; don't fix them yourself.
2. `PROJECT/.handoff/` exists: this is an update; go to **Update** below.
3. `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`: which exist, and whether each is a regular writable file or a symlink. Read their agent-workflow rules (state files, relay, roles, branches); note any that conflict with the handoff block in `TEMPLATE/template/AGENTS.md`, and project rules that sit only in `CLAUDE.md` or `GEMINI.md` (other agents don't read them).
4. Ignored paths: `git check-ignore -v .handoff/state.md docs/handoff/x AGENTS.md CLAUDE.md`.
5. Existing coordination:
   - `.agent-state.md`: tracked, or local-only (`.git/info/exclude`)? Its `Status` and `Task`.
   - Relay or handoff conventions in recent commits: `git log -30 --format='%h %s%n%b' | grep -iE 'relay|handoff|->|→'`.
   - Folders used for plans, specs, or handoff docs.
   - Unfinished work: the latest state or relay names a next role or step.

Report the findings to the user in a few bullets: what is there, what the install will add, and anything that needs a decision.

## Step 2. Ask (one at a time)
1. **Existing workflow** (only if Step 1 found unfinished work or a legacy `.agent-state.md`):
   - a. Migrate it: the current task becomes the `state.md` card. Recommended when the next step is clear.
   - b. Finish it under the old workflow first: stop the install now.
   - c. Install for new tasks only: the old workflow's history stays as is; a legacy `.agent-state.md` is still retired (Step 4.7).
2. **Branch**: all handoffs happen on one branch. Recommend the default branch, or the branch the project already works on. If it is not the current branch, ask the user to switch first.
3. **Roles**: list the defaults with one-line jobs (planner, reviewer, implementer, auditor). Propose a set that fits the project and map any existing role names (e.g. an existing "規劃者" becomes planner). The user can drop, rename, or add roles.
4. **Artifacts folder**: plans, specs, reviews, and questions. Recommend `docs/handoff/`, or the project's existing folder for such docs.
5. **Shared rules** (only if project rules sit only in `CLAUDE.md` or `GEMINI.md`): move the rules meant for all agents into `AGENTS.md` below the handoff block, so every agent reads them? Recommend yes.
6. **Ignored paths** (only if Step 1 found any): propose a `.gitignore` exception; handoff files must be committed.
7. **Push**: push the install commit now? Other machines and accounts get it only after a push.

## Step 3. Confirm
Show one summary: files to create, files to change, roles, branch, artifacts folder, migration, commit message, push yes/no. Change nothing until the user says yes. If the user changes an answer, show the summary again.

## Step 4. Install
1. `git pull --ff-only`; if it fails, ask.
2. Create `PROJECT/.handoff/` and copy from `TEMPLATE/template/.handoff/`: `PROTOCOL.md`, `memory.md`, `state.md`, `roles/_template/`, and only the chosen roles. For each new role, copy `_template/` and fill in `role.md` from the user's description.
3. Create the artifacts folder with an empty `.gitkeep` if it does not exist.
4. `AGENTS.md`:
   - absent: copy `TEMPLATE/template/AGENTS.md`;
   - present: append, starting on a new line, a blank line and the lines from `<!-- handoff:begin -->` to `<!-- handoff:end -->` of the template file. Skip if it already has `handoff:begin`.
5. `CLAUDE.md`:
   - absent: copy `TEMPLATE/template/CLAUDE.md`;
   - present: append `@AGENTS.md` on its own new line, unless it already has one.
   - Symlinks: don't edit through them. `CLAUDE.md` pointing to `AGENTS.md` needs nothing; otherwise tell the user what to add by hand.
6. Non-default settings go below the handoff block in `AGENTS.md`, one line each:
   - `- Handoff branch: <branch>`
   - `- Handoff artifacts: <folder>/ (replaces docs/handoff/ in PROTOCOL and roles)`
7. Shared rules (if 2.5 yes): move them into `AGENTS.md` below the handoff block and remove them from the old file. Old rules that conflict with the handoff block: tell the user, and change them only if the user agrees.
8. Migration (if 1a): fill `.handoff/state.md` from the old state:
   - `Status: handoff`, `Relay-To` set to the role that is next.
   - `Task`, `Pipeline` (completed steps ticked), a precise `Next action`, and `Artifacts` pointing to the existing docs.
   - In `Decisions`: `- YYYY-MM-DD: migrated from <source> at <commit>`.
9. Retire a legacy `.agent-state.md` (1a or 1c). If tracked: `git rm` it in the install commit. If local-only: tell the user it can be deleted.
10. Add `.gitignore` exceptions if agreed.

## Step 5. Verify and commit
1. Check that the `handoff:begin` block is in `AGENTS.md` and Claude reaches it (`@AGENTS.md` in `CLAUDE.md`, or a symlink).
2. Check that `git status -s` differs from Step 1 only by the files from the summary.
3. `git add` exactly those files, then `git commit -m "chore: install agent handoff" -- <those files>` so nothing else is committed. Push if agreed.
4. Tell the user how to start:
   - "start task: ..., pipeline: planner → implementer → user"
   - "接手" / "resume" in any agent to continue
   - "驗收通過" / "accepted" to close a task

   For headless or sandboxed agents, point the user to "Tool environment notes" in this repo's `README.md`.

## Update (`.handoff/` already exists)
`PROTOCOL.md` is the only template-owned file. Everything else in `.handoff/` belongs to the project: never touch `memory.md` files or the content of `state.md`.
1. Run the Step 1.1 git checks.
2. Diff `TEMPLATE/template/.handoff/PROTOCOL.md` against the project's copy. Summarize the changes for the user, including any local edits that the update would overwrite.
3. Diff roles (`TEMPLATE/template/.handoff/roles/` against `PROJECT/.handoff/roles/`), the field names in `state.md`, and the handoff block in `AGENTS.md`. For each difference, ask whether to apply it, one at a time.
4. Confirm the summary (Step 3), `git pull --ff-only`, apply, verify, and commit as `chore: update agent handoff`. Push if agreed.
