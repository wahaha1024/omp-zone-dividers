# omp-zone-dividers

Framed thinking divider and a matching **FINAL ANSWER** marker for [omp](https://omp.sh) transcripts, with eight switchable styles.

omp renders thinking blocks as plain italic text and glues them to the answer, so the boundary between "the model was thinking", "the model ran tools" and "this is the answer" disappears. This extension draws both boundaries.

```
╭─────────────────────────────────────────────────────────────────────────────╮
│                     ◕‿◕ THINKING DONE · 1.6k chars ◕‿◕                      │
╰─────────────────────────────────────────────────────────────────────────────╯

───────────────────────────────────────────────────────────────────────────────

◕‿◕ FINAL ANSWER ◕‿◕

The root cause is the resolver on the gateway side.
```

## Install

Requires omp 18.16+ and `extensionRenderers.assistantText` (the answer-side marker is gated by it):

```bash
omp config set extensionRenderers.assistantText true
```

### As a plugin (recommended)

```bash
omp plugin marketplace add wahaha1024/omp-zone-dividers
omp plugin install zone-dividers@omp-zone-dividers
```

Restart omp.

### As a single file

Copy [`index.ts`](./index.ts) to your user extension directory:

```bash
# Linux / macOS
cp index.ts ~/.omp/agent/extensions/zone-dividers.ts
# Windows (PowerShell)
copy index.ts "$env:USERPROFILE\.omp\agent\extensions\zone-dividers.ts"
```

Restart omp.

## Usage

```
/zone-style
```

Pick one of the eight styles; the choice is written to `~/.omp/agent/zone-dividers.json` and survives restarts. Cancelling the picker leaves the current style untouched.

| Style | Rule | Decoration | Rule color |
| --- | --- | --- | --- |
| S1 hairline | `─` | — | `mdHr` |
| S2 chip (default) | `─` | — (label gets a background fill) | `mdHr` |
| S3 double | `═` | — | `borderAccent` |
| S4 dotted | `┄` | `✻` both sides | `mdHr` |
| S5 banner | `─` | — (whole row filled) | `mdHr` |
| S6 bolt | `─` | `⚡` both sides | `warning` |
| S7 blocks | `░` | `▓▒` / `▒▓` | `borderMuted` |
| S8 kaomoji | `─` | `◕‿◕` both sides | `accent` |

## What you get

**Thinking divider** — three rows: a top rule, the label row (centered, with the style's decoration), a bottom rule. The label reads `THINKING DONE · <N> chars` and uses `k` notation above 1000 (`1.6k`). Width is `terminal width - 1`, capped at 80 columns.

**Answer marker** — two full-width rules with `FINAL ANSWER` between them, carrying the same decoration as the thinking side. Added whenever an assistant text block starts, including right after a thinking block.

Both are display-only: the session, exports and model context keep the reply as written.

## Behavior details

- **Consecutive thinking blocks** get a single divider, drawn after the last one in the run.
- **`hideThinkingBlock: true`** — the host never calls the thinking hook, so the divider disappears entirely; the answer marker stays.
- **`symbolPreset: ascii`** — thinking-side rules and decorations fall back to `-`, `=`, `:`, `#`, `*`, `^_^`. The answer-side decoration stays Unicode: the text renderer receives no theme, so it cannot see the preset.
- **Narrow terminals** — the divider keeps a floor of `label width + 4` columns so the label is never truncated; on terminals narrower than that the row exceeds the width by design.
- **Live style changes** — switching with `/zone-style` repaints thinking dividers in place; already-rendered answer markers keep their markdown.

## Why the two sides look different

Thinking blocks accept a component (`registerAssistantThinkingRenderer`), so the divider can measure columns, center the label and paint colors. The answer side only accepts markdown (`registerAssistantTextRenderer`): the host strips ANSI, reflows to the terminal width, and inserts blank lines between block elements. That is why the answer marker is a rule/label/rule sandwich rather than the same box.

## Uninstall

```bash
omp plugin uninstall zone-dividers@omp-zone-dividers
```

or delete `~/.omp/agent/extensions/zone-dividers.ts`, `~/.omp/agent/zone-dividers.json`, and optionally `omp config reset extensionRenderers.assistantText`.

## License

MIT
