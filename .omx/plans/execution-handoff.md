# Execution Handoff Plan

작성일: 2026-03-20

## 목적

현재까지 만든 planning set을 실제 실행팀이 바로 사용할 수 있도록
착수 순서와 handoff reading order를 고정한다.

이 문서는 구현 지시가 아니라 **execution-ready plan handoff** 문서다.

## Recommended Reading Order

### 1. Strategy / decision context

먼저 아래 문서를 읽는다.

```bash
sed -n '1,260p' docs/ONEDEX_SUBACCOUNT_DECISION.md
sed -n '1,260p' docs/PROJECT_TEST_AND_ISSUES_REPORT.md
sed -n '1,260p' docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md
```

목적:

- 현재 후보 구조
- 테스트 결과
- 거래소별 WS capability
- mainline-now / target 방향

### 2. High-level roadmap

그 다음 아래 문서를 읽는다.

```bash
sed -n '1,260p' .omx/plans/30m-nonnegative-pnl-roadmap.md
sed -n '1,260p' .omx/plans/donor-ws-architecture-execution-plan.md
```

목적:

- 3축 전체
  - donor extraction
  - WS / microstructure exploitation
  - Wintermute-style architecture

### 3. Detailed planning set

그 다음 아래 문서를 순서대로 읽는다.

```bash
sed -n '1,260p' .omx/plans/full-history-donor-matrix-plan.md
sed -n '1,260p' .omx/plans/commit-universe-index.md
sed -n '1,260p' .omx/plans/evidence-grading.md
sed -n '1,260p' .omx/plans/donor-matrix.md
sed -n '1,260p' .omx/plans/mainline-candidate-comparison.md
sed -n '1,260p' .omx/plans/target-architecture.md
sed -n '1,260p' .omx/plans/pnl-positive-kill-criteria.md
```

목적:

- donor universe
- 증빙 강도
- mainline 후보 비교
- target architecture
- go/no-go 기준

## Current Execution Baseline

- immediate baseline lane: `Nado 1DEX ETH-SOL pair`
- target lane: `1DEX + subaccount` same-asset hedge
- venue expansion order: `Nado -> GRVT -> Variational`

## Immediate Next Steps

### Step 1. Populate commit universe index

작업:

- related ref 전체에서 commit inventory 생성
- branch family / strategy family / exchange set / artifact presence tagging
- commit 하나씩 정복하는 방식이 아니라, 전체 donor universe 를 가로질러 기능 단위 체리피킹이 가능하도록 인덱싱한다
- 현재 전략 family 는 `alternate`, `mean reversion`, `breakout` 으로 보되, 현재 baseline 은 `alternate` 임시 기본선으로만 취급한다

Acceptance Criteria:

- relevant commit universe가 빠짐없이 인덱싱됨
- family별로 후보를 집계할 수 있음
- commit aesthetics 가 아니라 feature extraction 용도로 바로 쓸 수 있는 인덱스가 된다

### Step 2. Grade evidence strength

작업:

- 각 candidate commit에 `A/B/C/D`
- `baseline-working / baseline-profitable / ws-infra / economics / risk-flatness` 태그 부여

Acceptance Criteria:

- message-only plus와 artifact-backed plus가 엄격히 분리됨
- working baseline과 profitable baseline이 분리됨

### Step 3. Build full donor matrix

작업:

- 기능 축으로 donor를 배치
  - market-data
  - execution
  - risk-flatness
  - subaccount-leverage
  - accounting-economics
  - ops-observability
- donor를 `함수 / bounded feature` 단위로 잘라 extraction queue 작성
- 각 donor마다 `source behavior -> target contract -> required tests` 연결
- donor 채택 1차 기준은 economics 가 아니라 아래 인프라 안정성 축으로 둔다
  - one-leg fill 방지
  - residual position 방지
  - flat-close determinism
  - 기존 포지션 / 기존 주문 체크
  - 청산 전후 position truth 검증
  - orphaned position 방지
- market-order-heavy execution donor 는 기본적으로 reject 또는 donor-only 로 본다
- limit-first 구조와 보호주문(stop-loss / trailing-stop) 옵션을 살릴 수 있는 donor 를 우선 평가한다

Acceptance Criteria:

- 각 기능에 1차 donor / 2차 donor / reject source가 존재
- mainline에 바로 들어갈 것과 donor-only가 분리됨
- donor 1개마다 테스트 없는 통합이 불가하도록 queue가 정의됨
- 인프라 안정성을 악화시키는 donor 는 PnL 개선이 있어도 바로 promote 되지 않는다

### Step 4. Freeze mainline-now and mainline-target

작업:

- `mainline-now`
- `mainline-target`
- `donor-research`
- `fallback`
를 결정

Acceptance Criteria:

- execution team이 어느 lane를 메인으로 구현할지 명확히 알 수 있음
- fallback path가 정해짐

### Step 5. Start architecture-led execution

작업:

- target architecture 기준으로 execution backlog 작성
- donor 1개 추출 -> 테스트 -> 검증 -> 통합 순서를 backlog 기본 단위로 사용
- baseline stabilization → WS graft → accounting correctness → subaccount feasibility 순으로 추진
- 테스트 ladder 는 `$100 notional`, `3 iter` 로 시작하고, 개선 신호가 있으면 `5 -> 10 -> 15 -> 20` iter 로 확장한다
- 20 iter 기준으로 아래를 확인한다
  - flat-close 유지
  - residual position 없음
  - one-leg fill 악화 없음
  - 1시간당 거래 발생 `<= 2회` 조건에서도 유지되는지
  - PnL 개선이 확실한지

Acceptance Criteria:

- 구현 순서가 donor 순서와 일치함
- `PnL > 0` 달성 전까지 어떤 실험을 중단해야 하는지 명확함
- TDD/SDD gate를 통과하지 않은 donor가 메인라인에 들어가지 않음
- 각 donor/feature 는 commit -> PR -> merge -> planning update 루프로 추적 가능하다

## Non-Negotiables

- `PnL > 0`가 목표다
- `$200/day`도 위험하므로 low-fee는 충분조건이 아니다
- stop-only는 실패다
- flatten 없는 stop은 메인라인 금지
- one-leg fill 재발 시 해당 lane은 baseline 탈락
- market order는 기본 메인라인 execution doctrine이 아니다
- WebSocket capability 는 공통 기능만이 아니라 venue-specific 기능 inventory 까지 확보 후 최대 구현을 검토한다
- BBO 기반 venue-role assignment 는 WebSocket / execution quality 평가 항목으로 본다

## Final Check

- planning set complete: yes
- execution-ready reading order defined: yes
- next-step sequencing defined: yes
- numeric go/no-go rules available: yes
