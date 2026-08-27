---
language:
- zh
license: cc-by-sa-4.0
task_categories:
- text-generation
pretty_name: SFT-General-Simplified.Chinese-10K
size_categories:
- 10K<n<100K
annotations_creators:
- machine-generated
language_creators:
- found
source_datasets:
- original
tags:
- sft
- instruction-tuning
- simplified-chinese
- wikipedia
- reasoning
- datasets
- 10k
- training
- OysterCore
configs:
- config_name: default
  data_files:
  - split: train
    path: data/train.jsonl.gz
---

# SFT-General-Simplified.Chinese-10K 概况介绍

`SFT-General-Simplified.Chinese-10K` 是一个面向简体中文监督微调（SFT）研究的指令—回答数据集，针对数据源以及数据质量做出了严格把控和筛选，共 10,050 条记录。内容从中文维基百科开放转储的一手条目中提取并进行规则化任务构造；没有使用 Belle、MOSS-SFT、Alpaca-Chinese 或其他既有 SFT/指令数据集作为数据源。

> **重要边界：** 本数据集通过自动清洗、三层次筛选、定向数据收集、双逻辑 18 维评分和独立发布审计，但没有做到逐句人工事实核验，也没有完成真实模型训练消融实验。自动分数是筛选指标，并非事实正确率、法律意见或模型效果保证。高风险场景使用前请另行人工复核。

## 训练集数据概览

| 项目 | 结果 |
|---|---:|
| 记录数 | 10,050 |
| 语言 | 中文（简体规范化目标） |
| 数据分片 | `train` |
| 内容来源 | 中文维基百科开放转储 |
| 源内容与发布许可 | CC BY-SA 4.0 |
| 自动评分均值 | 93.2686 / 100 |
| 自动评分中位数 | 93.3333 / 100 |
| 自动评分范围 | 90.0000–96.6667 |
| 90–94 分 | 8,985（89.403%） |
| 95–100 分 | 1,065（10.597%） |

领域分布包括法律与公共治理、历史与文化、地理与环境、科学与技术、生活与健康、中文语言与文学、社会与人文以及综合百科。领域标签来自自动规则，仅供统计与采样参考。

## 训练集数据结构

发布分片位于 `data/train.jsonl.gz`。解压后每行是一个完整 JSON 对象：

```json
{
  "id": "1",
  "messages": [
    {"role": "user", "content": "指令文本"},
    {"role": "assistant", "content": "回答文本及来源边界"}
  ],
  "source": "new_high_quality_18d",
  "source_url": "https://zh.wikipedia.org/wiki/...",
  "license": "CC-BY-SA-4.0",
  "revision_date": "YYYY-MM-DD",
  "crawl_date": "YYYY-MM-DD",
  "domain": "领域标签",
  "quality_score_18d": 93.3333,
  "quality_dimensions_18d": {},
  "metadata": {}
}
```

`messages` 可直接映射到常见聊天模板。`source_url` 和 `revision_date` 用于追溯来源页面及编辑历史；`quality_score_18d` 和各维度分数均为自动评估结果。

## 加载方法

```python
from datasets import load_dataset

dataset = load_dataset("OysterCore/SFT-General-Simplified.Chinese-10K")
print(dataset)
print(dataset["train"][0]["messages"])
```

流式加载可以减少本地缓存占用：

```python
from datasets import load_dataset

dataset = load_dataset(
    "OysterCore/SFT-General-Simplified.Chinese-10K",
    split="train",
    streaming=True,
)
first_record = next(iter(dataset))
print(first_record["messages"])
```

也可以直接读取下载后的压缩分片：

```python
import gzip
import json

with gzip.open("data/train.jsonl.gz", "rt", encoding="utf-8") as file:
    for line in file:
        record = json.loads(line)
        messages = record["messages"]
```

## 构建流程

1. 从中文维基百科开放转储提取带来源和修订日期的页面文本；
2. 执行许可白名单、长度、中文主体、PII、广告、乱码和结构检查；
3. 清理模板、异常空白、不可打印字符、空占位符和格式残留；
4. 使用 OpenCC 并结合术语表进行繁体字形和部分地区用语规范化；
5. 仅依据来源文本构造带条件、证据和输出约束的指令—回答记录；
6. 使用两套独立启发式逻辑进行 18 维评分，并逐维取较低分；
7. 对 95 分及以上记录执行附加对抗检查；
8. 执行精确去重、近重复控制、UTF-8 严格解码和发布包一致性验证。

## 18 维自动质量评估

评估维度为：指令价值、事实可靠性、推理与思维链价值、训练信号清晰度、领域代表性与多样性、语法与表达规范性、安全与合规性、语言自然度、长度适配性、知识价值与新颖性、结构与可读性、信息密度、幻觉诱导风险、过拟合风险、指令多样性贡献、梯度贡献效率、输出可控性和负样本免疫力。

准入规则要求总分不低于 90，所有维度不低于 4，且信息密度为 5。最终自动评估显示 10,050 条全部通过。详细统计见 [`quality/QUALITY_REPORT.md`](quality/QUALITY_REPORT.md) 和 [`quality/quality_summary.json`](quality/quality_summary.json)。

请勿将这些启发式分数解释成经过人工专家逐条认证的标签。“梯度贡献效率”等指标尤其需要真实训练实验才能进一步验证。筛选数据必须满足合法合规这一硬性要求。

## 许可、署名和修改声明

源内容来自中文维基百科贡献者，并依据 [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) 使用。发布内容进行了摘取、清洗、简体规范化、结构整理、指令包装和筛选，因此本发布包采用 **CC BY-SA 4.0**。

合理署名通过以下方式提供：

- 本 Dataset Card 与 [`NOTICE.md`](NOTICE.md) 明确说明来源、许可和修改；
- 每条记录保留原页面 `source_url` 与 `revision_date`；
- 原贡献者列表可通过对应维基百科页面的“查看历史”获得。

共享本数据集或其适配版本时，应保留合理署名、原材料链接和许可链接，注明所做修改，并在 CC BY-SA 4.0 要求适用时采用相同许可。不得暗示 Wikimedia Foundation、维基百科或原贡献者认可本数据集、自动评分、下游模型或具体使用方式。详见 [`LICENSE`](LICENSE) 和 [`NOTICE.md`](NOTICE.md)。

## 适用场景

- 简体中文 SFT 和聊天模板研究；
- 可追溯来源的知识整理与证据约束训练；
- 中文数据清洗、过滤和质量评估实验；
- 领域采样、训练配比和消融研究。

## 不建议的使用场景

- 将本数据集当作无需核验的事实金标准；
- 医疗、法律、金融等高风险自动决策；
- 人物画像、身份识别或隐私推断；
- 单独用作模型事实正确率测试集；
- 违反 CC BY-SA 4.0、适用法律或第三方权利的用途。

## 已知限制

- 来源集中于中文维基百科，体裁与来源机构的多样性有限；
- 指令由规则构造，尽管模板经过多样性控制，仍可能存在风格偏斜；
- 百科页面可能含过时、争议、不完整或编辑错误的信息；
- 自动简体规范化不能保证所有专名、引文和语境都符合单一地区的用语偏好；
- 数据中可能出现历史冲突、政治、疾病等敏感但具有百科性质的主题；
- 当前仅提供训练集。建议按 `source_url` 分组后自行划分验证集，避免来源泄漏；
- 尚未通过基线模型训练、Loss 曲线比较或人工盲评证明实际增益。

## 删除、更正与安全反馈

若发现疑似隐私、权利、许可、事实或安全问题，请在仓库 Discussion 中仅提供记录 `id`、`source_url` 和简短问题描述；不要再次公开粘贴敏感信息。维护者应在确认后通过新版本删除或更正，并在变更记录中说明。

## 引用建议

使用本数据集时，请引用数据集仓库，并保留相关记录指向的中文维基百科来源。可使用：

```bibtex
@dataset{sft_general_simplified.chinese_10k_2026,
  title  = {SFT-General-Simplified.Chinese-10K},
  author = {OysterCore},
  year   = {2026},
  note   = {Derived from Chinese Wikipedia content under CC BY-SA 4.0}
}
```

## 版本记录

- `1.0.0`：10,050 条；标准 `messages` 格式；逐条来源与许可字段；采用多层次严格自动筛选和18维度自行评估；发布前独立审计。
