# Agent Instructions

<!-- handoff:begin -->
This repo uses the handoff protocol in `.handoff/PROTOCOL.md`. Hard rules (details there):
1. Conversation mode is the default: questions and discussion write nothing. Task mode starts only when the user assigns a role, says "resume" / "接手", or opens a task; then follow PROTOCOL §3.
2. One active agent at a time. Any conflict, ambiguity, or uncertainty: ask the responsible role; if you don't know who, ask the user. Never guess or force.
3. In task mode, start with `git pull --ff-only` and never stop without committing and pushing.
4. Never write anything about this project to provider-native memory; use the `.handoff/` memory files.
5. Use `.handoff/state.md`; never create `.agent-state.md` (migrate a legacy one into it, then retire it).
<!-- handoff:end -->

<!-- Project-specific rules below. -->
