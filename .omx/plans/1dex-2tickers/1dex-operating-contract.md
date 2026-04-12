# 1DEX Operating Contract

작성일: 2026-04-11

## 목적

이 문서는 `1DEX` 레인의 기술 규약 문서다.

이 문서가 소유하는 것:

- truth source rules
- failure taxonomy
- validation gate
- kill criteria
- flatten-required semantics

방향/단계/우선순위는 `1dex-2tickers/1dex-plan.md`가 소유한다.

## Non-Negotiables

### 0. 실전에서는 원인분석보다 현재 포지션 확인이 먼저다

- 실전 중 포지션이 남았을 가능성이 보이면
  - 원인분석부터 하지 않는다
  - 먼저 실제 포지션 조회를 다시 한다
  - 실제 포지션이 남아 있으면 강제청산/강제정리를 먼저 한다
- 웹소켓 값과 REST 값이 다르면
  - 더 위험한 쪽으로 가정하고 멈춘다
  - 그 다음 실제 포지션을 다시 조회한다
- 실제 포지션이 `0`으로 확인되면
  - 그때부터 원인분석으로 넘어간다
- 즉 순서는 항상 아래다.
  1. 현재 포지션 확인
  2. 남아 있으면 정리
  3. 그 다음 원인분석

### 0.5. 어려운 말보다 실제 상태를 먼저 쓴다

- `truth divergence`, `operationally integrated`, `authority mismatch` 같은 말보다
  - "웹소켓 값과 REST 값이 다르다"
  - "실제 포지션은 0인데 봇은 남았다고 본다"
  - "북뎁스가 없는데도 진입을 계속했다"
  처럼 먼저 쓴다
- 어려운 표현이 필요하면 glossary에 같이 풀어 적는다

### 1. Position Truth Before Speed

- fill event가 빨라도 position truth가 흔들리면 baseline 탈락
- WS-only position truth 금지

### 2. WS-First, Not WS-Only Truth

- BBO / BookDepth / order/fill events: WS-first
- critical position truth: WS + REST verification

### 3. Stop Without Flatten Is Failure

- stop-only는 mitigation이 아니다
- flatten 없는 stop은 failure다

### 4. Infra Before Economics

- flatness blocker 해소 전 economics 승격 금지
- truth-source blocker 해소 전 volume 논의 금지

## Planes

### Market Data Plane

필수:

- BBO
- BookDepth
- stale detection
- sequence gap detection
- REST fallback

금지:

- REST-only pricing baseline

### Execution Plane

필수:

- visible order placement path
- cancel/replace observability
- fill confirmation
- hidden fallback loop 금지

### Position Truth Plane

필수:

- fill reconciliation
- post-fill verification
- pre-exit verification
- restart/reconnect verification

### Risk / Flatness Plane

필수:

- one-leg fill handling
- emergency unwind
- orphan cleanup
- residual exposure detection

## Failure Taxonomy

### F1. One-leg fill

정의:

- 한쪽 leg만 체결되고 반대 leg는 reject / timeout / no-fill

판정:

- `P0`

필수 대응:

- deterministic unwind 또는 emergency flatten

### F2. Hedge fail after primary fill

정의:

- primary fill 이후 hedge fail
- false success 또는 position accumulation 발생

판정:

- `P0`

필수 대응:

- actual API position 기준 unwind
- cycle success false 처리

### F3. Position truth drift

정의:

- local state와 actual position 불일치
- restart / reconnect / partial fill 이후 mismatch

판정:

- `P0`

필수 대응:

- critical checkpoint REST verification

### F4. Close path incompleteness

정의:

- stop 되었지만 flat 하지 않음
- orphan position 잔존

판정:

- `P0`

필수 대응:

- close retry path
- orphan cleanup
- flatten-required stop

### F5. WS connected but not operationally integrated

정의:

- WS는 붙어 있으나 실제 decision / truth / risk path에 충분히 연결되지 않음

판정:

- `P1`

필수 대응:

- WS/BBO/BookDepth/order/fill/position 경로 명시

### F6. Stale market data

정의:

- stale BBO / BookDepth / quote 기반 order placement

판정:

- `P1`

필수 대응:

- stale detection
- sequence gap detection
- fallback rule 명시

### F7. Subaccount truth ambiguity

정의:

- health / leverage / isolated-unified 상태 해석 불가
- same-asset long/short truth 불명확

판정:

- `P0` for `1DEX + subaccount`

필수 대응:

- subaccount truth model
- signer/account ownership 정리
- dynamic health 변화가 정상동작인지 incident 인지 구분 규칙 명시

## Truth Source Rules

### Pair / Nado pair absorption phase

authoritative source:

- `git ref: origin/DN_pair_eth_sol_nado`
- target file: `hedge/DN_pair_eth_sol_nado.py`

현재 truth source map 초안:

- `ETH BBO truth`
  - `eth_client.fetch_bbo_prices(...)`
  - 성격: query-based top-of-book
- `SOL BBO truth`
  - `sol_client.fetch_bbo_prices(...)`
  - 성격: query-based top-of-book
- `ETH fill truth`
  - `place_simultaneous_orders()` 의 order result
  - `handle_emergency_unwind()` 및 post-order handling 으로 follow-up 검증
- `SOL fill truth`
  - `place_simultaneous_orders()` 의 order result
  - `handle_emergency_unwind()` 및 post-order handling 으로 follow-up 검증
- `ETH position truth`
  - `eth_client.get_account_positions()`
  - 성격: critical checkpoint authority
- `SOL position truth`
  - `sol_client.get_account_positions()`
  - 성격: critical checkpoint authority
- `Close truth`
  - ETH / SOL 두 다리 모두 verified flat 일 때만 성공으로 판정

필수 체크포인트:

- pre-trade
- post-fill
- pre-exit
- post-close

phase 1 에서 바로 해야 할 audit:

- 어떤 truth 가 polling 인지
- 어떤 truth 가 order result / position recheck 인지
- 어떤 checkpoint 에서 API verification 이 강제되는지
- POST_ONLY / IOC 에서 `OPEN`, `FILLED`, partial/no fill 을 어떻게 구분할지

source map usage rule:

- source map은 현재 로직을 미시적으로 최적화하기 위한 목록이 아니다
- source map의 목적은:
  - 최전선 필터와 truth source를 비교하고
  - 폐기할 것과 남길 것을 빠르게 가르는 것
- 따라서 `entry 인프라의 최전선 필터`가 live에서 계속 막히면
  - 그 로직은 `reject`
  - 소스맵은 그 reject 판정을 빠르게 내리기 위한 도구로 쓴다

### Grouped macro-flow map

#### BUILD group

소속 함수:

- `execute_build_cycle()`
- `place_simultaneous_orders()`
- `_wait_for_optimal_entry()`
- `_check_spread_profitability()`

이 그룹의 역할:

- BUILD 진입 가능 여부를 판단
- ETH / SOL 양 다리 진입을 동시에 발사
- spread / timing / sizing 을 entry decision 에 반영

이 그룹에서 중요한 truth:

- pre-build flatness truth
  - 양 다리 `get_account_positions()`
- build price truth
  - 양 다리 `fetch_bbo_prices(...)`
- build fill truth
  - `place_simultaneous_orders()` 의 `OrderResult`

이 그룹의 현재 약점:

- POST_ONLY `OPEN` 을 fill truth 로 간주하는 해석이 섞여 있다
- 즉 `accepted` 와 `actually filled` 가 혼합될 수 있다
- 실제 mainnet 1-cycle에서 `min-spread-bps 0`이어도 no fill 이 나왔으므로, 현재 `POST_ONLY BUILD entry`는 live baseline 후보로 보기 어렵다

이 그룹에서 고를 최적 로직 기준:

- BUILD 전 flatness 확인
- simultaneous order 유지
- partial / one-leg fill 을 success 로 흡수하지 않기
- build success 는 단순 order acceptance 가 아니라 후속 검증이 동반되어야 함

현재 verdict:

- `POST_ONLY BUILD entry`는 live baseline 기준 `reject`
- BUILD 그룹은 지금 `entry quality 최적화`보다 `one-leg / no-fill을 어떻게 다룰지`가 우선이다

#### UNWIND group

소속 함수:

- `handle_emergency_unwind()`
- `emergency_unwind_eth()`
- `emergency_unwind_sol()`
- `execute_unwind_cycle()`

이 그룹의 역할:

- partial / one-leg fill 시 즉시 exposure 를 줄인다
- UNWIND 주문을 발사하고 verified flat 까지 확인한다

이 그룹에서 중요한 truth:

- pre-unwind position truth
  - 양 다리 `get_account_positions()`
- unwind decision truth
  - 한쪽만 체결됐는지, 두 다리가 모두 성공했는지
- close truth
  - retry 포함 `get_account_positions()` verified flat

이 그룹의 현재 강점:

- `handle_emergency_unwind()` 와 `execute_unwind_cycle()` 라는 명시적 복구/종료 경로가 이미 있다

이 그룹의 현재 약점:

- partial / open / failed 상태의 경계가 더 엄격해야 한다
- close success 를 `order success` 로 볼지 `verified flat` 으로 볼지 항상 후자를 기준으로 잠가야 한다

이 그룹에서 고를 최적 로직 기준:

- one-leg / partial fill 은 즉시 unwind path 로 이동
- unwind 는 actual position 기준
- stop-only 금지
- 최종 성공 판정은 verified flat

우선순위:

- 이 그룹이 현재 `1DEX` 인프라의 최우선 그룹이다
- BUILD 그룹보다 먼저 계약을 고정한다

#### Cross-cutting truth / risk group

소속 함수:

- `get_account_positions()` 를 쓰는 모든 checkpoint
- `fetch_bbo_prices(...)` 를 쓰는 모든 pricing / timing 판단
- cycle-level gate 와 kill criteria 전체

이 그룹의 역할:

- BUILD 와 UNWIND 두 그룹이 공유하는 authority 를 고정
- 어떤 상태값을 실제 truth 로 볼지 잠근다

이 그룹에서 중요한 truth:

- BBO truth
- position truth
- close truth
- gate truth

이 그룹의 현재 약점:

- BBO 는 query-based path 의존이 강하다
- connected 와 operationally integrated 의 차이를 아직 더 엄격히 적어야 한다

이 그룹에서 고를 최적 로직 기준:

- query path, position recheck path, final close verification path 가 서로 충돌하지 않게 명시
- BUILD/UNWIND 어느 쪽도 local optimism 으로 성공 판정하지 않게 고정

### Forced liquidation / forced flatten family

`1DEX`에서 우선 고정할 forced liquidation family:

1. partial / one-leg fill 발생 시 강제 청산
2. UNWIND 실패 시 강제 청산 또는 강제 flatten
3. BUILD 시작 전 residual position 존재 시 진입 금지 + 강제 flatten 경로 검토

현재 contract 해석:

- `partial / one-leg fill` 은 `handle_emergency_unwind()` family 에 직접 연결된다
- `UNWIND 실패` 는 `execute_unwind_cycle()` 이후 verified flat 실패로 판단한다
- `BUILD 시작 전 잔여 포지션` 은 `execute_build_cycle()` 의 pre-build position check 에서 먼저 걸러야 한다

중요:

- `forced liquidation` 은 메인 방법이 아니다
- 본체는 건강한 `BUILD / UNWIND` 가 99%+ 상황에서 정상적으로 작동하는 구조여야 한다
- 강제 청산은 `수수료 손실 확률이 매우 높은 긴급 fallback` 이므로 2차 안전장치로만 둔다
- 따라서 평가 기준도 `forced liquidation 이 있다` 가 아니라 `forced liquidation 없이도 본체가 건강하게 돈다` 가 먼저다

## Live test interpretation rule

현재처럼 실전 1-cycle에서:

- spread filter 6bps -> skip
- spread filter 0bps -> BUILD 시도 but POST_ONLY no fill

이면 읽는 법은 아래다.

1. 환경 문제인지 먼저 분리
2. 환경 문제가 아니면 `entry filter / entry mode`를 본다
3. no fill이 반복되면 `POST_ONLY BUILD entry`는 live baseline 에서 내린다
4. 그 다음에야 `IOC` 같은 대체 진입 경로를 비교한다

### Commit-backed pair reading

`origin/DN_pair_eth_sol_nado` 기준으로 지금 확인된 중요한 구현 사실:

- `place_simultaneous_orders()` 는 ETH / SOL 양 다리를 동시에 발사하려는 중심 함수다
- order 이전에 양쪽 position 을 먼저 조회한다
- BBO 는 ETH / SOL 모두 `fetch_bbo_prices()` query path 를 쓴다
- partial / one-leg fill 은 `handle_emergency_unwind()` 로 복구를 시도한다
- `execute_unwind_cycle()` 는 양 다리를 다시 발사한 뒤, position recheck 와 retry 로 flat close 를 검증한다

즉 absorption phase 의 첫 과제는:

- code quality 감상이 아니라
- 이 commit 이 실제로 어떤 truth / unwind / close contract 를 가지는지 고정하는 것이다

### Current WS / data gap snapshot

현재 확인된 gap:

- `DN_pair_eth_sol_nado.py` 기준 BBO 는 query path 의존이 강하다
- position truth 는 `get_account_positions()` 기반 API recheck 가 핵심이다
- simultaneous order 는 있지만, POST_ONLY `OPEN` 상태를 fill truth 로 볼지 follow-up confirmation 을 더 둘지가 아직 약점이다
- WS 연결 로그는 보이지만, `decision/truth path` 안에서 어떤 WS stream 이 authority 인지는 현재 문서상 불충분하다

따라서 현재 absorption phase 의 WS 과제는:

1. `connected` 와 `operationally integrated` 를 구분
2. ETH / SOL pair baseline 에서 어떤 WS stream 이 실제 execution / truth / risk plane 에 공급되는지 고정
3. query-based BBO 와 position recheck 를 어디까지 유지하고, 어디를 WS-first 로 바꿀지 명시

### Subaccount optimization phase

subaccount truth model draft:

- `identity tuple`
  - main wallet
  - linked signer
  - subaccount name
- `position truth`
  - subaccount 단위 actual position state
- `risk truth`
  - risk engine 기준 health / leverage / margin state
- `close truth`
  - same-asset long/short 두 다리 모두 subaccount 기준으로 verified flat 이어야 함

추가 규칙:

- displayed leverage / health 변화 자체를 곧바로 runtime bug 로 취급하지 않는다
- risk engine 기반 동적 상태와 bot의 position/risk truth 를 분리해서 본다
- linked signer / main wallet / trading key ownership 을 문서상 고정한다
- UI/체감값보다 query 가능한 risk/position source 를 authority 로 둔다

## Close / Unwind Donor Checklist

pair baseline commit 과 `dn-build-unwind-fix` 에서 지금 바로 가져와야 할 donor 규율:

1. primary fill 후 hedge fail 이면 cycle success 로 처리하지 않는다
2. hedge fail 이후 residual position accumulation 을 허용하지 않는다
3. unwind 는 intent parameter 가 아니라 actual API position 기준으로 계산한다
4. stop 만 하고 flat 하지 못하는 path 는 실패로 본다
5. drift monitoring 과 실제 auto-trading recovery 는 분리해서 다룬다

추가 acceptance:

- one-leg / partial fill 발생 시 `handle_emergency_unwind()` 같은 명시적 복구 path 가 있어야 한다
- `execute_unwind_cycle()` 이후 ETH / SOL 양 다리 모두 verified flat 이어야 한다
- BUILD 시작 전에 residual position 이 있으면 진입 금지 또는 강제 flatten policy 가 분명해야 한다
- hedge fail path 가 log 상 명확히 보임
- deterministic emergency unwind 또는 manual-intervention-needed signal 둘 중 하나가 확실히 남음
6. binary success/failure 가 아니라 `flatness`, `manual intervention`, `residual position` 기준으로 acceptance 를 잡는다

이 체크리스트는 absorption phase 에 바로 적용한다.

## Validation Gates

### Gate A. Absorption Smoke

- `3 cycles`

통과 조건:

- no crash
- all positions flat at end
- no manual intervention
- canonical smoke는 `Stage 3` 기준으로 본다

### Gate B. Absorption Stability

- `10 cycles`

통과 조건:

- one-leg fill 재발 금지
- flat-close success = 100%
- deterministic close/unwind path
- restart/reconnect 후 truth consistency
- `Stage 4` 는 cycle behavior / 반복 실행 검증으로 읽는다

### Gate C. Subaccount Feasibility

통과 조건:

- subaccount truth confirmed
- health / leverage interpretation deterministic
- same-asset close path validated
- stop without flatten 없음

### Gate D. Economics Entry

전제:

- Gate A/B 통과
- subaccount를 mainline-target으로 볼 경우 Gate C 통과

그 뒤에만 economics / volume workstream 허용

## Kill Criteria

아래 중 하나라도 만족하면 해당 lane은 baseline 승격 금지다.

- residual position + auto flatten 실패
- one-leg fill 후 manual intervention 필요
- position truth inconsistent
- stop without flatten
- subaccount / leverage / health 상태 해석 불가
- WS 기능이 있어도 execution quality 개선이 없음

## Research Facts Preserved From Legacy Memo

이 문서가 흡수한 legacy 고유 facts:

- `1DEX pair` 는 true hedge 가 아니라 pair-basis-risk 를 가진 구조다
- `1DEX + subaccount` 는 same-asset hedge 에 더 가깝다
- Nado subaccount / health / leverage 는 동적으로 흔들릴 수 있다
- linked signer 분리는 구현상 고려 대상이다
- wash / sybil risk 때문에 단순 왕복 구조는 정책 리스크를 가진다

## Strong Donor Anchors

- `../.omc/plans/dn-build-unwind-fix.md`
- `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
- `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`

## Immediate Technical TODO

1. pair baseline truth source map 을 file:line 기준으로 더 촘촘히 고정
2. Nado query-based BBO 를 언제까지 baseline 으로 허용할지 결정
3. Lighter timeout fallback filled-path 를 truth source 에서 분리
4. close success 판정을 verified flat 기준으로 강제
5. subaccount identity tuple 과 authority queries 를 정리

## Final Rule

`1DEX`는 지금 economics lane이 아니다.

`1DEX`는 지금:

- truth lane
- flatness lane
- close/unwind lane
- subaccount feasibility lane

으로 다뤄야 한다.

## See Also

- `.omx/plans/1dex-lessons.md`
- `.omx/plans/1dex_execution_insights.md`

## Test Status Rule

- `Stage 2` 는 현재 pair baseline 기준 canonical test가 아니다
- `Stage 3` 를 현재 canonical smoke로 본다
- `Stage 4` 를 cycle behavior verification으로 본다
