# 12 · 角色、搭景、畫布與卸載示例

狀態：文件範例，非現有執行接口。所有人物、項目、ID、版本和 hash 都是虛構示意，不可當真實工具參數。

例子依 [03](03-vault-data.md)、[04](04-assets-prompts.md)、[05](05-modules.md)、[07](07-agent-api.md)。片段僅展示相關字段，完整 Schema 在實作階段建立。

## 一、沒有圖片也能建立角色

`Assets/Characters/角色A/asset.md` 的作者稿：

```markdown
---
schemaVersion: 1
id: character_demo
kind: asset
title: 角色 A
assetType: character
attributes:
  appearance:
    scar:
      side: left
      location: eyebrow_tail
    hair: tied
    wardrobe: neutral
protectedPaths:
  - appearance.scar.side
  - appearance.scar.location
mutablePaths:
  - appearance.hair
  - appearance.wardrobe
---

成年男性角色。外觀身份以已發布視覺 profile 為準。
服裝和髮型可隨劇情變化；固定疤痕不可左右互換。
```

`Prompts/identity.prompt.md`：

```markdown
---
schemaVersion: 1
id: prompt_character_identity_demo
kind: prompt
title: 角色 A 固定身份
role: identity
modality: visual
locale: zh-Hant
locked: true
subjectRef:
  objectId: character_demo
variables: {}
---

成年男性，肩部寬闊、腰線收窄。眉骨清晰，鼻樑挺直，
下頜輪廓俐落。左眉尾有一道細淡舊疤。
不同鏡頭保持相同骨相與疤痕位置。
```

衣服和髮型沒有被鎖進以上正文。工作服、休閒服、披髮可以另建 Variant；固定 identity block 不被它們覆蓋。

`Prompts/negative.prompt.md`：

```markdown
---
schemaVersion: 1
id: prompt_character_negative_demo
kind: prompt
title: 角色 A 禁止變化
role: negative
modality: visual
locale: zh-Hant
locked: true
subjectRef:
  objectId: character_demo
variables: {}
---

不要把左眉尾疤痕改到右側。不要增加另一道對稱疤痕。
不要擅自改變年齡段或人物面部身份。
```

發布兩個 block 後，profile 使用服務返回的精確引用。以下是資料形狀，hash 不是實際檔案雜湊：

```json
{
  "schemaVersion": 1,
  "id": "profile_character_visual_demo",
  "kind": "prompt_profile",
  "title": "角色 A 視覺設定",
  "modality": "visual",
  "locale": "zh-Hant",
  "orderedBlockRefs": [
    {
      "objectId": "prompt_character_identity_demo",
      "revisionId": "revision_identity_demo",
      "contentHash": "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    },
    {
      "objectId": "prompt_character_negative_demo",
      "revisionId": "revision_negative_demo",
      "contentHash": "sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
    }
  ],
  "variables": {},
  "attachmentRefs": []
}
```

附件列表為空是合法狀態。後續加入正／側面圖不要求安裝導演台，也不要求接入圖片模型。

## 二、變體只改允許的欄位

下面的 Variant 精確引用一個已發布角色版本；示例省略另行發布的附加衣服文字與附件：

```json
{
  "schemaVersion": 1,
  "id": "variant_workwear_demo",
  "kind": "variant",
  "title": "工作服",
  "baseRef": {
    "objectId": "character_demo",
    "revisionId": "revision_character_demo",
    "contentHash": "sha256:cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc"
  },
  "variantGroup": "wardrobe",
  "patches": [
    {"path": "appearance.wardrobe", "value": "workwear"}
  ],
  "promptBlockRefs": [],
  "attachmentRefs": []
}
```

path 是受 schema 驗證的具名資料路徑，不是可執行表达式。不能寫 `appearance.scar.side=right`，也不能同時選兩個 wardrobe 互斥變體。實际視覺文字由作者另外填入 state prompt block；軟件不從 workwear 一詞自动生成服裝描寫。

## 三、搭景固定，雨夜是局部狀態

`Assets/Sets/公寓/Prompts/structure.prompt.md`：

```markdown
---
schemaVersion: 1
id: prompt_apartment_structure_demo
kind: prompt
title: 公寓固定空間
role: identity
modality: visual
locale: zh-Hant
locked: true
subjectRef:
  objectId: set_apartment_demo
variables: {}
---

狹長的單人公寓。從玄關面向室內，左側是開放式小廚房，
右側是一張雙人沙發；床位在房間深處，窗戶位於床的左側。
家具形狀和主要位置在不同鏡頭保持一致。
```

「雨夜、窗上有雨痕、只開床邊暖燈」放在 weather／lighting 變體或本場 state block，不改掉固定空間。「角色 A 坐在沙發左端」是本鏡 layout，不把全劇公寓永久綁到這個角色。

場景可只有以上文字和平面參考圖。三維 `*.swdoc.json` 僅是可選附件，並非 set 的必填字段。

## 四、固定 prompt 與本鏡文字組合

鏡頭使用角色與公寓的精確版本，選定工作服、雨夜，另有一份鏡頭 context。調用形狀：

```json
{
  "projectId": "project_demo",
  "arguments": {
    "shotId": "shot_demo",
    "modality": "visual",
    "locale": "zh-Hant",
    "variables": {}
  }
}
```

這是 `prompts.resolve` 的讀取輸入，不是發送到模型的請求。前置條件是 shot_demo 的身份、變體及 block refs 都已存在且精確鎖定；服務不得用這段文字臆造缺失文件。

預期文字結構如下，內容均應來自已保存的作者稿／快照：

```text
[1 品質與共同風格]
本項目已明確保存的影像要求。

[2 身份]
角色 A 的固定身份。
公寓的固定空間。
已選工作服與雨夜狀態的原始描述。

[3 場次上下文]
角色剛回到家，正在等待電話。

[4 空間布局]
角色坐在沙發左端，手機放在右手邊。

[5 表演與台詞]
他看一眼手機，沒有接起。

[6 鏡頭與燈光]
正面中景，固定機位，只開床邊暖燈。

[7 固定約束]
保持身份、疤痕位置和公寓主要布局。

[8 負面描述]
已引用負面 block 的原始文字。
```

章節名是組裝器格式；具體内容不能為了湊格式臆造。sourceMap 將每段對應到 source object、revision 與 block。空章節可省略，固定 block 不自动縮寫。

更新人設後，此鏡頭仍使用舊 pin；使用者看到影響範圍，選擇要升級的鏡頭再提交變更。外部模型是否照做，不能由保存成功推導。

## 五、同一角色出現在兩處

最小畫布示例；實際版本還可保存樣式與額外選項：

```json
{
  "schemaVersion": 1,
  "id": "board_demo",
  "title": "第 1 集",
  "nodes": [
    {
      "nodeId": "node_character_left_demo",
      "type": "asset",
      "objectRef": {"objectId": "character_demo"},
      "x": 0,
      "y": 0,
      "width": 280,
      "height": 320
    },
    {
      "nodeId": "node_character_right_demo",
      "type": "asset",
      "objectRef": {"objectId": "character_demo"},
      "x": 800,
      "y": 0,
      "width": 280,
      "height": 320
    }
  ],
  "edges": []
}
```

這裡有兩個 nodeId，只有一個 character_demo。刪除左邊卡片不删除角色。兩處顯示的 prompt 都讀同一份正式資料；需要固定預覽時改用精確 revision 引用。

## 六、導演台卸載

卸載前：畫布有角色、搭景、prompt、Stage 工程、分鏡圖、預演影片和審核。Stage 已保存，沒有未決任務。

卸載後：只移除 director-stage 安裝包、可執行能力與可再生快取。所有上述正式資料保留，Stage 卡片改顯示「需要導演台模組」。prompts.resolve、board.edit、review 和 delivery 仍能使用；stage.edit／stage.capture 返回 MODULE_NOT_INSTALLED。

重裝支持同 formatVersion 的版本，可重新編輯該工程。新包不支持該格式時保持只讀，不借重裝偷偷轉換文件。這整條流程不依賴任何 AI。
