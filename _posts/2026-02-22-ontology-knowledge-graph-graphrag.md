---
title: "온톨로지, 지식 그래프, GraphRAG — 데이터 저장 중심에서 의미 모델 중심으로"
date: 2026-02-22 09:00:00 +0900
categories: [Data, Graph]
tags: [ontology, knowledge-graph, graphrag, rag, rdf, owl, sparql, llm]
---

> 2026-02-22에 발표한 자료를 글로 정리했다. [발표 자료 PDF 내려받기](/assets/pdf/ontology-graphrag.pdf)
{: .prompt-info }

## 1. 문제 정의

- 대규모 데이터 환경에서 semantic mismatch(의미상 불일치)가 발생한다
- LLM 응답의 사실성과 검증 가능성에 한계가 있다
- 벡터 유사도 중심 검색은 제약 조건을 처리하지 못한다

결과적으로 **"의미 모델 + 규칙 + 검증"을 결합한 구조**가 필요하다.

## 2. Ontology의 정의

> Gruber(1993): An explicit specification of a conceptualization

- **Conceptualization** — 도메인에 대한 추상적 개념화
- **Explicit specification** — 개념/관계/제약의 명시적 기술
- **Formal semantics** — 기계 추론 가능한 형식 논리 기반 표현

### FOL(First-Order Logic)

First-Order Logic은 복잡한 지식을 논리적으로 표현하는 언어다. 데이터에 의미와 맥락을 부여하고, 규칙 기반 자동 추론을 가능하게 하고, 복잡한 질의와 검색을 지원하고, 데이터 통합과 일관성을 확보한다.

흐름으로 보면 **데이터 → 지식 → 추론** 순으로 올라간다. JSON 데이터가 온톨로지를 거쳐 논리 표현(FOL)이 되는 셈이다.

### 온톨로지와 FOL은 뭐가 다른가

둘은 상호보완적이지만 역할이 명확히 갈린다. 온톨로지는 **무엇이 존재하는지**를 정의하고, FOL은 **무엇이 사실인지**를 표현한다.

| | 온톨로지 | FOL |
| --- | --- | --- |
| 역할 | 의미 구조 정의 (Concepts, Relations) | 의미 있는 사실 표현 (Facts, Rules) |
| 예시 | `Class: Station`, `Property: hasPopulation` | `Population(Gangnam, 92000)`<br>`∀x (HasCongestionLevel(x, 붐빔) → VeryCrowded(x))` |
| 목적 | 데이터의 개념화 (Conceptualization) | 추론 가능한 명제화 (Formalization) |
| 구현 언어 | RDF, OWL, RDFS | Prolog, 술어 논리식 |
| 질문 관점 | 무엇이 존재하는가? (What exists?) | 무엇이 사실인가? (What is true?) |

### Ontology 구성요소

| 요소 | 설명 | 가족 관계도 예시 |
| --- | --- | --- |
| 클래스 (Class) | 개념이나 사물의 종류를 정의하는 추상적 집합 | '사람', '남자', '여자' |
| 인스턴스 (Instance) | 클래스에 속하는 실제적인 개체 | '홍길동', '성춘향' |
| 속성 (Property) | 인스턴스가 가지는 특징이나 값 | '홍길동'의 '나이: 25세' |
| 관계 (Relation) | 인스턴스나 클래스 간의 연결 | '홍길동'은 '성춘향'과 `isFriendOf` |
| 공리 (Axiom) | 반드시 참이 되는 규칙이나 제약 조건 | '모든 남자는 사람이다' |

### TBox와 ABox — 지식의 두 기둥

지식 베이스는 두 층으로 나뉜다.

- **TBox (Terminological Box)** — 용어/개념/관계를 정의하는 스키마 계층. 지식의 설계도, 세상의 문법책이다. "강남역은 장소이다"라는 문장이 아니라 '장소'라는 **개념 자체**를 정의한다. 재사용성·일관성·논리 검증이 여기서 나온다.
- **ABox (Assertional Box)** — TBox가 정의한 구조에 따라 기록된 구체적인 사실들. 설계도로 지은 실제 건물이다. "강남역은 장소이다"처럼 개별 개체에 대한 사실을 저장한다. 확장성·유연성·실시간 반영이 여기서 나온다.

TBox는 **무엇을 말할 수 있는가의 틀**이고, ABox는 **그 문법에 맞춰 쓰인 구체적인 문장들**이다.

TBox 쪽 설계 요소는 이렇게 나뉜다.

- **Classes** — `Place`(장소), `Forecast`(예측), `CongestionLevel`(혼잡도), `SubClassOf`(계층 관계)
- **Properties** — `hasForecast`·`hasCongestionLevel`(Object Property), `avgPopulation`(Data Property)
- **Constraints** — Domain/Range(속성의 적용 범위), Disjointness(배타적 관계), Cardinality(수량적 제약)

설계할 때 챙긴 것들: 명명 규칙과 URI 일관성 유지, Domain/Range 명확히 설정하고 필요시 Disjoint 관계 정의, OWL의 개방 세계 가정(Open World Assumption) 고려, 필요한 경우 폐쇄 세계 제약을 명시적으로 모델링.

ABox 쪽은 인스턴스 선언과 사실 진술로 채운다.

```text
# Instances
gangnam_station rdf:type Place
forecast_1500   rdf:type Forecast
level_crowded   rdf:type CongestionLevel

# Property Assertions
gangnam_station hasForecast        forecast_1500
forecast_1500   hasCongestionLevel level_crowded
forecast_1500   avgPopulation      92000
```

ABox 작성 시 챙길 것: 인스턴스에 명확한 라벨과 주석 부여(`rdfs:label`, `rdfs:comment`), 동일 실체 정합성 관리(`owl:sameAs` 적절한 사용), 시간 관련 메타데이터 포함(생성일·수정일·버전), 데이터 품질 검증(SHACL, OWL 제약 활용).

### Protégé로 실제 입력하기

Object Property Assertion은 두 인스턴스 간의 관계를 정의해 지식그래프의 연결을 만든다.

```turtle
:forecast_POI014_20250926T160000 :forPlace :place_POI014 .
```

Domain/Range를 헷갈리지 않는 게 중요하다. `forPlace`의 Domain은 `Forecast`, Range는 `Place`다. 즉 Forecast가 Place를 참조하는 방향이고, Place에서 Forecast를 가리키면 안 된다.

Protégé 경로는 `Individuals 탭 > Individuals by class > Forecast 클래스 선택 > 인스턴스 클릭 > Object property assertions 섹션 > + 버튼 > forPlace 선택 > place_POI014 선택`이다.

Data Property Assertion을 넣으면 이런 TTL이 나온다.

```turtle
:forecast_POI014_20250926T160000
  :forecastCongestionLevel "붐빔" ;
  :forecastPopulationMax 96000 ;
  :forecastPopulationMin 94000 ;
  :forecastTime "2025-09-26T16:00:00"^^xsd:dateTime ;
  :forPlace :place_POI014 .
```

타입 지정은 이렇게 한다 — 문자열은 기본 타입이라 큰따옴표만 쓰면 되고, 정수는 드롭다운에서 `xsd:integer`, 날짜/시간은 ISO 형식(`YYYY-MM-DDThh:mm:ss`)으로 입력한 뒤 `xsd:dateTime`, 불리언은 `"true"`/`"false"` 입력 후 `xsd:boolean`을 고른다. 한글 텍스트에는 Language Tag `@ko`를 붙인다.

## 3. 시맨틱 웹 표준 스택

시맨틱 웹의 비전은 데이터에 의미(semantic)를 부여해 기계가 정보를 이해하고 추론할 수 있게 하는 것이다. 기계가 데이터의 맥락을 이해하고, 서로 다른 데이터셋 간 통합이 쉬워지고, 질의와 검색의 정확도가 오르고, 자동화된 추론과 의사결정을 지원한다.

![시맨틱 웹 기술 스택](/assets/img/posts/ontology-graphrag/13.png)
_Unicode·XML 위에 RDF, RDFS, Rules, OWL, Logic, Trust가 쌓인다_

- **URI/IRI** — 자원 식별
- **RDF** — 트리플 구조 기반 사실 표현
- **RDFS** — 기본 스키마/분류 체계
- **OWL** — 고급 논리 제약 및 추론
- **SPARQL** — 그래프용 SQL
- **SHACL** — 제약 기반 데이터 검증

### RDF — 지식의 원자

모든 지식을 subject–predicate–object로 표현한다. "강남역 – 위치한다 → 서울특별시" 같은 트리플 하나가 사실 하나다. 이 원자적 사실들이 모여 거대한 지식 네트워크를 형성한다.

- 그래프 데이터 모델 — 노드와 엣지로 구성된 네트워크
- URI 기반 식별 — 모든 자원과 관계를 고유 URI로 명확히 식별해 전역 참조
- 스키마-프리 확장성 — 새로운 정보를 언제든 추가 가능
- 웹 스케일 연결 — 수십억 개의 사실을 서로 연결

### OWL — 지식의 문법

OWL은 분류 체계를 지원하고(클래스 계층 구조와 상속 관계), 제약 조건을 표현하고(도메인/범위·속성·논리적 제약), 추론 규칙을 지원한다(명시적 지식에서 암묵적 지식을 도출).

```text
Student ⊑ Person
Professor ⊑ Person

Father ≡ Male ⊓ (hasChild some Person)
```

그 효과는 세 가지다 — **일관성 검사**(데이터가 정의된 제약과 충돌하지 않는지 자동 확인), **자동 분류**(인스턴스를 속성에 기반해 적절한 클래스로 자동 배치), **숨겨진 관계 발견**(직접 명시되지 않은 관계를 추론 엔진이 도출).

### SPARQL — 지식을 향한 질문

SQL이 테이블을 대상으로 한다면, SPARQL은 **그래프를 대상으로 특정 패턴과 일치하는 부분을 찾는 방식**으로 동작한다.

```sparql
PREFIX ex: <http://example.org/>
SELECT ?place WHERE {
  ?place ex:locatedIn ex:Seoul ;
         ex:congestion "붐빔" .
}
```

세미콜론으로 같은 주어에 대한 여러 조건을 간결하게 나열하고, 그래프 패턴으로 여러 단계의 관계를 추적하고, `OPTIONAL`·`FILTER`로 조건부 매칭을 한다. 페더레이션(FEDERATED) 쿼리로 여러 엔드포인트의 데이터를 통합 검색할 수 있고, OWL 온톨로지와 연계하면 추론 질의도 가능하다.

### 정리 — 온톨로지와 시맨틱 웹

- **RDF = 뼈대** — 지식 체계의 기본 구조를 이루는 트리플 형태의 기초 프레임워크
- **OWL = 두뇌** — 논리적 규칙과 제약으로 추론과 일관성 검사를 가능하게 하는 지능
- **SPARQL = 신경계** — 정보를 질의하고 응답을 전달하는 통신 체계
- **URI/Linked Data = 혈관** — 자원을 식별하고 시스템 전체에 연결하는 순환 체계

온톨로지가 도메인 지식의 의미론적 설계도라면, 시맨틱 웹은 이 설계도를 웹 규모로 실행하는 기술 인프라다.

## 4. 온톨로지와 RDBMS의 구조적 차이

| | RDBMS | 온톨로지 |
| --- | --- | --- |
| 기본 단위 | 테이블의 행(row) — 미리 정의된 스키마에 맞춰 저장된 레코드 | 트리플(Triple) — 주어-서술어-목적어 형태의 지식 원자 |
| 관계 표현 | 외래 키(FK) — 다른 테이블을 가리키는 숫자 ID로 간접 연결 | 명시적 관계(URI) — 의미를 가진 이름으로 직접 연결 |
| 스키마 변화 | 고정적 — 구조 변경이 어렵고 큰 비용 (`ALTER TABLE`) | 유연함 — 레고 블록처럼 새 개념과 관계를 쉽게 추가 |
| 의미 수준 | 데이터 수준 — 값 중심의 저장과 구조적 표현 | 개념과 논리 수준 — 의미와 맥락 표현 |
| 질의 언어 | SQL — 구조화된 테이블 대상 | SPARQL — 그래프 패턴 기반 |
| 초점 | 저장과 관리 | 의미 연결과 추론 |

RDBMS는 효율적인 저장과 관리에 최적화돼 있지만, 데이터의 의미와 맥락을 표현하고 확장하는 데는 한계가 있다 — 데이터 사일로, 추론 능력 부재, 표준화 어려움, 스키마 변경 비용.

온톨로지는 그 네 가지를 각각 보완한다.

1. **데이터 통합** — 서로 다른 스키마도 공통 개념(URI)으로 의미적 통합
2. **추론** — 논리 규칙을 통해 명시적으로 저장되지 않은 새로운 지식을 자동 도출
3. **지식 재사용** — 표준 온톨로지를 공유 어휘로 활용해 일관성과 연속성 확보
4. **확장성** — 테이블 구조 변경 없이 새로운 개념과 관계를 유연하게 추가

### 데이터 사례 비교

RDBMS는 개별 테이블 중심이라 `place_id`, `area_id` 같은 인공적 키로만 연결된다. 데이터베이스는 그 ID가 무엇을 의미하는지 알지 못하고, 의미는 애플리케이션 로직에 의존한다.

```sql
CREATE TABLE population (
  place_id INT,
  timestamp DATETIME,
  count INT,
  FOREIGN KEY (place_id) REFERENCES place(id)
);

CREATE TABLE commercial_area (
  area_id INT,
  place_id INT,
  business_count INT,
  FOREIGN KEY (place_id) REFERENCES place(id)
);
```

온톨로지는 모든 정보가 `:Place`, `:Road` 같은 의미 있는 개념을 중심으로 연결된다. 데이터 간의 의미적 연결이 데이터 자체에 내장돼 도시 지식망(Urban Knowledge Graph)을 형성한다.

```turtle
:강남역 a :Place .
:강남역 :locatedIn :서울특별시 .
:강남역 :hasPopulation "45000"^^xsd:integer .
:강남역 :hasCommercialArea :강남상권 .
:강남상권 :hasBusinessCount "2300"^^xsd:integer .
```

## 5. 지식 그래프

![지식 그래프 시각화](/assets/img/posts/ontology-graphrag/23.png)
_기업-산업 분류를 BELONGS_TO 관계로 연결한 지식 그래프_

- Entity–Relation–Attribute 구조의 그래프형 지식베이스
- 식별 가능한 개체와 의미론적 관계를 연결
- 온톨로지 기반일 때 논리 제약과 추론 가능성을 확보

## 6. RAG

![RAG 흐름도](/assets/img/posts/ontology-graphrag/24.png)
_질의 → 유사도 검색 → 연관 항목 → 프롬프트 증강 → LLM 생성 → 응답_

- **Retrieval(검색)** — 텍스트 유사도 분석
- **Augmented(증강)** — 프롬프트 증강
- **Generation(생성)** — LLM 생성

### 벡터 유사도만으로 부족한 지점

"화재 원인은 무엇인가?"라는 질의를 생각해보자.

![벡터 공간에서의 RAG 검색](/assets/img/posts/ontology-graphrag/25.png)
_질의 벡터와 가까운 문서만 걸린다_

질의는 "화재"라는 단어로 화재 상황 보고서와 가까워진다. 하지만 정작 원인을 설명하는 제품 설명서("완전 충전 후 전원을 분리하지 않으면 화재의 위험이 있습니다")는 벡터 공간에서 멀리 떨어져 있어 걸리지 않는다.

### GraphRAG

![벡터 공간에 그래프 연결을 얹은 GraphRAG](/assets/img/posts/ontology-graphrag/26.png)
_문서 간 관계 링크를 따라가면 유사도만으로는 닿지 않던 근거에 도달한다_

문서들이 그래프로 연결돼 있으면, 유사도로 찾은 노드에서 관계를 따라 이동해 실제 원인을 담은 문서까지 도달한다.

### RAG와 GraphRAG 비교

- **Standard RAG** — 유사 문서 검색 + 생성
- **GraphRAG** — 그래프 기반 검색(개체/관계/제약) + 생성

차이는 세 가지다.

- Similarity retrieval → **Constraint-aware retrieval**
- 비정형 근거 → **구조화된 근거**
- **설명 가능성 강화**

## 8. Ontology AG (온톨로지 증강 생성)

![DIKW 피라미드와 Ontology AG](/assets/img/posts/ontology-graphrag/28.png)
_Data → Information → Knowledge → Wisdom_

- **Ontology** — 정보의 연계, 추론(inferencing)
- **Augmented** — 설명·가설·정책·신뢰도, 추리(reasoning)
- **Generation** — 자연어 출력, LLM

흐름은 이렇게 된다.

```text
질문 → 온톨로지 기반 해석/질의(SPARQL) → 지식 그래프 조회 → LLM 답변 생성
```

RAG에 온톨로지와 지식그래프를 얹은 형태다.

## 9. 도구 및 프레임워크

![온톨로지 파이프라인](/assets/img/posts/ontology-graphrag/29.png)
_Protégé → Jena/RDFLib → Fuseki/GraphDB → SPARQL → LLM_

- **Ontology editor** — Protégé
- **RDF framework** — Apache Jena, RDFLib
- **Triple Store** — Fuseki, GraphDB, Blazegraph
- **Property Graph** — Neo4j, Kuzu, Apache AGE
