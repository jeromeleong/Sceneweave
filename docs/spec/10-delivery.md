# 10 · 開發順序與交付門檻

狀態：設計計劃；未實作、未執行應用測試。依據：[00](00-decisions.md)。

## 本輪範圍

本輪只確立 Markdown 規格、開發指引、範例和驗收矩陣。源碼、依賴、build、測試、安裝器和使用者資料不變。倉庫目前仍有內建 AI 和舊導演台實作，不能因本文件寫了「移除」就認為已刪除。

## 直接重構的定義

在同一倉庫內建立新的核心與模組架構，可分工作包與提交完成；「分段實作」不等於保存舊相容。新產品不提供 `.director` 匯入器、舊 API alias、mixed-format decoder、舊 UI 容器或新舊双寫。

旧演算法可以作為移植材料与比對來源；公開型別、工程格式、Svelte UI 及調用契約按新規格完成。未知旧文件僅標示 unsupported，不主動破壞或清除。

## 工作包

| 包 | 內容 | 門檻 |
| --- | --- | --- |
| W01 | Svelte 核心外殼、CommandService、Vault 原型、第一張真畫布 | 零模組啟動與保存；不存在 Stage import |
| W02 | 原生資料格式、ID、監看、索引、revision、交易與恢復 | 外部改檔、跨文件失敗與搬遷可驗證 |
| W03 | 角色／搭景、PromptBlock、Profile、變體、組裝和快照 | 固定文字保真、精確 pin、無模型調用 |
| W04 | 集／場／鏡頭、集合、表格／分鏡條、Take、審核 | 多視圖一份資料，跨集引用不複製 |
| W05 | Module SDK／Host、安裝／停用／卸載、通用 fallback | 真正移除包和能力但保留項目所有內容 |
| W06 | CLI、HTTP、MCP 同源及權限／事件／持久 receipt | 外部 agent 不看 UI 完成核心流程 |
| W07 | 官方可選導演台：Svelte UI、新 Stage schema、算法移植 | 缺模組不影響核心，輸出可追溯 |
| W08 | 可選剪輯器與媒體處理模組 | 專業時間線與核心資料分開，能力缺失可降級 |
| W09 | 安全、性能、打包、獨立發布與更新驗證 | core-only 與模組發行包各自驗收 |

命令契約、CLI 測試入口與模組邊界從 W01 開始設計，W06 是完整化，不是最後加一層自然語言代理。W05 可用最小測試模組證明卸載，不能等導演台全部完成才發現核心耦合。

第一個可用核心可以在 W01–W06 完成後獨立發布。官方導演台不是核心發布前置条件，也不因它未完成就把核心說成空殼。

## 現有內容的處理方向

| 路徑範圍 | 開發階段處理 |
| --- | --- |
| src/main.ts、src/app-context.ts、src/ui | 用新 Svelte 外殼、命令和模块會話取代，不包裝舊 DOM 應用 |
| src/engine.ts、animation、cinematography、lighting 等 | 可選擇移植算法到 director-stage，核心不引用 |
| src/model.ts、scenes、storage、project-save | 新格式重做，沒有 legacy reader／遷移器 |
| src/automation | 可借鑑驗證、預檢、冪等，重新綁定新對象／命令 |
| desktop/ai-host.cjs、ai-conversation.cjs、providers.cjs、director-prompt.cjs、model-limits.cjs、src/ui/ai-panel.ts | 新產品移除模型執行與配置，不僅隱藏 UI |
| desktop/mcp-* | 保留有價值安全概念，接入共用命令，不維護舊 director_* alias |
| skills/director-desk | 作歷史來源；新外部指南按能力和新格式重寫，不當內建代理 |
| desktop 發布／更新設定 | 新產品身份、目錄和發布源，拒絕沿用舊自動更新 |
| LICENSE、NOTICE、合法素材 | 保留來源與授權；不因更名刪除歸屬 |

此表是重構決策，不是逐檔實作完成證明。實際移植前讀取相關程式、授權與測試，不照路徑名稱盲刪。

## 不做的工作

不造模型商店、生成積分、AI 聊天、不替外部 agent 選模型、不加語意搜索、不做舊工程轉換、不先重造完整 Premiere／Blender、不把雲端帳號或多人即時編輯當首版必需。

## 每包交付證據

提交需列範圍、改動檔案、真正執行的命令與環境、通過／失敗、未驗證平台和發布門檻。設計文件不算成功執行證據。測試失敗不能以刪測試或隱藏功能混過。

新增模組需獨立 artifact、manifest、Schema、權限、通用工程示例和卸載測試。新增 command 需輸入／輸出契約、錯誤和同源 UI／CLI／HTTP／MCP 測試。

## 發布門檻

核心單獨安裝、斷網使用、無 key、無模組、不可自動跑項目程式；本地和服務端根權限正確；崩潰恢復、卸載保全、提示詞 pin 和已交付版本不可變。

目標支援平台逐個提供實際測試結果，不由 Electron 可跨平台推出全部安裝包已驗證。精確版本、簽名、codec 與硬件性能在發布表中記錄。
