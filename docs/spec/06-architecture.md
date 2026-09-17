# 06 · 技術與套件架構

狀態：V1 目標契約；未實作。依據：[00](00-decisions.md)、[05](05-modules.md)。

## 技術選擇

所有自有使用者介面使用 Svelte 5、TypeScript 和 Vite，包括核心、模組管理、導演台、剪輯器、對話框與設定。使用 runes 管理 UI 反應狀態；正式資料仍由命令層管理，不能以 `$state` 取代文件版本。

無限畫布互動採 `@xyflow/svelte`，包在可替換的 canvas adapter 內。產品自己的 `swcanvas` schema 不依賴該庫的 store 結構。Three.js 是可選導演台的內部實作，不能滲入核心物件型別。

Electron 是首個桌面宿主，Node 是本地服務／CLI 執行環境；保留 Web UI 連接同一服務的能力。這不代表要把後端、文件系統或三維數學寫成 Svelte 元件。不要為符合「全 Svelte」而把純算法綁死在 UI 生命週期。

現有 DOM 面板不作新產品的發布中間態。可以移植算法與素材，但面板、事件管理和產品資料契約需按新架構重做。沒有 `.director` reader、舊 API adapter 或 legacy bridge。

## 套件布局

```text
apps/
  workbench/             Svelte 核心 UI
  desktop/               Electron 外殼與受控平台橋接
  service/               本地／自託管服務
  cli/                   JSON 輸出的命令列
packages/
  domain/                ID、引用、schema、版本與製作物件
  vault/                 文件、索引、監看、交易、恢復
  commands/              命令、權限、預檢、receipt
  canvas/                Board 操作、空間索引與 Svelte adapter
  assets-prompts/        資產、變體、固定文字組裝
  module-sdk/            無專業工具依賴的公開契約
  module-host/           安裝、能力、隔離與生命週期
  transports/            HTTP、CLI／MCP 映射與事件
  ui/                    共用 Svelte 元件和設計 token
modules/
  director-stage/        可選；Svelte UI、Three.js、Stage schema
  video-editor/          可選；Svelte UI、多軌工程與渲染
  media-processing/      可選；確定性 FFmpeg 等 worker
```

這是目標結構，不是聲稱這些目錄已存在。模組以獨立發行工件交付。monorepo 開發依賴可以包含所有工作區，但 core-only 安裝、打包和運行必須不帶專業模組及它們的重型依賴。

## 依賴方向

UI／CLI／HTTP／MCP → CommandService → domain／vault／assets-prompts。

專業模組 → module-sdk → 宿主窄接口。module-host 只認 manifest、schemas、一般工程外殼及權限，不能 import DirectorStage 的 Scene／Camera 類別。

核心不向 module import；宿主依安裝註冊資訊載入獨立 entrypoint。對某模組的操作透過命名能力分派，不把 Stage 的 command switch 寫入 core。

## 程序邊界

一個 Vault 的写入權由 service coordinator 管理。核心 UI 和專業模組 UI 是客戶端。計算密集操作可在 worker；原生處理在受控子進程；需畫面的任務在 renderer。Node worker 本身不是安全沙盒。

UI 的可丟棄狀態包括選取、hover、拖動預覽、分頁與 viewport。工程、prompt、相機參數、採用 Take、審核是持久資料。一次拖動正式提交是一個命令，不由多個 store callback 分別寫文件。

## 三種部署模式

桌面版使用完整本地文件能力。瀏覽器連接本地或自託管 service，操作服務端已授權根目錄，而不是假裝能任意讀客戶端磁碟。純靜態瀏覽器 demo 可以檢視／試用，但不能冒充完整 folder-first 發行版。

不在 V1 加入雲端必登入、帳號計費或遠端多人即時 CRDT。資料接口要支持多客戶端，但首版使用明確版本衝突，不承諾任意文本自動合併。

## 非 AI 媒體

核心提供普通檔案探測與可用的瀏覽器播放／圖片預覽，不內建語音辨識或語義分析。完整 FFmpeg、三維渲染、時間線編碼是可選 worker 模組。只有安裝相應能力時才可調用；既有輸出的瀏覽不需要原模組。

素材的受限 codec、平台差異與性能資源必須返回具體診斷。輸出不是在核心 UI 偷偷阻塞主線程。

## 首版必測邊界

靜態 import graph 不得有 domain／core→module 邊。core-only build 不能包含 Three.js、Stage 模組程式或不可卸載的完整剪輯器。刪掉全部 modules 發行工件仍可跑核心端到端測試。

所有自有 UI 的新實作由 Svelte 組件構成；直接 WebGL Canvas、播放器和第三方編輯器引擎可以由 Svelte 封裝，這與保留整套旧 DOM 應用不同。

來源與技術依據見 [13](13-sources-status.md)；精確依賴版本在實作提交與 lockfile 中驗證鎖定，本文件不把未測試的新版本冒充已支持。
