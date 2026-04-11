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

## Preserved Analysis

### `POST_ONLY`

의미:

- maker 성격의 resting order를 노리는 방식
- 수수료는 예뻐 보이지만, 실제 체결이 안 나면 아무 의미가 없다

실전 결과:

- `min-spread-bps 6` -> skip
- `min-spread-bps 0` -> BUILD 시도 but 양 다리 no fill

판정:

- `POST_ONLY = reject as current live baseline entry mode`

이유:

- 두 다리 모두 no fill 이면 BUILD 자체가 시작되지 못한다
- 즉 이 로직은 `entry viability`를 만들지 못한다
- baseline 은 최소한 진입을 만들어야 하는데, 현재 POST_ONLY 는 그 문턱조차 못 넘는다
- 따라서 `POST_ONLY` 는 더 이상 baseline 후보가 아니다

### `IOC`

의미:

- Immediate-Or-Cancel
- 즉시 체결 가능한 만큼만 체결하고, 나머지는 바로 취소한다

실전 결과:

- real run 1:
  - `ETH fill / SOL no fill`
- real run 2:
  - `ETH fill / SOL no fill`
- 두 번 다 one-leg fill 재현
- 사후 포지션은 ETH/SOL 모두 `0`

판정:

- `IOC = reject as current live baseline entry mode`

이유 분석:

1. entry timing이 좋아서 들어간 게 아니라 timeout 후 진입했다
2. `BookDepth` 데이터가 실전에서 실제로 안 붙어 있어 slippage/sizing gate가 비활성에 가깝다
3. IOC는 특성상 유동성 비대칭이 있으면 한쪽만 체결되고 나머지는 취소될 수 있다
4. 현재 build gate는 paired fill viability를 충분히 선별하지 못한다
5. 그래서 문제는 `UNWIND`만이 아니라, BUILD 단계가 스스로 one-leg risk를 만들어낸다는 점이다

즉:

- `POST_ONLY`보다 진입성은 낫다
- 하지만 현재 상태로는 `unsafe`

### WS / REST

현재 읽기:

- read-only mainnet sanity 자체는 성공했다
- 하지만 실전 entry path에선 `REST-heavy` 경향이 여전히 강하다
- `WS connected` 와 `operationally integrated` 는 다르다

핵심 해석:

- WS를 쓴다고 해도 build paired fill 문제를 자동으로 해결해주지 않는다
- `BookDepth`와 paired viability gate가 실제로 살아 있어야 의미가 있다

## Current takeaway

- `POST_ONLY`는 예쁘지만 체결 안 나와서 탈락
- `IOC`는 진입은 되지만 BUILD 단계에서 one-leg fill을 반복적으로 만들어 탈락
- 지금 핵심은 새로운 entry mode를 더 파는 게 아니라
  - `paired fill viability`
  - `UNWIND / close / flatness`
  - `WS / BookDepth 실전 연결성`
  이 세 가지를 다시 짜는 것이다
