# Function Extraction Queue Plan

작성일: 2026-03-20

## 목적

`commit universe -> evidence grading -> donor matrix` 까지의 planning set을
실제 실행 가능한 **함수/feature 단위 extraction queue** 로 변환한다.

이 문서의 목표는 아래를 고정하는 것이다.

- 어떤 donor를 먼저 뽑을지
- donor를 어떤 contract slot에 꽂을지
- donor 1개당 어떤 테스트를 먼저 통과해야 하는지

즉 "좋은 커밋을 안다" 수준을 넘어서,
**실제로 어떤 donor function을 어떤 순서로 추출할지** 까지 내려쓰는 계획이다.

## Queue Construction Rules

queue item은 반드시 아래 형식을 가진다.

- `queue_id`
- `functional_axis`
- `source_family`
- `source_commit_candidates`
- `feature_unit`
- `target_contract_slot`
- `required_tests`
- `promotion_gate`
- `fallback_if_fail`

허용 단위:

- 단일 함수
- 작은 메서드 묶음
- 하나의 handler
- 하나의 risk guard
- 하나의 accounting unit

금지 단위:

- strategy file 전체
- exchange adapter 전체
- branch 전체

## Prioritization Principle

queue는 아래 우선순위로 정렬한다.

### P0. Flatness / Truth

가장 먼저 들어갈 donor:

- clean start
- pre-trade reconciliation
- post-fill verification
- residual exposure detection
- emergency flatten

이유:

- `PnL > 0` 이전에 생존 조건이 먼저다

### P1. Market Data / WS

다음 donor:

- WS-first BBO
- BookDepth snapshot/delta handlers
- stale BBO detection
- liquidity gating

이유:

- 좋은 pricing 없이는 maker ratio와 entry quality를 못 올린다

### P2. Execution

다음 donor:

- maker attempt logic
- cancel/replace loop
- fill confirmation state machine
- taker fallback policy

이유:

- WS data 위에서 execution quality를 올려야 한다

### P3. Economics / Accounting

다음 donor:

- fee-inclusive pnl attribution
- spread threshold logic
- per-cycle economics logging
- maker/taker ratio accounting

이유:

- economics는 마지막에 얹는 것이 아니라, 실행 단계부터 측정돼야 한다
- 다만 flatness / truth 없이 economics donor부터 얹으면 오판 가능성이 높다

### P4. Subaccount Feasibility

별도 lane:

- signer / subaccount truth
- leverage / health interpretation
- same-asset long/short state verification

이유:

- `1DEX + subaccount`는 mainline-target이지만, 아직 feasibility lane이다

## Queue Waves

### Wave 1. Mainline Survival Pack

대상:

- risk-flatness donors
- position truth donors

Acceptance Criteria:

- donor queue의 첫 묶음이 모두 `P0`에 속한다
- 각 item마다 `unit + regression + lane benchmark` 테스트가 붙는다

### Wave 2. Pricing Edge Pack

대상:

- WS BBO
- BookDepth
- stale detection
- liquidity skip

Acceptance Criteria:

- market-data donor가 execution donor보다 먼저 queue에 배치된다
- REST-only pricing은 donor target에서 제외된다

### Wave 3. Execution Quality Pack

대상:

- cancel/replace
- fill confirmation
- maker/taker mode switching

Acceptance Criteria:

- execution donor가 market-data donor 이후에만 promotion 가능하다
- fallback policy가 명시되지 않은 donor는 queue 진입 불가다

### Wave 4. Economics Promotion Pack

대상:

- fee logging
- pnl attribution
- threshold calibration

Acceptance Criteria:

- economics donor는 cycle-level accounting contract가 정의된 뒤에만 queue에 들어간다
- `expected-positive` donor는 evidence 없이 promotion 불가다

### Wave 5. Subaccount Promotion Pack

대상:

- 1DEX + subaccount feasibility donors

Acceptance Criteria:

- mainline-now와 섞이지 않는다
- feasibility proof 전까지 donor-only / research-first로 유지된다

## Per-Item Test Pack

각 queue item은 아래 3종 테스트를 갖는다.

### 1. Source Behavior Test

- 원 donor가 실제로 어떤 동작을 했는지 고정
- live artifact, committed csv/md, replay, code-path assertion 중 하나 필요

### 2. Target Contract Test

- target architecture 안에서 이 donor가 만족해야 할 interface / state contract 검증

### 3. Lane Benchmark Test

- donor 통합 후 lane-level KPI 변화 확인
- 최소 측정:
  - one-leg fill rate
  - flat-close success
  - fee-inclusive pnl%
  - maker fill ratio

## Promotion Rules

queue item은 아래를 모두 만족할 때만 `integrated` 상태가 된다.

1. source behavior understood
2. target contract documented
3. test pack green
4. lane benchmark에서 regression 없음
5. kill criteria 미충족

아래 중 하나면 `rejected` 또는 `donor-only`:

- test pack 부재
- source evidence weak
- lane regression 유발
- function scope가 너무 커서 attribution 불가

## Deliverables

1. donor extraction queue schema
2. priority wave ordering
3. per-item test pack rules
4. promotion / rejection rules

## Acceptance Criteria

- commit family가 실제 extraction queue로 내려온다
- donor 최소 단위가 함수/feature로 강제된다
- 각 queue item에 target contract와 test pack이 연결된다
- mainline execution이 batch graft가 아니라 sequential promotion으로 진행되게 된다

## Next

- `.omx/plans/execution-backlog-sequencing.md`
- `.omx/plans/benchmark-harness.md`
