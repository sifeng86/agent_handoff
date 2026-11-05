# Steps

Each step shows `.handoff/state.md` after the step and the commit that carries it. `Relay-To` in `state.md` decides whose turn it is; the `Relay-To:` commit trailer is an optional log.

---

## 00 — user opens the task

使用者對任一 agent 說：「開任務：JWT auth，範圍 login + middleware，不做 refresh token，pipeline 用標準流程。」這是任務模式，agent 寫入任務卡。

```
Status: handoff
Relay-To: planner
Task: Add JWT auth to the API
Pipeline:
- [ ] planner: plan
- [ ] reviewer: review plan
- [ ] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Write a plan for JWT auth. Scope: login endpoint + auth middleware. Out of scope: refresh tokens.
Completed:
Decisions:
Artifacts:
```

```
chore(handoff): open task JWT auth

Relay-To: planner
```

---

## 01 — planner writes the plan

`claude-opus-5-5`，Claude A 帳號，host-1。使用者說「接手」：`Relay-To` 是 planner，所以它是 planner。

```
Status: handoff
Relay-To: reviewer
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [ ] reviewer: review plan
- [ ] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Review docs/handoff/2026-11-04-plan-jwt.md against the goal in Task and the current code in src/api/.
Completed: plan
Decisions:
- HS256 with secret from env (plan §3)
Artifacts: docs/handoff/2026-11-04-plan-jwt.md
```

```
docs(plan): JWT auth plan

Next unchecked item is the plan review.

Relay-To: reviewer
Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 02 — reviewer reviews the plan

`gpt-6-astra`，Codex 帳號，host-2。換 provider、換機器：`git pull` 之後說「接手」就好。結論是 `changes-requested`；下一個沒打勾的項目本來就是作者 planner，所以照常往下交。

```
Status: handoff
Relay-To: planner
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [ ] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Decide on each finding in docs/handoff/2026-11-04-review-plan-jwt.md, then write the spec.
Completed: plan, review (changes-requested)
Decisions:
- HS256 with secret from env (plan §3)
Artifacts: docs/handoff/2026-11-04-plan-jwt.md, docs/handoff/2026-11-04-review-plan-jwt.md
```

```
docs(review): plan review for JWT auth, changes requested

Changes requested; the author (planner) is the next item anyway.

Relay-To: planner
```

---

## 03 — planner writes the spec

回到 `claude-opus-5-5`，Claude A 帳號，host-1。

```
Status: handoff
Relay-To: implementer
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Check docs/handoff/2026-11-04-spec-jwt.md against the code, then implement step S1.
Completed: plan, review, spec (review findings resolved in spec §7)
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md
```

```
docs(spec): JWT auth spec, review findings resolved

Relay-To: implementer
Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 04 — implementer checkpoint

`claude-sonnet-5-5`，Claude A 帳號，host-3。開工前先依 PROTOCOL §3.5 對照程式碼檢查 spec，沒問題才動手。完成 S1 後 checkpoint，`Relay-To` 不變。

```
Status: in-progress
Relay-To: implementer
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Implement S2 (auth middleware) per spec §3 S2.
Completed: plan, review, spec, S1 (token issue on POST /login, tests pass)
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md
```

```
feat(auth): issue JWT on login (S1)

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 05 — quota runs out; another account continues

Claude A 的 quota 用完。使用者在同一台主機用 Claude B 開 session，說「接手」。這是同一個角色換人，**不是交棒**。
上一個 session 留下沒 commit 的修改：先看 diff，再接著做（PROTOCOL §3.4），絕不丟棄。S2 做到一半再 checkpoint 一次：

```
Status: in-progress
Relay-To: implementer
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Finish S2: apply middleware to routes in spec §3 S2; expired-token test still missing.
Completed: plan, review, spec, S1, S2 middleware core
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md
```

```
wip(auth): JWT middleware core (S2 partial)

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 06 — implementer asks the planner

spec 要求所有 `/api/*` 都套用 middleware，但 `/api/health` 是外部監控在呼叫，沒辦法帶 token。不能用猜的，所以去問寫 spec 的人。交棒前先把不受答案影響的部分做完（S3 的其他測試）。

```
Status: handoff
Relay-To: planner
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Answer docs/handoff/2026-11-04-question-jwt-health.md, then hand back to implementer.
Completed: plan, review, spec, S1, S2 except route wiring, S3 tests not involving /api/health
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-question-jwt-health.md
```

```
docs(question): /api/health cannot carry a token

Spec conflict with the uptime monitor; the planner owns the spec.

Relay-To: planner
Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 07 — planner escalates to the user

讓一個 endpoint 不需要驗證，屬於安全和範圍的取捨，該由使用者決定。planner 把兩個選項寫進問題文件，交給 user。

```
Status: blocked
Relay-To: user
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: User: choose option 1 (keep /api/health public) or 2 (monitor token) in docs/handoff/2026-11-04-question-jwt-health.md.
Completed: plan, review, spec, S1, S2 except route wiring, S3 tests not involving /api/health
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-question-jwt-health.md
```

```
docs(question): escalate /api/health exposure to user

Making an endpoint public is a security/scope decision.

Relay-To: user
Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 08 — user decides and goes straight to the planner

使用者在 planner 的對話裡說：「選 1，/api/health 保持公開。」planner 先 `git pull`，看到 `Relay-To: user`，往前讀就知道是哪個問題。它把決定記進 `Decisions`、`Relay-To` 改成自己、修訂 spec（rev 2），再交給 implementer。

```
Status: handoff
Relay-To: implementer
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [ ] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Finish S2 per revised spec §3 S2 (/api/health excluded), then the rest of S3.
Completed: plan, review, spec (rev 2), S1, S2 except route wiring, S3 tests not involving /api/health
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
- 2026-11-04 user: keep /api/health public (option 1)
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-question-jwt-health.md
```

```
docs(spec): exclude /api/health from JWT middleware (rev 2)

User chose option 1; back to the implementer to finish S2.

Relay-To: implementer
Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 09 — implementer finishes

```
Status: handoff
Relay-To: planner
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [x] implementer: implement
- [ ] planner: verify
- [ ] user: accept
Next action: Verify docs/handoff/2026-11-04-result-jwt.md against spec §5 acceptance criteria.
Completed: plan, review, spec (rev 2), implement (S1-S3)
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
- 2026-11-04 user: keep /api/health public (option 1)
Artifacts: docs/handoff/2026-11-04-spec-jwt.md, docs/handoff/2026-11-04-result-jwt.md
```

```
feat(auth): protect API routes with JWT middleware (S2, S3)

Relay-To: planner
Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

---

## 10 — planner verifies

自己重跑 spec §5 的驗證指令，不只看 result 報告。只剩最後的 `user` 項目，所以設為 `done`。

```
Status: done
Relay-To: user
Task: Add JWT auth to the API
Pipeline:
- [x] planner: plan
- [x] reviewer: review plan
- [x] planner: spec
- [x] implementer: implement
- [x] planner: verify
- [ ] user: accept
Next action: User: accept, or ask for an audit.
Completed: plan, review, spec (rev 2), implement, verify (pass)
Decisions:
- HS256, secret from JWT_SECRET; token TTL 15 min; 401 body per spec §4
- 2026-11-04 user: keep /api/health public (option 1)
Artifacts: docs/handoff/2026-11-04-verify-jwt.md
```

```
docs(verify): JWT auth verified against spec §5

Relay-To: user
Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

## 11 — auditor (ad hoc)

使用者說：「你是 auditor，審 JWT auth 這個任務。」auditor 不在 Pipeline 裡，所以**不握棒子**：不動 `state.md`，它停留在步驟 10 的內容。auditor 先只看目標、程式碼和產出，之後才讀 state 和 review；只 commit 自己的報告，然後直接回報給使用者。

```
docs(audit): independent audit of JWT auth
```

使用者若採納審計意見，就開新任務（回到步驟 00），例如只有 `implementer: implement`、`planner: verify`、`user: accept` 三個項目的 checklist。

---

## 12 — user asks a role to remember something

任何時候對任何 agent 說：「implementer 記住：測試一律用 `pnpm test -- --silent`。」
即使是在對話模式，這也是使用者明確要求的寫入：agent 只寫 `.handoff/roles/implementer/memory.md`（見 [roles/implementer/memory.md](roles/implementer/memory.md)），**不動 `state.md`，不寫原生 memory**，寫完立即 commit 並 push：

```
docs(memory): implementer test command

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
```

之後任何帳號、任何機器、任何 provider 的 implementer 開工時都會讀到。
