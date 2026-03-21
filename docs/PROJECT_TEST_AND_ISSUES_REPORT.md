# Project Test And Issues Report

작성일: 2026-03-19

이 문서는 특정 커밋명이나 전략명을 직접 언급하지 않고, 현재 Backpack-GRVT 계열 프로젝트에 대해 실제 테스트 상황, WebSocket 활용 상태, 주요 문제점, 해석을 정리한 보고서다.

## 1. 프로젝트 목적

이 프로젝트의 목적은 Delta-Neutral 구조로 거래량을 만들면서:

- maker rebate
- volume-based points
- spread cost
- fee / funding
- residual delta risk

를 함께 관리하는 것이다.

실무적으로는 아래 두 가지가 동시에 성립해야 한다.

- 세션 종료 시 포지션이 flat 또는 허용 오차 이내여야 한다
- 수수료/리베이트까지 포함한 손익이 최소 near-flat 이상이어야 한다

## 2. 공식 WebSocket Capability 대비 현재 구독 상태

### 2.1 Backpack

공식적으로 제공되는 주요 WebSocket 계열:

- `account.orderUpdate`
- `account.positionUpdate`
- `account.rfqUpdate`
- `bookTicker.<symbol>`
- `depth.<symbol>` 및 주기 variants
- `kline.<interval>.<symbol>`
- `liquidation`
- `markPrice.<symbol>`
- `ticker.<symbol>`
- `openInterest.<symbol>`
- `trade.<symbol>`

현재 테스트한 구현에서 실제 구독 중인 스트림:

- `account.orderUpdate.<symbol>`

현재 구현에서 사용하지 않는 주요 WebSocket 기능:

- `account.positionUpdate`
- `bookTicker`
- `depth`
- `trade`
- `markPrice`
- `ticker`

해석:

- 현재 Backpack는 사실상 "내 주문 체결 감시" 용도로만 WS를 쓰고 있다
- BBO는 REST 조회 또는 상위 로직 재시도에 의존한다
- 즉 maker primary venue로는 WS 활용이 충분하지 않다

### 2.2 GRVT

공식적으로 제공되는 주요 market data WebSocket 계열:

- `v1.mini.s`
- `v1.mini.d`
- `v1.ticker.s`
- `v1.ticker.d`
- `v1.book.s`
- `v1.book.d`
- `v1.trade`
- candlestick 계열

공식적으로 제공되는 주요 trading WebSocket 계열:

- `v1.order`
- `v1.fill`
- `v1.position`
- `v1.deposit`
- `v1.transfer`
- `v1.withdrawal`

현재 테스트한 구현에서 실제 구독 중인 스트림:

- `v1.order`

현재 구현에서 사용하지 않는 주요 WebSocket 기능:

- `v1.fill`
- `v1.position`
- `v1.book.s`
- `v1.book.d`
- `v1.ticker.*`
- `v1.trade`

해석:

- 현재 GRVT도 사실상 "주문 상태/체결 결과 감시" 용도로만 WS를 쓴다
- BBO는 REST `fetch_order_book` 경유
- 포지션은 REST 또는 order-event 기반 로컬 추적으로 관리한다
- 즉 hedge venue로는 동작하지만, WS capability를 충분히 활용하는 상태는 아니다

### 2.3 현재 구조의 의미

현재 구조는 아래처럼 요약된다.

- WS는 order update only
- BBO는 REST
- 포지션 truth는 REST 또는 local fill tracking

이 구조의 장점:

- 구현이 단순하다
- 주문 체결 감시는 실시간성이 있다

이 구조의 단점:

- BBO stale risk
- cancel/reprice churn 증가
- fill stream / position stream 미사용
- 포지션 truth가 늦게 맞춰질 수 있음

## 3. 소규모 라이브 테스트 개요

실제 라이브 테스트는 모두 작은 사이즈로 진행했다.

공통 조건:

- 종목: `ETH`
- 명목 규모: 약 `$100`
- 수량: `0.049 ETH`
- 반복: `3 iterations`

이번 보고서에는 서로 다른 두 종류의 구현을 테스트한 결과를 같이 정리한다.

- 테스트 A: 보수적 검증형 구현
- 테스트 B: POST_ONLY hedge 시도 + MARKET fallback 구현

## 4. 테스트 A 결과

### 4.1 결과 요약

- `2/3` 사이클 완료
- `1/3` 실패
- 실패 시 실제 잔여 포지션 발생
- 테스트 종료 후 수동 청산 필요

### 4.2 관측된 문제

- primary leg는 체결되었지만 hedge leg가 reject/timeout 되는 케이스 발생
- bot은 stop 하긴 하지만 잔여 포지션을 자동으로 flat하게 만들지 못했다
- 실제 잔여 상태:
  - Backpack long 잔존
  - GRVT flat

### 4.3 수익성

완료된 2개 사이클 기준:

- Gross PnL: 음수
- rebate 포함 known-net: 여전히 음수
- 수동 청산분까지 포함하면 최종 실손익은 더 악화

### 4.4 해석

이 구현은 아래 donor 가치가 있었다.

- clean start
- pre-iteration hedge verification
- stale order 감지
- 운영 로깅

하지만 치명적 약점도 분명했다.

- hedge reject 시 emergency flatten이 불완전함
- 따라서 안정성 프레임은 좋지만 실전 완결성은 부족

## 5. 테스트 B 결과

### 5.1 결과 요약

- `3 iterations`
- 내부적으로 `6 cycles` 완료
- 최종 포지션:
  - Backpack `0`
  - GRVT `0`
  - Net Delta `0`

즉 flat 종료는 성공했다.

### 5.2 손익

테스트 결과:

- Total Volume: `$623.89`
- Total Gross PnL: `-$0.1250`
- Average Edge: `-2.00 bps`

즉 gross 기준으로는 음수다.

### 5.3 소요 시간

iteration 단위:

- 1회차: `35.71s`
- 2회차: `10.30s`
- 3회차: `15.91s`
- 평균: `20.64s / iteration`

사이클 관점:

- `6 cycles / 3 iterations`
- 약 `10.3s / cycle`

### 5.4 POST_ONLY hedge의 실제 동작

이 구현의 핵심 아이디어는:

- hedge를 먼저 `POST_ONLY`로 시도
- 3초 내 미체결 시 `MARKET`로 fallback

이번 라이브 테스트에서 확인된 사실:

- POST_ONLY 시도는 실제로 발생했다
- 그러나 실체결은 전부 `MARKET fallback`으로 끝났다
- export된 metrics에서도 hedge entry/exit order type은 모두 `MARKET`
- fee_saved 플래그도 모두 `False`

즉:

- 로직은 작동했다
- 하지만 기대했던 maker fee 절감은 실현되지 않았다

### 5.5 해석

이 구현은 테스트 A보다 flat 종료 품질이 확실히 좋았다.

장점:

- 최종 flat 종료 성공
- POST_ONLY 실패 시 fallback이 있어 청산 완결성이 높음
- cycle time도 비교적 짧음

단점:

- 결국 MARKET fallback에 많이 의존
- fee saving이 실제로는 거의 발생하지 않음
- 그래서 기대했던 plus PnL 재현에는 실패

## 6. 두 테스트를 함께 보면 보이는 것

### 6.1 안정성

보수적 검증형 구현:

- 사전 검증과 정지 로직은 좋음
- 하지만 hedge 실패 시 residual position 처리 완결성이 약함

POST_ONLY + fallback 구현:

- flat 종료는 더 잘함
- execution completion 측면에선 더 실전적

즉:

- "청산 완결성"은 POST_ONLY + fallback 구현이 더 낫다
- "안전 프레임과 운영 통제"는 보수적 검증형 구현이 더 낫다

### 6.2 수익성

공통점:

- 작은 실테스트에서는 둘 다 명확한 plus를 재현하지 못했다

차이:

- 보수적 검증형 구현은 reject/timeout 리스크가 더 크게 드러남
- POST_ONLY + fallback 구현은 손실은 작지만 여전히 음수

즉:

- 안정성 donor와 수익성 donor를 분리해서 봐야 한다
- 한 구현이 둘 다 만족하진 못하고 있다

## 7. 주요 문제점

### 7.1 WebSocket 활용 부족

- Backpack도 order update만 구독
- GRVT도 order update만 구독
- BBO와 position stream을 거의 활용하지 않음

문제:

- REST polling/lookup 비중이 높아짐
- cancel/reprice가 잦아짐
- stale pricing 가능성 증가

### 7.2 POST_ONLY의 실효성 부족

아이디어는 좋지만 실제로는:

- 3초 안에 fill이 잘 안 남
- 대부분 MARKET fallback
- 따라서 maker fee 절감 효과가 실전에서 잘 안 나타남

### 7.3 잔여 포지션 처리

일부 구현은 hedge 실패 시:

- stop 은 하지만
- 강제 flatten 은 미완

이 문제는 가장 위험하다.

### 7.4 cancel 흐름의 거칠음

라이브 로그에서 반복적으로 보인 패턴:

- cancel 요청 직후 "order not found"
- 이미 거래소 쪽 상태가 바뀌었는데 local state machine이 뒤따르는 구조

즉 order lifecycle 동기화가 매끄럽지 않다.

### 7.5 수익성 가정과 live 체결의 괴리

문서/아이디어 상으로는:

- POST_ONLY hedge
- spread filter
- order size 최적화

가 plus를 만들 수 있다고 보였지만,

실제 small live test에선:

- POST_ONLY가 잘 안 붙고
- market fallback이 늘어나고
- gross edge가 음수 또는 near-flat에 머문다

## 8. 현재 시점의 실무 판단

### 8.1 보존 가치가 있는 것

- clean start
- pre-trade verification
- dynamic contract discovery
- active cancel-replace 아이디어
- POST_ONLY attempt + fallback 구조
- trade metrics export
- final flatness 확인 로직

### 8.2 그대로 가져가면 안 되는 것

- order update만 WS로 쓰는 구조
- BBO를 REST에 과하게 의존하는 구조
- hedge reject 후 잔여 포지션을 남기는 stop-only 동작
- "POST_ONLY를 시도했으니 fee saving이 날 것"이라는 낙관적 가정

### 8.3 다음 단계

우선순위는 아래가 맞다.

1. BBO / position / fill stream 활용 확대 여부 결정
2. hedge reject 시 무조건 flat으로 가는 emergency path 강화
3. POST_ONLY price placement / timeout tuning 재검증
4. small live test 반복으로 fee-saving 재현 가능성 확인
5. plus가 문서상 존재하더라도 live reproduction 없으면 donor 로직만 보존

## 9. 결론

현재 Backpack-GRVT 프로젝트는:

- flat 종료를 안정적으로 만드는 방향으로는 분명 진전이 있었고
- 일부 구현은 이전보다 청산 완결성이 좋아졌다

하지만:

- WebSocket capability 활용은 여전히 낮고
- 수익성은 라이브 소규모 테스트에서 재현되지 않았으며
- 핵심 알파로 기대한 POST_ONLY hedge도 실제 fill로 잘 이어지지 않았다

즉 현재 단계의 가장 현실적인 해석은 아래다.

- 안정성/운영 로직은 donor 가치가 있다
- 수익성 로직은 live 재현 검증이 더 필요하다
- 이 프로젝트는 "완성된 winner" 보다는 "좋은 부품이 섞인 후보군"에 가깝다
