# dnbot

Clean execution repository for rebuilding a delta-neutral DN bot mainline from donor history.

## What This Repo Contains

- `docs/`: cross-machine handoff and decision documents
- `.omx/plans/`: planning set for donor extraction, WS/microstructure usage, and target architecture
- `src/`: future mainline implementation surface
- `tests/`: future donor-level and integration tests
- `benchmarks/`: future smoke/stability/baseline harnesses
- `donors/`: future donor mapping notes and extraction artifacts

## Current Intent

This repository starts from planning and handoff artifacts only.
It does not treat the legacy 2dex repository as the execution base.
The legacy repository is treated as a donor/archive source.

## First Reading Order

1. `docs/CROSS_MACHINE_HANDOFF_2026-03-21.md`
2. `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`
3. `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
4. `docs/ONEDEX_SUBACCOUNT_DECISION.md`
5. `.omx/plans/30m-nonnegative-pnl-roadmap.md`
6. `.omx/plans/donor-ws-architecture-execution-plan.md`
7. `.omx/plans/function-extraction-queue-plan.md`
8. `.omx/plans/target-architecture.md`
9. `.omx/plans/pnl-positive-kill-criteria.md`
10. `.omx/plans/wintermute-preaudit.md`

## Goal

`$2k` seed, `30 days / $30M volume`, `fee-inclusive PnL > 0`.
