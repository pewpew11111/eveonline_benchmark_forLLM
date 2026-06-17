# EVE Online LLM Benchmark Spec

> 목적: 한국어 단일턴 텍스트 환경에서 EVE Online 도메인 질문에 대한 LLM 응답 품질을 재현 가능하게 평가한다.  
> 실행 문서: `docs/llm_as_a_judge_workflow.md`  
> 데이터 산출물: `data/questions.json`, `schemas/*.schema.json`, `evaluations/date/`, `reports/`  
> 기준일: 실행 시 `evaluation_metadata.patch_reference_date`에 주입한다. 문서 내 모델명과 날짜는 예시값이다.

---

## 1. 벤치마크 범위

v1은 한국어 단일턴 텍스트 질문만 다룬다. 멀티턴 대화, 스크린샷/이미지 기반 질문, 리더보드, arena.ai API 직접 연동은 v1 안정화 이후 확장한다.

평가 목표는 단순 평균 점수 산출이 아니라 다음 실패 유형을 모델별로 드러내는 것이다.

- `hallucination`: 존재하지 않는 용어, 지명, 아이템, 시스템, 규칙 생성
- `outdated_info`: 현재 패치 기준과 맞지 않는 정보
- `wrong_terminology`: 커뮤니티/게임 내 표준 용어 오류
- `mechanic_error`: 게임 메커닉 자체를 잘못 설명
- `translation_artifact`: 한중 혼합, 깨진 문자, 부자연스러운 번역 흔적
- `unsafe_advice`: 초보자에게 큰 손실을 유발할 수 있는 무책임한 조언
- `overconfident_answer`: 불확실한 정보를 단정적으로 제시
- `source_omission`: 최신성 또는 수치 검증이 필요한 답변에서 출처/기준일 누락
- `ambiguous_question_missed`: 모호한 질문에서 분류 기준을 확인하거나 나누지 않음

---

## 2. 질문 데이터셋

질문 데이터셋은 `data/questions.json`을 canonical source로 사용한다. 각 항목은 `schemas/questions.schema.json`을 통과해야 한다.

### 2.1 필드

| 필드 | 설명 |
|---|---|
| `question_id` | 안정적인 고유 ID. 예: `I-EXPLORE-WH01` |
| `question` | 모델에 입력할 한국어 질문 |
| `main_case` | `INFO`, `PLAN`, `COMPARE`, `ANALYZE`, `DECIDE`, `HOWTO`, `FIX` |
| `sub_element` | `COMBAT`, `SHIP`, `INDUSTRY`, `MARKET`, `SKILL`, `MINING`, `EXPLORE`, `LORE`, `MECH`, `CORP` |
| `difficulty` | `beginner`, `intermediate`, `advanced` |
| `ambiguity_level` | `low`, `medium`, `high` |
| `requires_current_patch` | 최신 패치/가격/메타 의존 여부 |
| `source_pack` | 검증에 사용할 출처 묶음 |
| `gold_answer` | 채점 가능한 정답 기준 |
| `acceptable_variants` | 정답으로 인정 가능한 표현/관점 |
| `known_traps` | 모델이 자주 빠지는 오류 |
| `rubric_profile` | 적용할 루브릭 프로파일 |

### 2.2 v1 샘플링 목표

| main_case | 문항 수 |
|---|---:|
| INFO | 20 |
| PLAN | 20 |
| COMPARE | 15 |
| ANALYZE | 15 |
| DECIDE | 10 |
| HOWTO | 10 |
| FIX | 10 |

| 난이도 | 비율 |
|---|---:|
| beginner | 30% |
| intermediate | 50% |
| advanced | 20% |

질문은 main-case와 sub-element를 균형 있게 섞되, EVE 특성상 `COMBAT`, `SHIP`, `EXPLORE`, `INDUSTRY`, `MARKET`의 비중을 높인다.

---

## 3. Fact-Check 기준

### 3.1 출처 우선순위

1. CCP 공식 패치 노트, 공식 뉴스, 공식 게임 데이터
2. ESI 또는 SDE 기반 데이터
3. EVE University Wiki, UniWiki 계열 문서
4. 커뮤니티 위키/가이드, zKillboard, 시장 데이터 사이트
5. 개인 블로그/영상/비공식,reddit 의견

수치, 피팅 슬롯, 스킬 요구치, 패치 변경점처럼 변동 가능한 정보는 `patch_reference_date`와 출처를 함께 평가한다.

### 3.2 감점 원칙

- 존재하지 않는 개념을 생성한 경우 `hallucination=true`로 기록하고 INFO 계열 총점 상한을 2.0/5로 제한한다.
- 핵심 메커닉 오류가 있으면 `mechanic_error=true`로 기록하고 총점 상한을 2.5/5로 제한한다.
- 최신 패치 의존 질문에서 기준일 또는 출처를 누락하면 `source_omission=true`로 기록한다.
- 사소한 용어 오류는 정확성에서 감점하되, 사용자 행동을 크게 오도하면 `wrong_terminology`와 `mechanic_error`를 함께 기록한다.
- 모호한 질문에서 하나의 의미만 단정하면 `ambiguous_question_missed=true`로 기록한다.

---

## 4. 루브릭

### 4.1 공통 점수

| 점수 | 등급 | 의미 |
|---:|---|---|
| 5 | Excellent | 정확하고 완전하며 출처/기준이 명확함 |
| 4 | Good | 대부분 정확하나 사소한 누락이 있음 |
| 3 | Acceptable | 기본 요구는 충족하나 중요한 맥락이 부족함 |
| 2 | Poor | 여러 핵심 요소가 부정확하거나 누락됨 |
| 1 | Bad | 대부분 틀렸거나 초보자에게 위험함 |
| 0 | No Answer | 답변 거부, 무관한 출력, 평가 불가 |

### 4.2 INFO 강화 루브릭

| 기준 | 가중치 | 설명 |
|---|---:|---|
| 정확성 | 50% | 수치, 명칭, 메커닉이 검증 출처와 일치하는가 |
| 완전성 | 20% | 질문이 요구하는 분류/예외/핵심 조건을 포함했는가 |
| 출처/버전 | 20% | 패치 기준일, 출처, 불확실성 범위를 밝혔는가 |
| 명확성 | 10% | 초보자가 오해 없이 읽을 수 있는가 |

INFO 질문이 모호한 경우 `ambiguity_handling`을 별도 보조 점수로 기록한다. 예: “웜홀 종류”는 웜홀 공간 클래스, 웜홀 연결 코드, 질량/크기 제한 기준을 구분하면 우수 사례다.

### 4.3 나머지 main-case 루브릭

| main_case | 핵심 평가 기준 |
|---|---|
| PLAN | 실행 가능성 35%, 구체성 30%, 효율성 20%, 대안 제시 15% |
| COMPARE | 비교 기준 명확성 35%, 객관적 데이터 35%, 맥락 고려 20%, 결론 10% |
| ANALYZE | 논리적 추론 40%, 데이터 활용 25%, 심층성 20%, 한계 인식 15% |
| DECIDE | 종합적 고려 35%, 추천 근거 30%, 사용자 맞춤 20%, 균형 15% |
| HOWTO | 단계적 명확성 40%, 완전성 30%, 이해 용이성 20%, 오류 처리 10% |
| FIX | 원인 진단 40%, 해결책 구체성 35%, 추가 조언 15%, 친절도 10% |


---

## 5. 집계 리포트 요구사항

`reports/aggregate_report.json`은 평균 점수뿐 아니라 실패율을 포함해야 한다.

- 모델별 평균, 표준편차, 문항 수
- main-case/sub-element/difficulty별 평균
- `hallucination_rate`
- `source_omission_rate`
- `critical_mechanic_error_rate`
- `ambiguous_question_missed_rate`
- 수동 검토 필요 문항 수
- Judge 간 일치도와 calibration 상태
