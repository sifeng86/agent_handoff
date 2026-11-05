# Agent Instructions: agent_handoff (template development)

This file is for agents developing this repo. It is **not** the template's entry file; that is `template/AGENTS.md`, copied into other projects.

This repo does not install its own template: progress is tracked with the global `.agent-state.md` rule, not `.handoff/state.md`.

## Project decisions
<!-- Rules that constrain future work. One line each, max 20 lines. Rewrite or delete superseded ones; rationale stays in git history. -->
- Git + Markdown only. No CLI, scripts to install, or other tooling.
- One active agent at a time (plus its own subagents). Don't add machinery to guard against concurrent agents.
- Ask, don't guess: conflicts, ambiguity, and uncertainty go to the responsible role, else the user. The user may hand the baton to any role at any time.
- `state.md` is the single source of truth: `Relay-To` says whose turn it is, `Pipeline` is an advisory checklist. Commit trailers are logs only. No cursors or transition tables: agents judge where to hand off.
- Roles are folders (`role.md` + `memory.md`). Roles and pipelines are chosen per project and per task; never hardcode them in PROTOCOL.
- `reviewer` (in-loop, back to the author) and `auditor` (independent, reports to the user) are separate roles.
- Provider-native memory never holds project content; repo memory files only.
- Conversation mode writes nothing. Only task mode (role assigned, or resume) updates handoff files.
- Ad-hoc roles outside the `Pipeline` never touch `state.md`.
- In an installed project, only `.handoff/PROTOCOL.md` is template-owned. Install must never overwrite existing files.
- Agent-facing files are in English; explanations to the user are in Traditional Chinese.
- When PROTOCOL or a role changes, update `README.md` and `examples/jwt-walkthrough/` so they do not contradict it.
- `default_instruction/` is the user's local copy of global instructions. Not part of this project; never commit it.
