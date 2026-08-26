# MemoryCurator 规则文件

配合 `MemoryCurator_character.md` 使用,本文只写"怎么做"。角色定位、触发别名、决策日志引用都在 character 文件里。
## 1. 征询话术
触发别名(`precipitation`)命中后,停下来问一句,只问一次。三种语气任挑其一,效果相同:

1. "本次会话是否值得沉淀?回答'是'则写入 `pending/`,回答'否'或忽略则跳过。"
2. "停下来确认一下:这次会话要不要归档?回'是'就写 pending,回'否'或沉默就当没发生过。"
3. "会话结束征询。是否值得记下来?是 → `pending/<YYYY-Wxx>/<session-id>.md`,否 → 不落盘。"

收到"是"再动手,收到"否"或不回答就沉默退出。中间不允许追加追问。
## 2. 沉淀格式模板

写到 `pending/<YYYY-Wxx>/<session-id>.md` 时,严格按下面这个结构填,字段不要换位置:

```markdown
# Session <session-id> :: <ISO date>

## 摘要
<one-paragraph summary>

## 关键决策
- <decision 1>
- <decision 2>

## 待沉淀到 stable/ 的候选主题
- <topic A>: <one sentence>
- <topic B>: <one sentence>

## 不沉淀的部分(噪音/闲聊)
- <noise items>
```

`<YYYY-Wxx>` 用 ISO 周编号(例如 `2026-W34`)。这一层只求诚实不求精炼,精简留给周复盘。
## 3. 周过滤 checklist

周复盘是用户驱动的流程,脚本只是搬运工,所有判断都要人来拍板。完整流程见 `.omo/memory/README.md` 的"周过滤流程"一节,脚本行为以 `.omo/memory/scripts/weekly_review.sh` 为准。

执行顺序:先跑 `weekly_review.sh --dry-run` 列出当周 `pending/<YYYY-Wxx>/` 下未处理文件(只看不搬);逐个打开 pending 文件,按下方 checklist 核对;审完去掉 `--dry-run` 再跑一次,执行迁移。脚本不会判断主题归属,审一份就要心里有数"这份属于哪个 topic"。

每个 pending 文件的判定标准:

- [ ] pending/ 下当周文件已全部审阅
- [ ] 每个文件判定为: 沉淀 / 拆分 / 丢弃
- [ ] 沉淀的文件已写到 `stable/<topic>.md`(≤200 行)
- [ ] `INDEX.md` 与 `stable/index.md` 已更新
- [ ] pending/ 当周目录已清空

主题命名用自解释的动名词短语(`path-id-convention`、`memory-leak-patterns`),不要用 `notes`、`misc` 这种含糊词。
