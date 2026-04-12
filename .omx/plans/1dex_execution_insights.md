# 1DEX Execution Insights

작성일: 2026-04-12

## 목적

이 문서는 `1DEX`에서 나온 order / execution / WS / REST 관련 실전 인사이트를 보존하는 파일이다.

이 파일이 소유하는 것:

- order mode 실전 해석
- WS / REST execution 의미
- live run에서 얻은 구체적 결론

이 파일은 phase / gate / source-map count 를 소유하지 않는다.

## 왜 이 이름인가

`orders`만 붙이면 너무 좁다.
현재 문제는 주문 타입만이 아니라:

- `POST_ONLY`
- `IOC`
- `BBO`
- `BookDepth`
- `WS-first`
- `REST fallback`

이 같이 얽혀 있기 때문이다.

그래서 `execution insights`가 가장 맞다.

## Grouping Rule

이 파일은 앞으로 `BUILD group insights` 와 `UNWIND group insights` 로 나눠서 읽는다.

이유:

- 지금 우리가 부딪히는 실전 문제는 전부 "order mode의 취향 차이"가 아니라
- BUILD에서 무엇을 gate로 보고
- 무엇을 authority로 보고
- 무엇을 blocker로 승격할지의 문제이기 때문이다

또한 아래 두 가지를 분리해서 읽는다.

1. `risk priority`
   - 어느 failure가 더 치명적인가
2. `batch execution order`
   - 이번 배치에서 무엇을 먼저 잘라야 하는가

현재 `1DEX`에서는:

- risk priority 는 여전히 `UNWIND` 쪽이 더 높다
- 하지만 batch execution order 는 `BUILD-first` 다

즉, 지금 배치에서 `BUILD`를 먼저 자세히 적는 것은
`UNWIND`가 덜 중요해서가 아니라
현재 비교 후보 3개가 전부 BUILD 층에 있기 때문이다.

## BUILD Group Insights

### 1. 왜 지금 BUILD group을 먼저 자르나

지금 시점의 핵심 사실은 이미 두 가지다.

1. `POST_ONLY` 는 baseline 탈락
2. `IOC` 도 baseline 탈락

그런데 이 두 탈락을 자세히 읽어보면,
문제의 중심은 "어떤 주문 타입이 더 좋으냐"가 아니다.

실제 중심은 아래다.

- BUILD gate 가 무엇을 허용하고 무엇을 막는지
- BUILD 직전 market-data authority 를 무엇으로 둘지
- sizing authority 가 실제로 depth 신호를 먹고 있는지

즉, 지금 batch 의 질문은:

- `POST_ONLY` 대신 `IOC`를 더 다듬을까

가 아니라

- 왜 BUILD가 no-fill 또는 one-leg를 스스로 만들어내는가
- 그걸 gate/data/sizing 층에서 어떻게 다시 자를 것인가

다.

### 2. `spread / timing gate family`

이 family 는 지금 BUILD 단계에서 가장 헷갈리기 쉬운 층이다.

겉으로 보면:

- spread threshold 를 보고
- timing 을 보다가
- 좋을 때 들어가는 로직처럼 보인다

하지만 donor 코드 읽기로 보면 실제 성격은 더 애매하다.

#### donor code reading

1. `_check_spread_profitability()` 는 이름과 달리 코드 주석상 profitability filter 보다 sanity check에 가깝다.
2. `_wait_for_optimal_entry()` 는 threshold 미달이어도 timeout 후 진입한다.

이 두 가지를 합치면 현재 gate semantics 는 이렇게 읽힌다.

- "정말 수익성 있는 entry만 통과시킨다"
  보다는
- "조금 기다려보다가, 끝까지 안 오면 결국 entry를 release한다"

에 가깝다.

#### 왜 이게 중요한가

이 차이는 단순한 용어 문제가 아니다.

- profitability gate 라고 믿으면
  - threshold 수치를 미세하게 조정하는 쪽으로 가게 된다
- entry-release gate 라고 보면
  - timeout 정책
  - advisory vs hard block
  - WS/depth 결합 판단
  같은 더 구조적인 문제로 넘어가게 된다

즉, 지금 `spread/timing`은 "몇 bps가 맞나"를 테스트하기 전에
"이 gate가 무슨 역할을 하는가"부터 다시 정의해야 한다.

#### 현재 결론

- spread/timing family 는 `reject`가 아니라 `keep but retune`
- 다만 retune의 초점은 숫자 미세조정이 아니라 gate semantics 재정의다

### 3. `POST_ONLY`가 왜 탈락했는가

`POST_ONLY`는 개념상 나쁜 주문 방식이 아니다.

좋게 보면:

- 비용 구조가 예쁘고
- maker 성격이 강하고
- 장기 economics 관점에서 매력 있어 보인다

하지만 지금 `1DEX`의 phase는 economics phase가 아니다.
지금은 BUILD viability phase다.

#### 실전 결과

- `min-spread-bps 6` -> skip
- `min-spread-bps 0` -> BUILD 시도 but 양 다리 no fill

#### 해석

- 두 다리 모두 no fill이면 BUILD 자체가 시작되지 못했다
- 즉 이 로직은 "좋은 진입"이 아니라 "진입 부재"를 만들었다
- 그래서 현재 문맥에서 `POST_ONLY`는
  - 비용 최적화 후보가 아니라
  - no-fill source 다

#### 현재 결론

- `POST_ONLY = reject as current live baseline entry mode`

### 4. `IOC`가 왜 탈락했는가

`IOC`는 `POST_ONLY` 다음 후보처럼 보이기 쉽다.

그 이유는 단순하다.

- `POST_ONLY`보다 체결 가능성이 높아 보이기 때문이다

하지만 실전은 더 까다롭다.

#### 실전 결과

- real run 1: `ETH fill / SOL no fill`
- real run 2: `ETH fill / SOL no fill`
- 두 번 다 one-leg fill 재현
- 사후 포지션은 ETH/SOL 모두 `0`

#### 핵심 해석

1. entry timing이 좋아서 들어간 게 아니라 timeout 후 진입했다
2. `BookDepth` 데이터가 실전에서 실제로 안 붙어 있어 slippage/sizing gate가 비활성에 가깝다
3. IOC는 유동성 비대칭이 있으면 한쪽만 체결되고 다른 쪽은 취소될 수 있다
4. 현재 build gate는 paired fill viability를 충분히 선별하지 못한다

즉:

- `IOC`는 진입성은 높였지만
- BUILD가 건강한 paired fill 을 만들었다고 보기는 어렵다
- 오히려 BUILD 단계가 스스로 one-leg risk를 만들어냈다

#### 현재 결론

- `IOC = reject as current live baseline entry mode`

### 5. `websocket-first BBO / BookDepth family`

이 family 를 지금 `compare-next` 1순위로 두는 이유는
"WS가 멋져 보여서"가 아니다.

핵심은 authority 문제다.

#### donor code reading

1. `estimate_slippage()` 는 BookDepth handler 가 없으면 `999999`를 반환한다
2. 그런데 sizing 경로에서는 `No BookDepth data`여도 target quantity 로 계속 진행한다

이게 뜻하는 바는 명확하다.

- BookDepth는 코드 표면상 중요하다고 선언되어 있지만
- 실제 BUILD release / sizing authority 에서는 아직 hard blocker가 아니다

#### 왜 이게 중요한가

지금 `1DEX`는 order mode를 더 바꾸기 전에
아래를 먼저 정해야 한다.

- BBO를 정말 WS primary로 올릴 것인가
- BookDepth가 없으면 BUILD를 멈출 것인가
- BookDepth가 있으면 sizing/viability에 실제로 반영되는가

즉 이 family 는
"실시간 데이터 써보자"가 아니라
"현재 BUILD authority를 다시 세우자"는 family 다.

#### 현재 결론

- `websocket-first BBO / BookDepth family = keep / compare-next`
- 지금 배치에서 가장 먼저 비교할 build family다

### 5.5. WS 꺼진 런 복기

이건 반드시 남겨야 하는 복기다.

#### 왜 WS가 꺼졌나

- 원인은 간단했다.
- `sortedcontainers`가 없어
  - Nado WebSocket 모듈 import가 실패했고
  - 런타임이 자동으로 `REST fallback`으로 내려갔다

실제 표시:

- `Import failed: No module named 'sortedcontainers'`
- `WEBSOCKET_AVAILABLE set to False`

#### WS가 꺼진 상태에서 무슨 일이 있었나

- `ENTRY`는 `optimal_spread`가 아니라 `timeout`으로 열렸다
- `BookDepth`가 없었는데도
  - `No BookDepth data ... using target quantity`
  로 계속 진입을 시도했다
- 초기 양다리 주문은 둘 다 timeout/no-fill
- 그 뒤 retry로 들어갔고
- retry에서는 ETH만 fill, SOL은 계속 실패했다

즉 이 런은 이렇게 읽는다.

- WS가 꺼져서
- BookDepth가 실제 gate 역할을 못 했고
- BUILD가 건강한 paired fill 대신
- timeout/no-fill/retry/one-leg 쪽으로 흘렀다

#### 이 런이 남긴 결론

- 이 런은 credential 문제로 실패한 게 아니다
- `WS 미활성 + BookDepth 부재 + timeout release`가 겹쳐서
  BUILD가 약한 상태로 열린 것이다
- 따라서 `WS / BookDepth`를 살리기 전 로직 비교는 의미가 약하다

### 5.6. WS 켠 런 복기

#### 무엇이 달라졌나

- `sortedcontainers` 설치 후
  - `WEBSOCKET_AVAILABLE = True`
  - ETH/SOL 둘 다 `CONNECTED`
  - `book_depth` subscribe도 실제 메시지를 받았다

#### WS 켠 뒤 실전에서 무슨 일이 있었나

- 여전히 `ENTRY`는 `timeout`으로 열렸다
- 하지만 이번엔 `BookDepth`가 살아 있어서
  - ETH slippage `0.0 bps`
  - SOL slippage `0.0 bps`
  로 계산됐다
- BUILD는 양다리 모두 first try fill
- UNWIND도 양다리 모두 fill

즉:

- WS를 살리니
  - BUILD 자체는 이전보다 훨씬 정상적으로 돌아갔다
- 그래서 `No BookDepth data`는
  - 로직 문제이기도 하지만
  - 실제로는 WS 미활성의 영향도 컸던 것으로 읽힌다

### 5.7. 그런데 왜 봇은 멈췄나

이 부분은 어렵게 말할 필요가 없다.

실제 상황은 이거다.

- UNWIND 주문까지는 다 나갔다
- REST로 다시 조회하면 ETH/SOL 포지션은 둘 다 `0`이었다
- 그런데 웹소켓 포지션 값은 SOL이 `-1.2`처럼 남아 있다고 봤다
- 봇은 "포지션이 아직 남아 있을 수 있다"고 보고 멈췄다

즉 지금 문제를 쉬운 말로 쓰면:

- "실제 포지션은 0인데
  봇 안의 웹소켓 값은 아직 SOL이 남아 있다고 본다"

이다.

#### 중요한 운영 해석

- 이 상황에서 먼저 할 일은 root-cause 분석이 아니다
- 먼저 실제 포지션이 남았는지 다시 조회하는 것이다
- 이번에는 read-only 재확인 결과
  - ETH position `0`
  - SOL position `0`
  였다
- 그래서 이번 건은 "실제 잔여 포지션"이 아니라
  "웹소켓 포지션 값 해석/동기화 문제" 쪽에 더 가깝다

#### 이 런이 남긴 결론

- WS를 살리면 BUILD/UNWIND fill은 훨씬 좋아진다
- 하지만 이제 다음 blocker는
  - `웹소켓 포지션 값과 REST 포지션 값이 안 맞는 문제`
  - `PNL 상태값 누락`
  쪽으로 올라왔다

### 6. `per-leg pricing-mode family`

이 family 는 이름만 보면 그럴듯해서
바로 다음 핵심 해답처럼 보일 수 있다.

하지만 지금은 아니다.

이유는 간단하다.

- 아직 gate semantics 도 덜 정리됐다
- BookDepth availability도 blocker인지 아닌지 확정 안 됐다
- market-data authority 도 WS 쪽으로 충분히 올라가지 않았다

이 상태에서 leg별 pricing mode를 만지면
문제를 더 미세한 층으로 내려서 착시를 만들 수 있다.

그래서 현재 읽기는:

- 이 family는 버릴 후보는 아니다
- 하지만 지금 바로 first compare-next 후보도 아니다
- 먼저 `spread/timing`
- 그 다음 `websocket-first BBO / BookDepth`
- 그 다음에야 들어갈 후보다

## UNWIND Group Insights

### 1. 왜 지금 상세 분량은 BUILD보다 적은가

`UNWIND`는 여전히 더 위험한 영역이다.
이건 안 바뀐다.

하지만 이번 batch 에서 `UNWIND`보다 `BUILD`를 먼저 길게 적는 이유는:

- 현재 비교 후보 3개가 전부 BUILD 층에 있고
- 지금 탈락한 것도 `POST_ONLY`, `IOC`라는 BUILD entry mode 들이며
- donor 코드에서 새로 확인된 blocker도 `spread/timing`, `timeout`, `No BookDepth data` 쪽이기 때문이다

즉:

- `UNWIND`가 덜 중요해서가 아니다
- 지금 batch 의 실질적인 선택지가 BUILD 쪽이라서 그렇다

### 2. 그래도 BUILD 해석에서 UNWIND가 왜 계속 따라다니는가

이유는 하나다.

- one-leg fill 을 BUILD 성공으로 보면 안 되기 때문이다

즉, 지금 BUILD 문서에서 `UNWIND`가 계속 언급되는 것은
우리가 곧바로 UNWIND batch 로 간다는 뜻만은 아니다.

오히려 뜻은 이렇다.

- BUILD는 자기 결과를 스스로 성공이라고 선언할 수 없다
- one-leg / residual / hedge fail 이 나오면
  - 그 순간부터 UNWIND/flatness contract 가 판정을 가져간다

그래서 현재 `UNWIND`의 역할은:

- 이번 batch 의 상세 비교 주인공은 아니지만
- BUILD 해석의 실패 판정 기준을 계속 제공하는 것

이다.

## Cross-group takeaway

지금 `1DEX`에서 핵심은 새로운 entry mode를 더 파는 것이 아니다.

지금 핵심은:

- `spread/timing`이 실제로 무슨 gate인지 다시 정의하고
- `BookDepth unavailable`를 warning으로 둘지 blocker로 올릴지 정하고
- WS/BBO/BookDepth가 진짜 authority인지 확인하고
- 그 다음에야 per-leg pricing으로 내려가는 것이다

짧게 말하면:

- `POST_ONLY`는 no-fill로 탈락
- `IOC`는 one-leg로 탈락
- 다음은 order mode가 아니라 BUILD semantics를 다시 자르는 단계다
- 그리고 WS를 켠 뒤에는
  - `No BookDepth data`보다
  - `웹소켓 포지션 값과 REST 포지션 값이 안 맞는 문제`
  가 더 큰 실전 blocker로 올라왔다

## Runbook Link

- 첫 실행형 런북:
  - `.omx/plans/1dex_spread_timing_execution_plan.md`
