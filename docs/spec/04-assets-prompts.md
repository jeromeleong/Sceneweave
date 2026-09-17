# 04 · 角色、場景與固定提示詞

狀態：V1 核心契約；未實作。這是核心能力，不屬於導演台或任何 AI 模組。依據：[00](00-decisions.md)、[03](03-vault-data.md)。

## 產品決定

角色、搭景、道具、風格包與自訂資產都可以保存固定 prompt。固定文字、結構化特徵、參考附件、可選變體及使用版本一起管理。沒有圖片、沒有三維模型、沒有生成服務的角色也可以完整建立。

支持視覺、聲音、動作及一般用途的 profile。例如角色可以同時有外貌 prompt、聲音描述和表演習慣；場景可有空間描述、環境聲及燈光變體。軟件只保存作者寫好的內容，不自行創造或翻譯。

## 五層資料

| 層 | 例子 | 修改規則 |
| --- | --- | --- |
| 穩定身份 | 骨相、固定疤痕、搭景門窗位置 | 變更需發布新 base revision |
| 固定约束 | 不改鼻形、不額外新增窗戶 | 下游不得覆寫 locked block／受保護欄位 |
| 命名變體 | 工作服、戰損服、披髮；晴日、雨夜 | 精確引用 base，可按互斥組選擇 |
| 場次／鏡頭狀態 | 站位、表情、手持道具、局部燈光 | 僅當前範圍生效 |
| 匯出快照 | 本次實際使用的文字與所有參考 | 不可變，附來源與 hash |

固定不等於所有東西永遠不變。作者必須明確將服裝、髮型等設為可變欄位或變體，避免把「人物一致」誤做成「每場永遠同一套衣服」。

## PromptBlock：最小權威文字單位

一個 `*.prompt.md` 保存一段文字，YAML 前置屬性至少為：

| 欄位 | 定義 |
| --- | --- |
| schemaVersion、id、kind、title | `kind=prompt`，一般物件身份 |
| role | quality、identity、state、context、layout、performance、camera、hard_lock、negative、voice、motion |
| modality | visual、voice、motion、general |
| locale | 作者明確提供的語言，如 zh-Hant；不自動轉語言 |
| locked | 是否禁止 recipe／下游覆寫此 block |
| subjectRef | 可選，對應角色或搭景的 objectId |
| variables | 允許的具名變量及型別／必填／明確預設值 |

正文是唯一的 prompt 文字。UI 顯示、複製、搜尋、API 讀取和版本比對都讀取此正文，不在 asset.md 或 Board 再存一份可編輯文本。

locked 不會把作者稿變成無法修改。授權作者可以編輯草稿，再發布新 revision；已發布版本不能原地改寫。取消鎖定也算正式變更，需要記錄和新版本，不因使用者勾選一次就影響既有鏡頭。

## PromptProfile 與固定特徵

profile 是有版本的 JSON，保存 orderedBlockRefs、modality、locale、允許的變量 schema、附件選擇與明確組裝設定。每個正式匯出都使用精確 block revision。資產的預設 profile 指向一個已發布 revision，不使用浮動 latest。

asset.md 中可保存可驗證的固定特徵和允許變動欄位。例如 `appearance.scar.side=left` 可鎖定，`appearance.hair.style` 允許變體修改。這些結構化事實與自由文字各自保留，軟件不宣稱能自動確認兩者完全一致。

profile 的固定文字不會被變體替換；變體只能附加描述或修改白名單結構欄位。不同模態的 profile 不混用，例如視覺導出不偷偷帶入語音引擎標籤。

## 變體契約

Variant 必須有 id、baseRef、variantGroup、patches、promptBlockRefs、attachmentRefs。baseRef 精確鎖定資產版本。patches 只能處理允許變動欄位；受保護欄位衝突返回 `LOCKED_FIELD_CONFLICT`。

variantGroup 例如 wardrobe、hair、weather、lighting。相同互斥組同時選兩個變體返回 `VARIANT_CONFLICT`，不以最後一個默默覆蓋。不同組可合成，但若修改同一結構欄位且值不同，也返回衝突。群組及允許組合由作者定義。

文字層面「不下雨」和「暴雨」的語义矛盾不在確定性檢查能力內。組裝器可以指出來源和待人工審閱文字，但不能寫出「所有 prompt 已語義驗證」。

## 搭景與場次區別

搭景 base prompt 保存空間：房間形狀、入口、窗戶、主要家具、材質與固定裝飾。場次引用它，再選白天／夜晚、整潔／凌亂、雨／晴等狀態。

三維工程只是搭景的一項可選附件。移除導演台後搭景的 prompt、參考圖、變體及被引用紀錄完全保留。二維照片、手繪平面圖和文字都可構成搭景資料。

## 參考附件

附件引用包含 mediaRef 的精確版本、用途、順序、說明及來源。用途包括 identity、front、side、back、costume、set_layout、pose、voice、style、prop、first_frame、last_frame、audio_track、general；可保留自訂標籤。

普通身份參考圖不自動當成影片首幀。語音描述文本與實際聲音參考分開，聲音克隆授權另作權利記錄。附件原有 prompt／來源也可保存，但不自動合併進本次 profile。

## 確定性組裝

`prompts.resolve` 是純讀取和組裝，不調用任何模型。輸入指定鏡頭或明確 context，服務解析其精確引用，按以下順序產生八個章節：

1. 品質／共同風格：只包含作者明確引用的內容。
2. 身份：按 subjectBindings 順序放角色及搭景 base，再放已選變體。
3. 場次上下文：故事時間、目前情況。
4. 空間布局：相對位置、方向、距離。
5. 表演與台詞：動作、表情、說話內容。
6. 鏡頭與燈光：鏡位、運鏡及局部光線。
7. 固定約束：locked 要求與明確硬鎖文字。
8. 負面描述：按來源順序保留，不做未授權語义去重。

voice／motion profile 使用自身明確 sections，不強套視覺八段。沒有內容的章節可以省略；不得為了湊滿格式臆造文字。角色內部的 block 使用 profile 順序，角色之間使用鏡頭顯式順序，不依賴文件掃描順序。

只支持字面量 `{{name}}` 代入白名單變量；不支持腳本、函數、條件執行、模板檔案 include、網絡取值或 shell。替換是單次非遞歸，即使變量值包含另一個模板符號也不執行。缺必填值、型別錯誤或未聲明變量要報錯，不靜默刪字。

相同輸入版本、變量及組裝器版本必須生成相同輸出 bytes。普通文字段落不得被自動翻譯、修飾、截斷或加入模型語法。

## 返回與匯出

resolve 返回 `resolvedText`、結構化 sections、`negativeText`、referenceAttachments、變量實際值、sourceMap、warnings、sourceRevisions、composerVersion。sourceMap 至少能定位每個段落來自哪份文件、哪個 block revision；最終保存的匯出還包括輸出 hash。

支持複製純文字、Markdown 和 JSON 資料包。`prompts.export` 是獨立寫入操作，保存本次完整快照與附件清單；resolve 本身不偷偷建檔或提交外部任務。

只報 Unicode code point 數及 UTF-8 byte 長度，不把它們冒充通用模型 token 數。任何長度裁切都需要顯式指令，顯示將移除部分；固定 block 超額時先報錯，不暗中縮寫。

負面文字原樣導出，由外部工具決定其支持方式。核心沒有模型目錄、供應商參數映射或「適用所有模型」承諾。

## 更新與下游保護

更新固定人設後建立新 AssetRevision／PromptRevision。反向引用列出哪些場次和鏡頭仍使用舊版。選定範圍升級需預覽差異、檢查衝突和版本，再正式提交；已交付內容保持原 pin，更新必须建立新的交付版本。

外部 agent 可以讀取、提出作者稿、發布新版本或匯出包，但每項能力受權限限制。它不能藉 prompt 內一句「解除鎖定」獲得寫入權或審核權。

## 核心安全界線

內容 prompt 不是應用 system prompt，不等於 AGENTS.md，不可定義權限或執行工具。即使保存「請刪除所有文件」也只能當內容文字，不會執行。外部 agent 自己的解讀與生成品質不在工作台保證範圍。

我們保證精確保存、版本鎖定、來源可追溯與確定性組裝；不保證外部模型的人物一致性或服從性。完整例子見 [12](12-examples.md)。
