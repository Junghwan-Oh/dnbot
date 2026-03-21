# Cross-Machine Handoff

작성일: 2026-03-21

## 목적

다른 머신에서 지금까지의 분석, 테스트 해석, planning set을 그대로 이어받아
donor extraction / WS 활용 / target architecture 작업을 계속할 수 있게 하는
컨텍스트 보존 문서다.

## 현재 목표

- seed: `$2,000`
- 목표 거래량: `30일 / $30M`
- hard requirement: `fee-inclusive PnL > 0`

핵심 의미:

- `$1M/day` 처리량 자체보다 `PnL > 0`가 더 중요하다
- `$200/day` 비용도 10일이면 seed 대부분을 태우므로 충분조건이 아니다
- 목표는 "낮은 비용"이 아니라 **net positive economics** 다

## 현재 전략 결론

현재까지의 결론은 아래와 같다.

1. `2DEX` 와 `1DEX(Nado)` 모두 아직 완성된 mainline은 아니다
2. `2DEX` 는 execution continuity donor 가치가 높다
3. `Nado` 는 WS / BBO / BookDepth donor 가치가 높다
4. `1DEX pair` 는 mainline 전략이 아니라 donor/research lane으로 보는 편이 맞다
5. 최종 target은 `1DEX + subaccount` same-asset hedge 이다
6. 단, subaccount / leverage / health truth가 검증되기 전까지는 feasibility lane이다

## 지금까지 만든 핵심 문서

### 분석 / 결정 문서

- `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
- `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`
- `docs/ONEDEX_SUBACCOUNT_DECISION.md`
- `docs/ALTERNATE_HANDOFF_CONTEXT.md`
- `docs/ALTERNATE_BRANCH_TIMELINE_REPORT.md`
- `docs/ALTERNATE_DIFF_REPORT_B027C98_VS_3C64AC3.md`

### 계획 문서

- `.omx/plans/30m-nonnegative-pnl-roadmap.md`
- `.omx/plans/donor-ws-architecture-execution-plan.md`
- `.omx/plans/full-history-donor-matrix-plan.md`
- `.omx/plans/commit-universe-index.md`
- `.omx/plans/evidence-grading.md`
- `.omx/plans/donor-matrix.md`
- `.omx/plans/function-extraction-queue-plan.md`
- `.omx/plans/mainline-candidate-comparison.md`
- `.omx/plans/target-architecture.md`
- `.omx/plans/pnl-positive-kill-criteria.md`
- `.omx/plans/wintermute-preaudit.md`
- `.omx/plans/execution-handoff.md`

## 읽는 순서

다른 머신에서 바로 읽을 때는 아래 순서를 권장한다.

1. `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`
2. `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
3. `docs/ONEDEX_SUBACCOUNT_DECISION.md`
4. `.omx/plans/30m-nonnegative-pnl-roadmap.md`
5. `.omx/plans/donor-ws-architecture-execution-plan.md`
6. `.omx/plans/commit-universe-index.md`
7. `.omx/plans/evidence-grading.md`
8. `.omx/plans/donor-matrix.md`
9. `.omx/plans/function-extraction-queue-plan.md`
10. `.omx/plans/target-architecture.md`
11. `.omx/plans/pnl-positive-kill-criteria.md`
12. `.omx/plans/wintermute-preaudit.md`

## 현재 planning set의 핵심 원칙

1. donor는 commit 통째로 가져오지 않는다
2. donor는 `함수 / bounded feature` 단위로 추출한다
3. donor 1개마다 테스트 후에만 통합한다
4. `SDD-first`: target contract를 먼저 고정한다
5. `TDD gate`: source behavior / target contract / lane benchmark를 먼저 통과한다
6. `No batch grafting`: 여러 donor를 한 번에 섞지 않는다

즉 현재 계획은 **function-scoped extraction -> test -> verify -> integrate** 루프를 기본 규칙으로 둔다.

## 현재 계획의 판정

- execution-worthy: yes
- strict TDD/SDD readiness: conditional yes
- institutional-grade planning quality: near-pass
- `PnL > 0` 가능성: meaningful but unproven

정확한 해석:

- 이 planning set은 실행할 가치가 있다
- 하지만 economics를 아직 증명한 것은 아니다
- `100+ cycles fee-inclusive >= 0` 전에는 메인라인 확정 금지다

## 다음 작업 우선순위

1. commit universe 실제 인덱싱
2. evidence grading 실제 적용
3. donor matrix를 real queue로 채우기
4. donor 1개씩 tests-first extraction
5. lane benchmark 후 promote / reject

## Git / PR 주의사항

현재 브랜치는 `feature/2dex` 이다.

중요:

- `push` 는 worktree 자체를 올리는 게 아니라 **commit** 을 올린다
- 따라서 uncommitted 변경은 push만으로 올라가지 않는다
- 하지만 현재 워크트리는 이미 매우 dirty할 수 있으므로, 무심코 `git add .` 하면 관련 없는 파일까지 섞일 수 있다

권장:

1. handoff 전용 브랜치를 새로 만든다
2. 가능하면 **새 worktree 또는 clean branch** 에서 handoff 문서/계획만 담는다
3. 최소한 아래 파일만 선택적으로 add / commit 한다
   - `docs/CROSS_MACHINE_HANDOFF_2026-03-21.md`
   - `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
   - `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`
   - `docs/ONEDEX_SUBACCOUNT_DECISION.md`
   - `docs/ALTERNATE_*`
   - `.omx/plans/*.md`

## 권장 PR 성격

이 PR은 구현 PR이 아니라 아래 성격으로 보는 게 맞다.

- research handoff
- planning artifact PR
- donor extraction roadmap PR

즉 목적은 코드를 바꾸는 것이 아니라,
다른 머신 / 다른 세션 / 다른 실행팀이 같은 컨텍스트에서 이어받게 하는 것이다.
