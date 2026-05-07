# 中文参考文献.skill

> 「不要让参考文献格式，消耗你真正写论文的时间。」

把中文或英文文献题名，变成可核验的中文论文参考文献条目和 BibTeX。

不是根据标题猜一条格式，也不是把搜索结果随手拼起来。这个 skill 会先核验公开元数据，再按通用中文论文参考文献习惯输出；找不到明确来源时，它会告诉你没找到，而不是编一个看起来像真的结果。

适合这些场景：

- 写中文论文、课程报告、开题报告、基金材料，需要整理参考文献。
- 只有文献题名，需要补全作者、年份、期刊/会议、DOI、URL。
- 需要同时拿到中文参考文献条目和 BibTeX。
- 不想把 CNKI、Crossref、DBLP、出版社页面和 BibTeX 格式来回手动对齐。

看效果 · 安装 · 它格式化什么 · 工作原理 · 仓库结构

---

## 效果示例

你只需要把题名丢给 Codex：

```text
帮我把下面这些文献补全成中文论文参考文献格式，并输出 BibTeX：

1. Semi-supervised classification with graph convolutional networks
2. Focal loss for dense object detection
3. 理想的图书馆员和信息专家的素质与形象
```

它会先核验公开来源，再输出：

````text
标准参考文献：[1] Kipf T N, Welling M. Semi-supervised Classification with Graph Convolutional Networks[J]. International Conference on Learning Representations, 2017. DOI:10.48550/arXiv.1609.02907.
BibTeX：
```bibtex
@article{kipf-2017-semi-supervised-classification,
  author = {Kipf, Thomas N and Welling, Max},
  title = {Semi-supervised classification with graph convolutional networks},
  journal = {International Conference on Learning Representations},
  year = {2017},
  doi = {10.48550/arXiv.1609.02907}
}
```
核验来源：https://openreview.net/forum?id=SJU4ayYgl
````

如果没有可靠匹配，它会明确返回：

```text
未找到与“definitely not a real paper title”明确对应的公开文献。
```

如果多个来源冲突，它会给出歧义说明，而不是替你拍脑袋选一个。

---

## 安装

把仓库放到 Codex 可发现的 skills 目录：

```bash
git clone https://github.com/Zechang-Xiong/chinese-reference-formatter.git ~/.codex/skills/chinese-reference-formatter
```

如果你设置了 `CODEX_HOME`：

```bash
git clone https://github.com/Zechang-Xiong/chinese-reference-formatter.git "$CODEX_HOME/skills/chinese-reference-formatter"
```

然后在 Codex 里直接调用：

```text
Use $chinese-reference-formatter to format these literature titles as Chinese academic references with BibTeX.
```

或者用中文说：

```text
帮我把这些文献题名补全为中文参考文献格式，并给 BibTeX。
```

---

## 它格式化什么

这个 skill 默认面向通用中文论文参考文献写法，接近常见 GB/T 7714 数字顺序制习惯。它支持：

类型 | 标识 | 典型字段
--- | --- | ---
期刊论文 | `J` | 作者、题名、期刊、年、卷、期、页码、DOI
会议论文 | `C` | 作者、题名、会议/论文集、年、页码
图书/专著 | `M` | 作者、书名、版次、出版地、出版社、年份
析出章节 | `M` | 章节作者、章节题名、图书题名、出版社、页码
学位论文 | `D` | 作者、题名、保存地、保存单位、年份
报告 | `R` | 作者、题名、机构、年份
标准 | `S` | 标准编号、标准名称、发布机构、年份
专利 | `P` | 申请者、专利名、国别、专利号、发布日期
报纸 | `N` | 作者、题名、报纸名、日期、版次
在线资源 | `EB/OL` | 作者、题名、发布日期、引用日期、URL

作者规则：

- 3 名及以内作者全部列出。
- 超过 3 名作者，中文文献使用 `等`，英文文献使用 `et al`。
- 中文作者名按来源保留。
- 西文作者使用姓在前、名缩写在后，例如 `Einstein A`。

---

## 工作原理

输入一个或多个题名后，它做四件事：

1. 读取规则
   先读取 `references/chinese-reference-rules.md`，确认当前格式约束。

2. 核验元数据
   按文献类型检索公开来源，比对题名、作者、年份、载体、DOI/ISBN/URL。

3. 解决冲突
   DOI、出版社、官方期刊、会议、标准、专利、大学仓储等权威来源优先；如果来源冲突且无法判断，就返回歧义说明。

4. 格式化输出
   把已确认的元数据规范成 JSON，交给 `scripts/format_reference.py` 渲染为中文参考文献条目和 BibTeX。

这一步是有意拆开的：检索负责“别编”，formatter 负责“格式一致”。

---

## 直接运行 formatter

`scripts/format_reference.py` 只格式化已经核验过的元数据，不负责联网检索。

```bash
python scripts/format_reference.py --start 1 < metadata.json
```

输入可以是单个对象、对象数组，或 `{ "items": [...] }`：

```json
{
  "type": "article",
  "title": "Semi-supervised classification with graph convolutional networks",
  "authors": ["Kipf, Thomas N", "Welling, Max"],
  "journal": "International Conference on Learning Representations",
  "year": 2017,
  "doi": "10.48550/arXiv.1609.02907",
  "sources": ["https://openreview.net/forum?id=SJU4ayYgl"]
}
```

常用字段：

字段 | 说明
--- | ---
`type` | 文献类型，如 `article`、`inproceedings`、`book`、`thesis`、`standard`、`online`
`title` | 文献题名
`authors` | 作者列表，支持字符串或 `{ "family": "...", "given": "..." }`
`journal` / `booktitle` | 期刊名或会议/论文集名
`year` | 出版年份
`volume` / `issue` / `pages` | 卷、期、页码
`doi` / `url` | DOI 或 URL
`access_date` | 在线资源引用日期
`sources` | 核验来源 URL 列表
`status` | `not_found` 或 `ambiguous` 时输出对应提示

---

## 诚实边界

这个 skill 明确不做几件事：

- 不凭题名猜测文献信息。
- 不把聚合站结果自动当成权威来源。
- 不默认翻译题名或作者名。
- 不把学校、期刊或会议的特殊细则强加到所有用户身上。
- 不自动追加到项目 `.bib` 文件，除非你明确要求。

一个参考文献工具最重要的能力不是“看起来格式对”，而是知道什么时候不能确定。

---

## 仓库结构

```text
chinese-reference-formatter/
├── README.md
├── SKILL.md                              # skill 入口
├── agents/
│   └── openai.yaml                       # Codex 展示信息
├── references/
│   └── chinese-reference-rules.md        # 中文参考文献规则
└── scripts/
    ├── format_reference.py               # JSON -> 参考文献 + BibTeX
    ├── test_format_reference.py          # formatter 行为测试
    └── test_skill_content.py             # skill 内容回归测试
```

---

## 开发和测试

formatter 仅依赖 Python 标准库。测试使用 `pytest`：

```bash
python -m pytest scripts
```

当前测试覆盖：

- 英文期刊/会议条目。
- 三名以上作者规则。
- 中文期刊条目。
- 在线资源访问日期。
- 未匹配文献提示。
- skill 身份和通用性检查。

---

## English

Chinese Reference Formatter is a Codex skill for turning verified bibliographic metadata into Chinese academic references and BibTeX.

It is designed for common GB/T 7714-style Chinese bibliography workflows. It verifies public metadata first, formats only confirmed records, and reports uncertainty instead of inventing missing references.

Install:

```bash
git clone https://github.com/Zechang-Xiong/chinese-reference-formatter.git ~/.codex/skills/chinese-reference-formatter
```

Use:

```text
Use $chinese-reference-formatter to format these literature titles as Chinese academic references with BibTeX.
```
