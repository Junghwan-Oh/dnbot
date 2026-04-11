# 1DEX Source Map

작성일: 2026-04-11

## 목적

이 문서는 `1DEX`에서 무엇을 계속 볼지, 무엇을 버릴지, 무엇을 다음 단계 비교 후보로 둘지를 빠르게 판정하기 위한 source map 이다.

핵심 원칙:

- code-only archaeology 금지
- `plan + code + tests + live result` 같이 본다
- 소스맵은 미시 최적화용 목록이 아니라 `keep / reject / extract / compare-next` 판정판이다

## Authority

- authoritative git ref: `origin/DN_pair_eth_sol_nado`
- main target file: `hedge/DN_pair_eth_sol_nado.py`

## Scope Count

### Commit universe

- `1DEX Nado` branch total commits: `63`
- current mode:
  - commit-by-commit 전수독해가 아니라 cluster-first
- progress:
  - commit count identified: `63 / 63`
  - commit-by-commit reviewed: `0 / 63`
  - remaining if we ever do full archaeology: `63`

### Build-stage source map

이 문서에서 count 하는 것은 `중복 호출 수`가 아니라 `고유 컴포넌트 블록`이다.

- total mapped build-stage components: `13`
- mapped: `13 / 13`
- remaining for build-stage map: `0`

세부:

- build-stage function blocks: `5`
- ws capability blocks: `6`
- rest/query truth blocks: `2`

### Build candidate universe

- total build candidate groups: `7`
- group mapping complete: `7 / 7`

그룹:

1. simultaneous pair execution core
   - status: `keep`
2. spread / timing gate family
   - status: `keep but retune`
3. POST_ONLY maker-first family
   - status: `reject`
4. IOC taker-immediate family
   - status: `reject`
5. websocket-first BBO / BookDepth family
   - status: `keep / compare-next`
6. per-leg pricing-mode family (`eth_mode` / `sol_mode`)
   - status: `dormant / spec-only`
7. continuous execution / 100-trades family
   - status: `late-phase / not now`

### Verdict progress

- entry mode verdicts total: `2`
- verdict complete: `2 / 2`
- completed:
  - `POST_ONLY` = `reject`
  - `IOC` = `reject`

## Build Group Map

### Build candidate groups: `7`

1. `simultaneous pair execution core`
   - source hints:
     - `place_simultaneous_orders()`
     - `execute_build_cycle()`
   - status:
     - `keep`

2. `spread / timing gate family`
   - source hints:
     - `_wait_for_optimal_entry()`
     - `_check_spread_profitability()`
     - `spread-filter-optimization.md`
   - status:
     - `keep but retune`

3. `POST_ONLY maker-first family`
   - source hints:
     - `use_post_only`
     - `spread-filter-optimization.md`
     - `spread-filter-usage.md`
   - status:
     - `reject`

4. `IOC taker-immediate family`
   - source hints:
     - `place_ioc_order()`
     - `use_ioc`
     - `nado.py`
   - status:
     - `reject`

5. `websocket-first BBO / BookDepth family`
   - source hints:
     - `nado-dn-pair-websocket-first.md`
     - `nado-websocket-high-performance.md`
     - `WEBSOCKET_COMPARISON_REPORT.md`
   - status:
     - `keep / compare-next`

6. `per-leg pricing-mode family`
   - source hints:
     - `eth_mode`
     - `sol_mode`
     - `bbo_minus_1 / bbo_plus_1 / bbo / aggressive / market`
   - status:
     - `dormant / spec-only`

7. `continuous execution / 100-trades family`
   - source hints:
     - `dn-bot-1M-volume-1day-lossless.md`
   - status:
     - `late-phase / not now`

### Build-stage function blocks: `5`

1. `execute_build_cycle()`
   - 역할:
     - BUILD 진입 전 safety / timing / spread gate
   - 판단:
     - keep

2. `_wait_for_optimal_entry()`
   - 역할:
     - 일정 시간 동안 entry timing 탐색
   - 판단:
     - compare-next

3. `_check_spread_profitability()`
   - 역할:
     - spread filter
   - 판단:
     - keep as concept
     - current threshold behavior must be re-tuned

4. `calculate_order_size_with_slippage()`
   - 역할:
     - BookDepth 기반 sizing / slippage gate
   - 판단:
     - keep

5. `place_simultaneous_orders()`
   - 역할:
     - ETH / SOL 동시 발사
   - 판단:
     - keep
     - 가장 중요한 build-stage core

### WS capability blocks: `6`

1. `WEBSOCKET_AVAILABLE`
   - 의미:
     - WS capability feature gate
   - 판단:
     - keep

2. `NadoWebSocketClient`
   - 의미:
     - Nado WS client
   - 판단:
     - keep

3. `BBOHandler`
   - 의미:
     - BBO stream handler
   - 판단:
     - keep

4. `BookDepthHandler`
   - 의미:
     - BookDepth stream handler
   - 판단:
     - keep

5. `estimate_slippage()`
   - 의미:
     - BookDepth-based slippage estimate
   - 판단:
     - keep

6. `check_exit_capacity()`
   - 의미:
     - depth 기반 exit capacity check
   - 판단:
     - keep

### REST/query truth blocks: `2`

1. `fetch_bbo_prices(...)`
   - 현재 쓰임:
     - build price truth
     - spread filter
     - optimal entry timing
   - 판단:
     - keep
     - query-heavy 의존은 계속 점검

2. `get_account_positions()`
   - 현재 쓰임:
     - pre-build flatness
     - unwind verification
   - 판단:
     - keep
     - critical checkpoint authority

## Build-stage entry logic list

현재 build-stage에서 실제로 쓰는 entry logic family:

1. pre-build position clean check
2. optimal entry timing wait
3. spread profitability filter
4. BookDepth/slippage sizing
5. simultaneous order placement
6. mode split:
   - `POST_ONLY`
   - `IOC`
7. partial / one-leg fill handling via emergency unwind

build-stage entry logic total: `7`

## Current Live Verdict

### `POST_ONLY`

- live run result:
  - `min-spread-bps 6` -> skip
  - `min-spread-bps 0` -> BUILD 시도 but 양 다리 no fill
- current verdict:
  - `reject` as live baseline entry mode

이유:

- 체결이 안 나면 baseline 으로 볼 수 없다
- 현재는 `entry alpha` 가 아니라 `no fill` source 다

### `IOC`

- live run result:
  - real run 1: `ETH fill / SOL no fill`
  - real run 2: `ETH fill / SOL no fill`
- current verdict:
  - `reject as live baseline entry mode`

이유:

- one-leg fill 이 재현적으로 반복되면 baseline 으로 볼 수 없다
- `POST_ONLY`보다 진입성은 낫지만, 현재 상태로는 `unsafe`

## Unwind Group Map

### Core unwind blocks

1. `handle_emergency_unwind()`
   - one-leg / partial fill 시 강제 복구 분기
   - keep

2. `emergency_unwind_eth()`
   - ETH leg emergency close
   - keep

3. `emergency_unwind_sol()`
   - SOL leg emergency close
   - keep

4. `execute_unwind_cycle()`
   - verified flat까지 확인하는 핵심 close contract
   - keep
   - current top-priority logic

## Cross-cutting Truth / Risk Map

### Current top truths

- BBO truth:
  - `fetch_bbo_prices(...)`
- fill truth:
  - `OrderResult` + partial/unwind interpretation
- position truth:
  - `get_account_positions()`
- close truth:
  - verified flat

### Current top risk families

1. `UNWIND failure`
2. `one-leg / partial fill`
3. `BUILD with residual position`
4. `POST_ONLY no-fill`

## Current execution status

1. canonical docs established
2. stage2 dropped as legacy
3. stage3 canonical smoke green
4. stage4 cycle verification green
5. mainnet read-only sanity succeeded
6. real 1-cycle run succeeded to BUILD attempt
7. `POST_ONLY` live verdict fixed
8. `IOC` live verdict fixed
9. current blocker:
   - no viable current baseline entry family

## Next

1. keep `UNWIND group`
2. keep `BUILD one-leg handling`
3. reject `POST_ONLY` as baseline entry
4. reject `IOC` as current live baseline entry
5. compare-next:
   - websocket-first BBO / BookDepth family
   - per-leg pricing-mode family
6. do not assume current `POST_ONLY/IOC` baseline viability
