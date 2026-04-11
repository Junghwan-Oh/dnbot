# PnL Positive Kill Criteria Plan

> REFERENCE ONLY
>
> This is no longer the authoritative operating contract for `1DEX`.
> For `1DEX` work, use:
> - `.omx/plans/1dex-plan.md`
> - `.omx/plans/1dex-operating-contract.md`
>
> This document remains as a cross-lane numeric gate / kill-criteria reference.

작성일: 2026-03-20

## 목적

메인라인 후보를 계속 밀지, donor-only로 내릴지, 거래소 자체를 버릴지를
**숫자 기준**으로 결정한다.

이 문서는 "느낌상 별로다" 수준을 넘어서,
`PnL > 0` 목표와 직접 연결된 go / no-go 기준을 정의한다.

## Core Economics Constraint

목표:

- seed: `$2,000`
- target volume: `$1M / day`
- monthly target: `$30M`
- hard requirement: `fee-inclusive PnL > 0`

핵심 의미:

- `$200/day` 비용도 충분히 위험하다
- `$200/day x 10일 = $2,000` 이므로, 단순 저비용으로는 부족하다
- **low fee** 가 아니라 **net positive** 가 필수 KPI다

## KPI Set

### 1. Execution Quality

- `one_leg_fill_rate`
- `cancel_replace_rate`
- `maker_fill_ratio`
- `fallback_to_taker_ratio`
- `flat_close_success_rate`

### 2. Risk / Flatness

- `residual_position_count`
- `net_delta_drift_events`
- `manual_intervention_count`
- `orphaned_position_incidents`

### 3. Economics

- `gross_pnl_pct`
- `fee_inclusive_net_pnl_pct`
- `fee_bps`
- `slippage_bps`
- `funding_bps`
- `net_bps_per_cycle`

### 4. Throughput

- `cycles_per_hour`
- `trade_opportunity_hit_rate`
- `executed_notional_per_cycle`

## Mandatory Test Tiers

### Tier 1. Smoke

- `3 cycles`
- 목적: 실행 자체 확인

통과 조건:

- no crash
- no residual position
- all positions flat at end

### Tier 2. Stability

- `10 cycles`
- 목적: 반복 운용 안정성 확인

통과 조건:

- `one_leg_fill_rate = 0`
- `flat_close_success_rate = 100%`
- `manual_intervention_count = 0`

### Tier 3. Baseline Validation

- `20+ cycles`
- 목적: baseline 후보 검증

통과 조건:

- `flat_close_success_rate >= 95%`
- `fee_inclusive_net_pnl_pct >= -2 bps`
- `residual_position_count = 0`

### Tier 4. Mainline Candidate

- `100+ cycles` 또는 하루 throughput 근사
- 목적: 메인라인 후보 검증

통과 조건:

- `flat_close_success_rate >= 99%`
- `one_leg_fill_rate <= 1%`
- `fee_inclusive_net_pnl_pct >= 0`

## Kill Criteria

아래 중 하나라도 만족하면 해당 lane 또는 거래소를 `mainline` 후보에서 내린다.

### Immediate Kill

- residual position이 남고 자동 flatten 실패
- one-leg fill 후 manual intervention 필요
- position truth가 일관되지 않음
- subaccount / leverage 상태가 해석 불가

### Economic Kill

- `20+ cycles` 기준 `fee_inclusive_net_pnl_pct <= -5 bps`
- `100+ cycles` 기준 `fee_inclusive_net_pnl_pct < 0`
- 수수료만 줄여도 설명 안 되는 구조적 손실 확인

### Reliability Kill

- `10 cycles` 내 one-leg fill 재발
- `flat_close_success_rate < 95%`
- `manual_intervention_count > 0`

### Exchange-Specific Kill

- 거래소 자체 health / leverage / margin truth가 계속 흔들림
- WS 기능이 있어도 실거래 안정성이 개선되지 않음
- sybil / wash risk를 피하면 trade cadence가 성립하지 않음

## Continue Criteria

다음 단계로 계속 진행할 수 있는 조건:

### Donor-only Continue

- 구조적으로 좋은 기능은 있음
- 하지만 baseline으로는 부족

예:

- WS BBO
- BookDepth
- liquidity-aware skip
- dynamic threshold

### Mainline Continue

- `10 cycles` 안정성 통과
- `20+ cycles`에서 near-flat 이상
- `100+ cycles` 확장에서 `PnL >= 0` 근접

### Mainline Promote

- `fee_inclusive_net_pnl_pct >= 0`
- `flat_close_success_rate >= 99%`
- `one_leg_fill_rate <= 1%`
- manual intervention 없음

## Current Interpretation Rules

현재 테스트를 읽는 규칙:

- `3 cycles` 성공은 working baseline의 힌트일 뿐이다
- `10 cycles` 실패면 mainline baseline으로는 탈락이다
- `10 cycles`는 통과했지만 `20+ cycles` 음수면 economics baseline 탈락이다
- speed가 빨라도 flatness가 약하면 탈락이다

## Lane-Specific Threshold Guidance

### 2DEX lane

현재 관찰된 리스크:

- drift 누적
- market fallback 비중 높음
- 느림

2DEX 유지 기준:

- `10 cycles`에서 residual position 없어야 함
- `20+ cycles`에서 drift가 통제돼야 함
- fallback-to-taker 비율이 낮아져야 함

### Nado lane

현재 관찰된 리스크:

- one-leg fill
- subaccount/leverage ambiguity
- no-trade over-filter 또는 fast-fail

Nado 유지 기준:

- `10 cycles`에서 one-leg fill 재발 금지
- close path가 deterministic 해야 함
- WS-connected 상태가 실제 execution quality 개선으로 이어져야 함

### 1DEX + subaccount lane

현재는 feasibility lane

promote 조건:

- subaccount truth confirmed
- leverage / health interpretation deterministic
- same-asset long/short close path validated

## Deliverables

1. numeric go/no-go rules
2. lane-specific kill criteria
3. interpretation rules for future tests

## Next

- handoff to execution lane only after these thresholds are accepted
