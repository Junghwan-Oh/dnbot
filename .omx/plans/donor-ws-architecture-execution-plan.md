# Donor / WS / Architecture Execution Plan

작성일: 2026-03-20

## 목적

`$2k` seed, `30일 / $30M 거래량`, `PnL > 0` 목표를 위해 아래 3축을 실행 가능한 순서로 구체화한다.

1. 기존 커밋/브랜치에서 donor 추출
2. WebSocket 기능과 시장 미시구조 최대활용
3. Wintermute quant lead 수준 메인라인 아키텍처 설계

이 문서는 구현팀이 바로 착수할 수 있는 handoff plan이다.

## Execution Discipline

이 계획은 donor를 통째로 섞는 방식이 아니라, **함수/기능 단위 추출 -> 테스트 -> 검증 -> 통합** 순서를 강제한다.

핵심 규율:

1. **SDD-first**
   - donor 혼합 전에 target architecture contract를 먼저 고정한다
   - 어떤 truth source, fallback, state contract에 donor를 꽂을지 먼저 정의한다
2. **Function-scoped extraction**
   - donor는 commit 단위가 아니라 함수/클래스/좁은 feature 단위로 분해한다
   - 예: `fill confirmation`, `position truth`, `cancel/replace`, `BBO handler`
3. **TDD gate before integration**
   - donor를 메인라인에 넣기 전에 해당 donor 기능의 기대 동작을 테스트로 먼저 고정한다
   - source behavior test, target contract test, regression test를 분리한다
4. **No batch donor grafting**
   - 여러 donor를 한 번에 섞지 않는다
   - donor 1개 또는 tightly-coupled donor set 1개씩만 검증 후 통합한다
5. **Promote by evidence**
   - 함수 하나라도 live 또는 replay 기준 검증 실패면 donor-only 또는 reject로 내린다

## Donor Extraction Loop

각 donor는 아래 루프로만 메인라인에 들어갈 수 있다.

1. donor 기능을 1개 함수/feature 단위로 정의
2. source commit에서 그 기능의 실제 behavior와 artifact를 고정
3. target architecture 안에서 필요한 contract를 문서화
4. contract 기준 테스트 작성
5. donor 추출 / 이식
6. 단위 테스트 + regression test + lane-level benchmark 수행
7. 통과 시에만 다음 donor로 진행

즉, 사용자가 말한 **"함수 하나 추출할 때마다 테스트하고 검증하면서 통합"** 이 기본 규칙이다.

## Phase Rule Before Step 1

현재 donor extraction 은 아래 두 phase 로 본다.

### Phase 1 — Trading Infrastructure Hardening
우선 추출 대상:
- execution continuity
- order lifecycle correctness
- flat-close reliability
- safety / verification gate
- WS / REST truth handling
- emergency flatten path
- telemetry / accounting integrity

평가 원칙:
- 이 단계의 practical baseline 은 우선 `-0.05%`
- `-0.1%` 이하 손실 로직은 direct strategy 용도로는 폐기
- 단, direct strategy 로는 폐기해도 execution / safety / telemetry donor 로서의 가치는 별도로 평가

### Phase 2 — PnL Hardening / Optimization
후속 추출 대상:
- strategy quality 개선 donor
- maker ratio 개선 donor
- spread / slippage / fee 최적화 donor
- venue-specific economics donor

Phase 2 메모:
- infra 안정화 이후에는 `0.02%` 수준의 tighter economics baseline 도 적용 가능하다
- 최종 목표는 항상 `PnL >= 0`

## Step 1. Donor Matrix 작성

### 작업

- donor 후보를 아래 그룹으로 재분류한다
  - `2DEX execution donors`
  - `Backpack/GRVT hardening donors`
  - `Nado WS / market-data donors`
  - `Nado working baseline donors`
  - `EdgeX public/private WS donors`
- 기능 단위로 표준화한다
  - entry gating
  - order placement
  - cancel/replace
  - fill confirmation
  - position truth
  - emergency unwind
  - accounting / metrics
  - market-data

### 산출물

- `.omx/plans/donor-matrix.md`

### Acceptance Criteria

- 각 donor 항목에 source commit/branch, 기능 설명, keep/adapt/reject 판단이 있다
- donor 최소 단위가 `함수 / 클래스 / bounded feature` 로 기록된다
- 1DEX/2DEX/Nado/Backpack/EdgeX가 같은 기능 축으로 비교된다
- mainline에 반드시 필요한 donor와 실험 donor가 분리된다

## Step 2. Benchmark Harness 고정

### 작업

- baseline 비교 체크리스트를 아래 KPI로 고정한다
  - gross pnl
  - fee-inclusive pnl
  - pnl%
  - cycles/hour
  - one-leg fill rate
  - flat-close success rate
  - residual position count
  - maker fill ratio
  - fallback-to-taker ratio
- 테스트 레벨을 3단으로 고정한다
  - smoke: `3 cycles`
  - stability: `10 cycles`
  - baseline: `20+ cycles`

### 산출물

- `.omx/plans/benchmark-harness.md`

### Acceptance Criteria

- baseline 후보를 동일 KPI로 비교할 수 있다
- 어떤 테스트가 “성공”인지가 숫자로 정의된다
- 1DEX/2DEX 결과를 동일 스케일로 읽을 수 있다
- donor를 메인라인에 넣기 전 필요한 `unit / regression / benchmark` 3단 테스트가 명시된다

## Step 3. WS + Market Microstructure 활용 계획

### 작업

- 거래소별 공식 capability와 현재 구현을 다시 대응시킨다
  - BBO
  - BookDepth
  - order updates
  - fills
  - positions
  - funding / liquidation / account state
- 다음 microstructure workstream을 우선순위화한다
  - WS-first BBO
  - BookDepth slippage skip
  - fill-probability-aware reprice
  - maker queue proxy
  - dynamic spread threshold
  - exit liquidity gating

### 산출물

- `.omx/plans/ws-microstructure-plan.md`

### Acceptance Criteria

- 각 거래소에서 “현재 쓰는 WS”와 “추가로 써야 할 WS”가 명확하다
- Nado donor에서 가져와야 할 market-data 기능이 분리된다
- 2DEX 메인라인에 이식할 microstructure 기능 순서가 정의된다
- WS donor도 `handler / filter / state feature` 단위로 잘려서 테스트 가능하다

## Step 4. Wintermute-Style Mainline Architecture 설계

### 작업

- 메인라인을 아래 원칙으로 설계한다
  - WS-first market data
  - deterministic position truth
  - event-driven order state
  - maker-first / taker-fallback
  - strict flatness guard
  - fee-aware entry gating
  - emergency flatten path
- candidate lane를 정의한다
  - `Mainline: 2DEX execution lane`
  - `Donor: Nado market-data lane`
  - `Feasibility: 1DEX + subaccount lane`

### 산출물

- `.omx/plans/target-architecture.md`

### Acceptance Criteria

- mainline truth source / fallback source가 컴포넌트별로 적혀 있다
- one-leg fill, stale BBO, drift, close failure에 대한 대응 경로가 있다
- 1DEX+subaccount feasibility와 2DEX fallback 관계가 문서화된다
- donor를 꽂을 target contract와 state boundary가 먼저 정의된다

## Step 5. PnL > 0 달성용 Go / No-Go 규칙

### 작업

- `PnL > 0`를 위한 workstream을 수치 기준으로 고정한다
  - maker fill ratio target
  - max acceptable fee rate
  - max one-leg fill rate
  - max residual position frequency
  - min trade opportunity frequency
  - sybil / wash-risk safe cadence
- kill criteria를 명시한다
  - one-leg fill with residual exposure 반복
  - fee-inclusive pnl materially negative
  - flat-close failure
  - exchange health / subaccount ambiguity unresolved

### 산출물

- `.omx/plans/pnl-positive-kill-criteria.md`

### Acceptance Criteria

- 계속 갈지 버릴지 결정하는 규칙이 숫자로 정의된다
- “거래소를 버려야 할 때”가 명확히 문서화된다
- `$200/day도 위험`이라는 economics constraint가 의사결정 규칙에 반영된다
- donor promotion이 테스트 통과와 숫자 기준을 동시에 만족할 때만 허용된다

## Handoff Commands

다음 단계에서 실행팀이 바로 열어볼 파일:

```bash
sed -n '1,260p' .omx/plans/30m-nonnegative-pnl-roadmap.md
sed -n '1,260p' docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md
sed -n '1,260p' docs/ONEDEX_SUBACCOUNT_DECISION.md
sed -n '1,260p' docs/PROJECT_TEST_AND_ISSUES_REPORT.md
```

다음 단계에서 작성할 계획 파일:

```bash
.omx/plans/donor-matrix.md
.omx/plans/benchmark-harness.md
.omx/plans/ws-microstructure-plan.md
.omx/plans/target-architecture.md
.omx/plans/pnl-positive-kill-criteria.md
```
