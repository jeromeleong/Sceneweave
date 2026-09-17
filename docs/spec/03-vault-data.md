# 03 · Vault、資料模型與保存

狀態：V1 目標契約；未實作。依據：[00](00-decisions.md)。

## 身份與單一來源

`projectId` 識別一個 Vault；`objectId` 識別其中的正式物件；`nodeId` 只識別某張畫布上的呈現實例；`revisionId` 指不可變版本；`contentHash` 驗證具體內容。名稱、EP001 等顯示編號、檔案路徑都不是永久身份。

一般引用為 `{objectId}`，只適合未鎖定作者稿的檢視。固定引用為 `{objectId, revisionId, contentHash}`。所有身份由建立命令分配；agent 不得由檔名臆造真實 ID。跨 Vault 引用還需明確 `projectId` 和已授權的解析來源。

本文的「正式」意指 authoritative，與「已審核通過」不同。普通作者稿也是正式資料，但可以修改；發布 revision 不可原地修改。

## 物件與權威文件

| 物件 | 權威文件 | 主要內容 |
| --- | --- | --- |
| Project | `sceneweave.project.json` | schemaVersion、id、title、內容根目錄與預設 Board |
| Episode | `episode.md` | YAML 屬性及正文；顯示編號、排序、製作資料 |
| Scene | `scene.md` | 場次、故事時間、出場、搭景引用、局部狀態 |
| Shot | `shot.md` | 鏡頭要求、subjectBindings、採用 Take、製作狀態 |
| Asset | `asset.md` | assetType、固定特徵、描述、profile 與變體引用 |
| MediaFile | 普通媒體＋相鄰 `.media.json` | 穩定身份、文件路徑、來源與內容雜湊 |
| PromptBlock | `*.prompt.md` | 型別屬性及一段權威 prompt 正文 |
| PromptProfile | `*.prompt-profile.json` | 精確 PromptBlock 引用、變量與組裝選項 |
| Variant | `*.variant.json` | 精確 base revision、結構化變化、附加 prompt 和附件 |
| Take | `take.json` | 某候選的媒體、用途、來源聲明和不可變輸入快照 |
| Board | `*.swcanvas.json` | 節點、布局、可視連線及正式關係引用 |
| Collection | `*.collection.json` | 手選 ID 或明確 query；排序規則 |
| EditorDocument | `*.swdoc.json`＋payload | 通用模組外殼、格式、依賴、預覽及輸出 |
| Review／Delivery | 各自 JSON 或 Markdown 記錄 | 精確版本、審核者、時間定位及交付清單 |

Markdown 前置屬性至少有 `schemaVersion`、`id`、`kind`、`title`；正文不另存一份資料庫副本。JSON 使用相同身份和版本原則。未知 major 拒絕寫入；未知擴展必須保留或只讀，不丟掉再重存。

媒體尺寸、波形、縮圖和全文索引是衍生資料，與實際文件 hash 綁定。雜湊不符時原有「已驗證尺寸」失效，先重新探測，不能把使用者改過的文件當旧版本。

## 預設文件夾

```text
我的劇集/
  sceneweave.project.json
  Bible/
  Assets/
    Characters/角色A/
      asset.md
      Prompts/identity.prompt.md
      Prompts/negative.prompt.md
      visual.prompt-profile.json
      Variants/工作服.variant.json
      References/
    Sets/公寓/
      asset.md
      Prompts/structure.prompt.md
      Variants/雨夜.variant.json
  Episodes/EP001/
    episode.md
    script.fountain
    Scenes/SC010/
      scene.md
      board.swcanvas.json
      Shots/SH010/shot.md
      Shots/SH010/Takes/T001/take.json
      Editors/main.swdoc.json
      Editors/payload/
  Boards/main.swcanvas.json
  Collections/
  Reviews/
  Deliveries/
  .sceneweave/
    revisions/
    operations/
    trash/
    index/
    cache/
    local/
```

目錄名稱可改、可用中文，內容根可配置。正式層級以最近的有型別父容器推導，不在同一物件再保存一套可能矛盾的 parentId；排序是明確欄位，不依賴字典序或畫布位置。自由項目可省略 Episodes。

檔案移動命令連同必要引用更新一起提交；外部搬移由身份比對辨識。衝突的重複 ID 不靠任選一份解決。整個 Vault 複製後可當同一項目的離線副本；作為獨立新項目時使用 fork 操作更新項目身份，兩份同身份副本不能同時註冊為獨立 writer。

## 修訂與內容雜湊

作者稿放在人可讀的目錄。發布時在 `.sceneweave/revisions/<objectId>/<revisionId>/` 保存不可變快照、其所有正式依賴的精確引用及 `manifest.json`。大媒體可用同 Vault 的 immutable blob／版本文件去重，但清單必須能收集完整依賴。

單檔 `expectedRevision` 使用目前原始 bytes 的 SHA-256。快照 `contentHash` 是實際保存的 manifest bytes 的 SHA-256；manifest 列出每個檔案的相對路徑、byte 長度和 SHA-256。服務必須固定 UTF-8、LF、欄位順序和按 UTF-8 路徑 bytes 排序的序列化；比較輸入版本不能依賴修改時間或檔名。

固定引用只解析指定版本，找不到就返回缺失錯誤，不退回 latest。修改作者稿不影響已發布快照。原始外部媒體若沒有被收集進 immutable storage，不得宣稱已鎖定可重現交付。

## 索引與外部編輯

SQLite 用於 ID→路徑、全文、反向引用、查詢與空間索引，不能是唯一儲存。刪除 index/cache 後應能從項目文件和版本清單恢復。history、operations、revisions、Reviews 和 Deliveries 不屬於快取。

監看器處理新增、刪除、重命名、外部修改、原子替換與事件丟失；必要時完整掃描。掃描狀態與資料新鮮度可查詢。讀取受影響而尚未重新索引的文件時，直接驗證檔案，不能用 stale index 批准正式寫入。

未保存的本機編輯與外部改動衝突时保存 recovery draft，顯示差異，不靜默選一方。外部改壞 YAML 或 JSON 不阻止 Vault 其他部分開啟；問題文件保留原 bytes 並標示不可結構化編輯。

## 寫入交易

同一 Vault 由一個寫入協調器管理。UI、CLI、HTTP、MCP、模組都提交到它；獨立 CLI 寫入也要取得相同 lease／lock。進程中止後先恢復未完成 transaction，再接受寫入。

交易包含 read-set、write-set、預期 hash、暫存內容、inverse 資料與狀態日誌。預檢不代表預留版本；commit 時重新檢查。對受控 writer 提供一致提交，對任意外部工具直接改檔不能承諾強跨文件原子性。檢查仍需在替換前後執行，無法安全協調時返回 conflict。

磁碟滿、权限失敗、中途當機、部分替換都有可恢復日誌；不能遇到錯誤就刪除不認識的新文件。成功後返回每份文件的新 hash 和 transaction receipt。

## 刪除、搬遷與依賴

刪除卡片不涉及實體文件。正式文件預設移到項目回收區，保留原路徑和引用影響。永久刪除需額外權限及人類確認，已被交付或 revision 使用的內容不可被一般垃圾回收移除。

跨項目共享採明確匯入／固定快照為預設，可選外部唯讀庫引用需另外授權並標示離線依賴。交付前收集外部引用、檢查授權與 missing files。關閉程式後搬走整個項目，可以重建而無需舊資料庫。
