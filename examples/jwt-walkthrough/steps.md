# Steps

Each step shows `.handoff/state.md` after the step and the commit that carries it. Only handoff commits have `Relay-*` trailers.

---

## 00 — user opens the task

使用者自己編輯，或叫任一 agent 寫入。

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: planner
Next action: Write a plan for JWT auth. Scope: login endpoint + auth middleware. Out of scope: refresh tokens.
Completed:
Remaining: plan, review, spec, implement, verify
Decisions:
Artifacts:
Last agent: user
```

```
chore(handoff): open task JWT auth

Relay-To: planner
Relay-Doc: .handoff/state.md
```

---

## 01 — planner writes the plan

`claude-opus-5-5@claude-a/host-1`。開 session 說「你是 planner」或「接手」。

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: reviewer
Next action: Review docs/handoff/2026-11-04-plan-jwt.md against the goal in Task and the current code in src/api/.
Completed: plan
Remaining: review, spec, implement, verify
Decisions: HS256 with secret from env (see plan §3)
Artifacts: docs/handoff/2026-11-04-plan-jwt.md
Last agent: claude-opus-5-5@claude-a/host-1
```

```
docs(plan): JWT auth plan

Relay-To: reviewer
Relay-Doc: docs/handoff/2026-11-04-plan-jwt.md

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 02 — reviewer reviews the plan

`gpt-6-astra@codex/host-2`。換 provider、換機器：只需要 `git pull`，然後說「接手」。

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: planner
Next action: Decide on each finding in docs/handoff/2026-11-04-review-plan-jwt.md, then write the spec.
Completed: plan, review (changes-requested)
Remaining: spec, implement, verify
Decisions: HS256 with secret from env (see plan §3)
Artifacts: docs/handoff/2026-11-04-plan-jwt.md, docs/handoff/2026-11-04-review-plan-jwt.md
Last agent: gpt-6-astra@codex/host-2
```

```
docs(review): plan review for JWT auth, changes requested

Relay-To: planner
Relay-Doc: docs/handoff/2026-11-04-review-plan-jwt.md
```

---

## 03 — planner writes the spec

回到 `claude-opus-5-5@claude-a/host-1`。

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: implementer
Next action: Check docs/handoff/2026-11-04-spec-jwt.md against the code, then implement step S1.
Completed: plan, review, spec (review findings resolved in spec §6)
Remaining: implement S1-S3, verify
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md
Last agent: claude-opus-5-5@claude-a/host-1
```

```
docs(spec): JWT auth spec, review findings resolved

Relay-To: implementer
Relay-Doc: docs/handoff/2026-11-04-spec-jwt.md

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 04 — implementer checkpoint

`claude-sonnet-5-5@claude-a/host-3`。完成 S1，checkpoint：**不寫 Relay 行**。

```
Status: in-progress
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: implementer
Next action: Implement S2 (auth middleware) per spec §3.2.
Completed: plan, review, spec, S1 (token issue on POST /login, tests pass)
Remaining: S2, S3, verify
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md
Last agent: claude-sonnet-5-5@claude-a/host-3
```

```
feat(auth): issue JWT on login (S1)

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 05 — quota runs out; another account continues

Claude A 的 quota 用完。在同一台（或任何一台）主機用 Claude B 開 session 說「接手」。
同一角色換人**不是交棒**：不寫 Relay 行，從 `Next action` 繼續。S2 做到一半時再 checkpoint 一次：

```
Status: in-progress
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: implementer
Next action: Finish S2: apply middleware to routes listed in spec §3.2; expired-token test still missing.
Completed: plan, review, spec, S1, S2 middleware core
Remaining: S2 route wiring + expired-token test, S3, verify
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md
Last agent: claude-sonnet-5-5@claude-b/host-3
```

```
wip(auth): JWT middleware core (S2 partial)

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 06 — implementer pushes back

Spec 要求套用到 `/health`，但它被外部監控呼叫、沒辦法帶 token。不自己猜，退回給 planner。

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: planner
Next action: Answer docs/handoff/2026-11-04-question-jwt-health.md, then hand back to implementer.
Completed: plan, review, spec, S1, S2 except route wiring
Remaining: answer question, S2 route wiring, S3, verify
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-question-jwt-health.md
Last agent: claude-sonnet-5-5@claude-b/host-3
```

```
docs(question): /health cannot carry a token

Relay-To: planner
Relay-Doc: docs/handoff/2026-11-04-question-jwt-health.md

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 07 — planner answers

修訂 spec（§3.2 排除 `/health`），回答寫在問題文件最後。Pipeline 位置不變，回到 implementer。

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: implementer
Next action: Finish S2 per revised spec §3.2 (/health excluded), then S3.
Completed: plan, review, spec (rev 2), S1, S2 except route wiring
Remaining: S2 route wiring, S3, verify
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4; /health stays public
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-question-jwt-health.md
Last agent: claude-opus-5-5@claude-a/host-1
```

```
docs(spec): exclude /health from JWT middleware

Relay-To: implementer
Relay-Doc: docs/handoff/2026-11-04-question-jwt-health.md

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 08 — implementer finishes

```
Status: handoff
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: planner
Next action: Verify docs/handoff/2026-11-04-result-jwt.md against spec §5 acceptance criteria.
Completed: plan, review, spec (rev 2), S1, S2, S3
Remaining: verify
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4; /health stays public
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-result-jwt.md
Last agent: claude-sonnet-5-5@claude-b/host-3
```

```
feat(auth): protect API routes with JWT middleware (S2, S3)

Relay-To: planner
Relay-Doc: docs/handoff/2026-11-04-result-jwt.md

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 09 — planner verifies

自己重跑 spec §5 的驗證指令，不只看 result 報告。

```
Status: done
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: user
Next action: Decide whether to run an audit or close the task.
Completed: plan, review, spec (rev 2), implement, verify (pass)
Remaining:
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4; /health stays public
Artifacts: docs/handoff/2026-11-04-verify-jwt.md
Last agent: claude-opus-5-5@claude-a/host-1
```

```
docs(verify): JWT auth verified against spec §5

Relay-To: user
Relay-Doc: docs/handoff/2026-11-04-verify-jwt.md

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 10 — auditor (optional)

使用者說：「你是 auditor，審 JWT auth 這個任務。」auditor 先只看目標、程式碼和產出，之後才讀 state 和 review。

```
Status: done
Task: Add JWT auth to the API
Pipeline: planner -> reviewer -> planner -> implementer -> planner -> user
Current: user
Next action: Decide which findings in docs/handoff/2026-11-04-audit-jwt.md to adopt.
Completed: plan, review, spec (rev 2), implement, verify (pass), audit
Remaining:
Decisions: HS256, secret from JWT_SECRET; token TTL 15 min; 401 body format per spec §4; /health stays public
Artifacts: docs/handoff/2026-11-04-verify-jwt.md, docs/handoff/2026-11-04-audit-jwt.md
Last agent: gpt-6-astra@codex/host-2
```

```
docs(audit): independent audit of JWT auth

Relay-To: user
Relay-Doc: docs/handoff/2026-11-04-audit-jwt.md
```

使用者若採納審計意見，就開新任務（回到步驟 00），例如 `Pipeline: planner -> implementer -> planner -> user`。

---

## 11 — user asks a role to remember something

任何時候對任何 agent 說：「implementer 記住：測試一律用 `pnpm test -- --silent`。」
agent 寫進 `.handoff/roles/implementer/memory.md`（見 [roles/implementer/memory.md](roles/implementer/memory.md)），**不寫原生 memory**，立即 commit 並 push：

```
docs(memory): implementer test command

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

之後任何帳號、任何機器、任何 provider 的 implementer 開工時都會讀到。
