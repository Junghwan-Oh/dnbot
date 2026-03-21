# Alternate Diff Report: `b027c98` vs `3c64ac3`

**Scope**: direct code-structure comparison between
- `b027c98` `feat(grvt): STORY-V3 Phase 2 complete - 10 iterations verified`
- `3c64ac3` `feat(alternate): Add DN_alternate_grvt_paradex strategy`

**Intent**:
- `b027c98` = stability-focused evolution of the original hedge bot
- `3c64ac3` = alternate strategy extraction and re-architecture

---

## 1. Executive Summary

`b027c98` and `3c64ac3` are not just two commits on the same line of development.
They represent two different answers to the same question.

- `b027c98` asks: "How do we make the existing bot safer, measurable, and scalable?"
- `3c64ac3` asks: "How do we split alternate into its own strategy engine and trade cross-exchange spread directly?"

In code terms:

- `b027c98` is a **hardening commit**
- `3c64ac3` is a **strategy refactor + new engine commit**

---

## 2. File-Level Diff

| Area | `b027c98` | `3c64ac3` | Interpretation |
|------|-----------|-----------|----------------|
| Main bot | `perpdex/hedge/hedge_mode_bp.py` heavily modified | `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` newly created | `b027c98` improves existing engine, `3c64ac3` forks a new one |
| Exchange layer | `perpdex/hedge/exchanges/grvt.py` patched | full `perpdex/strategies/alternate/exchanges/*` copied in | `3c64ac3` isolates alternate from shared hedge engine |
| Sizing | `helpers/progressive_sizing.py` added | same helper carried into alternate package | `3c64ac3` inherits the scaling concept |
| Rebate | `helpers/rebate_tracker.py` added | same helper carried into alternate package | rebate becomes part of strategy package |
| Docs | post-mortem + stories added | no runtime report, only later README in `6ccaa43` | `b027c98` has better on-branch narrative evidence |

---

## 3. Core Architectural Change

### `b027c98`: improve the old Backpack-first hedge loop

The main class is still a single `HedgeBot` built around Backpack maker orders and GRVT market hedges.

Relevant code:
- class definition and state: [hedge_mode_bp excerpt](#reference-snippets)
- Progressive sizing + rebate tracker initialization
- clean start and hedge verification logic

Characteristics:
- one main loop
- one operational story
- retrofitted reliability features
- strategy logic is still relatively simple and procedural

### `3c64ac3`: create a dedicated alternate strategy engine

The new class is `DNHedgeBot` and strategy identity is fixed to `"alternative"`.

Characteristics:
- explicit `primary_exchange`, `hedge_exchange`
- explicit `primary_mode`, `hedge_mode`
- spread gate via `min_spread_bps`
- reusable exchange factory inside the alternate package
- trade summary and cumulative gross PnL are first-class outputs

This is a larger conceptual jump than the raw insertions suggest.
It is not just "more code"; it moves alternate from a configured mode into a dedicated strategy runtime.

---

## 4. Entry Logic Diff

### `b027c98`

Entry logic is still operationally simple:
- get current price
- compute dynamic size from progressive sizing
- place Backpack post-only order
- react to WebSocket fill
- hedge on GRVT market path

The key idea is:
- keep the execution loop stable
- let sizing and rebate tracking improve outcomes over time

This makes `b027c98` easy to reason about operationally, but its edge model is weak.
It is still closer to a robust volume/hedge engine than a true cross-exchange alpha engine.

### `3c64ac3`

Entry logic becomes spread-aware.

Code path:
- `check_arbitrage_opportunity()` computes spread in bps from both venues
- trade enters only if `spread_bps >= min_spread_bps`

This is the single most important conceptual improvement.

Instead of:
- "enter because the loop says so"

it becomes:
- "enter only if the cross-exchange edge is above threshold"

That is exactly why the later `GRVT-Paradex` runtime family can produce positive sessions at all.

---

## 5. Position Truth Source Diff

### `b027c98`

Position truth is conservative.

It explicitly verifies hedge completion by:
- fetching GRVT position via REST
- fetching Backpack position via REST
- comparing actual vs tracked positions
- rejecting imbalance above threshold

This is slower, but safer.

It also adds `ensure_clean_start()`:
- cancel stale GRVT orders
- cancel stale Backpack orders

That one change directly addresses the documented V4 misdiagnosis and stale-order problem.

### `3c64ac3`

Position truth becomes WebSocket-first.

It uses:
- `get_ws_position()` for both exchanges
- periodic `reconcile_positions()`
- REST fallback only when WS positions look stale or suspicious

This is better for latency and more suitable for alternate spread trading.
But it also introduces a real risk:
- if reconciliation is not strong enough, you can finish profitable while still not flat

That is exactly what the later runtime logs show.

---

## 6. Hedge and Close Logic Diff

### `b027c98`

Close/hedge behavior:
- Backpack remains maker-centric
- GRVT hedge uses market-like taker execution
- post-fill hedge verification is explicit
- stale-order cleanup is explicit

The design goal is:
- do fewer clever things
- verify more

### `3c64ac3`

Close/hedge behavior is more aggressive and more strategy-aware:
- `place_hedge_order()` reprices during waiting
- close path uses aggressive BBO-biased pricing
- `emergency_close_primary()` uses market or aggressive limit fallback
- `force_close_all_positions()` exists as a hard recovery path

This is more sophisticated.
It also means there are more moving parts that can go wrong.

So the tradeoff is clear:
- `b027c98`: simpler, slower, safer
- `3c64ac3`: faster, more adaptive, harder to fully flatten

---

## 7. Runtime/Operations Diff

### `b027c98`

Strong operational support:
- Telegram control
- status file
- progressive sizing telemetry
- rebate milestone tracking
- CSV logging with phase/rebate data

This is "operator-friendly" code.

### `3c64ac3`

Operationally leaner inside the strategy core:
- cumulative PnL printed per cycle
- spread and bps become direct runtime metrics
- exchange abstraction is cleaner

This is "strategy-developer-friendly" code.

---

## 8. Why `b027c98` Still Feels More Stable

`b027c98` wins on:
- stale-order protection
- REST-based truth checks
- hedge completion verification
- fewer implicit assumptions
- stronger branch-internal documentation

It is a safer base commit because it distrusts its own runtime state more.

That is a good trait for an early production bot.

---

## 9. Why `3c64ac3` Unlocks Better PnL

`3c64ac3` wins on:
- spread-gated entry
- explicit bps-based decisioning
- exchange-agnostic strategy packaging
- faster WebSocket-first state handling
- better cross-exchange edge expression

That is why the later `GRVT-Paradex` runtime family can print:
- `18 cycles`, `+$0.1505`, `+2.82 bps`
- `7 cycles`, `+$0.0982`, `+4.77 bps`

But those same runs still end with non-zero net delta.

So `3c64ac3` improves the **profit engine** more than the **flat-exit engine**.

---

## 10. Direct Verdict

### If we care most about production safety

Choose `b027c98`.

Why:
- stronger verification
- cleaner recovery from stale orders
- better operator controls
- better documented validation story

### If we care most about future alternate upside

Choose `3c64ac3`.

Why:
- introduces the real alternate strategy engine
- spread threshold logic is the key profitability unlock
- later positive runtime logs are downstream of this design

### If we want the best actual path forward

Take:
- the **execution safety** of `b027c98`
- the **spread-aware entry model** of `3c64ac3`

That hybrid is the version worth building toward.

---

## 11. Recommended Merge Direction

From `b027c98`, keep:
- `ensure_clean_start()`
- REST-backed hedge verification
- progressive sizing integration
- rebate tracker integration
- operator-friendly CSV / status / Telegram support

From `3c64ac3`, keep:
- `min_spread_bps` gate
- explicit spread-based entry decision
- exchange-agnostic alternate package structure
- `print_trade_summary()` style cumulative PnL reporting
- reconcile loop with WS-first, REST-fallback behavior

Do **not** carry over unchanged:
- ending sessions with non-zero net delta
- aggressive repricing without a stronger flatness guarantee

---

## 12. Reference Snippets

### `b027c98`

- Bot class and state init:
  - `perpdex/hedge/hedge_mode_bp.py` lines 36-60
- Progressive sizing and rebate tracker:
  - `perpdex/hedge/hedge_mode_bp.py` lines 181-203
- Backpack BBO and maker open flow:
  - `perpdex/hedge/hedge_mode_bp.py` lines 968-1018
- Hedge verification:
  - `perpdex/hedge/hedge_mode_bp.py` lines 1475-1536
- Clean start:
  - `perpdex/hedge/hedge_mode_bp.py` lines 1576-1613
- Main trading loop:
  - `perpdex/hedge/hedge_mode_bp.py` lines 1613-1868

### `3c64ac3`

- Alternate bot class and strategy identity:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 43-70
- Trade summary / cumulative PnL:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 168-205
- Spread gate:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 283-309
- WS-first positions and reconcile:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 416-487
- Hedge order execution:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 602-710
- Emergency close:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 941-1010
- Main trading loop:
  - `perpdex/strategies/alternate/DN_alternate_grvt_paradex.py` lines 1049-1188

