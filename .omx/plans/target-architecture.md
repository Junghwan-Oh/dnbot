# Target Architecture Plan

작성일: 2026-03-20

## 목적

`$2k` seed, `30일 / $30M 거래량`, `PnL > 0` 목표를 위해
메인라인이 가져야 할 **production-grade architecture** 를 정의한다.

이 문서는 구현이 아니라 구조 계획이다.
핵심 목표는 아래 3가지를 동시에 만족하는 것이다.

- flatness
- fee-inclusive non-negative economics
- high-frequency repeatability

## Design Principles

### 1. Position Truth Before Speed

속도보다 먼저 포지션 진실원천을 고정한다.

- fill event가 아무리 빨라도
- 실제 position truth 가 흔들리면
- 그 시스템은 메인라인 후보가 아니다

### 2. WS-First Market Data, Not WS-Only Truth

시장 데이터는 WS-first가 맞다.
하지만 포지션 truth는 무조건 WS-only로 두지 않는다.

- BBO / BookDepth / trade events: WS-first
- position truth: WS + REST verification

### 3. Maker-First, Taker-When-Risk-Critical

메인 경제성은 maker에 의존해야 한다.
하지만 리스크 상황에서는 taker fallback을 허용한다.

### 4. Every Cycle Must Close Cleanly

`PnL > 0` 이전에 `flat close`를 먼저 만족해야 한다.
one-leg fill 을 못 다루면 scale은 금지한다.

## Target System Layers

### Layer 1. Market Data Plane

책임:

- BBO
- BookDepth
- spread state
- microstructure features
- queue / slippage proxies

원칙:

- WS-first
- REST fallback
- stale detection
- sequence gap detection

source donors:

- Nado WS line
- GRVT WS/BBO routing line
- EdgeX public depth line

### Layer 2. Execution Plane

책임:

- order placement
- cancel/replace
- maker attempt
- taker fallback
- fill confirmation

원칙:

- active cancel-and-replace
- fee-aware mode switching
- no hidden fallback loops
- every fallback is observable

source donors:

- 2DEX restoration line
- 2DEX alternate line
- Nado working baseline

### Layer 3. Position Truth Plane

책임:

- actual position state
- fill reconciliation
- residual position detection
- subaccount truth

원칙:

- WS local tracking allowed
- REST verification mandatory at critical checkpoints
- pre-trade / post-fill / pre-exit reconciliation

source donors:

- `b027c98` hardening line
- Nado close/subaccount fixes
- WS position tracking fixes

### Layer 4. Risk / Flatness Plane

책임:

- one-leg fill handling
- emergency unwind
- flatness guard
- orphaned position cleanup
- drift kill switch

원칙:

- stop is not enough
- stop without flatten is failure
- residual exposure is first-class incident

source donors:

- `b027c98`
- 2DEX hardening
- Nado close-order / orphan cleanup

### Layer 5. Accounting / Economics Plane

책임:

- gross pnl
- fees
- rebate
- funding
- net pnl
- pnl%
- cycle-level attribution

원칙:

- every cycle has fee-inclusive accounting
- estimated pnl and realized pnl must be separated
- no strategy promotion without fee-inclusive results

source donors:

- 2DEX metrics / analysis line
- Nado fee logging line
- economics commits

## Candidate Lane Mapping

### Mainline-now

- `2DEX execution lane`

이유:

- 현재까지는 trade continuity 가 가장 높다
- 느리고 market fallback 많지만, 적어도 실행 흐름이 더 예측 가능하다

### Mainline-target

- `1DEX + subaccount lane`

이유:

- 같은 API
- 같은 거래소
- 같은 자산
- 구조적으로 가장 단순한 진짜 hedge

단, 아직 feasibility 검증이 필요하다.

### Donor Research Lane

- `Nado WS / BookDepth / BBO lane`
- `EdgeX public/private WS lane`
- `Backpack execution donor lane`

## Required Capabilities for Mainline

메인라인이 되려면 아래 능력이 모두 필요하다.

1. `BBO` WS-first
2. `BookDepth` for slippage gating
3. `order updates` real-time
4. `position truth` verified
5. `maker ratio` measurable
6. `fallback-to-taker ratio` measurable
7. `emergency flatten` deterministic
8. `fee-inclusive net pnl` cycle-level

## Explicit Non-Goals

메인라인에서 당장 비목표로 두는 것:

- pair-based pseudo-neutral strategy를 메인 전략으로 채택
- pure REST-only market data
- outcome evidence 없이 expected-positive commit을 baseline으로 채택
- stop-only safety (flatten 없는 stop)

## Architecture Comparison Rules

아키텍처 단계에서 후보를 바꿀 수 있는 조건:

- `1DEX + subaccount` feasibility confirmed
- one-leg fill control proven
- fee-inclusive pnl 이 0에 수렴

아키텍처 단계에서 후보를 버리는 조건:

- subaccount truth 불안정
- close path 불완전
- WS market-data 붙여도 position truth 실패

## Deliverables

1. `target architecture memo`
2. `lane ownership model`
3. `truth source map`
4. `fallback map`
5. `kill criteria linkage`

## Next

- `.omx/plans/pnl-positive-kill-criteria.md`
