# 1DEX Glossary

작성일: 2026-04-11

## 목적

이 문서는 `1DEX` 문맥에서 반복 등장하는 용어를 빠르게 읽고 맞추기 위한 glossary 다.

포함 원칙:

- 단순 사전 정의가 아니라
- 우리 대화 / 현재 1DEX 문맥에서 어떤 의미로 쓰는지까지 같이 적는다

## POST_ONLY

- 의미:
  - 즉시 체결되지 않도록 resting order 로 올리려는 주문 방식
  - 보통 maker 쪽을 노리는 limit-style 주문으로 이해하면 된다
- 우리 문맥:
  - entry fee 를 줄이기 위해 BUILD 에서 썼다
  - 하지만 real run 에서는 `min-spread-bps 0`이어도 no fill 이 나왔다
  - 그래서 현재 live baseline entry 로는 부적합하다고 본다

## IOC

- 의미:
  - Immediate-Or-Cancel
  - 즉시 체결 가능한 만큼만 체결하고, 나머지는 취소하는 주문 방식
- 우리 문맥:
  - POST_ONLY 보다 체결 가능성은 높고, 수수료는 더 불리할 수 있다
  - 현재 다음 entry comparison 후보로 본다

## fallback

- 의미:
  - 본체 로직이 원하는 경로로 안 될 때 쓰는 대체 경로
- 우리 문맥:
  - POST_ONLY 실패 후 IOC 로 넘어가는 것
  - forced liquidation / forced flatten 같이 긴급 safety path 로 넘어가는 것

## BBO

- 의미:
  - Best Bid / Best Offer
  - 최우선 매수호가 / 매도호가
- 우리 문맥:
  - 진입 가격 판단의 최전선 데이터
  - BBO truth 가 틀리면 BUILD 가격 판단이 틀어진다

## spread filter

- 의미:
  - 현재 spread 가 기준 이상일 때만 진입하게 막는 필터
- 우리 문맥:
  - 원래는 PnL 보호용
  - 지금은 진입 자체를 막는지 검증 대상
  - `6bps` 에서는 skip, `0bps` 에서는 BUILD 시도까지 갔음

## flatness

- 의미:
  - 포지션이 0 또는 허용 오차 이내인 상태
- 우리 문맥:
  - `UNWIND 성공`의 최종 기준
  - stop 여부보다 flatness 가 더 중요하다

## truth

- 의미:
  - 봇이 실제 상태라고 믿는 최종 기준
- 우리 문맥:
  - 예쁘거나 빨라 보이는 데이터가 아니라
  - 의사결정에 권한을 주는 상태값

## truth source

- 의미:
  - 봇이 무엇을 실제 상태로 믿는지, 그 출처
- 우리 문맥:
  - BBO truth
  - fill truth
  - position truth
  - close truth

## BBO truth

- 의미:
  - 현재 얼마에 사고팔 수 있는지를 봇이 믿는 기준
- 우리 문맥:
  - ETH / SOL 양 다리의 `fetch_bbo_prices(...)` 경로

## fill truth

- 의미:
  - 주문이 실제로 체결됐는지, 얼마나 체결됐는지에 대한 기준
- 우리 문맥:
  - `accepted` 와 `actually filled` 를 구분해야 한다
  - 지금 pair baseline 에서는 POST_ONLY `OPEN` 처리 규칙이 핵심 약점이다

## position truth

- 의미:
  - 현재 실제 보유 포지션이 얼마인지에 대한 기준
- 우리 문맥:
  - `get_account_positions()` 기반 recheck 가 authority 다

## close truth

- 의미:
  - 포지션이 진짜 닫혔는지에 대한 기준
- 우리 문맥:
  - 주문 성공이 아니라
  - ETH / SOL 양 다리 모두 verified flat 인지로 본다

## plane

- 의미:
  - 책임 층 / 역할 층
- 우리 문맥:
  - Market Data Plane
  - Execution Plane
  - Position Truth Plane
  - Risk / Flatness Plane

## source map

- 의미:
  - 어떤 truth 를 어디서 읽고, 어디서 검증하는지 적는 지도
- 우리 문맥:
  - 미시 최적화용 목록이 아니다
  - 현재 로직을 비교해서 `keep / reject / extract` 를 빠르게 결정하는 도구다

## data gap snapshot

- 의미:
  - 현재 구현에서 데이터 경로가 어디서 비거나 약한지 요약한 스냅샷
- 우리 문맥:
  - 연결은 됐는데 decision path 에 안 들어간 경우
  - query path 의존이 과한 경우
  - close truth 가 약한 경우를 잡는 데 쓴다

## connected

- 의미:
  - 채널이 붙어 있다는 뜻
- 우리 문맥:
  - WS connected 로그가 있다고 해서 충분하지 않다

## operationally integrated

- 의미:
  - 연결된 데이터를 실제 decision / truth / risk path 에 쓰고 있다는 뜻
- 우리 문맥:
  - 그냥 연결됐다는 것보다 훨씬 강한 상태다

## forced liquidation / forced flatten

- 의미:
  - 본체가 망가졌을 때 노출을 빨리 제거하는 긴급 청산 경로
- 우리 문맥:
  - 메인 방법이 아니라 2차 fallback
  - 수수료 손실 가능성이 크므로 여기에 의존하면 baseline 탈락
