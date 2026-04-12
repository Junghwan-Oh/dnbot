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

## Historical Best-Version Screening

이번 source map에서는 commit screening의 첫 질문을 바꾼다.

- 잘못된 질문:
  - "최근 websocket 커밋부터 잡자"
- 수정된 질문:
  - "`md / csv / log`에 explicit success record가 남은 best version이 무엇이냐"

screening rule:

1. commit message보다 `md / csv / log` evidence를 우선한다
2. explicit success record가 있는 commit을 먼저 고른다
3. 같은 수준이면 그 다음에 최신 쪽을 우선한다

### screened anchor commits: `8`

1. `7e11a6f`
   - 역할:
     - subaccount / close order bug fix
   - 읽기:
     - necessary infra fix
     - explicit cycle success record는 약하다
   - verdict:
     - `important prerequisite`
     - `not first anchor`

2. `c4f7c75`
   - 역할:
     - emergency unwind / tick-size rounding fix
   - evidence:
     - `.omc/plans/emergency-unwind-fix.md`
     - `.omc/plans/tick-size-rounding-fix.md`
   - 읽기:
     - design / repair evidence는 있다
     - explicit successful cycle record는 약하다
   - verdict:
     - `repair milestone`
     - `not first anchor`

3. `0c93e82`
   - 역할:
     - position accumulation 방지용 emergency unwind 강화
   - 읽기:
     - close-side safety에는 중요
     - explicit successful cycle record는 약하다
   - verdict:
     - `important safety fix`
     - `not first anchor`

4. `3b483fe`
   - supporting documentation:
     - `23231c6` `docs/tp-precision-fix-summary.md`
   - explicit success record:
     - ETH TP order placed
     - SOL TP order placed
     - static TP hit
     - SOL WS precision correction
   - documented remaining issue:
     - `WS lag at unwind verification`
   - verdict:
     - `provisional best 1DEX version anchor`

5. `9c3be2b`
   - supporting documentation:
     - `docs/bug-fixes-summary.md`
   - evidence:
     - core logic tests `9 / 9` passing
     - TP mocks `3 / 3` still broken
   - 읽기:
     - test evidence는 좋다
     - live best-version evidence는 `3b483fe`보다 약하다
   - verdict:
     - `supporting guardrail commit`
     - `not first anchor`

6. `4ff3ae2`
   - supporting documentation:
     - `docs/websocket-position-tracking-fix.md`
   - evidence:
     - `SOL ~$400 accumulation` incident-response
     - `23 tests passing`
   - 읽기:
     - 중요한 post-mortem / repair batch다
     - 하지만 baseline anchor라기보다 incident-response history다
     - 게다가 이 family는 `position_change authority` 가정을 강하게 두는데, 2026-04-12 mainnet read에서는 그 가정을 바로 truth로 쓰기 어렵다
   - verdict:
     - `extract lessons`
     - `not first anchor`

7. `43b36fb`
   - 역할:
     - WS fill / retry handling 재작업
   - 읽기:
     - useful experimental lane
     - explicit md/csv success record가 없다
   - verdict:
     - `keep as experiment`
     - `not best-version anchor`

8. `93bdfce`
   - 역할:
     - WS startup / build retry handoff hardening
   - evidence:
     - unit / bootstrap tests 강화
     - 2026-04-12 live에서는 `no-fill clean skip` 와 `one-leg -> unwind -> retry re-entry` 문제가 남았다
   - verdict:
     - `keep as current experimental head`
     - `not best-version anchor`

### Current anchor verdict

- provisional best `1DEX` version:
  - `3b483fe`
- why:
  - explicit success record가 있는 가장 강한 commit family
  - remaining problem도 `BUILD failure general`이 아니라 `unwind verification lag`로 더 좁게 적혀 있다
- do not anchor on:
  - `4ff3ae2`
  - `43b36fb`
  - `93bdfce`
  because those are better read as `incident-response / experimental lanes`, not `best known stable version`

## Scope Count

### Commit universe

- `1DEX Nado` branch total commits: `65`
- current mode:
  - commit-by-commit 전수독해가 아니라 cluster-first
- progress:
  - commit count identified: `65 / 65`
  - commit-by-commit reviewed: `8 / 65`
  - remaining if we ever do full archaeology: `57`

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

## Build controller assumption

현재 build family 비교는 `alternate` cycle controller 위에서 읽는다.

- `BUY_FIRST`
  - ETH long / SOL short BUILD
- `SELL_FIRST`
  - ETH short / SOL long BUILD

즉 지금 compare-next 대상은 controller 자체를 뒤엎는 후보가 아니라,

- 같은 alternating controller 위에서
- build gate 를 어떻게 다듬을지
- 어떤 market-data / pricing family 를 올릴지

를 비교하는 것이다.

## Current Live Verdict

### `POST_ONLY`

- live run result:
  - `min-spread-bps 6` -> skip
  - `min-spread-bps 0` -> BUILD 시도 but 양 다리 no fill
- current verdict:
  - `reject` as live baseline entry mode

이유:

- live run 2회 기준 BUILD 자체가 시작되지 못했다
- `min-spread-bps 6` 에서는 gate skip
- `min-spread-bps 0` 으로 풀어도 양 다리 모두 no fill 이었다
- 즉 지금 이 family 는 수수료 최적화 후보가 아니라 BUILD 미개시 source 다
- baseline 문턱을 못 넘으므로 `reject`

### `IOC`

- live run result:
  - real run 1: `ETH fill / SOL no fill`
  - real run 2: `ETH fill / SOL no fill`
- current verdict:
  - `reject as live baseline entry mode`

이유:

- run 1, run 2 모두 `ETH fill / SOL no fill` 이 반복됐다
- 즉 현재 BUILD path 가 paired fill 을 만들지 못하고 one-leg risk 를 직접 생성한다
- `POST_ONLY`보다 진입성은 낫지만, baseline 은 BUILD 단계에서 one-leg 를 만들면 안 된다
- unwind 가 나중에 복구하더라도 BUILD baseline 성공으로 볼 수 없으므로 `reject`

## Remaining build families to review

### why these 3 families are related but not one bundle

현재 남은 3개 family는 서로 연결돼 있지만, 서로 다른 층이다.

1. `spread / timing gate family`
   - BUILD release semantics 층
2. `websocket-first BBO / BookDepth family`
   - market-data / sizing authority 층
3. `per-leg pricing-mode family`
   - leg별 가격 공격성 조정 층

즉:

- 이 셋은 같이 언급될 수는 있어도
- 같은 실험 단위로 한 번에 뭉개서 테스트하면 안 된다
- `gate -> authority -> pricing` 순서로 쪼개야 한다
- 이유는 원인 분리를 위해서다
- gate semantics도 안 정해진 상태에서 pricing-mode를 건드리면, 실패 원인이 어느 층인지 다시 흐려진다

### 1. `spread / timing gate family`

- 역할:
  - BUILD 에 들어갈지 말지를 정하는 front gate
  - `_wait_for_optimal_entry()`
  - `_check_spread_profitability()`
  - `--min-spread-bps`
- 현재 판정:
  - `keep but retune`
- 이유:
  - gate 라는 개념 자체는 필요하다
  - 하지만 현재 20bps -> 6bps -> 0bps 식의 고정 threshold 조정만으로는 paired fill 문제를 해결하지 못했다
  - 즉 이 family 는 유지하되, `profitability hard filter` 하나로 BUILD viability 를 대표시키면 안 된다
- 현재 해석:
  - 이 family 는 수익성 필터이면서 동시에 진입 차단기다
  - 따라서 지금은 `economics gate` 보다 `entry viability gate` 성격으로 다시 써야 한다
- donor code reading:
  - `_check_spread_profitability()` 는 코드 주석상 profitability filter 가 아니라 sanity check 다
  - `_wait_for_optimal_entry()` 는 threshold 미달이어도 timeout 후 진입을 허용한다
  - 따라서 현재 문제는 "spread threshold가 너무 높다"보다 "gate semantics가 섞여 있다" 쪽에 더 가깝다
- 다음 판정 질문:
  - hard block 으로 둘지
  - soft advisory/logging 으로 내릴지
  - WS / depth 신호와 결합한 composite gate 로 올릴지
- 현재 batch acceptance:
  - `spread` 를 profitability hard gate로 문서화하지 않기
  - timeout 진입은 `optimal_spread`와 분리해서 읽기
  - BUILD entry release 조건을 spread 단독에서 WS/BBO/BookDepth 결합 판단으로 옮길지 결정
- 세부 테스트 슬라이스:
  1. `semantic split`
     - 목적:
       - profitability gate 와 sanity gate 를 분리
     - 왜 하는가:
       - 지금 가장 큰 혼선이 수치가 아니라 용도 해석이기 때문이다
  2. `timeout release`
     - 목적:
       - timeout 진입을 "좋은 진입 발견"과 분리
     - 왜 하는가:
       - timeout 경로를 그대로 두면 gate 평가가 계속 왜곡된다
  3. `policy lock`
     - 목적:
       - hard block / soft advisory / composite gate 중 하나로 고정
     - 왜 하는가:
       - 다음 실험 배치가 같은 애매한 문구를 반복하지 않게 하기 위해서다

### 2. `websocket-first BBO / BookDepth family`

- 역할:
  - REST 조회 위주의 현재 build path 를 WS-primary data path 로 끌어올리는 family
  - BBO 로 tactical pricing
  - BookDepth 로 sizing / paired fill viability / slippage 추정
- 현재 판정:
  - `keep / active-first`
- 이유:
  - 현재 live blocker 는 단순 spread 값이 아니라 paired fill viability 부족이다
  - 실행 로그에서도 BookDepth data 부재가 직접 드러났다
  - 이 family 는 바로 그 부족한 `build 전선의 미시 체결 가능성 판단`을 보완하는 유력 후보다
- 현재 해석:
  - 이 family 가 살아야 `POST_ONLY`/`IOC` 같은 order mode 비교도 의미가 생긴다
  - order mode 보다 먼저, build 직전 market-data authority 를 올리는 family 로 읽는 편이 맞다
- donor code reading:
  - `BBO`, `BookDepth`, `fill` stream 은 실제로 수신된다
  - 반면 `position_change.amount`는 봇 수량 단위와 바로 맞지 않는다
  - 따라서 현재 blocker 는 "WS transport"보다 "WS 이벤트별 truth ownership"과 "phase handoff" 쪽이다
- 다음 판정 질문:
  - BBO 를 WS primary / REST fallback 으로 확실히 올릴 수 있는가
  - BookDepth 를 실제 sizing / viability gate 에 연결할 수 있는가
  - live 에서 `No BookDepth data` 상태를 없앨 수 있는가
- 현재 batch acceptance:
  - `position_change`를 수량 truth에서 제외
  - `fill`을 수량 truth로 고정
  - stale WS 값 때문에 false residual stop이 나지 않게 phase handoff를 고정
  - `No BookDepth data` 없는 WS-centered 3-cycle smoke 통과
- 세부 테스트 슬라이스:
  1. `WS transport`
     - 목적:
       - `best_bid_offer`, `book_depth`, `fill`이 실제로 들어오는지 확인
     - 왜 하는가:
       - transport가 죽은 상태에선 그 뒤 판단이 모두 무의미하다
  2. `truth split`
     - 목적:
       - `fill` vs `position_change` 중 어느 것이 수량 truth인지 고정
     - 왜 하는가:
       - 지금 가장 큰 오판이 `position_change.amount`를 수량처럼 읽은 것이기 때문이다
  3. `phase handoff`
     - 목적:
       - order result, fill WS, close verification의 연결을 고정
     - 왜 하는가:
       - 이 부분이 꼬이면 false residual stop과 과잉 retry가 생긴다
  4. `3-cycle smoke`
     - 목적:
       - 위 1-3이 한 번이 아니라 짧은 배치에서 버티는지 확인
     - 왜 하는가:
       - source map 항목을 “실제 개선 확인”으로 올리려면 반복 evidence가 필요하다
  5. `10-cycle stability`
     - 목적:
       - WS path가 baseline 후보로 갈 만한지 확인
     - 왜 하는가:
       - 진짜 완료 판정은 단발이 아니라 반복 런 기준이어야 한다

#### donor documents

- 아래 문서/코드 표면은 `dnbot` 내부가 아니라 external donor checkout `2dex/` 기준이다.
- `2dex/.omc/plans/nado-dn-pair-websocket-first.md`
  - WS 를 optional 이 아니라 PRIMARY 로 보라는 설계 문서
  - public/private stream 분리
  - WS primary + REST fallback 구조를 명시
- `2dex/.omc/plans/websocket_rest_trading_optimization.md`
  - BBO 와 BookDepth 를 단순 연결이 아니라 trading decision 에 어떻게 쓰는지 설명
  - spread monitoring, sizing, slippage, paired fill viability 의 donor 문서
- `2dex/.omc/plans/nado-websocket-high-performance.md`
  - stream latency / reconnect / event routing / high-performance 관점 donor
  - build 전선에서는 특히 BBO / BookDepth / Fill / PositionChange 의 역할 정리용
- `2dex/docs/WEBSOCKET_COMPARISON_REPORT.md`
  - SDK/raw 구현 비교
  - connected 와 operationally integrated 를 구분해서 읽는 근거

#### actual code surfaces

- `2dex/hedge/exchanges/nado.py`
  - `fetch_bbo_prices()`
    - BBO truth 를 읽는 donor client 표면
  - `get_bbo_handler()`
    - BBO handler authority 유무를 노출하는 표면
  - `estimate_slippage()`
    - BookDepth 를 sizing/slippage path 에 연결하는 API
  - `check_exit_capacity()`
    - depth 기반 exit viability API
- `2dex/hedge/exchanges/nado_websocket_client.py`
  - 실제 WS client
  - `subscribe()` / reconnect / authenticate / routing 표면
- `2dex/hedge/exchanges/nado_bbo_handler.py`
  - BBO latest price, spread monitor, momentum detector
- `2dex/hedge/exchanges/nado_bookdepth_handler.py`
  - local order book state
  - incremental delta 처리
  - slippage / liquidity estimation
- `2dex/hedge/DN_pair_eth_sol_nado.py`
  - `_check_spread_profitability()`
    - spread / timing gate family 의 economics gate 표면
  - `_wait_for_optimal_entry()`
    - entry timing gate 표면
  - `No BookDepth data` / `BookDepth unavailable` warnings
    - live gap 이 실제로 surfaced 되는 donor runtime 표면

#### current gap reading

- donor checkout 기준 코드 표면상 WS/BBO/BookDepth path 는 존재한다
- 하지만 `dnbot`는 planning repo 이므로, 여기서는 donor 경로를 실행 표면이 아니라 evidence anchor 로만 써야 한다
- 하지만 live run 에서는 `No BookDepth data` 가 나왔고, current BUILD 는 여전히 query-heavy 였다
- 따라서 지금 판정은 `구현 없음`이 아니라 `구현은 있으나 build 전선에 충분히 통합되지 않음`이다
- compare-next 의 초점은 새 기능 추가보다:
  - BBO authority 를 실제로 WS 쪽으로 끌어올릴 수 있는지
  - BookDepth availability 를 실제 live 에서 확보할 수 있는지
  - sizing / viability gate 가 정말 depth 신호를 먹는지
  로 잡아야 한다

### 3. `per-leg pricing-mode family`

- 역할:
  - ETH 와 SOL leg 를 같은 주문 모드로 고정하지 않고, leg 별로 다르게 가격/공격성을 조정하는 family
  - `eth_mode`
  - `sol_mode`
  - `bbo_minus_1`, `bbo_plus_1`, `bbo`, `aggressive`, `market`
- 현재 판정:
  - `dormant / spec-only`
- 이유:
  - branch plan 수준에서는 설계가 있었지만, 현재 HEAD/live baseline 에서는 실제 비교군으로 올라오지 않았다
  - 그래도 one-leg risk 가 leg 비대칭에서 온다면, 장기적으로는 이 family 가 paired fill 개선에 기여할 수 있다
- 현재 해석:
  - 당장 첫 compare-next 후보는 아니다
  - 먼저 WS/BBO/BookDepth family 로 market-data truth 를 올린 뒤, 그 다음 leg 별 공격성 조정 family 로 보는 순서가 맞다
- 다음 판정 질문:
  - asymmetric liquidity 를 leg 별 mode 차등으로 줄일 수 있는가
  - `aggressive`/`market` 를 baseline 본체가 아니라 bounded fallback 으로만 둘 수 있는가
  - subaccount lane 으로 넘어가도 재사용 가능한 family 인가
- 세부 테스트 슬라이스:
  1. `mode inventory`
     - 목적:
       - donor 에 실제 존재하는 mode surface를 빠짐없이 분리
     - 왜 하는가:
       - 아직 spec-only라서 먼저 비교군 정의가 필요하다
  2. `asymmetry hypothesis`
     - 목적:
       - one-leg risk 가 leg별 liquidity 비대칭에서 오는지 확인
     - 왜 하는가:
       - 이 가설이 맞을 때만 pricing family 가 다음 단계 의미가 생긴다
  3. `bounded fallback`
     - 목적:
       - aggressive/market 계열을 baseline 본체가 아니라 fallback 범위로 제한할지 결정
     - 왜 하는가:
       - 더 공격적인 mode가 baseline economics를 망치지 않도록 경계가 필요하다

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

1. finish `websocket-first BBO / BookDepth family`
2. run `3-cycle smoke`
3. run `10-cycle stability`
4. only after 1-3, return to `spread / timing gate family`
5. only after 1-4, review `per-leg pricing-mode family`
6. keep `BUILD one-leg handling`
7. keep `UNWIND group`
8. reject `POST_ONLY` as baseline entry
9. reject `IOC` as current live baseline entry

## Execution Runbook

- 첫 실제 실행 런북:
  - `.omx/plans/1dex_spread_timing_execution_plan.md`
- WS-first 실전 런북:
  - `.omx/plans/1dex_websocket_execution_plan.md`
