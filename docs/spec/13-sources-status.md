# 13 · 資料來源、基線與實作狀態

記錄日期：2026-09-17。本文件區分倉庫證據、外部技術參考與 Sceneweave 自訂設計。

## 倉庫基線

本次讀取的 main 基線是 `eee9234ae38ffc61a7d4576298a0f23739474597`，commit 的 tree 為 `c851724e5c733a987bb8cc9fd0a37b638ce5cd26`。倉庫為 jeromeleong/Sceneweave。

| 來源 | 可支持的事實 | 不可由此推出 |
| --- | --- | --- |
| [基線 commit](https://github.com/jeromeleong/Sceneweave/commit/eee9234ae38ffc61a7d4576298a0f23739474597) | Release 0.4.8 基線與版本身份 | 不能證明新重構已執行 |
| [基線 README](https://github.com/jeromeleong/Sceneweave/blob/eee9234ae38ffc61a7d4576298a0f23739474597/README.md) | 原產品定位導演台，描述三維預演、內建 AI、MCP 和每場提示詞 | 不能把它當 Sceneweave 核心／模組或角色 prompt 新契約 |
| [基線 package.json](https://github.com/jeromeleong/Sceneweave/blob/eee9234ae38ffc61a7d4576298a0f23739474597/package.json) | 名稱 director-desk、Three.js／Mediabunny 及現有工具依賴 | 不能證明新 Svelte 應用或 core-only 包已存在 |
| [基線樹](https://github.com/jeromeleong/Sceneweave/tree/eee9234ae38ffc61a7d4576298a0f23739474597) | 可定位現有 src、desktop、skills、tests 及授權文件 | 單看路徑不代表已測試每個算法 |

本次文檔提交不修改這些執行實作。旧 README 透過固定 commit 仍可查閱；新根 README 明確說明設計與程式狀態不同。

## 使用者確立的產品約束

文件夾優先、面向劇集而可自由組合、無限畫布是核心、不內建 AI、外部 agent 可精確操作、直接重構且不做舊相容、所有自有 UI 使用 Svelte，以及導演台可卸載、角色和場景可保存固定 prompt。

这些是本產品決策，不是從其他產品接口抄來的限制。角色／場景 prompt 的檔案設計、版本規則及八段組裝是此文檔的規範；不能宣稱與某服務未公開的 API 完全相同。

## 外部技術參考

以下官方資料在本輪查閱。它們只支持相應技術背景，Sceneweave 的目標契約仍由 00–12 定義。

| 來源 | 本方案採用的部分 | 限制 |
| --- | --- | --- |
| [Svelte 官方 Overview](https://svelte.dev/docs/svelte/overview) | 元件式、編譯式 UI；自有介面統一 Svelte | 不是後端或三維數學引擎 |
| [Svelte Flow 官方](https://svelteflow.dev/) | `@xyflow/svelte` 的節點互動、平移縮放、自訂 Svelte 節點 | 不擁有項目資料模型；並非影片製作平台 |
| [JSON Canvas 1.0](https://jsoncanvas.org/spec/1.0/) | 參考文字／文件／連線／分組的開放表示方式 | V1 採自有 swcanvas；不承諾 JSON Canvas 相容或往返 |
| [MCP Tools 2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) | 能力發現、Schema、結構化結果和工具列表變更 | 不是內建 AI；不表示這是所有客戶端支持的唯一或最新版本 |

不把 MiniMax Design 或 LibTV 在其他會話中的模型名、工具清單、特定 nodeId 或當時連接狀態寫成此倉庫已具備的事實。它們只是產品方向的比較背景，本輪不發布競品全部能力清單。

## 狀態詞彙

| 標記 | 意義 | 所需證據 |
| --- | --- | --- |
| Design | 文檔已定義目標行為 | 本套版本化規格 |
| Implemented | 已有相應程式 | 實际路徑、commit、Schema／功能 |
| Executed | 已實際運行驗證 | 命令、日期、環境、結果與 fixture |
| Release-gated | 發布前尚需通過條件 | 未完成項及對應驗收 ID |

本套功能目前統一為 Design，不宣稱 Implemented 或 Executed。应用 build、單元測試、視覺測試、模組卸載測試和安裝包驗證在本輪沒有執行。

在此文檔外另行進行的 Markdown／JSON 語法檢查，只驗證文件，不可把它改報成應用測試。

## 本輪文檔交付

根 README、AGENTS、docs/README 及 00–13 規格共 17 份 Markdown 文件。內容覆盖核心、工作區、畫布、資料、固定 prompt、可卸載模組、Svelte 架構、API、製作流程、安全、工作包、驗收、完整示例與來源。

後續修改以真實 diff、commit、PR 和驗收結果記錄；不虛構已刪除 AI、已完成 Svelte 重構、已拆出插件或已跑完測試。
