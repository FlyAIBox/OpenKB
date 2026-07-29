# Slides

`openkb deck new` 将 Wiki 内容转为精美的**单文件 HTML 幻灯片** —— CSS/JS 全部内联，无外部资源，支持键盘导航，可直接全屏打开或分享。

> **本文件夹中的真实产物：** [`attention-intro.html`](attention-intro.html)
> —— 由 `openkb deck new` 从一篇论文生成的 9 页 deck，主题为
> `openkb-deck-neon`。在浏览器中打开（← → 导航，`F` 全屏）。它是单个自包含文件。

```bash
openkb deck new <name> "<intent>"
```

- `<name>` — kebab-case slug；输出位于 `output/decks/<name>/index.html`。
- `<intent>` — deck 主题；生成器据此从 Wiki 选取并排序内容。

```bash
openkb deck new attention-intro \
  "A short intro deck on the Transformer and self-attention for an ML reading group"
```

```text
Generating deck 'attention-intro' via skill openkb-deck-neon (default)...
Deck written to output/decks/attention-intro/index.html
```

该命令生成了 [`attention-intro.html`](attention-intro.html)。你也可以在 [`openkb chat`](../chat/) 内用 `/deck new …` 驱动。

---

## 选项

| 标志 | 效果 |
| --- | --- |
| `--skill <name>` | 选择 deck 主题/skill（默认 `openkb-deck-neon`）。 |
| `--critique` | 运行二轮 critic（`openkb-html-critic`），在保存前审查并修复 CSS/UX 问题。 |
| `-y, --yes` | 不提示直接覆盖已有 deck（旧版备份到 `output/decks/<name>-workspace/iteration-N/`）。 |

```bash
# Warm editorial look, with a quality pass:
openkb deck new okf-pitch "Pitch OpenKB to a data team" \
  --skill openkb-deck-editorial --critique
```

---

## 主题

Deck 由 **deck skill** 渲染 —— 一个定义视觉方向与 slide 语法的 SKILL.md。OpenKB 自带两个（无需安装）：

| Skill | 外观 |
| --- | --- |
| [`openkb-deck-neon`](../../skills/openkb-deck-neon/SKILL.md) *(default)* | 深色「Aurora Glass」—— 近黑背景，青/天蓝/品红/琥珀 neon，玻璃拟态面板。 |
| [`openkb-deck-editorial`](../../skills/openkb-deck-editorial/SKILL.md) | 暖色「Editorial Monocle」—— 奶油色、衬线、砖红强调；印刷页质感。 |

想要自定义外观？将第三方 deck skill 放入 `~/.openkb/skills/` 或
`<kb>/skills/`，用 `--skill` 选择（同名 skill 会覆盖内置版）。

---

## 语法约束防止 deck 退化

每个 deck skill 声明输出必须满足的 slide 语法。`openkb-deck-neon` 主题要求 `cover` 与 `closing`，至少 4 种不同 slide 类型，且同类型连续不超过 2 页：

```yaml
od:
  mode: deck
  output_path_template: "output/decks/{slug}/index.html"
  deck_grammar:
    kind_attr: data-type
    required: [cover, closing]
    allowed: [cover, chapter, thesis, quote, compare, data, closing]
    min_distinct: 4
    max_consecutive_same: 2
```

[`attention-intro.html`](attention-intro.html) 以 9 页满足约束，涵盖
`cover · thesis · quote · compare · data · closing` —— 这正是避免退化成千篇一律 bullet 墙的原因。
