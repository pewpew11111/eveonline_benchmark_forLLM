# EVE Online LLM Benchmark

> A reproducible benchmark evaluating LLM response quality on EVE Online domain questions in Korean single-turn text environments.
>
> 한국어 단일턴 텍스트 환경에서 EVE Online 도메인 질문에 대한 LLM 응답 품질을 재현 가능하게 평가하는 벤치마크 프로젝트

<details>
  <summary>Click to expand English README</summary>
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
</details>

## 개요

이 프로젝트는 **EVE Online**이라는 복잡한 MMORPG 도메인에서 LLM의 응답 품질을 체계적으로 평가하기 위한 벤치마크입니다.

단순 평균 점수 산출이 아닌, **다음 실패 유형을 모델별로 드러내는 것**에 초점을 둡니다:

| 실패 유형 | 설명 |
|---|---|
| `hallucination` | 존재하지 않는 용어, 지명, 아이템, 시스템, 규칙 생성 |
| `outdated_info` | 현재 패치 기준과 맞지 않는 정보 |
| `wrong_terminology` | 커뮤니티/게임 내 표준 용어 오류 |
| `mechanic_error` | 게임 메커닉 자체를 잘못 설명 |
| `translation_artifact` | 한중 혼합, 깨진 문자, 부자연스러운 번역 흔적 |
| `unsafe_advice` | 초보자에게 큰 손실을 유발할 수 있는 무책임한 조언 |
| `overconfident_answer` | 불확실한 정보를 단정적으로 제시 |
| `source_omission` | 최신성 또는 수치 검증이 필요한 답변에서 출처/기준일 누락 |
| `ambiguous_question_missed` | 모호한 질문에서 분류 기준을 확인하거나 나누지 않음 |

## 프로젝트 상태

**v1.0-alpha** — 첫 번째 실행 완료 (2026-06-07), 총 7개 모델 평가, 4개 태스크

### 평가 모델 (1차 실행)

| 모델 | 평균 점수 (5점) |
|---|---|
| gpt-5.5 | 4.30 |
| gpt-5.5-instant | 3.58 |
| claude-opus-4-8 | 3.55 |
| claude-sonnet-4-6 | 2.95 |
| grok-4.3 | 1.40 |
| minimax-m2.7 | 1.40 |
| ernie-5.1-preview | 1.30 |

## 폴더 구조

```
├── data/                    # 질문 데이터셋과 회귀 세트
│   ├── questions.json       # 캐노니컬 질문 데이터셋
│   └── regression_set.json  # 회귀 테스트용 예시 세트
├── docs/                    # 벤치마크 스펙과 워크플로우 문서
│   ├── eveonline_benchmark_plan.md  # 벤치마크 스펙 (분류 체계, 루브릭, 사실 확인 기준)
│   ├── llm_as_a_judge_workflow.md   # LLM-as-a-Judge 실행 파이프라인
│   └── yt_workflow.md               # YouTube 영상 기반 질문 생성 워크플로우
├── evaluations/             # 개별 평가 결과 (날짜별)
│   └── YYYY-MM-DD/
│       ├── task_N_input.md     # 태스크 입력
│       ├── task_N_eval.json    # 태스크 평가 결과
│       └── summary.json        # 당일 평가 요약
├── inputs/                  # (예정) 원문 질문과 모델 답변이 들어 있는 입력 문서
├── reports/                 # 집계 리포트
│   └── aggregate_report.json
├── schemas/                 # JSON Schema 파일
│   ├── questions.schema.json
│   ├── evaluation.schema.json
│   └── aggregate_report.schema.json
├── yt_sum/                  # YouTube 영상 요약 (질문 생성 참고 자료)
├── .gitignore
├── LICENSE
└── README.md
```

## 실행 파이프라인

벤치마크는 5단계 파이프라인으로 실행됩니다:

```
Phase 1: 준비     → 데이터셋 로드, JSON Schema 검증, Judge/모델 설정
Phase 2: 응답 수집 → 대상 LLM에 질문 전송, 원문 응답 저장
Phase 3: 절대 평가 → main_case별 루브릭 적용, error_taxonomy 기록
Phase 4: 보조 비교 → 동일 질문 내 답변 A/B 비교 (절대 점수 변경 없음)
Phase 5: 집계 및 QA → 평균/실패율 산출, 회귀 세트 통과 확인
```

## 데이터 포맷

### 질문 데이터셋 (`data/questions.json`)

| 필드 | 설명 |
|---|---|
| `question_id` | 고유 ID (예: `I-EXPLORE-WH01`) |
| `question` | 모델에 입력할 한국어 질문 |
| `main_case` | 질문 유형: `INFO`, `PLAN`, `COMPARE`, `ANALYZE`, `DECIDE`, `HOWTO`, `FIX` |
| `sub_element` | 도메인 요소: `COMBAT`, `SHIP`, `INDUSTRY`, `MARKET`, `SKILL`, `MINING`, `EXPLORE`, `LORE`, `MECH`, `CORP` |
| `difficulty` | 난이도: `beginner`, `intermediate`, `advanced` |
| `ambiguity_level` | 모호성: `low`, `medium`, `high` |
| `gold_answer` | 채점 가능한 정답 기준 |
| `rubric_profile` | 적용할 루브릭 프로파일 |

### 평가 결과 (`evaluations/YYYY-MM-DD/task_N_eval.json`)

LLM-as-a-Judge 평가 결과 구조:

- `weighted_score_5`: 5점 만점 가중 점수
- `score_100`: 100점 환산 점수
- `error_taxonomy`: 9가지 실패 유형 플래그
- `needs_human_review`: 수동 검토 필요 여부

## 라이선스

MIT License — 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

## 관련 링크

- [EVE University Wiki](https://wiki.eveuniversity.org/)
- [EVE Online 공식 사이트](https://www.eveonline.com/)
- [ESI (EVE Swagger Interface)](https://esi.evetech.net/)