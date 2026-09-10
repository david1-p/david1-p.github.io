---
title: "스프링과 FastAPI 비교 기록"
date: 2026-08-29 09:00:00 +0900
categories: [Backend, Framework]
tags: [spring, fastapi, java, python]
---

## 왜 비교하게 됐나

파이썬으로 FastAPI를 먼저 쓰다가 스프링을 공부하기 시작했다.
처음엔 완전히 다른 세계 같았는데, 개념 이름만 다르고 하는 일이 같은 게 꽤 많았다.
헷갈리는 걸 정리해두려고 쓴다.

## 개념 대응표

이름만 다르고 역할이 같은 것들을 먼저 정리했다.

| 하는 일 | 스프링 | FastAPI |
| --- | --- | --- |
| 라우팅 | `@GetMapping("/items/{id}")` | `@app.get("/items/{id}")` |
| 요청 본문 검증 | DTO + Bean Validation (`@Valid`) | Pydantic 모델 |
| 의존성 주입 | 생성자 주입 + 스프링 컨테이너 | `Depends(...)` |
| 요청 전후 가로채기 | 필터 / 인터셉터 | 미들웨어 |
| 전역 예외 처리 | `@RestControllerAdvice` | `@app.exception_handler(...)` |
| 설정 바인딩 | `application.yml` + `@ConfigurationProperties` | `.env` + `BaseSettings` |
| API 문서 | springdoc-openapi (의존성 추가) | 기본 내장 (`/docs`) |
| 테스트 | `@SpringBootTest`, `MockMvc` | `TestClient` |

## 같은 코드, 두 언어

같은 엔드포인트를 양쪽으로 써보면 구조가 거의 겹친다.

```java
@RestController
@RequestMapping("/items")
public class ItemController {

    private final ItemService itemService;

    public ItemController(ItemService itemService) {  // 생성자 주입
        this.itemService = itemService;
    }

    @PostMapping
    public ItemResponse create(@Valid @RequestBody ItemRequest request) {
        return itemService.create(request);
    }
}
```

```python
router = APIRouter(prefix="/items")

@router.post("")
def create(
    request: ItemRequest,                       # 검증
    service: ItemService = Depends(get_service) # 주입
) -> ItemResponse:
    return service.create(request)
```

컨트롤러는 검증된 입력을 받아 서비스에 넘기고, 서비스가 실제 일을 한다.
이 계층 구조는 양쪽 다 똑같다. 그래서 스프링을 배울 때 "이건 FastAPI의 뭐다"로 대응시키면 빠르게 붙는다.

## 여기서부터 갈린다

### 1. 검증 시점 — 컴파일 vs 런타임

자바는 타입이 컴파일 시점에 검사된다. 타입이 안 맞으면 애초에 빌드가 안 된다.
파이썬의 타입 힌트는 실행 시점에 강제되지 않는다. FastAPI가 동작하는 이유는 Pydantic이 **런타임에** 그 힌트를 읽어서 직접 검사하기 때문이다.

```python
def create(request: ItemRequest) -> ItemResponse:
```

여기서 `ItemRequest`는 주석이 아니라 실행되는 코드다. FastAPI가 이 힌트를 보고 요청 본문을 파싱하고 검증한다.
반대로 서비스 계층 내부에서 잘못된 타입을 주고받는 건 아무도 안 막아준다. 자바는 막아준다.

### 2. 동시성 모델 — 스레드 vs 이벤트 루프

가장 크게 다른 지점이다.

**스프링 MVC**는 요청 하나에 스레드 하나를 붙인다. DB를 기다리는 동안 그 스레드는 블로킹된 채로 논다.
스레드 풀이 꽉 차면 나머지 요청은 대기한다. 대신 코드는 위에서 아래로 읽히고, 디버깅도 스택 트레이스만 보면 된다.

**FastAPI**는 이벤트 루프 위에서 돈다. `async def`로 쓴 엔드포인트는 `await` 지점에서 제어권을 넘겨서, 스레드 하나가 여러 요청을 왔다 갔다 처리한다.
I/O 대기가 많은 서비스에서 적은 자원으로 많은 동시 요청을 받을 수 있다.

여기 함정이 있다. FastAPI에서 `async def` 안에 블로킹 코드를 쓰면 이벤트 루프 전체가 멈춘다.

```python
@app.get("/items")
async def bad():
    return requests.get(...)   # 블로킹 — 루프 전체가 멈춘다

@app.get("/items")
async def good():
    async with httpx.AsyncClient() as c:
        return await c.get(...)

@app.get("/items")
def also_fine():
    return requests.get(...)   # 그냥 def면 스레드풀로 보내준다
```

`def`로 선언하면 FastAPI가 알아서 별도 스레드풀에서 돌려준다. 어설프게 `async`를 붙이는 것보다 그냥 `def`가 나을 때가 많다.

양쪽 다 반대편 모델도 가지고 있긴 하다. 스프링에는 이벤트 루프 기반의 WebFlux가 있고,
자바 21 이후로는 가상 스레드를 켜서 블로킹 코드를 그대로 두고도 스레드 비용을 줄일 수 있다.

### 3. DI — 컨테이너가 관리하느냐, 그냥 함수 호출이냐

스프링의 DI는 컨테이너가 있다. 애플리케이션이 뜰 때 빈을 만들어 싱글톤으로 들고 있다가 주입한다.
그래서 시작 시점에 의존성 그래프가 다 검증되고, 순환 참조 같은 건 뜰 때 터진다.

FastAPI의 `Depends`는 컨테이너가 없다. **요청이 올 때마다 그 함수를 호출**하는 것뿐이다.

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

`yield` 앞은 요청 시작할 때, 뒤는 응답 끝난 뒤 실행된다. 요청 스코프 자원을 다루기엔 이 쪽이 훨씬 직관적이다.
대신 싱글톤이 필요하면 모듈 전역 변수나 `lru_cache`로 직접 만들어야 한다. 프레임워크가 관리해주지 않는다.

### 4. 트랜잭션 — 애노테이션 하나 vs 명시적 세션

스프링은 `@Transactional` 한 줄이면 프록시가 알아서 트랜잭션을 열고 닫는다.
편한 만큼 감춰져 있어서, 같은 클래스 내부 호출에는 프록시가 안 먹는다는 것 같은 함정이 있다.

FastAPI + SQLAlchemy는 세션 수명을 위처럼 의존성으로 직접 관리한다. 감춰진 게 없어서 예측 가능하지만, 커밋/롤백을 내가 챙겨야 한다.

### 5. 프레임워크냐 조립이냐

스프링 부트는 스타터만 넣으면 보안, 트랜잭션, 캐시, 배치까지 정해진 방식으로 붙는다.
인증이 필요하면 Spring Security가 있고, 그 규약을 따르면 된다.

FastAPI는 Starlette(ASGI) + Pydantic 위에 얹은 얇은 층에 가깝다.
인증, 권한, 백그라운드 작업 같은 건 라이브러리를 고르고 직접 조립해야 한다. 자유롭지만 결정할 게 많다.

## 정리

| | 스프링 | FastAPI |
| --- | --- | --- |
| 성격 | 정해진 방식이 있는 프레임워크 | 조립해서 쓰는 얇은 층 |
| 타입 검사 | 컴파일 시점 | 런타임 (Pydantic) |
| 기본 동시성 | 요청당 스레드 (블로킹) | 이벤트 루프 (async) |
| DI 범위 | 컨테이너 관리 싱글톤 | 요청마다 함수 호출 |
| 트랜잭션 | `@Transactional` (암묵적) | 세션 직접 관리 (명시적) |
| 잘 맞는 곳 | 규모가 크고 오래 갈 서비스, 복잡한 도메인 | I/O 중심 API, 빠른 개발 |

큰 차이는 언어가 아니라 **감춰주는 정도**에 있는 것 같다.
스프링은 많이 감춰주는 대신 규약을 알아야 하고, FastAPI는 적게 감추는 대신 내가 조립해야 한다.

<!-- WebFlux / 가상 스레드 직접 써보고 나서 3번 항목 보강하기 -->
<!-- Spring Security vs FastAPI 인증 비교는 따로 한 편 -->
