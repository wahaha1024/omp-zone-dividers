# omp-zone-dividers

给 [omp](https://omp.sh) 转录加分区线：思考区结束处画一个方框，回答区开头放一个配套的 **FINAL ANSWER** 标记，共八种可切换样式。

omp 把思考块渲染成普通斜体文本并直接拼到回答上，于是"模型在想"、"模型调了工具"、"这是回答"三者的边界消失了。这个扩展把两条边界都画出来。

```
╭─────────────────────────────────────────────────────────────────────────────╮
│                     ◕‿◕ THINKING DONE · 1.6k chars ◕‿◕                      │
╰─────────────────────────────────────────────────────────────────────────────╯

───────────────────────────────────────────────────────────────────────────────

◕‿◕ FINAL ANSWER ◕‿◕

根因在网关侧 resolver。
```

## 安装

需要 omp 18.16+，并开启 `extensionRenderers.assistantText`（回答区标记受它控制）：

```bash
omp config set extensionRenderers.assistantText true
```

### 作为插件安装（推荐）

```bash
omp plugin marketplace add wahaha1024/omp-zone-dividers
omp plugin install zone-dividers@omp-zone-dividers
```

重启 omp。

### 单文件安装

把 [`index.ts`](./index.ts) 复制到用户扩展目录：

```bash
# Linux / macOS
cp index.ts ~/.omp/agent/extensions/zone-dividers.ts
# Windows (PowerShell)
copy index.ts "$env:USERPROFILE\.omp\agent\extensions\zone-dividers.ts"
```

重启 omp。

## 使用

```
/zone-style
```

在八种样式里选一个；选择写入 `~/.omp/agent/zone-dividers.json`，重启后保持。取消选择不改动当前样式。

| 样式 | 线 | 装饰 | 线色 |
| --- | --- | --- | --- |
| S1 细线 hairline | `─` | — | `mdHr` |
| S2 胶囊 chip（默认） | `─` | —（标签段加底色） | `mdHr` |
| S3 双线 double | `═` | — | `borderAccent` |
| S4 点线星芒 dotted | `┄` | 两侧 `✻` | `mdHr` |
| S5 色带 banner | `─` | —（整行加底色） | `mdHr` |
| S6 闪电 bolt | `─` | 两侧 `⚡` | `warning` |
| S7 方块渐变 blocks | `░` | `▓▒` / `▒▓` | `borderMuted` |
| S8 颜文字 kaomoji | `─` | 两侧 `◕‿◕` | `accent` |

## 渲染结果

**思考分区**——三行：上框线、标签行（居中，带该样式的装饰）、下框线。标签文案是 `THINKING DONE · <N> chars`，超过 1000 用 `k` 记法（`1.6k`）。宽度为「终端宽度 - 1」，上限 80 列。

**回答标记**——两条整行横线夹住 `FINAL ANSWER`，装饰符号与思考侧一致。每个 assistant 文本块开头都会加，包括紧跟思考块的情况。

两者都是纯展示：会话记录、导出和模型上下文保持原文不变。

## 行为细节

- **连续思考块**只画一条分区线，落在该段最后一个思考块之后。
- **`hideThinkingBlock: true`** —— 宿主不再调用思考钩子，分区线整体消失；回答标记仍在。
- **`symbolPreset: ascii`** —— 思考侧的线与装饰降级为 `-`、`=`、`:`、`#`、`*`、`^_^`。回答侧的装饰仍是 Unicode：文本渲染器拿不到 theme，读不到预设。
- **窄终端** —— 分区线保底 `标签宽 + 4` 列，标签不会被裁剪；终端比这更窄时该行会超出宽度，这是设计上的取舍。
- **实时换样式** —— `/zone-style` 切换后，思考分区线原地重绘；已经画出的回答标记保留其 markdown。

## 两侧为何不同形

思考块接受组件（`registerAssistantThinkingRenderer`），所以能按列宽计算、居中、上色。回答侧只接受 markdown（`registerAssistantTextRenderer`）：宿主会剥掉 ANSI、按终端宽度重排、并在块级元素之间插入空行。这就是回答标记是「横线 / 标签 / 横线」而不是同一个方框的原因。

## 卸载

```bash
omp plugin uninstall zone-dividers@omp-zone-dividers
```

或删除 `~/.omp/agent/extensions/zone-dividers.ts`、`~/.omp/agent/zone-dividers.json`，需要时再 `omp config reset extensionRenderers.assistantText`。

## 许可证

MIT
