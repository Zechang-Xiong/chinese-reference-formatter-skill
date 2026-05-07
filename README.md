# Chinese Reference Formatter

通用中文参考文献格式化 Codex skill。它把中文或英文文献题名整理为中文论文参考文献条目，并同时输出 BibTeX。

这个 skill 面向 GB/T 7714 风格的常见中文学术写作场景。它的核心原则是先核验公开元数据，再格式化输出；如果无法确认唯一文献，就明确返回未找到或歧义说明，而不是根据题名猜测。

## 功能

- 根据文献题名补全并格式化中文参考文献条目。
- 支持输出对应 BibTeX。
- 覆盖期刊、会议、图书、章节、学位论文、报告、标准、专利、报纸和在线资源等类型。
- 对作者数量、英文作者缩写、英文题名大小写、在线访问日期和 DOI 做统一处理。
- 对无法核验或来源冲突的条目保留明确提示。

## 目录结构

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   └── chinese-reference-rules.md
└── scripts/
    ├── format_reference.py
    ├── test_format_reference.py
    └── test_skill_content.py
```

- `SKILL.md`：Codex skill 入口，定义何时使用、检索来源优先级、格式化流程和冲突处理规则。
- `references/chinese-reference-rules.md`：通用中文论文参考文献规则。
- `scripts/format_reference.py`：把已核验的标准化 JSON 元数据渲染为参考文献和 BibTeX。
- `agents/openai.yaml`：OpenAI/Codex 侧展示名称和默认提示。
- `scripts/test_*.py`：formatter 行为和 skill 内容回归测试。

## 安装

将仓库放到 Codex 可发现的 skills 目录中：

```bash
git clone https://github.com/Zechang-Xiong/chinese-reference-formatter.git "$CODEX_HOME/skills/chinese-reference-formatter"
```

如果没有设置 `CODEX_HOME`，请使用你的 Codex skills 根目录，例如：

```bash
git clone https://github.com/Zechang-Xiong/chinese-reference-formatter.git ~/.codex/skills/chinese-reference-formatter
```

安装后，在对话中可以直接说：

```text
Use $chinese-reference-formatter to format these literature titles as Chinese academic references with BibTeX.
```

## 使用方式

给 Codex 提供一个或多个文献题名。skill 会先查询公开来源核验题名、作者、年份、期刊或会议、DOI/ISBN/URL 等信息，再调用 formatter 输出结果。

示例请求：

```text
帮我把下面这些文献补全成中文论文参考文献格式，并输出 BibTeX：
1. Semi-supervised classification with graph convolutional networks
2. Focal loss for dense object detection
```

输出格式：

````text
标准参考文献：[1] ...
BibTeX：
```bibtex
...
```
核验来源：...
````

## 直接运行 formatter

`scripts/format_reference.py` 接受已经核验过的元数据 JSON。它不会联网查询文献，只负责格式化。

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

| 字段 | 说明 |
| --- | --- |
| `type` | 文献类型，如 `article`、`inproceedings`、`book`、`thesis`、`standard`、`online` |
| `title` | 文献题名 |
| `authors` | 作者列表，支持字符串或 `{ "family": "...", "given": "..." }` 对象 |
| `journal` / `booktitle` | 期刊名或会议/论文集名 |
| `year` | 出版年份 |
| `volume` / `issue` / `pages` | 卷、期、页码 |
| `doi` / `url` | DOI 或 URL |
| `access_date` | 在线资源引用日期 |
| `sources` | 核验来源 URL 列表 |
| `status` | `not_found` 或 `ambiguous` 时输出对应提示 |

## 开发和测试

formatter 仅依赖 Python 标准库。测试使用 `pytest`：

```bash
python -m pytest scripts
```

测试覆盖：

- 英文期刊/会议条目的中文参考文献格式。
- 三名以上作者的 `et al` / `等` 规则。
- 中文期刊条目、在线资源访问日期、未匹配文献提示。
- skill 身份和通用性内容检查。

## 设计约束

- 不能只凭题名猜测文献信息。
- DOI、出版社、官方期刊、会议、标准、专利、大学仓储等权威来源优先。
- 来源冲突且无法判断时，输出歧义说明。
- 保留题名和作者原语言，不默认翻译。
- 不追加或修改项目 `.bib` 文件，除非用户明确要求。
