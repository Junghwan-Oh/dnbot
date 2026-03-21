# DEX WebSocket Integration Status And Plan

작성일: 2026-03-19

이 문서는 현재 워크스페이스의 DEX별 WebSocket / API 통합 상태를 정리하고, 앞으로 만들 통합 버전에서 어떤 기능을 가져오고 어떤 기능을 버릴지 결정하기 위한 실무 문서다.

기존 `ALTERNATE_*` 문서의 긴 서사는 여기서 주가 아니다.
여기서는 실제 구현 판단에 필요한 것만 남긴다.

## 1. 범위와 즉시 결정

- `Paradex`는 완전 폐기한다.
- `Backpack`도 신규 구현 대상에서는 제외한다.
- 다만 `Backpack`은 과거 구현이 꽤 잘 되어 있었기 때문에 donor 코드 추출용으로만 유지한다.
- 앞으로 구현할 DEX 레인은 다음 4개다.
  - `GRVT`
  - `Variational`
  - `Nado`
  - `EdgeX`
- 단, `Variational`은 현재 기준 `API/WS trading` 레인이 아니라 `브라우저 / web component control` 레인으로 본다.

## 2. 이 문서의 판단 기준

이 문서에서 WebSocket 활용도를 평가할 때 기준은 아래와 같다.

- 공식 문서상 제공 기능 전체를 먼저 확인한다.
- 그 다음 현재 코드가 실제로 무엇을 구독/활용하는지 본다.
- 마지막으로 "모든 기능을 다 썼는가"가 아니라, DN bot의 핵심 목적에 맞게 필요한 기능을 제대로 썼는가를 본다.

DN bot 기준 핵심 기능은 아래 순서다.

- maker 진입을 위한 신뢰 가능한 BBO / top-of-book
- 내 주문 fill / cancel / partial fill 이벤트
- 포지션 drift / net delta 확인
- 비상시 청산 / 운영 경보

### 2.1 Bot 목적과 성공식

이 프로젝트의 1차 목적은 "Perp DEX Point Farming DN Bot" 이다.
즉, 거래량을 가능한 한 무손실에 가깝게 크게 만들고, 리베이트 / 포인트 / 스프레드 비용을 통제하는 것이 핵심이다.

핵심 성공식은 과거 계획 문서 기준으로 아래 형태다.

```text
실제 수익 = maker rebate + DEX points - spread cost - fees - funding - residual delta cost
```

여기서 가장 중요한 운영 invariant는 아래다.

- 세션 종료 시 `Position Imbalance = 0` 또는 매우 작은 허용 오차 이내
- 하드코딩 심볼 금지, `get_contract_attributes()` 같은 동적 조회 사용
- 새 주문 전에 stale order / 잔여 포지션 검증

즉 이 문서의 모든 판단은 "fill rate" 하나가 아니라 아래 3가지를 같이 본다.

- flatness
- maker/taker execution quality
- 순손익에 가까운 방향성

## 3. 전제: 현재 워크스페이스의 코드 상태

현재 워크스페이스에는 `hedge/hedge_mode_*.py` 진입점 파일은 있지만, 이 파일들이 참조하는 `exchanges.*` 패키지 구현 파일은 이 스냅샷에 보이지 않는다.

따라서 현재 워크스페이스 기준 평가는 아래 두 층으로 나뉜다.

- 현재 스냅샷의 `hedge_mode_*` 파일에서 직접 확인 가능한 WS/API 사용 패턴
- git history 상 donor commit (`b027c98`, `3c64ac3`)에서 직접 확인 가능한 함수/구조

즉, "현재 구현 상태"는 entrypoint 중심 평가이고, "추출할 donor 기능"은 history 기반 평가다.

## 4. Historical Donor 기준

### 4.1 `b027c98`에서 가져올 것

`b027c98`은 `Backpack + GRVT` 조합이지만, 여기서 중요한 건 거래소 조합 자체가 아니라 다음 안전성 기능이다.

- `ensure_clean_start()`
- `verify_hedge_completion()`
- `_check_hedge_confirmation_after_fill()`
- `ProgressiveSizingManager`
- `RebateTracker`
- pre-iteration verification gate
- Telegram / status file / CSV / funding / balance monitoring

이 커밋의 핵심은 "새 주문을 넣기 전에 실제 포지션이 맞는지 먼저 확인한다"는 운영 규율이다.

신규 통합 버전에서는 위 기능들을 `exchange-agnostic safety layer`로 재구성해서 재사용하는 것이 맞다.

또한 `2DEX_HEDGE_MODE_COMPREHENSIVE_PLAN` 기준으로 아래 교훈도 같이 가져와야 한다.

- 심볼 하드코딩 금지
- `TICKER_CONTRACT_MAP` 같은 환각성 매핑 금지
- 모든 거래소는 `get_contract_attributes()` 또는 동등한 동적 contract discovery 경로를 우선 사용
- WebSocket 제거를 단순화라고 착각하지 말 것

즉 신규 통합 버전에서는 "symbol discovery"도 safety layer 일부로 취급해야 한다.

### 4.2 `3c64ac3`에서 가져올 것

`3c64ac3`는 원래 `Paradex + GRVT` 대안 전략 엔진이지만, 여기서 `Paradex` 자체는 버린다.
대신 아래 전략 레이어 개념만 donor로 유지한다.

- `PriceMode`
- `min_spread_bps`
- `check_arbitrage_opportunity()`
- `print_trade_summary()`
- `max_position`
- `emergency_close_primary()`의 아이디어
- `force_close_all_positions()`의 아이디어
- primary / hedge 역할 분리 구조

이 커밋의 핵심은 "그냥 돈는 루프"에서 "spread가 기준 이상일 때만 들어가는 전략 엔진"으로 바뀐 점이다.

### 4.3 `2DEX` 복원 이력에서 가져올 것

`RESTORATION_HISTORY_2DEX_HEDGE_MODE.md` 와 `FILL_RATE_TEST_PHASE1_RESULTS.md` 는 `exchange-agnostic execution donor` 관점에서 중요하다.

여기서 donor로 볼 가치는 아래 패턴이다.

- active cancel-and-replace pattern
- WebSocket BBO priority + REST fallback
- 명시적 중간 상태 체크
- `get_bbo()` helper 캡슐화
- fill timeout tuning

이 문서들이 남긴 핵심 결론은 아래다.

- 단순 polling-only 구조는 fill rate와 latency를 동시에 악화시킬 수 있다
- 공식 WS가 있는 거래소에서 WS를 제거하는 건 "단순화"가 아니라 종종 `under-engineering`이다
- BBO는 가능하면 WS 우선, REST fallback 구조가 맞다

확정적으로 기록된 수치도 있다.

- fillTimeout 5s -> 10s 실험에서 fill rate는 `20% -> 45%` 로 개선
- 같은 테스트군에서 `Position Imbalance = 0` 은 유지

즉 신규 통합 버전의 maker execution layer는 아래 원칙을 따라야 한다.

- WebSocket BBO first
- stale price 감지 후 active cancel-and-replace
- explicit state machine
- timeout은 짧게 두되, retry/cancel logic은 적극적으로 둔다

## 5. 현재 구현 상태: DEX별 요약

### 5.1 GRVT

현재 워크스페이스에서 보이는 관련 파일:

- `hedge/hedge_mode_grvt.py`
- `hedge/hedge_mode_grvt_v2.py`

현재 확인된 상태:

- `hedge_mode_grvt.py`
  - GRVT order update는 WebSocket 사용
  - BBO는 REST 기반
  - 포지션은 API 조회 기반
  - 상대 거래소(Lighter) order book은 WebSocket 사용
- `hedge_mode_grvt_v2.py`
  - GRVT order update WebSocket 사용
  - GRVT order book WebSocket 추가
  - spread history / threshold / BBO logging 추가
  - 여전히 포지션 truth는 API 조회 비중이 큼

판단:

- 현재 로컬 구현 중 `GRVT`는 4개 타깃 중 가장 성숙하다.
- `v1`은 보수적이고 안정적이다.
- `v2`는 orderbook WS까지 붙어서 전략 donor 가치가 더 높다.
- 최종 통합 버전의 `GRVT` 베이스는 `hedge_mode_grvt_v2.py` 쪽이 맞다.
- 단, `b027c98`식 verification gate를 반드시 다시 붙여야 한다.

### 5.2 Nado

현재 워크스페이스에서 보이는 관련 파일:

- `hedge/hedge_mode_nado.py`

현재 확인된 상태:

- Nado 쪽은 `NadoClient.connect()`를 호출하긴 하지만, 현재 `hedge_mode_nado.py` 자체에서 Nado native WS 채널을 적극적으로 소비하는 구조는 보이지 않는다.
- BBO는 `fetch_nado_bbo_prices()`로 조회
- 포지션은 `get_nado_position()` API 조회
- 체결/취소 판단도 주로 polling loop에서 `get_order_info()`를 반복 조회하는 구조다.
- 즉, 현재 버전은 "Nado를 WS exchange로 적극 활용"한 구현이 아니라, "REST/polling 중심 구현"에 가깝다.

판단:

- `Nado`는 현재 워크스페이스 기준으로는 구현은 있지만 WS 활용도는 낮다.
- 그래도 post-only open / cancel / retry 루프는 donor로 쓸 가치가 있다.
- 신규 구현에서는 Nado를 단순 조회 대상이 아니라, 공식 WS stream을 제대로 붙이는 대상으로 승격해야 한다.
- 특히 `best_bid_offer` + `order_update` + `position_change` 조합으로 "REST-polling 중심 구현"을 끝내는 것이 목표다.

### 5.3 EdgeX

현재 워크스페이스에서 보이는 관련 파일:

- `hedge/hedge_mode_edgex.py`

현재 확인된 상태:

- edgeX 공식 SDK의 `WebSocketManager` 사용
- public / private websocket 둘 다 연결
- public depth를 사용해 local order book 유지
- private `trade-event` 중 `ORDER_UPDATE`를 사용해 fill 처리
- BBO는 websocket 우선, REST fallback
- 포지션은 API 조회

판단:

- 현재 로컬 구현 중 `EdgeX`는 native websocket 활용도가 꽤 높다.
- 특히 public depth + private order update 조합은 maker primary venue로서 괜찮은 패턴이다.
- donor 가치가 큰 부분은 다음이다.
  - SDK 기반 public/private 분리
  - websocket 우선 + REST fallback
  - depth consumer 구조
- `RESTORATION_HISTORY` 기준으로도, 이런 구조는 이후 공통 maker execution layer 설계의 좋은 출발점이다.

### 5.4 Variational

현재 워크스페이스에서 보이는 관련 코드:

- 없음

현재 판단:

- 로컬 구현은 전무하다.
- 공식 문서 기준 현재 공개 API는 read-only 성격이 강하고, public docs에서 trading API는 아직 일반 사용자 대상 공개 상태가 아니다.
- 다만 사용자 판단대로 `web component control`로는 통합 가능성이 있다.

결론:

- `Variational`은 제외가 아니라, 별도 레인으로 유지한다.
- 단, `API/WS trading integration` 문맥에서는 아직 1순위 구현 대상이 아니다.
- 통합 문서상 분류는 다음이 맞다.
  - `GRVT / Nado / EdgeX`: API/WS 통합 레인
  - `Variational`: 브라우저 제어 레인

### 5.5 Backpack

현재 결정:

- 신규 구현 대상에서는 제외
- donor-only

유지 이유:

- private order update + public depth 구성이 명확했다
- fill event를 hedge trigger로 연결하는 구조가 좋았다
- `b027c98` safety layer와 결합돼 donor 가치가 높다

## 6. 공식 문서 기준: DEX별 WS/API Capability

### 6.1 GRVT

공식 문서 기준 market data websocket stream:

- `v1.mini.s`
- `v1.mini.d`
- `v1.ticker.s`
- `v1.ticker.d`
- `v1.book.s`
- `v1.book.d`
- `v1.trade`
- candlestick 계열

공식 문서 기준 trading websocket stream:

- `v1.order`
- `v1.fill`
- `v1.position`
- `v1.deposit`
- `v1.transfer`
- `v1.withdrawal`

현재 로컬 구현이 실제로 쓰는 것:

- `hedge_mode_grvt.py`: 사실상 `order update` 중심
- `hedge_mode_grvt_v2.py`: `order update` + `order book`

평가:

- 공식 capability 대비 구현률은 높지 않다.
- 하지만 DN bot 핵심 기능 기준으론 `order`와 `book`부터 제대로 붙이는 것이 우선이므로 방향 자체는 맞다.
- 부족한 부분은 `fill`과 `position` stream을 아직 구조적으로 활용하지 않는 점이다.
- `2DEX_HEDGE_MODE_COMPREHENSIVE_PLAN` 기준으로도, order / BBO / position을 WS에서 받을 수 있는 거래소에서 polling-only로 내려가는 것은 목표 아키텍처가 아니다.

통합 예정 기능:

- 필수
  - `v1.order`
  - `v1.book.s` 또는 `v1.book.d`
- 강력 권장
  - `v1.fill`
  - `v1.position`
- 선택
  - `v1.ticker.*`
  - candlestick

통합 판단:

- `GRVT`는 4개 타깃 중 최우선 구현 대상이다.
- 현재 donor 기반으로 가장 빨리 production-like 통합 버전을 만들 수 있는 거래소다.

### 6.2 Nado

공식 문서 기준 subscriptions stream:

- `order_update`
- `trade`
- `best_bid_offer`
- `fill`
- `position_change`
- `book_depth`
- `liquidation`
- `latest_candlestick`
- `funding_payment`
- `funding_rate`

공식 문서 기준 execute 지원:

- place order
- cancel orders
- cancel product orders
- withdraw collateral

현재 로컬 구현이 실제로 쓰는 것:

- 현재 워크스페이스 코드에서 Nado native WS stream 소비 흔적은 약하다
- 실질적으로는 API/polling 중심

평가:

- 공식 capability는 충분하다.
- 구현만 아직 그 capability를 거의 활용하지 못하고 있다.
- 특히 DN bot 입장에서 `best_bid_offer`, `order_update`, `fill`, `position_change`는 바로 써야 할 채널이다.
- 따라서 `Nado`는 "지금 상태가 donor"가 아니라 "공식 capability를 아직 못 따라간 상태"로 보는 게 정확하다.

통합 예정 기능:

- 필수
  - `best_bid_offer`
  - `order_update`
  - `position_change`
- 강력 권장
  - `fill`
  - `book_depth`
- 선택
  - `funding_rate`
  - `funding_payment`
  - `latest_candlestick`

통합 판단:

- `Nado`는 "공식 문서 capability는 충분한데 현재 로컬 구현이 뒤처진 케이스"다.
- 실제 신규 구현 가치가 높다.

### 6.3 EdgeX

공식 문서 기준 private websocket:

- 구독 없이 자동 push
- trading/custom message 포함
- event 종류:
  - `Snapshot`
  - `ACCOUNT_UPDATE`
  - `DEPOSIT_UPDATE`
  - `WITHDRAW_UPDATE`
  - `TRANSFER_IN_UPDATE`
  - `TRANSFER_OUT_UPDATE`
  - `ORDER_UPDATE`
  - `FORCE_WITHDRAW_UPDATE`
  - `FORCE_TRADE_UPDATE`
  - `FUNDING_SETTLEMENT`
  - `START_LIQUIDATING`
  - `FINISH_LIQUIDATING`

공식 문서 기준 public websocket:

- `ticker.{contractId}`
- `ticker.all`
- `ticker.all.1s`
- `kline.{priceType}.{contractId}.{interval}`
- `depth.{contractId}.{depth}`
- `trades.{contractId}`
- `metadata`

현재 로컬 구현이 실제로 쓰는 것:

- private `ORDER_UPDATE`
- public `depth`
- websocket BBO 우선 + REST fallback

평가:

- 공식 capability 대비 구현률은 중간 수준
- 하지만 maker-primary bot 기준으론 중요한 채널은 이미 잘 선택했다
- 추가 가치가 있는 건 `ACCOUNT_UPDATE`와 `FUNDING_SETTLEMENT`다
- 따라서 `EdgeX`는 새 통합 버전의 public/private WS 분리 표준 샘플로 삼기 좋다.

통합 예정 기능:

- 필수
  - private `ORDER_UPDATE`
  - public `depth`
- 강력 권장
  - private `ACCOUNT_UPDATE`
  - private `FUNDING_SETTLEMENT`
- 선택
  - `ticker`
  - `trades`
  - `metadata`

통합 판단:

- `EdgeX`는 현재 코드상 WS 활용도가 가장 균형 잡혀 있다.
- 신규 통합 버전에서 `GRVT` 다음 순위 후보로 적합하다.

### 6.4 Variational

공식 문서 기준 현재 공개 상태:

- 현재 공개 API는 read-only Omni API 중심
- 공식 roadmap에는 `Enable read-only Omni API`와 `Enable API trading`이 별도 항목으로 존재한다
- 즉 public docs 기준 API trading은 아직 일반 공개 상태가 아니다

현재 문서 기준 결론:

- `Variational`은 지금 당장 `API/WS trading` 대상으로 넣기 어렵다
- 다만 사용자 판단대로 `web component control`로는 별도 통합 시도 가능

통합 예정 기능:

- API/WS 직접 통합: 보류
- 브라우저 제어 레인:
  - 시세 수집
  - 주문 버튼 / 입력창 제어
  - 체결 / 포지션 / PnL DOM 파싱
  - 비상 중지 / 포지션 flatten UI 경로 확보

통합 판단:

- `Variational`은 "불가능"이 아니라 "레인이 다르다"
- 이 문서에서는 API/WS 레인과 브라우저 제어 레인을 분리해서 다룬다

## 7. 최종 구현 우선순위

### 7.1 API/WS 레인

1. `GRVT`
2. `EdgeX`
3. `Nado`

이 순서가 맞는 이유:

- `GRVT`: 현재 donor와 로컬 구현이 가장 풍부하다
- `EdgeX`: current implementation quality가 높다
- `Nado`: official capability는 좋지만 current implementation leverage가 낮다

### 7.2 브라우저 제어 레인

1. `Variational`

이 레인은 별도 모듈로 분리하는 것이 맞다.
같은 "exchange client" 인터페이스로 억지 통합하지 말고, `browser_operator` 또는 `ui_broker` 성격의 adapter로 따로 두는 편이 낫다.

## 8. 통합 대상 기능 리스트

### 8.0 Legacy adapter baseline

`ADDING_EXCHANGES.md` 기준으로 과거 시스템은 `BaseExchangeClient` 중심의 plugin architecture 를 전제로 설계돼 있었다.

핵심 baseline 메서드는 아래다.

- `connect()`
- `disconnect()`
- `setup_order_update_handler()`
- `fetch_bbo_prices()`
- `place_open_order()`
- `place_close_order()`
- `cancel_order()`
- `get_order_info()`
- `get_active_orders()`
- `get_account_positions()`
- `get_contract_attributes()`

향후 `adapter interface spec` 문서는 이 legacy baseline을 그대로 복사하지는 않더라도, 여기서 출발해 "DN bot 전용 contract"로 더 좁혀야 한다.

### 8.1 공통 Safety Layer

`b027c98` donor에서 가져올 대상:

- stale order clean start
- hedge completion verification
- post-fill verification gate
- progressive sizing
- rebate tracker
- funding / balance / alert / status logging

이 레이어는 새 통합 버전에서 거래소별 adapter 위에 놓는 공통 계층이 되어야 한다.

### 8.2 공통 Strategy Layer

`3c64ac3` donor에서 가져올 대상:

- `PriceMode`
- `min_spread_bps`
- spread gate
- cycle summary / gross edge accounting
- `max_position`
- emergency close / force close 설계 아이디어

단, `Paradex` 의존 경로는 전부 제거한다.

### 8.3 Exchange Adapter Layer

`GRVT`

- `hedge_mode_grvt_v2.py`의 orderbook websocket handling
- 기존 `hedge_mode_grvt.py`의 보수적 execution/monitoring
- `b027c98`식 verification gate
- 동적 contract discovery 원칙
- active cancel-and-replace + WS BBO 우선 원칙

`EdgeX`

- `hedge_mode_edgex.py`의 public/private websocket wiring
- websocket 우선 + REST fallback
- dynamic contract discovery / no hardcoded symbol 원칙

`Nado`

- 현재 `hedge_mode_nado.py`의 post-only loop는 donor
- 실제 신규 구현은 공식 stream 중심으로 재작성
- 목표는 `best_bid_offer + order_update + position_change`

`Variational`

- exchange adapter가 아니라 browser automation adapter

## 9. 버릴 것

- `Paradex` 관련 모든 구현
- `Backpack` 신규 구현 계획
- `3c64ac3`에서 `Paradex`를 전제로 한 직접 구현
- WS-first라고 가정하지만 실제 stream contract가 코드에 닫혀 있지 않은 부분
- 거래소별로 각자 다른 position truth policy를 유지하는 방식

## 10. 권장 문서 후속 작업

이 문서 다음 단계는 별도 문서를 무조건 늘리는 방식이 아니라, 현재 문서 안에 필요한 contract를 하위섹션으로 붙이는 방식이 우선이다.

### 10.1 Adapter interface spec

현재 판단:

- 별도 문서로 즉시 분리하지 않는다
- 이 문서의 하위 섹션으로 유지한다
- 실제 adapter 구현을 시작할 시점에만 독립 문서 분리를 다시 검토한다

하위섹션에 추가될 내용:

- `place_post_only_order`
- `place_taker_hedge`
- `get_bbo`
- `get_position_truth`
- `connect_order_stream`
- `connect_bbo_stream`
- `connect_position_stream`
- dynamic contract discovery contract
- normalized order / fill / position event schema
- WS-first / REST-fallback rules
- stale-order cleanup / pre-trade verification hooks
- force-close / emergency-close semantics
- browser automation adapter contract for `Variational`

### 10.2 Implementation queue

- `GRVT` first
- `EdgeX` second
- `Nado` third
- `Variational` browser lane separately

### 10.3 Backpack donor retest

목적:

- 말로 된 분석 100번보다, 작은 live test 3회가 donor 가치 판단에 더 정확한 진실을 준다
- 즉 `Backpack`을 앞으로 신규 통합 대상 거래소로 살릴지 보기보다, "보존 가치 있는 execution logic"을 선별하는 것이 목적이다

테스트 기준 커밋:

- 1차 기준: `b027c98`
- 추후 비교 기준: `d4c2942` + `c309256`

우선 검증할 것:

- 거래 안정성
  - fill event 수신
  - stale order 정리
  - 세션 종료 시 `Position Imbalance`
  - hedge verification 통과 여부
- 수익성
  - spread cost
  - rebate 반영 후 near-flat 여부
  - 방향별 체결 품질

문서상 현재 확인 가능한 historical baseline:

- `b027c98`: `10/10 iterations` at Phase 1 (`$30`) 검증 기록
- `2DEX` phase test: `0.01 ETH`, `5회` / `20회` fillTimeout 실험 기록
- `2DEX` batch script: `100 iterations` 테스트 runner 존재

즉 현재 저장소 문서만 기준으로는 "`ETH 100불 x 3회`가 기존의 정식 표준 프로토콜"이라고 단정할 근거는 없고, 그건 사용자 운영 관행으로 보는 편이 정확하다.

권장 실행:

- 작은 ETH donor retest 3회는 타당하다
- 단, 문서에는 fixed ritual로 박지 말고 "small live donor retest" 로 남긴다
- 결과가 좋으면 그 다음에 `d4c2942` + `c309256` 비교 확장으로 간다

## 11. 참고 소스

### 내부 문서 / 코드

- `docs/ALTERNATE_HANDOFF_CONTEXT.md`
- `docs/ALTERNATE_BRANCH_TIMELINE_REPORT.md`
- `docs/ALTERNATE_DIFF_REPORT_B027C98_VS_3C64AC3.md`
- `hedge/hedge_mode_grvt.py`
- `hedge/hedge_mode_grvt_v2.py`
- `hedge/hedge_mode_nado.py`
- `hedge/hedge_mode_edgex.py`

### 공식 문서

- GRVT market data websocket docs: <https://api-docs.grvt.io/market_data_streams/>
- GRVT trading websocket docs: <https://api-docs.grvt.io/trading_streams/>
- Nado streams docs: <https://docs.nado.xyz/developer-resources/api/subscriptions/streams>
- Nado events docs: <https://docs.nado.xyz/developer-resources/api/subscriptions/events>
- Nado executes docs: <https://docs.nado.xyz/developer-resources/typescript-sdk/user-guide/engine-client/executes>
- edgeX websocket docs: <https://edgex-1.gitbook.io/edgeX-documentation/api/websocket-api>
- Variational public API docs: <https://docs.variational.io/technical-documentation/api>
- Variational roadmap: <https://docs.variational.io/roadmap>
