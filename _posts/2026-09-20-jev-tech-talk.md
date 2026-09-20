---
title: "Jev 기술 발표 — TypeSafe 공식 문서 기반"
date: 2026-09-20 09:00:00 +0900
categories: [AI, LLM]
tags: [jev, typesafe, llm, choice, score, noul, rag]
---

TypeSafe 공식 문서를 바탕으로 정리한 Jev 기술 발표 자료입니다. 슬라이드는 화살표 키(← →)로 넘길 수 있습니다.

[전체 화면으로 보기]({{ '/assets/slides/jev-tech-talk.html' | relative_url }}){:target="_blank"}

<iframe src="{{ '/assets/slides/jev-tech-talk.html' | relative_url }}" style="width:100%;aspect-ratio:16/9;border:0;border-radius:8px" allowfullscreen loading="lazy"></iframe>

## 목차

1. 생성형 LLM과 Jev의 출력 목적
2. System One의 입력과 출력 계약
3. 코드가 소유하는 워크플로 제어
4. State 입력 계약
5. 공식 Choice 요청과 응답
6. Choice, Score, Noul
7. Choice의 확률 벡터 / Score의 확률가중 평균 / Noul의 P(yes)
8. Probability와 Confidence
9. 원자적 질문 설계
10. 코드 중심의 네 가지 아키텍처 패턴
11. 14문항 루브릭의 공식 자기일관성 실험
12. 한 요청에 질문을 묶는 이유
13. 검색 결과의 의미 기반 재순위
14. Jev 1.13의 공식 실패 모드
15. 운영 스펙과 한국어 도입 위험
16. Jev가 적합한 프로젝트
17. Shadow PoC 기반 도입 검증
