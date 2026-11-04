# agent_handoff Audit Report

日期：2026-11-04（Asia/Taipei）
基準：`7630c8c`，另檢視工作目錄中尚未追蹤的 `default_instruction/`。
範圍：README、全部模板協定／角色／記憶與狀態檔、JWT walkthrough、全域指令樣本及 Git 交接行為。

本專案是文件式工作流程模板，沒有應用程式、套件依賴或可執行的 JWT 實作。主要風險是交接狀態遺失、路由歧義與規則互相阻擋。發現 **7 項 P2、1 項 P3**；未識別 P0/P1 問題。建議先修正 P2，再進行真實專案跨環境試跑。

嚴重程度：P1＝應立即處理的重大風險；P2＝特定正常使用情境會失敗，應修正；P3＝較低影響的相容性／一致性問題。這些分級是工作流程可靠性評估，不代表已發現可利用的資安漏洞。

## F1 — P2：唯讀角色無法合法完成必要的狀態交接

- 證據：`template/.handoff/PROTOCOL.md:6,28-30,35-37`；`template/.handoff/roles/reviewer/role.md:12`；`template/.handoff/roles/auditor/role.md:14,19`。
- 問題：角色規則優先於 protocol，但 reviewer 只能修改 review file，auditor 只能修改 audit report。兩者因此都無法依 protocol 修改 `state.md`，auditor 同時又被要求交棒給 user。
- 影響：遵守角色限制會留下過期的 Current／Next action；更新狀態則違反角色限制。JWT 範例 steps 02、10 有更新 state，沒有解決規範上的衝突。
- 建議：唯讀限制明確豁免交接 metadata，列出允許修改的 state 欄位；或指定另一個主體負責 metadata，但不得依靠未定義的協作角色。
- 驗收：reviewer／auditor 都能產出報告並更新交接狀態，且不修改被審查的內容。

## F2 — P2：reviewer 回傳規則與公開推薦的 Pipeline 不一致

- 證據：`README.md:41,43`；`template/.handoff/PROTOCOL.md:42`；`template/.handoff/roles/reviewer/role.md:20`。
- 問題：protocol 要求前往 Pipeline 下一站；reviewer 則一律要求回到 artifact author。`implementer -> reviewer -> user` 在審查通過後，依角色規則仍須回 implementer。高風險 Pipeline 的 code reviewer 同樣會回 implementer，而非下一站 planner。
- 影響：公開推薦的正常流程無法依全部規則完成，可能多出未規劃的往返。
- 建議：依 verdict 分流：approve 前往下一站；changes-requested 回作者；明確規定修正後是否重審，以及如何回到原 Pipeline。
- 驗收：對 README 所列的兩種受影響 Pipeline，分別走過 approve 與 changes-requested 路徑，確認下一位負責人唯一。

## F3 — P2：等待腳本用歷史事件判斷當前責任，會誤醒或漏接

- 證據：`template/.handoff/PROTOCOL.md:83-90`。
- 問題：固定 `old=HEAD` 後，搜尋 `old..@{u}` 任何符合角色的訊息，沒有判斷最新 state 是否仍由該角色負責。若待處理交接已經 pull，範圍為空，也不會喚醒。
- 實測：同一區間先交給 reviewer、再交給 implementer，reviewer 的匹配結果仍為 True，但遠端 Current 已是 implementer；pull 後 Current 為 implementer，對 implementer 的匹配結果卻是 False。
- 影響：不負責下一步的 agent 被喚醒，或真正的接手者持續等待。
- 建議：先檢查最新遠端 state 的 Status／Current，另以已處理的交接 commit 或事件 ID 去重；喚醒後再次核對 ownership。fetch／upstream 錯誤也應有明確退出方式。
- 驗收：涵蓋已 pull 的交接、已被後續交接取代的訊息、無新交接及 fetch 失敗。

## F4 — P2：重跑安裝指令會覆寫進行中的狀態與客製內容

- 證據：`README.md:7-15`，尤其第 8 行 `cp -r template/.handoff template/docs <project>/`。
- 實測：第一次安裝後將目標 `.handoff/state.md` 設為 `Status: in-progress` 與既有 Task，再執行相同命令，內容被重設為模板的 `Status: idle`。
- 影響：已有同名 state、memory 或 role 檔案的專案重跑安裝時，這些檔案會被覆寫；未提交的資料無法由 Git 還原。
- 建議：安裝前檢查 `.handoff` 是否存在；存在時停止並提供人工合併流程。區分首次安裝與更新，避免把覆寫當作更新策略。
- 驗收：全新目標可安裝；已有客製內容的目標應停止或保留原內容，不得靜默重設狀態。

## F5 — P2：既有專案的安裝分支漏掉 state-relay override

- 證據：`README.md:11-12`；`template/AGENTS.md:3-5`；`default_instruction/AGENTS.md:26-28`。
- 問題：既有 AGENTS.md 的安裝方式只要求附加 protocol pointer，沒有帶入模板第 5 行禁止 `.agent-state.md` 的 override。搭配提供的全域指令樣本時，`.agent-state.md` 與 `.handoff/state.md` 都仍有維護要求。
- 實測：依 README 的既有專案分支安裝，AGENTS.md 有 protocol pointer，但沒有 state-relay override。
- 影響：同一任務可能產生兩套進度來源；不同環境依不同規則接手時容易讀到不一致的狀態。
- 建議：提供完整的可合併指令區塊，包含 pointer、唯一狀態來源與 legacy migration；或在 protocol 本身明確取代舊 state-relay 規則。
- 驗收：新舊專案兩種安裝分支，都只有一個明確的狀態來源。

## F6 — P2：重複角色與 push-back 缺少明確的 Pipeline 位置及返回規則

- 證據：`template/.handoff/state.md:3-5`；`template/.handoff/PROTOCOL.md:22-24,42,46-50`；`examples/jwt-walkthrough/steps.md:196-203`。
- 問題：Pipeline 多次出現 planner，但只存 Current 的角色名稱，沒有 step ID／cursor。push-back 要回上一位交棒者，卻未定義答覆後回哪一站、是否推進 Pipeline。範例宣稱「Pipeline 位置不變」，該位置與規則未在 protocol 定義。
- 影響：遇到非線性的退回、重審或中斷恢復時，下一個 agent 必須自行從自然語言與歷史推斷，可能跳過驗證或重做規劃。
- 建議：加入簡單的 step ID／cursor 與 return-to，明定 push-back 暫停原步驟、答覆後返回，以及哪些轉移會推進 cursor。維持純文件格式即可。
- 驗收：在每次交接後模擬新 session，僅依 protocol 與 state，就能唯一判定下一步；涵蓋兩次 push-back 和重複角色。

## F7 — P2：Small fix Pipeline 沒有產生 implementer 必需的 spec

- 證據：`README.md:40-41`；`template/.handoff/roles/implementer/role.md:6,12-15,19`。
- 問題：README 推薦直接從 implementer 開始，但角色要求輸入 spec、不得做 planning decisions，完成條件也依賴 acceptance criteria。文件未說明小修是否可使用使用者指令作為輕量 spec。
- 影響：沒有預先建立 spec 的一般小修，agent 只能退回詢問 planner；公開宣稱的簡化 Pipeline 不能自足。
- 建議：定義小修所需的最小輸入（範圍、預期行為、驗證方法），允許明確的使用者指令充當 spec；不完整時交回 user 補充或採用完整 Pipeline。
- 驗收：以沒有任何 artifact 的單點小修啟動 `implementer -> user`，能判定輸入是否足夠並完成，不必新增未列出的 planner。

## F8 — P3：範例 Relay 欄位不是 Git 可解析的 trailers

- 證據：`template/.handoff/PROTOCOL.md:37` 要求 final paragraph；`examples/jwt-walkthrough/steps.md:53-56,107-110,268-271` 在 Relay 與 Co-Authored-By 之間插入空白行。
- 實測：將相同格式交給 `git interpret-trailers --parse`，僅得到 Co-Authored-By，沒有 Relay-To／Relay-Doc。
- 影響：目前全文 grep 仍能匹配，但使用標準 Git trailer parser 的整合會漏掉交接資訊，且範例不符合自己的協定。
- 建議：將所有 trailers 放在同一個最末段落，中間不要空白行。
- 驗收：所有 commit 範例經 Git trailer parser 解析後，都能取得預期的 Relay 欄位。

## 其他觀察與驗證限制

- `Status: done` 有列入合法值，但 protocol 的一般完成交接一律寫 handoff；範例最終則寫 done。建議補上完成、阻擋與等待使用者決策的轉移表。
- JWT 範例的 `/api/health` 與 `/health` 名稱、spec §6／§7 引用有不一致，降低教學精確度；這些是示例文件問題，不是本 repository 的真實 auth 漏洞。
- 尚未追蹤的 `default_instruction/` 不會隨目前 commit 傳到其他 clone；本報告有檢視其內容，但不視為已發布的專案檔案。
- 正向設計包括：跨供應商的文字格式、角色職責分離、明確 acceptance criteria、planner 實際重跑驗證，以及不依賴私人記憶的交接方式。上述缺陷可以透過小幅修訂協定與範例處理，不必先引入 CLI。
- 驗證使用 `/tmp` 下隔離的 bare remote 與兩個 clones，沒有連線外部服務，也未對本專案 pull／push。完成了過期通知、已 pull 通知、既有專案安裝、重複安裝與 Git trailer parsing 的重現。
- 未執行真實跨 provider／帳號交接；未驗證工具供應商對 instruction／memory 的實際載入語意；沒有執行示例中的 pnpm 指令，因對應程式與測試不存在。
- 本次只產出 audit report 與依使用者要求更新工作狀態；未修正模板。優先順序建議：F1/F2/F6 的交接語意 → F4/F5 安裝安全 → F3 等待 → F7/F8 文件契約。
