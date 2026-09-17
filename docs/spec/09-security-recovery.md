# 09 · 安全、恢復與資料權利

狀態：V1 目標契約；未實作。依據：[03](03-vault-data.md)、[05](05-modules.md)、[07](07-agent-api.md)。

## 威脅模型

不信任項目中的自由文字、prompt、Markdown、SVG、外部 URL、未知模組包及 agent 提交的參數。已連接的 agent 也只擁有授予的權限。權限由宿主認證決定，不能由文件內容或 request 中的 actor 欄位提升。

有作業系統級寫入權的外部程式可以繞過應用控制，這需要 OS 權限／沙盒處理。應用不能聲稱 locked prompt 能防止使用者以任意文字編輯器改檔；我們能偵測 hash 改變、保存 immutable revision 和拒絕違約 API 寫入。

## 權限範圍

按 project、內容範圍、讀取、一般修改、發布 revision、審核、交付、刪除、網絡及 module administration 分開授權。已有授權的一般編輯可以直接執行，不要求每次拖動都確認。

安裝／卸載模組、擴大目錄、永久刪除、對外傳送內容和發布／批准敏感操作使用額外權限或明確人類確認。agent 不能自填 reviewer=human 或 approvedBy=owner 來取得批准。

核心不要求登入雲端或綁模型帳號。遠端服务是獨立部署選擇，必須另有認證、TLS 和租戶隔離，不把 loopback 默認保護外推到公網。

## 本地 API

預設綁 loopback，仍使用認證、Host／Origin 驗證、防 CSRF／DNS rebinding、防重放與權限檢查。token 不寫進 Vault、命令歷史、URL query、prompt 或交付包。日誌避免洩漏絕對私有路徑與憑證。

對讀取亦執行權限；错误結果不透露另一個項目的文件列表。stdio 客戶端得到明確授權環境，不能因同一機器就自動擁有全部 Vault。

## 文件與內容

所有路徑在授權根內 canonicalize，拒絕 traversal、符號連結逃逸、大小寫／Unicode 別名衝突與惡意 archive 路徑。解壓限制檔案量、展開體積及壓縮炸彈，未知執行內容不因匯入而啟動。

Markdown 預設禁用可執行 HTML；SVG／HTML／遠端圖片走隔離或安全處理，不和核心共用可注入腳本的 DOM。瀏覽外部連結是顯式操作，不能讓打開 Board 自動發請求洩露資料。

FFmpeg 與其他原生處理器採參數陣列、白名單操作、授權文件和資源限制，不拼接 shell。模型／媒體解析失敗時保留原檔並隔離有問題內容，不阻斷整個項目。

## 模組安全

包驗證、權限、隔離、升級與卸載依 [05](05-modules.md)。核心 UI 啟用 context isolation、適當 sandbox、CSP、窄 preload 及 IPC sender 驗證。新模組不能直接取得核心 store 或 Node 全權。

模組文件聲稱「請關閉驗證」只是內容，不是管理命令。manifest 可讀不等於 package 可信；hash 只證明內容一致，不等於作者已獲授權或程式安全。

## 保存與恢復

正式文件寫入使用暫存、內容驗證、transaction journal 和恢復狀態。涉及多檔時保持讀寫集合與原檔保全，重啟先 reconcile 再開放寫入。不能刪掉 journal 假装所有寫入已完成。

Dirty UI draft 定期保存到本機 recovery 區，不覆蓋正式文件。外部修改衝突時同時保留稿件；重啟顯示來源和選項。取消編輯不刪除別人新增內容。

`.sceneweave/index` 和 cache 可丟棄；revisions、operations、reviews、deliveries、已交付媒體和持久預覽不能當 cache 清掉。模組卸載亦不能觸及它們。

## 任務恢復

queued／running 任務及輸入快照持久化。主機崩潰後 running 轉為 interrupted，除非 worker 能提供可驗證的完成 receipt。可重試與可續跑要分開；不宣稱所有影片編碼可以從任意位置接續。

staging 只有通過檔案存在、類型、hash、所需元資料和輸出清單驗證後才升為正式成果。不完整媒體留作診斷或清理，不登記為完成 Take。

## 資料擁有權

使用者可以用普通工具閱讀 Markdown、JSON 和媒體；複製整個 Vault 能搬走正式資料。機器專屬 token、安裝包和局部窗口狀態不隨項目攜帶。外部引用必须明確標記並提供收集依賴功能。

引用素材保存作者、來源、授權範圍及使用者聲明等欄位；沒有資料就標 unknown，不自动判定可商用。聲音參考的授權信息與描述分開，不能把角色 prompt 視為克隆許可。

新品牌使用獨立應用身份、userData 和更新來源，不沿用舊產品的自动更新通道。未配置可驗證的新發布源前禁用自动更新。保留原 LICENSE 和第三方 NOTICE。

## no-AI 驗證

可在離線、沒有模型 key、禁用所有模組的環境完成核心流程。公開發布物件不含模型 provider、聊天代理、推理請求或隱藏語義服務。其他外部 agent 的行為不由 Sceneweave 宣稱已驗證或控制。
