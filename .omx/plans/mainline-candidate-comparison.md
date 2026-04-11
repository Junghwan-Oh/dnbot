# Mainline Candidate Comparison Plan

> REFERENCE ONLY
>
> This is no longer the authoritative decision document for `1DEX`.
> For `1DEX` work, use:
> - `.omx/plans/1dex-plan.md`
> - `.omx/plans/1dex-operating-contract.md`
>
> This document remains as a historical cross-lane comparison snapshot.

작성일: 2026-03-20

## 목적

현재 가능한 메인라인 후보 3개를 같은 비교축 위에 올려서,

- 무엇을 당장 메인라인으로 둘지
- 무엇을 donor-only 또는 feasibility lane으로 둘지
- 무엇을 폐기할지

를 결정하는 계획이다.

후보 3개:

1. `2DEX` 단일자산 delta-neutral
2. `1DEX pair` 노셔널 매칭형
3. `1DEX + subaccount` 단일자산 long/short 분리

## Comparison Axes

세 후보는 아래 축으로만 비교한다.

### 1. Flatness

질문:

- one-leg fill 발생 시 복구 가능한가
- 종료 시 포지션이 실제로 0이 되는가
- 3회 / 10회 / 20회 확장 시 residual risk가 어떻게 증가하는가

### 2. Execution Reliability

질문:

- trade가 실제로 계속 발생하는가
- no-trade / over-filter / one-leg-fill 중 어느 쪽 문제를 더 많이 보이는가
- maker fill과 taker fallback 비율은 어떤가

### 3. WS / Market-Data Utilization

질문:

- 공식 WS capability 중 핵심 기능을 얼마나 쓰는가
- BBO / BookDepth / Fill / Position stream 을 실제 trading decisions에 쓰는가
- REST fallback이 보조인지, 주 경로인지

### 4. Economics

질문:

- gross pnl%
- fee-inclusive net pnl%
- cycle당 손실/이익
- $1M/day scaling 가정 시 burn rate

### 5. Operational Simplicity

질문:

- 구현/운영 복잡도는 어떤가
- debugging cost는 어떤가
- 계정/키/경로/구조가 얼마나 단순한가

### 6. Exchange Risk

질문:

- 거래소 구조 자체가 bot 구현을 어렵게 만드는가
- leverage / health / subaccount ambiguity가 큰가
- wash / sybil risk가 큰가

## Current Provisional Read

### Candidate 1: 2DEX

현재까지 보이는 성격:

- 장점:
  - 거래가 잘 발생
  - 실행 흐름이 비교적 예측 가능
  - donor가 많음
- 단점:
  - 느림
  - market fallback 비중 높음
  - 확장 테스트에서 drift 누적
  - economics 약함

현재 provisional tag:

- `mainline-fallback`

### Candidate 2: 1DEX pair

현재까지 보이는 성격:

- 장점:
  - 같은 거래소 안에서 빠르게 돌 수 있음
  - WS / BookDepth donor 가치 높음
- 단점:
  - 진짜 delta-neutral이 아님
  - pair basis risk 존재
  - 메인라인 전략으로는 해석 불안정

현재 provisional tag:

- `donor-only`

### Candidate 3: 1DEX + subaccount

현재까지 보이는 성격:

- 장점:
  - 같은 API
  - 같은 거래소
  - 같은 자산
  - 구조적으로 가장 단순한 진짜 hedge
- 단점:
  - 아직 feasibility 미확인
  - subaccount / leverage / health ambiguity
  - exchange policy / sybil risk

현재 provisional tag:

- `mainline-target`

## Plan

### 1. Candidate fact sheet 작성

작업:

- 3개 후보마다 fact sheet 작성
  - 구조
  - 장점
  - 실패 패턴
  - known blockers

Acceptance Criteria:

- 후보마다 같은 형식으로 읽을 수 있다
- 감각적 선호가 아니라 evidence-based 비교가 가능하다

### 2. Test evidence mapping

작업:

- small / extended live test 결과를 각 후보에 연결
- `3회`, `10회`, 향후 `20회+` 결과를 동일 포맷으로 붙인다

Acceptance Criteria:

- 각 후보의 smoke/stability 상태가 한 눈에 보인다
- one-leg fill / no-trade / drift 누적 패턴이 분리된다

### 3. Economics scaling model

작업:

- 하루 `1M`, 월 `30M` 기준으로
  - gross
  - fees
  - net
  - burn rate
를 후보별로 모델링

Acceptance Criteria:

- `$200/day도 위험`이라는 조건이 비교에 반영된다
- `PnL > 0`가 안 되면 mainline 탈락이라는 판단이 숫자로 가능하다

### 4. Decision rules

작업:

- 최종 역할을 정한다
  - `mainline-now`
  - `mainline-target`
  - `fallback`
  - `donor-only`
  - `reject`

Acceptance Criteria:

- 3개 후보가 하나의 role로 정리된다
- "왜 이 후보를 지금 메인으로 두는지" 설명 가능하다

### 5. Handoff to Architecture

작업:

- candidate comparison 결과를 target architecture 문서로 넘긴다
- 메인라인과 donor lane를 고정한다

Acceptance Criteria:

- architecture 단계에서 다시 후보 논쟁을 반복하지 않는다
- lane 분리:
  - execution mainline
  - donor research
  - subaccount feasibility
가 명확하다

## Expected Output

- `.omx/plans/mainline-candidate-comparison.md` (this file)
- next:
  - `.omx/plans/target-architecture.md`
  - `.omx/plans/pnl-positive-kill-criteria.md`
