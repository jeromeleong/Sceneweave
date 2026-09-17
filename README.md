# Sceneweave

**文件夾優先、以無限畫布為核心、讓人類與外部 agent 共用的劇集製作工作台。**

> 本次更新只確立重構文檔，沒有完成產品重構。倉庫目前的執行程式仍源自 DirectorDesk；以下是 Sceneweave V1 的目標契約，不是現有功能清單。

## 已確立的方向

無限畫布是不可卸載的產品核心，而不是導演台的入口頁。文件、素材、角色、場景、固定提示詞、鏡頭及工程可以在画布自由組合。每部劇預設一個真實文件夾，每集與場次提供可下鑽的工作範圍，但不限制自由項目。

導演台、影片剪輯器及其他專業編輯器都是可選、可停用、可卸載的模組。移除模組不會刪除工程、素材、畫布卡片、固定提示詞或已輸出的成果。核心不得依賴 Three.js 或某個專業編輯器才能啟動。

角色、搭景、風格與道具可保存有版本的固定 prompt、負面描述、結構化固定特徵及參考附件。提示詞只是普通項目資料；組裝和匯出是確定性操作。Sceneweave 不內建 AI、模型渠道、生成服務、模型金鑰、推理代理或自動語义改寫。

所有自有介面使用 **Svelte 5 + TypeScript**。直接重構，不保留舊 UI 外殼、`.director` 讀取器、舊 API 相容層或資料遷移工具。可重用算法，但不延續舊產品的資料和介面契約。

## 正式文檔

從 [文檔索引](docs/README.md) 開始，開發者與 coding agent 另讀 [AGENTS.md](AGENTS.md)。

| 主題 | 文檔 |
| --- | --- |
| 不可違反的架構決策 | [設計基線](docs/spec/00-decisions.md) |
| 無限畫布與多層工作範圍 | [產品與工作區](docs/spec/01-product-workspaces.md)、[畫布契約](docs/spec/02-canvas.md) |
| 文件與正式資料 | [Vault 與資料模型](docs/spec/03-vault-data.md) |
| 角色、場景及固定提示詞 | [資產與提示詞](docs/spec/04-assets-prompts.md) |
| 真正可卸載的專業工具 | [模組契約](docs/spec/05-modules.md) |
| Svelte、程序與套件邊界 | [技術架構](docs/spec/06-architecture.md) |
| 外部 agent 精確操作 | [操作 API](docs/spec/07-agent-api.md) |
| 實施與驗收 | [交付順序](docs/spec/10-delivery.md)、[驗收矩陣](docs/spec/11-acceptance.md) |

## 現有程式與目標設計的區別

程式基線為 `eee9234ae38ffc61a7d4576298a0f23739474597`。該版本的依賴、AI 助手、導演台及發布設定仍然存在，本次文檔提交沒有刪改它們。不要用文檔中的新 CLI、API、模組或檔案格式去假定現有程式已支援。

舊程式的開發、演示與操作說明可查閱 [基線 README](https://github.com/jeromeleong/Sceneweave/blob/eee9234ae38ffc61a7d4576298a0f23739474597/README.md)。該歷史文件不是重構規範，其中的內建 AI、舊發布來源及舊工程格式不代表 Sceneweave 的目標能力。

本次未運行應用 build、單元測試、視覺測試或安裝包驗證。實作完成必須另附證據，不得把「規格已寫」標成「功能已完成」。

## 授權

保留原有 [MIT LICENSE](LICENSE) 與第三方授權聲明。重構及更名不移除原作者的權利與歸屬；素材另依其原始授權。
