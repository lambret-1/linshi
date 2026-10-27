
# 小说改编三阶段工作流

> 合并自：`volume-pass.md`、`outline-pass.md`、`episode-pass.md`  
> 工作流顺序：分卷摘要 → 骨架 → 分集梗概  
> 标签基线：以模板 JSON 为准。文档不要求模板外字段。  
> 分集梗概一次写完全部集数，不再分批。

---

## 模板标签基线

### 顶层

`source` / `lang` / `params` / `adaptation` / `characters` / `scenes` / `beats` / `episodes`

### `params`

`episodes` / `minutesPerEpisode` / `genre` / `adaptMode` / `preferences` / `thresholds`

### `adaptation`

`core` / `keep` / `cut` / `merge` / `risks`

- `keep[]`：`what` / `why` / `evidence`
- `cut[]`：`what` / `why`
- `merge[]`：`what` / `why`
- `risks[]`：`what` / `plan`

### `characters[]`

`id` / `name` / `role` / `tier` / `arc` / `from`

`tier` 枚举：`lead` / `support` / `functional`

### `scenes[]`

`id` / `name` / `primary` / `reusePlan`

### `beats[]`

`id` / `type` / `weight` / `episode` / `setup` / `payoff`

`weight` 枚举：`major` / `minor`

### `episodes[]`

`ep` / `synopsis` / `hook` / `suspense` / `sceneIds` / `characterIds` / `propIds` / `crowdPlan` / `warnings`

`warnings` 枚举：`雨戏` / `肢体接触` / `人群` / `手部特写`

---

## 一、分卷摘要 · 中间产物

输入为一卷原文（若干章），输出该卷的改编原料。只输出 JSON，不要解释，也不要加 Markdown 围栏。

后续做大纲的人看不到原文，只能看到这份摘要。因此线索必须写清楚、落在纸面上，不能指望“到时候再翻书”。

> 本阶段标签是中间产物标签，不进入 `outline.json`。

### 输出格式

```json
{
  "plotlines": [
    {
      "name": "沈知微翻案线",
      "events": "这一卷里该线发生了什么，按顺序，写具体",
      "hookPotential": "这条线里能当爽点用的时刻"
    }
  ],
  "characters": [
    {
      "name": "老周",
      "aliases": ["老伯"],
      "doing": "这一卷里他做了什么、和谁产生了关系"
    }
  ],
  "beatMaterial": [
    {
      "moment": "陆行远当众掏出口袋",
      "type": "身份揭破",
      "evidence": "口袋里是一方旧砚，不是枪"
    }
  ],
  "settings": ["渡船船舱", "县城当铺"]
}
```

### 规则

- `plotlines` 按线整理，不按章整理。章是物理切分，线才是改编单位。
- `beatMaterial.evidence` 必须逐字复制原文片段，并保持原文语言。爽点素材必须能指回原文，这是落实“禁止凭书名脑补”的关键。
- `characters.aliases` 要收齐这一卷中出现的所有称谓。跨卷归并角色时依赖该字段。
- `settings` 只列实际发生过戏的地点；只在叙述中提到、没有发生动作的地点不算。
- 本阶段不做取舍判断。砍线、合人是下一步的工作；这一步只负责让原料齐全。

---

## 二、骨架 · 砍线 / 合人 / 排爽点

输入为全部卷摘要（或短篇原文）加用户参数，输出 `outline.json` 的骨架四块：`adaptation` / `characters` / `scenes` / `beats`。分集梗概不在这一步写。骨架未拍板就写分集，方向一旦错误，后续全部作废。

### 顺序即方法

1. 砍线。短剧只装得下一条主线，至多再加一条副线。砍掉的线写入 `adaptation.cut`，每条写 `what` 与 `why`。若 `adaptMode` 不是“忠实”，一条线都不砍会被 `validate` 拦截。

2. 合人。把功能重复的角色合成一张脸，例如两个报信角色合并为一个。合并记录写入 `characters[].from`。随后划分 `tier`：
   - `lead`：主角组，1–5 人；
   - `support`：有名字的重要配角，不超过 10 人；
   - `functional`：功能性角色，不超过 10 人。

   功能性角色占脸不占名：`name` 使用称呼标签，例如“急诊医生”，不需要写人物弧。无名背景人物不进入角色表。

3. 收场景。主场景数量上限随集数变化：`4 + ⌈集数 / 10⌉`，并夹在 5–15 之间。例如 6 集给 5 个场景，60 集给 10 个场景。最终以 `validate` 报出的数量为准。这是 AI 短剧的宽松标准：场景由生成完成，多开几个景来提升观赏性是划算的，不要把戏都憋在一个屋里。原著中只出现一次的场景，要么砍掉，要么填写 `reusePlan`，说明复用哪个现有环境资产、通过更换时段或天气如何改造出来。场景字段使用：`id` / `name` / `primary` / `reusePlan`。

4. 排爽点。先确定 `major` 落在哪几集，再用 `minor` 填补空隙。硬规则：
   - 相邻爽点间隔不超过 3 集；
   - 开头和结尾不能出现爽点真空；
   - 最早的 `major` 不能放在最后一集。

   `type` 跟随题材：女频打脸和男频战神的爽点不是同一种东西，需参考 `params.genre`。爽点字段使用：`id` / `type` / `weight` / `episode` / `setup` / `payoff`。

5. 收道具。道具排在爽点之后，并非随意排序。叙事道具的判断标准是“它托起哪个爽点”；爽点未定，就分不清哪件物件是道具、哪件只是陈设。道具上限为 8 件，与主角数量处于同一量级。

   判断一件东西是否该进入道具表，问一句就够了：它坏了、丢了、被换掉，剧情会不会塌？

   会塌，就是叙事道具，例如皮箱、砚台、那封信；不会塌，就是场景陈设，例如船舱的挂灯、桌上的茶碗。场景陈设归 `novel-art` 场景锚点管理，不进入道具表。填不出功能的，不是道具，是背景。

   标签约束：`outline.json` 的模板只在 `episodes[].propIds` 中引用道具 id，未定义顶层道具表。因此不要在 `outline.json` 里自创 `props`、`function`、`beatIds` 等模板外字段。道具的功能说明与所托爽点写在项目道具表或备注中；`outline.json` 只保留 `propIds` 引用。

每一步都基于卷摘要中的原料。关键取舍在 `adaptation.keep` 中附 `evidence`（原文逐字片段）。大纲必须能指回原文，不能凭印象改编。

砍线和合人完成后，结论分别并入 `adaptation.cut[].why` 与 `adaptation.merge[].why`。不要写 `cutNote` / `mergeNote`，模板没有这两个字段。报告中的“关键决策”可从 `why` 提取。

### 自检

写完运行：

```bash
node scripts/novel-outline.mjs validate <outline.json> --stage beats
```

有违规就逐条修，改完重跑；直到通过，才允许进入分集阶段。

---

## 三、分集梗概 · 一次写完全部集数

骨架拍板之后才进入这一步。输入包括：拍板后的骨架四块、全部集数（`params.episodes`）以及这些集里落着的爽点。一次输出全部 `episodes` 条目，不再分批。只输出 JSON 数组，不要解释，也不要加围栏。

### 每集必填字段

| 字段 | 写什么 |
| --- | --- |
| `ep` | 集号 |
| `synopsis` | 这一集发生了什么。使用叙述体——出现 `「」“”` 引号对白就是在写剧本，`validate` 会拦截 |
| `hook` | 让人点下一集的那个问题 |
| `suspense` | 埋着但尚未解决的线 |
| `sceneIds` | 场景 ID 列表，只能使用骨架中登记过的场景 ID |
| `characterIds` | 角色 ID 列表，只能使用骨架中登记过的角色 ID |
| `propIds` | 道具 ID 引用列表，只能引用已登记的道具 ID；不要新增模板外道具字段 |
| `crowdPlan` | 三人及以上同框时的拆解方案 |
| `warnings` | 生成风险提示，枚举：`雨戏` / `肢体接触` / `人群` / `手部特写` |

### 硬规则

- 每个爽点（`beats[].episode`）对应那一集的梗概必须真正演出 `setup` / `payoff`。爽点表说第 13 集打脸，第 13 集梗概里就必须有打脸。
- 集号必须从第 1 集连续写到 `params.episodes`，不跳集、不越界、不留空。
- `sceneIds` / `characterIds` 只能使用骨架中登记过的 ID。
- 当同一场戏有三人及以上同框时，必须填写 `crowdPlan`：机位如何分配、谁用背影和过肩镜头带过。校验器以 `characterIds` 中登记的人数作为代理指标；若三人实际不同框，应在方案中写明事实，例如“三人分处两场，无同框，分场拍”，不要空着。
- 梗概中出现雨、拥抱、人群、手部特写这类生成难点时，要写入 `warnings`。枚举值包括：`雨戏` / `肢体接触` / `人群` / `手部特写`。宁可多报。
- `propIds` 只引用已有道具 ID；模板未定义顶层道具表时，不要在分集条目里自创 `props`、`function`、`beatIds` 等字段。

### 自检

写完运行：

```bash
node scripts/novel-outline.mjs validate <outline.json> --stage episodes
```

有违规就逐条修，改完重跑；直到通过。
```