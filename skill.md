# 小说改编短剧大纲
给你一本书或者章节摘要，输出一本完整短剧大纲

## 核心思路（字段表内版本）

1. **砍线**
   - 砍掉内容写入 `adaptation.cut`，每条必须有 `why`。
   - `params.adaptMode` 不是 `忠实` 时，`adaptation.cut` 不能为空。

2. **合人**
   - 合并记录写入 `characters[].from`。
   - 角色分档写入 `characters[].tier`：
     - `lead`：1–5 人，受 `params.thresholds.maxLeads` 约束；
     - `support`：≤10，受 `params.thresholds.maxSupport` 约束；
     - `functional`：≤10，受 `params.thresholds.maxFunctional` 约束。
   - `characters[].name`：功能性角色用称呼标签。
   - `characters[].arc`：`lead` / `support` 必填，`functional` 可省。

3. **收场景**
   - 主场景上限见 `params.thresholds.maxPrimaryScenes`：`4 + ceil(集数/10)`，夹在 `5–15`。
   - 只出现一次的场景必须写 `scenes[].reusePlan`。

4. **排爽点**
   - 先定 `beats[].weight = major` 的落点，再用 `minor` 填充。
   - 相邻爽点间隔 ≤ `params.thresholds.maxBeatGap`。
   - 开头结尾无真空；至少一个 `major`，最早的 `major` 不能落在最后一集。
   - `beats[].type` 跟随 `params.genre`。

5. **收道具**
   - `props[].function` 必填：物件在戏里承载什么。
   - `props[].beatIds` 可选，写了必须指向真实 `beats[].id`。
   - 上限见 `params.thresholds.maxProps`，默认 `8`。
   - 分集用 `episodes[].propIds` 引用。

6. **关键取舍与结论**
   - `adaptation.keep` 至少一条，每条 `{what, why}`；关键取舍可附 `adaptation.keep[].evidence`（原文逐字片段）。
   - `adaptation.cutNote`：砍线结论句，给了不能为空字符串。
   - `adaptation.mergeNote`：合人补注，通常写主角组入选理由，给了不能为空字符串。



### 字段说明表（规范回填版）

#### 1. 顶层与 params

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `source` | string | 是 | 来源作品名/书名 | `"书名"` |
| `lang` | string | 是 | 语言代码 | `"zh"` |
| `params` | object | 是 | 创作参数对象 | `{...}` |
| `params.episodes` | integer | 是 | 总集数，正整数；分集数量必须与它一致 | `60` |
| `params.minutesPerEpisode` | number | 是 | 单集时长（分钟） | `2` |
| `params.genre` | string | 是 | 题材，决定爽点类型，不许缺；投放平台差异不单设字段，直接体现在 `thresholds` 上 | `"女频逆袭"` |
| `params.adaptMode` | string | 是 | 只能三选一：`忠实` / `抽核` / `借壳` | `"抽核"` |
| `params.preferences` | array<string> | 否 | 用户点名要保的角色、戏 | `["保女主复仇线"]` |
| `params.thresholds` | object | 否 | 逐项覆盖质量门阈值；短篇建议收紧角色档 | `{...}` |
| `params.thresholds.maxLeads` | integer | 否 | 主角组上限，默认 `5` | `5` |
| `params.thresholds.maxSupport` | integer | 否 | 重要配角上限，默认 `10` | `10` |
| `params.thresholds.maxFunctional` | integer | 否 | 功能性角色上限，默认 `10` | `10` |
| `params.thresholds.maxBeatGap` | integer | 否 | 相邻爽点最大间隔，默认 `3` | `3` |
| `params.thresholds.maxPrimaryScenes` | integer | 否 | 主场景上限；缺省不是常数，随集数动态：`4 + ceil(集数/10)`，夹在 `5–15`；60集→`10` | `10` |
| `params.thresholds.maxProps` | integer | 否 | 叙事道具上限，默认 `8`；由 props 节可覆盖 | `8` |

#### 2. adaptation 改编说明

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `adaptation` | object | 是 | 改编策略对象 | `{...}` |
| `adaptation.core` | string | 是 | 一句话核心 | `"女主被弃后夺回身份与家业"` |
| `adaptation.keep` | array<object> | 是 | 至少一条；每条 `{what, why}`；可带 `evidence` | `[{...}]` |
| `adaptation.keep[].what` | string | 是 | 保留什么 | `"主线复仇"` |
| `adaptation.keep[].why` | string | 是 | 为什么保留 | `"支撑逆袭爽点"` |
| `adaptation.keep[].evidence` | string | 否 | 原文逐字片段；关键取舍要能指回原文，禁止凭书名脑补 | `"原文逐字段落…"` |
| `adaptation.cut` | array<object> | 条件 | 每条 `{what, why}`；`adaptMode` 不是忠实时不能为空 | `[{...}]` |
| `adaptation.cut[].what` | string | 条件 | 砍什么 | `"原著后30章支线"` |
| `adaptation.cut[].why` | string | 条件 | 为什么砍 | `"与主线爽点无关"` |
| `adaptation.merge` | array<object> | 否 | 每条 `{what, why}` | `[{...}]` |
| `adaptation.merge[].what` | string | 条件 | 合并什么 | `"闺蜜线与助理线合并"` |
| `adaptation.merge[].why` | string | 条件 | 为什么合并 | `"减少角色维护"` |
| `adaptation.risks` | array<object> | 否 | 每条 `{what, plan}` | `[{...}]` |
| `adaptation.risks[].what` | string | 条件 | 风险是什么 | `"反派动机弱化"` |
| `adaptation.risks[].plan` | string | 条件 | 应对方案 | `"第3集补一场动机戏"` |
| `adaptation.cutNote` | string | 否 | 砍线结论句；给了就不能是空字符串；说清砍完后故事终点变成什么 | `"全剧终点改为夺回家业，原著后30章基本不用"` |
| `adaptation.mergeNote` | string | 否 | 合人补注；通常写主角组入选理由，谁有完整转变弧；给了就不能是空字符串 | `"女主与女二合并，保留完整转变弧"` |

> 非字段备注：大爆点落点列表不用写，报告从 `beats` 自动算。

#### 3. characters 人物表

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `characters` | array<object> | 是 | 角色列表；无名背景人不进表、不追踪、不限量 | `[{...}]` |
| `characters[].id` | string | 是 | `C01` 格式，全局唯一；分集靠它引用 | `"C01"` |
| `characters[].name` | string | 是 | 姓名；功能性角色用称呼标签 | `"林晚"` / `"急诊医生"` |
| `characters[].role` | string | 是 | 定位一句话 | `"女主"` / `"搅局配角"` |
| `characters[].tier` | string | 是 | 三档之一：`lead` / `support` / `functional` | `"lead"` |
| `characters[].arc` | string | 条件 | `lead` / `support` 必填；`functional` 可省 | `"从隐忍到掌权"` |
| `characters[].from` | array<string> | 是 | 改动记录，非空数组：原著对应谁、合并了谁、纯原创写 `["原创"]` | `["原著：林晚","合并：闺蜜线"]` |

| tier | 是谁 | 上限 | 规则 |
|---|---|---|---|
| `lead` | 主角组（男女主 + 主反派） | 1–5 人 | `arc` 必填 |
| `support` | 有名字的重要配角（亲属、闺蜜、副反派） | ≤ 10 | `arc` 必填 |
| `functional` | 功能性角色（医生、秘书、店员） | ≤ 10 | 占脸不占名：`name` 用称呼标签；`arc` 可省 |

> 映射说明：`lead` → protagonist，`support` → supporting，`functional` → minor；反向 cast.json 也可映射。

#### 4. scenes 场景表

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `scenes` | array<object> | 是 | 场景列表 | `[{...}]` |
| `scenes[].id` | string | 是 | `S01` 格式，全局唯一 | `"S01"` |
| `scenes[].name` | string | 是 | 场景名 | `"医院走廊"` |
| `scenes[].primary` | boolean | 是 | 是否主场景；主场景上限随集数动态，见 `thresholds.maxPrimaryScenes` | `true` |
| `scenes[].reusePlan` | string | 条件 | 全剧只出现一次的场景必须带；写复用哪个现有环境资产、换什么时段天气改出来 | `"复用医院走廊，改夜雨"` |

> 覆盖约束：每个场景至少被 `episodes[].sceneIds` 使用一次。

#### 5. props 叙事道具（可选表）

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `props` | array<object> | 否 | 可选字段；没写照常通过全部质量门，`prop-cap` 与 `refs` 两道会明说跳过；写了就按下面查 | `[{...}]` |
| `props[].id` | string | 条件 | 写了 props 必填；`P01` 格式，全局唯一 | `"P01"` |
| `props[].name` | string | 条件 | 写了 props 必填 | `"玉佩"` |
| `props[].function` | string | 条件 | 写了 props 必填；这件物件在戏里承载什么；填不出来说明不是叙事道具，归场景陈设/美术场景锚点 | `"证明女主身份"` |
| `props[].beatIds` | array<string> | 否 | 它托起哪几个爽点；写了就必须指向 `beats` 里真实存在的 id | `["B03","B07"]` |

> 上限与边界：`maxProps` 默认 `8`，可进 `params.thresholds` 覆盖；只收有特写、跨集出现、承载剧情的。分集用 `episodes[].propIds` 引用；没有任何一集引用的道具会被 `refs` 门点名。尺度、锚点、状态变体、白底提示词归 `novel-art`，不在这里定。

#### 6. beats 爽点表

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `beats` | array<object> | 是 | 爽点/节拍列表 | `[{...}]` |
| `beats[].id` | string | 是 | `B01` 格式 | `"B01"` |
| `beats[].type` | string | 是 | 自由文本：打脸/揭破/反转… | `"打脸"` |
| `beats[].weight` | string | 否 | 只能 `major` / `minor`；缺省算 `minor` | `"major"` |
| `beats[].episode` | integer | 是 | 唯一数据源；分集不再列 `beatIds`，节奏条和资产清单都从这里算 | `3` |
| `beats[].setup` | string | 是 | 铺垫 | `"反派当众羞辱女主"` |
| `beats[].payoff` | string | 是 | 回收/爆发 | `"女主亮出身份反打"` |

> 硬规则：相邻爽点间隔 ≤ `maxBeatGap`；开头结尾无真空；至少一个 `major`，且最早的 `major` 不能落在最后一集。

#### 7. episodes 分集梗概

| 字段路径 | 类型 | 必填 | 说明/约束 | 示例 |
|---|---|---|---|---|
| `episodes` | array<object> | 是 | 分集列表 | `[{...}]` |
| `episodes[].ep` | integer | 是 | 从 `1` 连续编号，总数等于 `params.episodes` | `1` |
| `episodes[].synopsis` | string | 是 | 分集梗概；叙述体；出现 `「」『』“”` 引号对白就是写剧本，越界，validate 会拦 | `"女主被赶出家门后发现自己才是继承人"` |
| `episodes[].hook` | string | 是 | 开头钩子；必填，空了视为未完成；叙述体，禁止引号对白 | `"她被当众赶出家门"` |
| `episodes[].suspense` | string | 是 | 结尾悬念；必填，空了视为未完成；叙述体，禁止引号对白 | `"门外停下一辆来接她的车"` |
| `episodes[].sceneIds` | array<string> | 是 | 必填且必须指向已登记 `scenes[].id`；每个场景至少用一次 | `["S01","S02"]` |
| `episodes[].characterIds` | array<string> | 是 | 必填且必须指向已登记 `characters[].id`；每个角色至少出现一集 | `["C01","C02"]` |
| `episodes[].propIds` | array<string> | 否 | 可选；写了 `props` 才有意义；必须指向已登记道具 id；每件道具至少出现一集 | `["P01"]` |
| `episodes[].crowdPlan` | string | 条件 | `characterIds ≥ 3` 的集必须写；同框拆解方案。校验按人数判，是代理指标；如果三人实际不同框，把事实写成方案即可 | `"三人分处两场，无同框，分场拍"` |
| `episodes[].warnings` | array<string> | 否 | 梗概里扫到生成难点关键词（雨戏/肢体接触/人群/手部特写）就必须列进来 | `["雨戏","人群"]` |

#### 8. 全局校验与引用关系

| 规则 | 说明 |
|---|---|
| ID 唯一性 | `characters[].id`、`scenes[].id`、`props[].id`、`beats[].id` 各自全局唯一 |
| 引用存在 | `episodes[].sceneIds` / `characterIds` / `propIds` 必须指向已登记 id；`props[].beatIds` 必须指向真实 beats id |
| 覆盖约束 | 每个角色至少出现一集；每个场景至少用一次；每件道具至少出现一集 |
| 分集数量 | `episodes` 总数必须等于 `params.episodes`，`ep` 从 1 连续 |
| adaptation 条件 | `keep` 至少一条；`adaptMode` 不是忠实时 `cut` 不能为空；`cutNote` / `mergeNote` 给了不能是空字符串 |
| 空数组 | `preferences` / `cut` / `merge` / `risks` / `warnings` 建议用 `[]`，不要用 `null` |
| beats 硬规则 | 相邻爽点间隔 ≤ `maxBeatGap`；开头结尾无真空；至少一个 `major`，且最早 `major` 不能落最后一集 |
| 场景上限 | `maxPrimaryScenes` 缺省动态：`4 + ceil(集数/10)` 夹在 `5–15`；60集→`10` |
| props 可选 | 没写 `props` 时，`prop-cap` 与 `refs` 明说跳过；写了按 props 表校验 |
| 叙述体 | `synopsis` / `hook` / `suspense` 禁止 `「」『』“”` 引号对白 |

## 最终输出 json 结构
输出示例
```json
{
  "source": "书名",
  "lang": "zh",
  "params": { "episodes": 60, "minutesPerEpisode": 2, "genre": "女频逆袭", "adaptMode": "抽核", "preferences": [] },
  "adaptation": { "core": "…", "keep": [], "cut": [], "merge": [], "risks": [] },
  "characters": [ { "id": "C01", "name": "…", "role": "…", "arc": "…", "from": ["原著…", "合并：…"] } ],
  "scenes": [ { "id": "S01", "name": "…", "primary": true, "reusePlan": "…" } ],
  "beats": [ { "id": "B01", "type": "打脸", "weight": "major", "episode": 3, "setup": "…", "payoff": "…" } ],
  "episodes": [ { "ep": 1, "synopsis": "…", "hook": "…", "suspense": "…", "sceneIds": ["S01"], "characterIds": ["C01"], "propIds": ["P01"], "crowdPlan": "…", "warnings": [] } ]
}
```
