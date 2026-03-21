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

## Immediate Next Steps

### Step 1. Populate commit universe index

작업:

- related ref 전체에서 commit inventory 생성
- branch family / strategy family / exchange set / artifact presence tagging

Acceptance Criteria:

- relevant commit universe가 빠짐없이 인덱싱됨
- family별로 후보를 집계할 수 있음

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

Acceptance Criteria:

- 각 기능에 1차 donor / 2차 donor / reject source가 존재
- mainline에 바로 들어갈 것과 donor-only가 분리됨
- donor 1개마다 테스트 없는 통합이 불가하도록 queue가 정의됨

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

Acceptance Criteria:

- 구현 순서가 donor 순서와 일치함
- `PnL > 0` 달성 전까지 어떤 실험을 중단해야 하는지 명확함
- TDD/SDD gate를 통과하지 않은 donor가 메인라인에 들어가지 않음

## Non-Negotiables

- `PnL > 0`가 목표다
- `$200/day`도 위험하므로 low-fee는 충분조건이 아니다
- stop-only는 실패다
- flatten 없는 stop은 메인라인 금지
- one-leg fill 재발 시 해당 lane은 baseline 탈락

## Final Check

- planning set complete: yes
- execution-ready reading order defined: yes
- next-step sequencing defined: yes
- numeric go/no-go rules available: yes
