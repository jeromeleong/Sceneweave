# 05 · 可安裝、可卸載的模組契約

狀態：V1 目標契約；未實作。這是架構邊界，不是 UI 顯示開關。依據：[00](00-decisions.md)、[06](06-architecture.md)。

## 什麼算可卸載

移除模組的程式包、可選依賴、可執行能力及運行資源後，核心仍可啟動、打開所有畫布、管理文件、編輯 prompt、安排鏡頭和查看既有成果。導演台與 Three.js 不在核心 runtime／build import graph。

隱藏面板、延遲 import、關閉 WebGL、保留所有程式但改成 disabled，都不等於 uninstall。核心安裝包不能捆綁一個無法移除的導演台目錄。

## 安裝與項目資料分離

模組包放在應用管理的安裝目錄。Vault 只保存使用哪些模組的描述、工程、依賴和成果，不把可執行插件藏在項目中自動運行。

打開別人發來的項目不自动安裝、啟用、下載或執行任何模組。缺模組時仍提供通用工程卡片。安裝是一個明確的使用者管理操作，不是一般 agent 內容寫入權的延伸。

V1 官方發行的模組與使用者明確信任的本地包是首要範圍，不承諾任意第三方原生程式已安全沙盒化。原生 worker 需要受信任發布者與明確權限；沒有實際 OS 隔離不能宣稱它對文件系統完全沙盒。

## Manifest

每個模組有不可執行的 manifest，可在不啟動 UI 的情況下讀取：

| 欄位 | 要求 |
| --- | --- |
| id、version、moduleApiVersion | 穩定身份與接口版本，unsupported major 拒絕載入 |
| displayName、description、publisher | 人類可讀資訊，不能當權限證據 |
| integrity | 內容雜湊、可驗證來源；未驗證明確標示 |
| entrypoints | 可選 ui／worker／renderer，明確支持的環境 |
| documentTypes | formatId、formatVersion、extension、genericFallback |
| capabilities | 命令名、input/output Schema、單位、權限、是否需要 renderer |
| permissions | 文件範圍、網絡、處理器、資源與任務能力 |
| dependencies | 必須／可選模組，明確版本限制，拒絕循環依賴 |
| uiContributions | 編輯器、卡片呈現、檢視器、選單；不能改寫核心身份 |

manifest 不包含模型供應商或生成路由。模組公開能力的 Schema 是資料，不能要求外部 agent 打開 iframe 後才靠自然語言詢問。

## 通用工程外殼

`*.swdoc.json` 由核心理解，至少包含 schemaVersion、id、title、moduleId、formatId、formatVersion、payloadRefs、dependencyRefs、previewRefs、outputRefs。payload 是模組自己的工程文件，核心不擅自解析或改寫其專業語義。

payloadRefs 和 dependencies 使用穩定物件及精確版本，必要時帶相對路徑定位提示。相鄰 payload 檔案可以是 JSON、二進位或多檔資產；不得把唯一正式資料只存在插件的 localStorage、記憶體或私有資料庫。

previewRefs 指向保留下來的普通輸出，不依賴已卸載模組才能解碼。當前縮圖快取可以清理，但最後有效的持久預覽與已交付成果不能隨 uninstall 消失。

## 宿主接口

核心提供讀取帶權限的物件、提交命令、取得臨時媒體 URL、寫入 output staging、報告任務、註冊／撤回能力和記錄診斷。模組不能直接改核心 store、遍歷整個磁碟、任意 shell 或自行宣稱擁有更高權限。

所有正式寫入必須由 CommandService 驗證，renderer 只產生候選輸出。驗證成功才把 staging 成果登記到 Vault。模組不得以自己的 UUID 覆寫其他模組的 namespace。

Svelte UI 是獨立 bundle。預設使用隔離的 iframe／受控 web context；核心與模組用具 Schema 的 MessageChannel 通訊，檢查來源窗口、握手 nonce、會話和能力。來源為 opaque origin 時不能只靠 origin 字串作驗證。受信任本機 worker 亦使用窄接口，絕不等同自帶 OS 沙盒。

資料操作可以不啟動 UI。必須渲染的 capture／export 由有能力的受控 renderer 執行，未具備 GPU／codec 時返回 capability unavailable，不回傳虛構成功。

## 生命週期

狀態至少包括 absent、installed_disabled、enabled、draining、failed、quarantined。install 驗證包、版本及依賴但不自動啟用越權能力；enable 完成明確授權後註冊可執行能力。

關閉編輯器只卸載視圖，不移除模組。disable 停止新任務、收束既有工作、撤回能力，但保留包。uninstall 需要以下順序：

1. 列出所有已註冊 Vault 的受影響工程、開啟視圖、dirty draft、工作任務和依賴模組。
2. 阻止新任務與新 mount；對未保存內容提供保存、丟棄或取消選擇。
3. 要求使用者選擇等待／取消任務；取消超時可強制停止，但記錄 interrupted，不宣稱成功。
4. dispose Svelte 元件、listener、timer、GPU、codec、worker、臨時 URL，撤回能力。
5. 移除安裝包及專屬可再生快取，不刪工程、prompt、來源、receipt、預覽與輸出。
6. 所有相關卡片立即降級為核心 fallback，更新 capabilitiesRevision 和事件。

有其他啟用模組依賴它時，預設阻擋；可以明確選擇一起停用，不能静默連鎖卸載。其他正在執行的应用窗口必須協調，不能單一窗口刪除另一窗口仍使用的程式文件。

## 缺失、失敗與重裝

缺模組保留工程原始 bytes；允許通用讀取、複製、打包、改顯示名稱和移動畫布實例，不允許假裝修改相機或時間線。實際 API 返回 MODULE_NOT_INSTALLED 或 MODULE_DISABLED。

重新安裝支持該精確 formatVersion 的模組後可恢復專業操作。未支持的新格式仍只讀，不自动下載舊版本、不建立默認相容 reader。本契約不要求舊 DirectorDesk 格式支援。

模組崩潰不能帶走核心畫布或丟失已提交資料。無法安全完成的寫入回到 coordinator recovery；私有 worker 的崩潰只中斷其任務，留具體診斷。

## 更新與可重現性

新包先獨立驗證，再原子切換安裝位置；新權限需重新授權。不靜默升級所有工程格式。不認識既有 payload 时停止編輯，而非猜測轉換。

工程與輸出記錄真正使用的 moduleVersion、formatVersion、輸入 revision 和處理設定。重裝可以恢復編輯能力，但不承諾不同 GPU／codec／模組版本產生 byte-identical 的影片。

## 卸載驗收

必須測試核心單獨發行、零模組打開含 Stage 工程的項目、卸載前後文件雜湊、能力移除、dirty draft、繁忙任務、依賴阻擋、崩潰、重裝及 memory／GPU 資源釋放。详見 [11](11-acceptance.md)。
