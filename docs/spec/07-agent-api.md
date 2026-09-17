# 07 · 外部 agent 與共用操作 API

狀態：V1 目標契約；以下命令尚未實作。依據：[00](00-decisions.md)、[03](03-vault-data.md)、[05](05-modules.md)。

## 基本原則

人類 UI、CLI、HTTP 和 MCP 調用同一個 CommandService。直接讀文件也是可用入口，但競爭寫入優先使用受協調接口。「任何 agent」表示不綁特定模型或供應商，不表示所有客戶端無需配置即可使用。

所有命令明確指定 project、object／document 和 revision。不依賴目前打開的分頁、目前選取或使用者視口；背景修改另一集不得自動切走人類正在閱讀的畫布。

每個命令公開 inputSchema、outputSchema、permissions、讀寫性質、是否需 renderer、可逆性、單位、錯誤碼及實作狀態。所有 ID 必須來自讀取／建立結果，不從顯示名稱猜測。

## 入口

CLI 提供 `sceneweave capabilities --json` 和 `sceneweave invoke <command> --project <path> --args <file> --json`。stdout 只輸出結構化結果，診斷與進度用 stderr；失敗返回非零 exit code。

HTTP 以 `/v1/capabilities` 查目錄，`POST /v1/commands/<commandName>` 調用共用命令，以游標讀取 `/v1/events`。MCP 用工具列表和 Schema 映射相同命令，必要時以資源返回大文本或媒體，不把影片 base64 放进結果。

MCP 版本在實作時鎖定並檢查客戶端支持。WebMCP 非首版前置条件，不建立另一套資料／命令核心。

## 能力目錄

返回已安裝且目前可執行的能力、所需權限、環境限制、capabilitiesRevision。另有不可執行的 module catalog 用於顯示缺失／停用狀態，不能把目錄項目假裝成可用工具。

安裝、卸載、停用或故障後，capabilitiesRevision 改變並發事件。MCP 支持相應通知時發出 tools/list_changed；其他客戶端可重新查詢。若持有舊名稱，dispatcher 回具體 module missing／disabled／unknown 錯誤，不偷偷使用另一個模組替代。

## 核心命令目錄

| 範圍 | 命令 | 最少輸入／返回 |
| --- | --- | --- |
| 發現 | capabilities.list、schemas.get | scope／name；Schema、版本、權限與狀態 |
| 項目 | project.inspect | projectId；摘要、根、問題、新鮮度 |
| 查詢 | entities.query、entity.get | 類型／條件／cursor 或 objectId；分頁／版本 |
| 文件 | document.read、document.search | objectId、行窗口／query；精確範圍與 hash |
| 文件寫入 | document.patch、files.import、files.move | 前置版本、精確 patch／文件／目標；變更與 ID |
| 畫布 | board.get、board.query、board.edit | boardId、bounds／ops；節點、關係與版本 |
| 製作 | episode.create、scene.create、shot.create、entity.update | 正式父容器／欄位；新身份或修訂 |
| 資產 | assets.get、assets.publish、variants.validate | 資產／作者稿／變體；版本與明確衝突 |
| 提示詞 | prompts.get、prompts.resolve、prompts.export | profile／shot／context；文字、來源與快照 |
| 依賴 | references.list、dependencies.inspect | objectId、revision；上下游與 missing |
| 變更 | changes.preview、changes.commit、changes.get | 操作、read-set／previewId／requestId；diff／receipt |
| 任務 | jobs.get、jobs.cancel | jobId；狀態、進度、輸出或中止 |
| 審核 | review.submit、review.decide | 精確 target revision、身份授權；紀錄 |
| 交付 | delivery.package、delivery.inspect | 鎖定清單；文件、hash、依賴檢查 |
| 模組 | modules.list、modules.inspect | moduleId；靜態契約與生命週期 |
| 模組管理 | modules.install、modules.enable、modules.disable、modules.uninstall | 額外管理權與確認；影響／結果 |
| 診斷 | diagnostics.run | scope；可機械驗證的問題，不含主觀美學判斷 |

專業模組自行註冊命名空間，如 stage.read、stage.edit、stage.capture、edit.apply、media.transcode。核心只做 generic dispatch，不引用其專業型別。

## 命令 envelope

讀命令至少有 projectId 和 arguments。正式寫入增加 requestId、expectedRevisions；actor／授權由認證環境提供，不接受客戶端自填更高身份。

```json
{
  "projectId": "project_demo",
  "requestId": "request_demo_001",
  "expectedRevisions": {
    "shot_demo": "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  },
  "arguments": {
    "objectId": "shot_demo",
    "patch": {"selectedTakeId": "take_demo_003"}
  }
}
```

這是示例身份與 hash，不是有效線上物件。一般輸出為 `{ok, data, revisions, warnings, receiptId}`；錯誤為 `{ok:false, error:{code,message,details,retryable}}`。hash、資料量上限、時間單位不得藏在自然語言說明中。

## 預檢與並發

preview 解析 read-set／write-set、鎖定欄位、引用及權限，返回具體 diff、風險、預計受影響物件、有效期限及 previewId。commit 重新驗證版本與權限；preview 不是鎖或完成承諾。

同 requestId 和相同 canonical request 在重試時回原 receipt，不重複新建資產。同一鍵不同請求返回 IDEMPOTENCY_CONFLICT。冪等範圍是 principal＋project＋requestId；receipt 持久化，不能重啟就忘記。

單文件與跨文件操作檢查各自 hash，不用一個全劇全域 revision 令兩個無關鏡頭互相阻擋。成功後提供新 hash、持久操作記錄和可逆性；undo 也要檢查衝突，不覆蓋別人之後的編輯。

長任務先捕捉精確輸入 snapshot，再執行。完成後另行提交輸出；輸入被改不使舊結果偷偷成為新版本。外部生成只做顯式匯入，不在核心代為連接服務。

## prompt 資料包

`prompts.resolve` 可接受 shotId，或由呼叫者提供具體 subjectBindings、setBinding、variantRefs、blockRefs 和 variables。所有發布引用必須精確；缺引用、未解變量、互斥變體、固定欄位衝突要返回明確錯誤。

返回 sections、resolvedText、negativeText、附件用途／順序、sourceMap、sourceRevisions、composerVersion 和 warnings。resolve 不寫檔，export 才建立快照。缺少三維模組不能影響此能力。

`project.inspect`／場次 context 查詢可組裝劇本、相關人設、搭景、鏡頭與未解審核；只做引用解析，不生成 AI 摘要。支持 metadata-only，按需讀取長文本。

## 分頁與資料量

列表以穩定排序、limit 和 opaque cursor 讀取，返回 nextCursor、snapshotRevision／freshness。游標過期明確返回 CURSOR_EXPIRED，不把截斷內容當成全量。長文档支持行窗口、精確搜尋及 contentHash；媒體返回授權資源引用。

## 錯誤分類

至少包括 VALIDATION_ERROR、NOT_FOUND、REVISION_CONFLICT、IDEMPOTENCY_CONFLICT、PERMISSION_DENIED、LOCKED_FIELD_CONFLICT、VARIANT_CONFLICT、UNRESOLVED_VARIABLE、MISSING_REFERENCE、UNSUPPORTED_FORMAT、MODULE_NOT_INSTALLED、MODULE_DISABLED、CAPABILITY_UNAVAILABLE、BUSY、CANCELLED、IO_ERROR、RECOVERY_REQUIRED。

NOT_FOUND 不自動建立同名物件；CONFLICT 不強制覆寫；MODULE_NOT_INSTALLED 不自动安裝；PERMISSION_DENIED 不擴大權限。错误中不得洩漏未授權根目錄或 token。

## 任務與完成證據

任務狀態 queued、running、succeeded、failed、cancelling、cancelled、interrupted 均持久化。成功需要實際輸出、文件驗證與來源清單，不能只回一句 done。取消不可被誤標 succeeded。

文字修改返回 diff；匯入返回文件身份、hash 和實際元資料；渲染返回真實圖片／影片；機械診斷不替代視覺驗收。需要重啟的 renderer 不意味任务自动續跑；不支援續跑就標記 interrupted 並顯式重試。
