---
title: "Jev 기술 발표 — 생성형 LLM과 다른 의사결정 모델"
date: 2026-09-20 09:00:00 +0900
categories: [AI, LLM]
tags: [jev, typesafe, system-one, choice, score, noul, confidence, rerank]
---

TypeSafe 공식 문서(기준 모델 `jev-1.13.0`, 기준일 2026-09-20)를 바탕으로 정리한 기술 발표 자료입니다. 이 발표의 질문은 "텍스트 생성 모델과 다른 출력 계약이 소프트웨어 설계를 어떻게 바꾸는가"입니다. Jev를 더 작은 LLM이나 새로운 챗봇으로 소개하지 않습니다.

[슬라이드 전체 화면으로 보기]({{ '/assets/slides/jev-tech-talk.html' | relative_url }}){:target="_blank"} (← → 이동, N 발표자 노트, F 전체 화면)

<iframe src="{{ '/assets/slides/jev-tech-talk.html' | relative_url }}" style="width:100%;aspect-ratio:16/9;border:0;border-radius:8px" allowfullscreen loading="lazy"></iframe>

## 1. 생성형 LLM과 Jev의 출력 목적

| | 일반 LLM (Generate) | Jev / System One (Decide) |
|---|---|---|
| 출력 | 사람이 읽을 텍스트와 코드 | 코드가 직접 쓸 타입이 정해진 답 |
| 답의 공간 | 열린 답변 공간, 복합 추론 | 호출자가 답의 공간을 먼저 정의 |
| 구조화 | 구조화 출력도 생성 과정의 결과 | 선택지별 확률과 불확실성을 노출 |

공식 문서는 LLM을 인간이 읽을 텍스트를 만드는 시스템으로, System One을 소프트웨어가 직접 사용할 결정을 만드는 시스템으로 구분한다. Jev는 답변, 코드, 추론 설명을 생성하지 않는다.

> 출처: [Introduction](https://docs.typesafe.ai/introduction) · [System One](https://docs.typesafe.ai/concepts/system-one)

## 2. System One의 입력과 출력 계약

1. **INPUT — State**: 문자열, JSON 객체, 텍스트 배열로 판단 대상과 근거를 제공한다.
2. **EVALUATION — Typed questions**: Choice, Score, Noul 질문을 같은 State에 대해 독립적으로 평가한다.
3. **OUTPUT — Typed answers + probabilities**: 코드가 비교하고 정렬하고 임계값으로 분기할 수 있는 값(`choice`, `score`, `noul`, `probabilities`)을 반환한다.

질문 간 숨은 대화 문맥을 만들지 않고, **각 질문을 같은 State에 대해 독립적으로 평가**한다. System One의 공식 정의는 State를 평가하고 typed answers와 probabilities를 반환하는 AI 모델 부류다. 질문은 병렬로 처리되며, 한 질문의 결과가 다른 질문의 숨은 문맥이 되지 않는다.

> 출처: [System One](https://docs.typesafe.ai/concepts/system-one) · [Primitives](https://docs.typesafe.ai/primitives)

## 3. 코드가 소유하는 워크플로 제어

| 역할 | 맡는 일 | 기준 |
|---|---|---|
| 코드 | 계산, 검증, 정책 결합, 외부 API 호출과 부수효과 | 결정론적으로 표현 가능한 일 |
| **Jev** | 분류, 관련성, 위험, 적합성 같은 좁은 의미 판단 | 빠른 원자적 판단 |
| 일반 LLM | 답변 작성, 요약, 코드 생성, 계획과 다단계 추론 | 열린 생성과 숙고 |
| 사람 | 고위험 결정, 불확실 사례, 정책 예외와 최종 책임 | 판단 책임과 승인 |

TypeSafe의 권장 구조는 평범한 소프트웨어 워크플로를 먼저 만들고, AI가 필요한 지점에 System One을 삽입하는 방식이다. 송금, 삭제, DB 변경 같은 행동은 코드가 실행한다.

> 출처: [How to build with TypeSafe](https://docs.typesafe.ai/concepts/how-to-build-with-system-one) · [Intent routing](https://docs.typesafe.ai/patterns/intent-routing)

## 4. State 입력 계약

| 형식 | 용도 |
|---|---|
| String | 메시지나 짧은 문단 하나 |
| Object | 이름 있는 필드와 관련 레코드 |
| Array | 대화나 레코드의 시퀀스 |

공식 문서는 대부분의 요청에서 관계가 드러나는 JSON 객체를 권장한다.

```json
{
  "ticket": {
    "message": "같은 주문이 두 번 결제됐습니다."
  },
  "charges": [
    {"amount": 49000, "status": "captured"},
    {"amount": 49000, "status": "captured"}
  ],
  "refund_policy": "중복 결제는 환불 가능"
}
```

State에는 **사실과 문맥**을, Question에는 **판단 기준**을 둔다. 필요한 근거만 보내고, 질문에서 중첩 경로를 명시한다. Jev는 이미지, 음성, 영상을 직접 받지 않으므로(텍스트 입력만 지원) 먼저 텍스트나 구조화 필드로 변환해야 한다.

> 출처: [State](https://docs.typesafe.ai/concepts/state)

## 5. 공식 Choice 요청과 응답

**Input · State + typed question**

```python
state = "My running shoes arrived in the wrong size. Can I swap them for a size 10?"

questions = {
  "department": Choice(
    instructions="Which team should handle this?",
    criteria={
      "returns": "Exchanges, wrong or damaged items",
      "shipping": "Delivery status, delays, lost packages",
      "billing": "Charges, invoices, payment problems"
    }
  )
}
```

**Output · Typed answer + uncertainty**

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "department": {
      "type": "choice",
      "choice": "returns",
      "confidence": 1.0,
      "probabilities": {
        "shipping": 0.0,
        "returns": 1.0,
        "billing": 0.0
      }
    }
  },
  "usage": {"input_tokens": 328, "output_tokens": 34}
}
```

질문 ID `department`가 응답 키로 유지된다. 코드는 `choice`로 분기하고 `probabilities`와 `confidence`로 검토 정책을 정한다. 이 쉬운 티켓에서는 returns가 1.0이지만, 실제 경계 사례에서는 분포가 여러 옵션으로 갈리고 confidence가 낮아지므로 코드가 자동 처리와 사람 검토를 나눈다.

> 출처: [Choice](https://docs.typesafe.ai/primitives/choice) · 공식 SDK 요청 예시를 축약, 응답값은 원문 유지

## 6. Choice, Score, Noul

| Primitive | 답의 형태 | 예시 | 반환값 |
|---|---|---|---|
| **Choice** | 정해진 후보 중 하나 | 담당 부서, 문서 유형, 프로그래밍 언어 | `choice`, `probabilities`, `confidence` |
| **Score** | 설명 가능한 순서형 수준 | 버그 심각도, 고객 불만도, 기술 숙련도 | `score`, `legend`, `probabilities`, `confidence` |
| **Noul** | 하나의 명제가 참인가 | 환불 요청, 개인정보 포함, 사람 상담 요청 | `noul = P(yes)` |

코드가 바로 행동할 수 있는 답의 형태를 선택한다. Choice는 분기, Score는 순서형 기준과 임계값, Noul은 하나의 명제에 대한 yes 확률이다.

> 출처: [Primitives](https://docs.typesafe.ai/primitives)

## 7. Choice의 확률 벡터

공식 복합 티켓 응답값:

| 옵션 | 확률 |
|---|---|
| returns | 0.61 |
| billing | 0.35 |
| shipping | 0.04 |

→ `choice = returns`, `confidence = 0.42`

- 최고 확률 옵션이 선택 결과가 된다.
- 전체 분포의 합은 1이다.
- 두 번째 후보 0.35도 라우팅 정보로 활용할 수 있다.
- 옵션은 최대 255개까지 정의할 수 있다.

Choice의 probabilities는 개념적으로 벡터이며 API에서는 옵션 이름과 확률의 맵으로 표현된다. confidence는 이 벡터가 얼마나 한곳에 집중되는지를 하나의 숫자로 요약한다.

> 출처: [Choice](https://docs.typesafe.ai/primitives/choice)

## 8. Score의 확률가중 평균

| 수준 | 확률 |
|---|---|
| 0 Cosmetic | 0.00 |
| 1 Workaround | 0.57 |
| 2 Blocking | 0.43 |

`0×0.00 + 1×0.57 + 2×0.43 = 1.43` → **score 1.43** (수준 축 위의 위치), **confidence 0.35** (분포 집중도)

같은 score라도 분포가 다를 수 있다. 정확한 양이나 비율로 해석하지 않고 **probabilities와 함께 읽는다**. Score는 실제 수량을 복원하는 도구가 아니다. 공식 jaggedness 문서는 score level의 수치 보정이 약하므로 정확한 크기를 보간하지 말라고 경고한다. 2~10개 수준을 지원한다.

> 출처: [Score](https://docs.typesafe.ai/primitives/score)

## 9. Noul의 P(yes)

"고객이 사람 상담을 요청하는가?" → `noul = 0.99` (`P(yes)`, 별도 confidence 없음)

0.5는 "중간 정도"가 아니라 **yes와 no 사이의 불확실성**이다. 0.0(NO) — 정책에 따른 검토 구간(REVIEW) — 1.0(YES)으로 임계값을 둔다.

false positive가 비싸면 yes 임계값을 높이고, true positive를 놓치는 것이 비싸면 낮춘다. 여러 속성이 동시에 참일 수 있는 다중 라벨 체크리스트에는 여러 Noul을 같은 호출에 넣는다.

> 출처: [Noul](https://docs.typesafe.ai/primitives/noul)

## 10. Probability와 Confidence

- **probabilities**: 옵션·수준 전체에 분배된 정보
- **confidence** (0…1): 분포 집중도를 요약한 스칼라
- Noul은 이진 분포이므로 `P(yes)` 하나가 분포를 완전히 나타낸다.

해석에서 지켜야 할 경계:

- 최고 확률과 confidence는 같은 값이 아니다.
- confidence 0.8은 개별 답의 80% 정답 보증이 아니다.
- Calibration은 예측 집단에서 측정하는 통계적 성질이다.
- 실제 임계값은 자체 데이터와 행동 위험으로 정한다.

확률을 보여준다는 사실만으로 신뢰성이 보장되지 않는다. 프로젝트 데이터에서 confidence와 정확도의 관계를 측정하고, 업무 위험별로 다른 threshold를 둬야 한다.

> 출처: [Confidence](https://docs.typesafe.ai/confidence) · [System One](https://docs.typesafe.ai/concepts/system-one)

## 11. 원자적 질문 설계

**Broad question (나쁜 예)**: "이 티켓을 분석하고 최선의 처리를 결정해줘"

**Atomic questions**:

- Q1. 주요 의도는 무엇인가?
- Q2. 고객이 환불을 명시적으로 요청했는가?
- Q3. 정책이 이 요청을 지원하는가?
- Q4. 문제의 심각도는 어느 수준인가?
- CODE. 가중치, 임계값, 정책 예외를 결합한다.

독립 요소를 한 질문에 숨기면 무엇이 틀렸는지 찾기 어렵다. 질문을 분해하면 각각을 테스트하고, 코드에서 가중치만 바꾸며 정책을 조정할 수 있다.

> 출처: [Primitives](https://docs.typesafe.ai/primitives) · [How to build with TypeSafe](https://docs.typesafe.ai/concepts/how-to-build-with-system-one)

## 12. 코드 중심의 네 가지 아키텍처 패턴

1. **Speculative fan-out** — 필요할 수 있는 독립 질문을 한 요청에 보내고 코드가 사용할 답을 고른다.
2. **Confidence routing** — 답의 종류와 confidence를 함께 보고 자동 처리, 확인, 검토로 나눈다.
3. **Composite scoring** — 여러 Score를 정규화하고 업무 가중치는 코드에서 결합한다.
4. **Intent routing** — 결정론적 코드, 전문 LLM, 사람 중 적절한 처리기로 보낸다.

**답은 무엇인지** 알려주고, **confidence는 행동할지** 결정하는 두 번째 축이 된다. 공식 예시의 0.6, 0.85 같은 수치는 출발점일 뿐이다. 문서도 올바른 threshold는 도메인, 모델 성능, 오판 비용에 따라 정하라고 명시한다.

> 출처: [Patterns](https://docs.typesafe.ai/patterns) · [Confidence-gated routing](https://docs.typesafe.ai/patterns/confidence-routing)

## 13. 14문항 루브릭의 공식 자기일관성 실험

| 모델 | 평균 왕복 지연시간 |
|---|---|
| **jev-1.13.0** | **111ms** |
| gpt-5.4-mini · yes/no t=0 | 1,113ms |
| claude-haiku-4-5 · yes/no t=0 | 1,485ms |
| gpt-5.5 reasoning | 11,125ms |
| claude-opus-4-8 reasoning | 13,886ms |

- Jev 질문별 확률 표준편차 평균: **0.0102**
- `covered: 0.43–0.53` — Jev도 0.5 임계값을 넘나들었다.

벤더가 수행한 한 가지 과제의 결과이며, 독립적인 정확도 우월성 증거가 아니다. TypeSafe의 확률 변동은 비교 조건 중 낮았지만 0이 아니었다. 비용 표는 Cookbook이 명시한 과거 가격 가정을 사용하므로 현재 청구 금액으로 제시하지 않는다. 이 슬라이드는 속도와 자기일관성 실험이다.

> 출처: [Self-consistency: nouls](https://docs.typesafe.ai/cookbooks/consistency_noul_cookbook) · 2026-09-11 실행

## 14. 한 요청에 질문을 묶는 이유

| 조건 | 비용 | 시간 | 비고 |
|---|---|---|---|
| 1 call · 13 questions | **$0.000497** | 0.27초 | 53,777자 문서를 한 번 전송 |
| 13 calls · 1 question each | $0.006090 | 2.71초 | 같은 문서를 열세 번 전송 |

→ **12.2× 비용 절감**, **10.0× 총 시간 단축**(순차 실행 대비)

속도 비교는 13개 단일 호출의 지연시간을 합한 순차 실행 기준이다. 동시 실행하면 시간 차이는 줄지만 입력 토큰 비용 차이는 남는다. Primitives 페이지는 11.5배와 9.6배라고 요약하지만 Cookbook 본문 계산 결과는 12.2배와 10.0배다. 여기서는 실험 원문 표의 값을 사용했다.

> 출처: [Parallel questions](https://docs.typesafe.ai/cookbooks/parallel_questions) · 모델 `jev-1.12`, 조건당 5회

## 15. 검색 결과의 의미 기반 재순위

| | BM25 | BM25 + Jev re-rank |
|---|---|---|
| Top 1 | 5% | **18%** |
| Top 5 | 15% | **35%** |
| Top 10 | 38% | **62%** |

3,565개 법률 구절 · 40개 쿼리 · 상위 30개 재순위 · 1,200회 호출 · **$0.0645**

재순위는 1차 검색이 고르지 않은 구절을 새로 추가할 수 없다. Jev는 전체 검색을 대체하지 않는다. BM25 같은 빠른 검색이 후보를 좁히고, Noul 확률이 후보의 의미 관련성을 평가해 순서를 바꾸는 역할을 한다.

> 출처: [Re-ranking](https://docs.typesafe.ai/cookbooks/rerank_typesafe) · CLERC 데이터셋 공식 walkthrough

## 16. Jev 1.13의 공식 실패 모드

**코드가 더 잘하는 일**
- 수학과 숫자 정밀도
- 개수 세기
- 날짜·시간 비교
- Score로 정확한 수량 보간

**질문과 State의 위험**
- 문자 그대로의 해석
- 여러 단계의 간접성
- 무관한 정보가 많은 State
- 적대적 콘텐츠
- 서로 모순된 instructions와 criteria

**능력의 경계**
- 상식적 구조 불변성 미보장
- 질문과 부정 질문의 합이 1이 아닐 수 있음
- 텍스트·코드·설명 생성 미지원

산술은 코드에, 생성은 LLM에, Jev에는 **의미 판단만** 맡긴다. Jev는 결정론적이지 않으며 구조적 항등식도 자동 보장하지 않는다. 같은 결정을 한 가지 질문 형태로 평가하고, 합계·상호배타성·보수 관계는 코드에서 강제한다.

> 출처: [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13) · 2026-09-17 검토

## 17. 운영 스펙과 한국어 도입 위험

| 항목 | 값 |
|---|---|
| 모델 | jev-1.13.0 |
| 가격 | $42 / Btok · $0.042 / Mtok |
| 과금 | 입력 토큰만 · 출력 토큰 무료 |
| 레이트 리밋 | 250k tokens/s · 1,200 req/min |
| 컨텍스트 | 64k/request · state + 최장 질문 32k |
| 입력 | Text only |
| 데이터 | 요청·응답으로 학습하지 않음 |

**한국 프로젝트 핵심 리스크**: 영어가 주 학습 언어이며 정확도가 가장 높다. CJK를 포함한 다른 언어도 처리하지만 동등한 정확도를 보장하지 않는다. 한국어 실데이터 PoC와 confidence 기반 라우팅이 필요하다.

**운영 경고**: 공개 레이트 리밋은 예고 없이 바뀔 수 있다. 임계값을 튜닝했다면 이동하는 alias보다 버전 ID를 고정하고 응답 모델을 로그로 남긴다.

**보안과 계약**: 엔터프라이즈 고객에게 ZDR을 제공하며 공식 Legal 페이지에 DPA, Master Customer Agreement, Privacy Policy가 연결되어 있다. 공개 문서에 없는 인증, 리전, SLA는 계약 전에 별도 확인한다.

> 출처: [Models](https://docs.typesafe.ai/models) · [Legal](https://docs.typesafe.ai/legal)

## 18. Jev가 적합한 프로젝트

**적합**
- 반복량이 많은 분류와 라우팅
- 여러 독립 기준의 관련성·위험 판정
- 확률로 정렬하는 재순위
- 불확실 사례만 LLM이나 사람에게 보내는 게이트
- 코드가 최종 정책과 행동을 소유하는 워크플로

**부적합**
- 자유로운 답변과 문서·코드 생성
- 정확한 산술, 계산, 개수 세기
- 복합 다단계 추론을 한 번에 요구하는 과제
- 이미지·음성·영상의 직접 이해
- 언어별 자체 검증 없이 수행하는 고위험 자동화

Jev 도입의 기준은 모델 유행이 아니라 과제 형태다. 답의 공간이 닫혀 있고, 동일 판단이 반복되며, 불확실성을 다음 처리기로 라우팅할 수 있을 때 가치가 커진다.

> 출처: [Use case map](https://docs.typesafe.ai/concepts/use-case-map) · [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13)

## 19. Shadow PoC 기반 도입 검증

1. **과제 선택** — 저위험·고빈도의 좁은 판단 하나
2. **Gold set** — 한국어 실데이터와 경계 사례 라벨
3. **Shadow** — 기존 경로를 바꾸지 않고 결과 기록
4. **Threshold** — 업무 위험별 자동·확인·검토 구간
5. **Rollout** — 버전 고정, 재평가, 단계적 확대

> Jev는 판단하고, 코드는 계산·검증·행동하며, 일반 LLM은 생성·복합 추론하고, 사람은 고위험 사례를 책임진다.

성공 기준에는 정확도만 넣지 않는다. 지연시간, 비용, 자동 처리율, 사람 검토율, 임계값별 오류, 버전 변화까지 함께 측정한다. 공식 Cookbook 수치는 가설을 세우는 근거이고 최종 결정은 자체 PoC 결과로 내린다.
