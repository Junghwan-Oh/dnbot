# 1dex-sub

Execution and planning repository for the `1DEX subaccount` mainline.

## What This Repo Contains

- `docs/`: donor handoff and older cross-machine context
- `.omx/plans/1dex_execution_insights.md`: highest-signal live execution insights
- `.omx/plans/1dex-lessons.md`: steering lessons to reuse
- `.omx/plans/1dex_glossary.md`: shared terminology
- `.omx/plans/1dex-2tickers/`: archived `1DEX two-ticker` lane documents
- `src/`: future `1dex-sub` implementation surface
- `tests/`: future donor-level and integration tests
- `benchmarks/`: future smoke/stability/baseline harnesses

## Current Intent

This repository is now the document/mainline home for `1dex-sub`.

- `2dex` remains the donor and current execution sandbox
- `1DEX two-ticker` is preserved as a reference lane under `.omx/plans/1dex-2tickers/`
- the root `1dex-*` documents are the reusable pieces that should carry forward into `1dex-sub`

## First Reading Order

1. `.omx/plans/1dex_execution_insights.md`
2. `.omx/plans/1dex-lessons.md`
3. `.omx/plans/1dex_glossary.md`
4. `.omx/plans/1dex-2tickers/1dex-plan.md`
5. `.omx/plans/1dex-2tickers/1dex_source_map.md`
6. `.omx/plans/1dex-2tickers/1dex-operating-contract.md`
7. `docs/CROSS_MACHINE_HANDOFF_2026-03-21.md`
8. `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
9. `.omx/plans/target-architecture.md`
10. `.omx/plans/pnl-positive-kill-criteria.md`

## Goal

Build the `1DEX subaccount` mainline with donor-filtered lessons from prior `1DEX` and `2DEX` work.
