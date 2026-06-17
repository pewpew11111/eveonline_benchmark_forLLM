# LLM-as-a-Judge Workflow

> 기반 스펙: `docs/eveonline_benchmark_plan.md`  
> 목적: EVE Online 한국어 벤치마크를 재현 가능하게 실행하고, Judge 평가 편향과 오류를 추적한다.  
> 주의: 문서 내 모델명과 날짜는 예시값이다. 실제 실행값은 메타데이터에 기록한다.

---

## 1. 산출물 구조

v1은 세 종류의 산출물을 분리한다.

1. 벤치마크 스펙: 질문 분류, 루브릭, fact-check 기준, 데이터 스키마
2. 실행 워크플로우: 응답 수집, Judge 평가, 집계, QA 절차
3. 예시/회귀 세트: Bad/Good 답변, 평가 결과, fact-check 근거

필수 파일:

- `data/questions.json`
- `data/regression_set.json`
- `schemas/questions.schema.json`
- `schemas/evaluation.schema.json`
- `schemas/aggregate_report.schema.json`
- `evaluations/evaluations.json`
- `evaluations/date/task_01_input.md`
- `evaluations/date/task_01_eval.json`
- `evaluations/date/summary.json`
- `reports/aggregate_report.json`

---

## 2. 실행 파이프라인

```
Phase 1: 준비
├── data/questions.json 로드 및 JSON Schema 검증
├── source_pack 기준으로 gold_answer/fact-check 기준 확인
├── 대상 모델과 Judge 모델 메타데이터 설정
└── calibration/regression 세트 준비

Phase 2: 응답 수집
├── 각 질문을 대상 LLM에 전송
├── 원문 응답 저장
└── 오류/타임아웃/빈 응답 기록

Phase 3: 절대 평가
├── 질문 + gold_answer + acceptable_variants + known_traps 제공
├── main_case별 루브릭 적용
├── error_taxonomy 기록
└── JSON Schema 검증

Phase 4: 보조 pairwise 비교
├── 동일 질문 내 답변 A/B 비교
├── winner와 이유 기록
└── 절대 점수는 변경하지 않음

Phase 5: 집계 및 QA
├── 평균/분산/카테고리별 점수 산출
├── 실패율 산출
├── 회귀 세트 통과 여부 확인
└── 수동 샘플과 MAE 비교
```

---

## 3. Judge 프롬프트 요구사항

Judge는 먼저 각 답변을 독립적으로 평가한다. A/B 비교는 독립 평가 이후 보조 분석으로만 수행한다.

필수 입력:

- 질문 원문
- `main_case`, `sub_element`, `difficulty`, `ambiguity_level`
- `requires_current_patch`
- `source_pack`
- `gold_answer`
- `acceptable_variants`
- `known_traps`
- 모델 답변
- 적용 루브릭

필수 출력:

- `scores`
- `weighted_score_5`
- `score_100`
- `error_taxonomy`
- `reasoning`
- `rationale_assessment`
- `strengths`
- `weaknesses`
- `feedback`
- `confidence`
- `needs_human_review`

### 3.1 공통 Judge 지시문

```text
당신은 EVE Online 도메인의 엄격한 벤치마크 평가자입니다.
답변의 길이나 문장 유창성보다 사실 정확성, 게임 메커닉 일치, 출처/기준일 명시를 우선 평가하세요.

각 답변은 다른 답변과 비교하지 말고 먼저 독립적으로 채점하세요.
존재하지 않는 용어/지명/아이템/시스템을 만들면 hallucination=true로 기록하세요.
핵심 게임 메커닉을 잘못 설명하면 mechanic_error=true로 기록하세요.
최신 패치 의존 질문에서 출처 또는 기준일이 없으면 source_omission=true로 기록하세요.
질문이 모호한데 분류 기준을 나누지 않고 단정하면 ambiguous_question_missed=true로 기록하세요.

반드시 JSON만 출력하세요.
```

### 3.2 INFO 질문 추가 지시

```text
INFO 루브릭은 정확성 50%, 완전성 20%, 출처/버전 20%, 명확성 10%입니다.
hallucination이 있으면 총점은 최대 2.0/5입니다.
critical mechanic error가 있으면 총점은 최대 2.5/5입니다.
```

---

## 4. 데이터 포맷

### 4.1 평가 결과

평가 결과는 `schemas/evaluation.schema.json`을 통과해야 한다. 핵심 변경점은 `error_taxonomy` 표준화다.

```json
{
  "benchmark_version": "1.0",
  "evaluation_metadata": {
    "evaluation_date": "2026-06-17",
    "patch_reference_date": "2026-06-17",
    "judge_model": "example-judge-model",
    "calibration_version": "1.0"
  },
  "target_model": "example-target-model",
  "results": [
    {
      "question_id": "I-EXPLORE-WH01",
      "weighted_score_5": 3.75,
      "score_100": 75.0,
      "error_taxonomy": {
        "hallucination": false,
        "outdated_info": false,
        "wrong_terminology": false,
        "mechanic_error": false,
        "translation_artifact": false,
        "unsafe_advice": false,
        "overconfident_answer": false,
        "source_omission": true,
        "ambiguous_question_missed": false
      },
      "needs_human_review": false
    }
  ]
}
```

### 4.2 집계 리포트

`schemas/aggregate_report.schema.json`을 통과해야 한다.

필수 지표:

- overall ranking
- by main-case/sub-element/difficulty
- hallucination rate
- source omission rate
- critical mechanic error rate
- ambiguous question missed rate
- manual review count
- calibration MAE
- inter-judge agreement

---

## 5. Calibration 및 회귀 테스트

### 5.1 Calibration Set

초기 calibration set은 10~20개 문항으로 구성한다.

- 예시1: Bad 사례. 허구 스타 색상, W1~W6, Etherium Reach, 깨진 문자 등 감지 필수
- 예시2: Good 사례. 정확성은 높게 평가하되 출처/버전 누락은 감점 필수
- 나머지 8~18개: main-case와 난이도를 섞어 수동 평가 완료 후 추가

수용 기준:

- 수동 평가 대비 MAE <= 0.5
- 동일 질문 3회 반복 평가 총점 범위 <= 0.5
- Kendall Tau >= 0.7
- 예시1의 각 답변은 3.0/5 미만
- 예시2의 각 답변은 `source_omission=true`

### 5.2 Human Review 플래그

다음 중 하나라도 해당하면 `needs_human_review=true`로 설정한다.

- Judge confidence < 0.75
- judge 간 표준편차 > 1.0
- `hallucination=true`
- `mechanic_error=true`
- `unsafe_advice=true`
- gold answer와 acceptable variants만으로 판단이 어려움

---

## 6. 집계 규칙

집계는 평균 점수와 실패율을 모두 출력한다.

- `hallucination_rate = hallucination true / evaluated count`
- `source_omission_rate = source_omission true / evaluated count`
- `critical_mechanic_error_rate = mechanic_error true / evaluated count`
- `ambiguous_question_missed_rate = ambiguous_question_missed true / ambiguous question count`

Pairwise 비교 결과는 리포트의 qualitative section에만 반영하고 overall ranking 계산에는 사용하지 않는다.

---

## 7. summary.json

당일 태스크 완료 확정시 태스크 결과를 종합하여 해당 문서에 요약한다.
