# reading-comprehension-skill · 英语阅读讲解生成器

> 上传一篇英语阅读题（图片 / PDF / 文本），得到一份可打印的 **A4 解题讲义**：读文思路、信息点清单、逻辑链、逐题证据链、干扰项命名。
>
> An agent skill that turns one English reading passage into a 12-page printable handout explaining **how to solve it** — not just the answers. Chinese handout layout, English source text, fully local rendering (Pillow only).

[中文说明](#它解决什么问题) · [快速开始](#快速开始) · [目录结构](#目录结构) · [扩展](#扩展) · [License](#license)

![封面](docs/preview-01-cover.png)

---

## 它解决什么问题

学生要的不是「这篇讲了什么」，而是「我下次遇到同类题怎么办」。大多数解析只做三件事：翻译、报答案、说「与原文不符」。这个技能强制输出五件事：

| 输出 | 内容 | 为什么有用 |
|---|---|---|
| **段落功能** | 每段贴 `现象 / 证据 / 机制 / 主张 / 评价` 标签 | 先有地图再做题，不用通读两遍 |
| **信息点** | 10–15 条 `I1…In`：原文锚点 + 中文 + 对应题号 | 命题落点可视化，复习只背这张表 |
| **逻辑链** | 6 个节点 + **每一跳的依据词** | 说明「为什么能这么推」，不靠语感 |
| **逐题证据链** | 题干 → 定位句 → 同义替换 → 干扰项类型 | 正确答案是怎么被「造」出来的 |
| **干扰项命名** | 15 类错因（对象错位 / 认知状态反了 / 拼凑后文 …） | 记住类型才能迁移，而不是记住这道题 |

---

## 预览

| 逻辑链 | 逐题精讲 |
|---|---|
| ![逻辑链页](docs/preview-02-logic.png) | ![逐题页](docs/preview-03-question.png) |

![信息点清单](docs/preview-04-infos.png)

12 页结构：封面 → 材料档案 → 逐段精读（自动分页）→ 逻辑链 → 信息点 → 关键句拆解 → 四题精讲（每题一页）→ 信号词与易错 → 词汇与练习。

---

## 快速开始

### 1. 作为 Agent Skill 安装

```bash
git clone https://github.com/<your-name>/reading-comprehension-skill.git
# 放到本地 skill 目录（DSH / Claude Code 等按各自规则）
cp -r reading-comprehension-skill ~/.dsh/skills/reading-comprehension-skill
```

装好后对 agent 说「**讲讲这篇阅读怎么做**」「拆解这篇阅读」「出一份阅读讲解讲义」，它会按 `SKILL.md` 的 8 步流程走。

### 2. 直接当 CLI 用

```bash
pip install pillow            # 唯一依赖
cd reading-comprehension-skill

# 先看效果：内置样例（一篇环境类说明文，13 条信息点、4 道题）
python scripts/engine.py templates/content_sample.json --out ./out --pdf-name "sample.pdf"

# 内容体检（缺字段、干扰项没命名、中文配 mono 字体会变成 □）
python scripts/check_content.py templates/content_sample.json
```

输出：`out/<pdf-name>`（12 页 A4，300dpi）与 `out/pages/p01.png …`（逐页位图，方便预览）。

### 3. 换成自己的文章

```bash
cp templates/content_blank.json my_content.json
# 填：packet（材料档案）/ paras（逐段精读）/ questions（逐题）/ 可选 chain、infos、key
python scripts/check_content.py my_content.json
python scripts/engine.py my_content.json --out ./out --pdf-name "C篇阅读讲解.pdf"
```

- 允许不填 `chain` / `infos` / `key`：缺哪个键就跳过对应页面，最小可出 3 页。
- 用 Python 管理内容也行：文件里定义 `CONTENT = {...}`，脚本会加载它。

---

## 工作流（技能内部按这个顺序做）

```
材料读取 ──→ 段落功能 ──→ 信息点 ──→ 逻辑链 ──→ 逐题四步 ──→ 关键句拆解 ──→ 渲染讲义 ──→ 质检
（转写原文）  （贴 5 类标签）  （I1…In）   （6 节点+依据词） （定位/替换/排除）  （切 5 个角色）   （PDF/PNG）  （10 条清单）
```

技能正文 `SKILL.md` 有两条强约束：

1. **每个判断都要贴证据**：写「第几段第几句 / 哪个词」，不写「根据上下文可知」。
2. **每个错项都要命名**：`check_content.py` 会拦住「与原文不符」这类空理由。

---

## 目录结构

```
reading-comprehension-skill/
├── SKILL.md                      技能主文档：8 步工作流、写作规则、质检清单、边界情况
├── scripts/
│   ├── engine.py                 JSON → 12 页 A4 讲解讲义（页面函数 + 自动分页）
│   ├── kit.py                    渲染原语（Pillow：圆角、曲线、字距、PNG/SVG/PDF）
│   └── check_content.py          内容体检（结构、答案一致性、干扰项命名、字体/豆腐块风险）
├── templates/
│   ├── content_sample.json       完整样例（可直接渲染出 12 页）
│   └── content_blank.json        空白骨架（3 页最小讲义）
├── references/
│   ├── method.md                 四步法、定位优先级、段落功能、5 种文体逻辑链模板、信息点规则
│   ├── question-types.md         8 类题型 + 15 类干扰项（识别口诀）+ 解题实录模板 + 变式生成
│   ├── signals.md                信号词 60 条、题干动词 20 个、陷阱词、态度词
│   └── deliverable-spec.md       版式规格、content schema 全字段、排障表
└── docs/                         README 预览图
```

---

## 内容格式（一份 JSON 讲完一篇文章）

```jsonc
{
  "title": "C 篇阅读讲解",
  "answer_cards": [["Q28", "B", "旧认知被改写：once considered = were long thought"]],
  "packet":  { "meta_table": {...}, "order_table": {...} },              // §1 材料档案
  "paras":   [{ "id": "P1", "fn": "现象 + 反转", "notes": [[lead, body], ...] }],
  "chain":   { "steps": [[num, label, body, color], ...], "basis_table": {...} },
  "infos":   { "table": { "rows": [["I1", "five huge ocean garbage patches", "洋流交汇处", "28"]] } },
  "questions": [{
    "no": "Q28", "ans": "B",
    "loc": "P1 第 2–3 句　信号词 were long thought / But",
    "opts": [["A. formed by natural currents and waves", "×", "因果错位", "currents/waves 说的是位置与移动"],
             ["B. were once considered to have no creatures", "√", "——", "once considered = were long thought"]],
    "walk": [["读题干", "know about patches → 问「已知事实」"]]
  }]
}
```

约定：`opts` 每条 = `[选项文本, 标记(√/×), 干扰类型, 判断理由]`；`table.widths` 是相对权重。
完整字段见 [`references/deliverable-spec.md`](references/deliverable-spec.md)。

---

## 干扰项类型库（摘录）

| 类型 | 识别口诀 | 例子 |
|---|---|---|
| 因果错位 | 「问的是 why 还是 where？」 | 原文「洋流在此交汇」→ 选项「由洋流形成」 |
| 对象错位 | 「这个动作的宾语是谁？」 | 原文「塑料伤害 marine life」→ 选项「塑料影响 garbage patches」 |
| 认知状态反了 | 「是旧说还是新发现？」 | `were long thought lifeless` → 选项 `were known to support diverse ecosystems` |
| 拼凑后文 | 「这个词在本句出现过吗？」 | 用后文的 `digestive` / `habitat` 造选项 |
| 方向反转 | 「动词朝哪边？」 | `plastics block digestive systems` → 选项 `help digest plastics` |
| 绝对化 | 「原文有这么绝对吗？」 | `reduce plastic use` → 选项 `stop all plastic production` |
| 以偏概全 | 「原文列了几项？」 | `businesses/governments/individuals` → 选项 `only businesses` |

全部 15 类见 [`references/question-types.md`](references/question-types.md)。

---

## 设计要点（为什么这样做）

- **内容与排版分离**：文字全在一份 JSON，`engine.py` 只排版。换文章 = 换 JSON，不动代码。
- **纯 Pillow 渲染**：不依赖 LibreOffice / Chrome / 网络；任意平台 `pip install pillow` 即可出 PDF。
- **中文绝不配 mono 字体**：Cascadia / Consolas 无中文字形会渲染成 `□`，`check_content.py` 会直接报错。
- **避头尾换行**：行首不出现 `，。、；：？！）】」』`，行尾不留开引号。
- **答案三处一致**：封面速览、逐题页、易错自检必须同答案，脚本会校验 `ans` 与 `opts` 的 √ 是否一致。

---

## 已知限制

- 只处理**单篇**阅读；一次多篇请一篇一个 content.json，分别渲染。
- 扫描件靠视觉读图，不做 OCR；原文转写由 agent 完成，转写错误会污染下游（属流程风险，非脚本问题）。
- 版式为 A4 纵向固定栅格；如需双栏或其它开本，改 `engine.py` 里的 `ML/CW/BOTTOM` 与页面函数。
- 不生成标注图；需要「词元 / POS / 实体 / 依存」四层标注图时，配合 `corpus-annotation-figure` 技能一起用。

---

## 扩展

| 想加的东西 | 改哪里 |
|---|---|
| 新题型 / 新干扰项 | 往 `references/question-types.md` 追加一行，`SKILL.md` 加一句触发说明 |
| 新信号词 | `references/signals.md` 对应小节 |
| 新版块（如「长难句分析」页） | `engine.py` 加 `page_xxx()`，在 `render()` 里挂上，`deliverable-spec.md` 补字段说明 |
| 新主题色 | 各节 `accent` 字段；标题色来自 JSON，不用改代码 |

---

## License

MIT © 2026

渲染原语 `scripts/kit.py` 派生自同作者的 `corpus-annotation-figure` 技能（同为 MIT），保留出处即可。
