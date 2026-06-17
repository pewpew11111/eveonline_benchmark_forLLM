# EVE Online LLM Benchmark

> 한국어 단일턴 텍스트 환경에서 EVE Online 도메인 질문에 대한 LLM 응답 품질을 재현 가능하게 평가하는 벤치마크 프로젝트입니다.

> **참고**: 평가 대상 모델은 웹 검색(web search/retrieval) 기능 없이, 순수 사전 학습된 파라미터만으로 응답을 생성한 결과를 평가합니다. 검색 증강 생성(RAG) 또는 브라우징 기반 답변은 현재 평가 범위에 포함되지 않습니다.

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
├── README.md                # English version
└── README.ko.md             # 한국어 버전 (현재 문서)
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

---

> English version: [README.md](README.md)