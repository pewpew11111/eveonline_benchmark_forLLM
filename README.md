# EVE Online LLM Benchmark

> A reproducible benchmark evaluating LLM response quality on EVE Online domain questions in Korean single-turn text environments.

> **Note**: Evaluated models generate responses using only their pre-trained parameters, without web search or retrieval (RAG) capabilities. Search-augmented or browser-based answers are not within the current evaluation scope.

## Overview

This project provides a systematic benchmark to evaluate how well LLMs understand and respond to questions about **EVE Online**, a complex MMORPG known for its steep learning curve and deep gameplay mechanics.

The evaluation focuses on identifying **specific failure types** rather than just producing average scores:

| Failure Type | Description |
|---|---|
| `hallucination` | Inventing non-existent terms, locations, items, systems, or rules |
| `outdated_info` | Information inconsistent with the current patch |
| `wrong_terminology` | Incorrect community/in-game standard terminology |
| `mechanic_error` | Misexplaining core game mechanics |
| `translation_artifact` | Mixed Korean/Chinese, broken characters, unnatural translation traces |
| `unsafe_advice` | Irresponsible advice that could cause significant loss for beginners |
| `overconfident_answer` | Presenting uncertain information assertively |
| `source_omission` | Omitting sources or reference dates for answers requiring recency/verification |
| `ambiguous_question_missed` | Not clarifying classification criteria for ambiguous questions |

## Status

**v1.0-alpha** — First run completed on 2026-06-07, evaluating 7 models across 4 tasks.

### Evaluated Models (First Run)

| Model | Avg Score (5pt) |
|---|---|
| gpt-5.5 | 4.30 |
| gpt-5.5-instant | 3.58 |
| claude-opus-4-8 | 3.55 |
| claude-sonnet-4-6 | 2.95 |
| grok-4.3 | 1.40 |
| minimax-m2.7 | 1.40 |
| ernie-5.1-preview | 1.30 |

## Directory Structure

```
├── data/                    # Question dataset and regression set
│   ├── questions.json       # Canonical question dataset
│   └── regression_set.json  # Regression test examples
├── docs/                    # Benchmark spec and workflow documents
│   ├── eveonline_benchmark_plan.md  # Benchmark spec (taxonomy, rubric, fact-check)
│   ├── llm_as_a_judge_workflow.md   # LLM-as-a-Judge pipeline
│   └── yt_workflow.md               # YouTube-based question generation
├── evaluations/             # Individual evaluation results (by date)
│   └── YYYY-MM-DD/
│       ├── task_N_input.md     # Task input
│       ├── task_N_eval.json    # Task evaluation result
│       └── summary.json        # Daily evaluation summary
├── inputs/                  # (Planned) Raw questions and model responses
├── reports/                 # Aggregate reports
│   └── aggregate_report.json
├── schemas/                 # JSON Schema files
│   ├── questions.schema.json
│   ├── evaluation.schema.json
│   └── aggregate_report.schema.json
├── yt_sum/                  # YouTube video summaries (reference for question generation)
├── .gitignore
├── LICENSE
└── README.md
```

## Pipeline

The benchmark runs in 5 phases:

```
Phase 1: Setup      → Load dataset, validate JSON Schema, configure Judge/model
Phase 2: Collection  → Send questions to target LLM, save raw responses
Phase 3: Evaluation  → Apply rubric per main_case, record error_taxonomy
Phase 4: Comparison  → A/B comparison within same question (no score changes)
Phase 5: Aggregation → Calculate averages/failure rates, verify regression set
```

## Data Format

### Question Dataset (`data/questions.json`)

| Field | Description |
|---|---|
| `question_id` | Stable unique ID (e.g., `I-EXPLORE-WH01`) |
| `question` | Korean question for model input |
| `main_case` | Question type: `INFO`, `PLAN`, `COMPARE`, `ANALYZE`, `DECIDE`, `HOWTO`, `FIX` |
| `sub_element` | Domain element: `COMBAT`, `SHIP`, `INDUSTRY`, `MARKET`, `SKILL`, etc. |
| `difficulty` | `beginner`, `intermediate`, `advanced` |
| `ambiguity_level` | `low`, `medium`, `high` |
| `gold_answer` | Scorable reference answer |
| `rubric_profile` | Rubric profile to apply |

### Task Evaluation (`evaluations/YYYY-MM-DD/task_N_eval.json`)

- `weighted_score_5`: Weighted score out of 5
- `score_100`: Scaled score out of 100
- `error_taxonomy`: 9 failure type flags
- `needs_human_review`: Whether manual review is needed

## Related Links

- [EVE University Wiki](https://wiki.eveuniversity.org/)
- [EVE Online Official Site](https://www.eveonline.com/)
- [ESI (EVE Swagger Interface)](https://esi.evetech.net/)

---

> 한국어 버전은 [README.ko.md](README.ko.md)를 참조하세요.