# 30M Volume With Non-Negative PnL Roadmap

작성일: 2026-03-20

## Goal

`$2,000` seed로 `30일 / $30M 거래량`을 만들면서 `PnL > 0` 를 유지할 수 있는 실행 가능한 메인라인을 설계한다.

핵심 현실 제약:

- 하루 목표 거래량: `$1M`
- 월 목표 거래량: `$30M`
- 하루 비용이 `$200`만 나도 10일이면 seed의 대부분이 소진된다
- 따라서 `$200/day`도 충분히 좋은 숫자가 아니라, **실질적으로는 `PnL >= 0`가 필수**다
- 단기 운영 목표는 `1 day / $1M` 를 현실적인 중간 milestone 으로 두고 검증한다
- 현재 baseline 실행 가정은 `5x leverage`, long/short 모두 사용 가능한 perpetual DEX point farming DN bot 이다

즉 이 계획의 목적은 "낮은 비용"이 아니라 **non-negative economics** 를 달성할 수 있는 donor 추출과 시스템 설계를 만드는 것이다.

## Three Pillars

이 계획의 핵심은 아래 3가지를 하나로 합치는 것이다.

1. **기존 커밋에서 장점 추출**
   - donor를 기능 단위로 분해하고
   - 어떤 commit family 에서 무엇을 가져올지 결정한다

2. **WebSocket 기능 + 시장 미시구조 최대활용**
   - 단순 order update 수준이 아니라
   - BBO, BookDepth, Fill, Position, liquidity state, spread state를 최대한 활용하는 방향으로 설계한다

3. **Wintermute quant lead 수준의 메인라인 아키텍처**
   - 실행 엔진, truth source, fallback, risk guard, economics gate를 production 기준으로 재정의한다

즉 최종 계획은:

- donor extraction
- WS / market microstructure exploitation
- institutional-grade execution architecture

를 합쳐서 `PnL > 0` 이 가능한 메인라인을 만드는 로드맵이어야 한다.

실제 착수 우선순위는 아래 순서로 해석한다.

1. 매매 인프라 안정성 확보
2. WebSocket capability / execution quality 강화
3. economics / PnL 개선

즉 `PnL > 0` 는 최종 목표이지만, 초기 donor 채택의 1차 기준은 인프라 안정성이다.

## Evidence Base

이 계획은 아래 근거를 기반으로 한다.

- 로컬 분석 문서:
  - `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
  - `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`
  - `docs/ONEDEX_SUBACCOUNT_DECISION.md`
  - `docs/ALTERNATE_*`
  - `docs/2DEX_HEDGE_MODE_COMPREHENSIVE_PLAN.md`
  - `docs/RESTORATION_HISTORY_2DEX_HEDGE_MODE.md`
- live retest 결과:
  - 2DEX 소규모 / 확장 테스트
  - Nado baseline 소규모 / 확장 테스트
- 공식 문서:
  - Nado margin / subaccounts / fee / points 문서
  - GRVT market/trading websocket 문서
  - Backpack websocket 문서

## Plan

### 1. Economics Constraint Update

목적:

- 기존 의사결정 문서에서 "$1M에 $200 비용도 괜찮다"로 읽힐 수 있는 부분을 수정한다
- 목표가 `30M / month`, `PnL > 0` 임을 명확히 고정한다

작업:

- `docs/ONEDEX_SUBACCOUNT_DECISION.md` 에 seed burn math 추가
- `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md` 에 `PnL > 0`가 필수 KPI임을 명시
- gross / fee / slippage / funding / residual delta까지 포함한 economics 정의를 하나로 통일

Acceptance Criteria:

- 문서에 `1M/day`, `30M/month`, `seed burn` 수치가 명시된다
- "$200/day도 충분히 위험"이라는 점이 분명히 적힌다
- success metric 이 `low fee` 가 아니라 `net non-negative` 로 재정의된다

### 2. Pillar 1: Donor Extraction Matrix

목적:

- 모든 donor를 기능 단위로 분해해 어떤 라인에서 무엇을 가져올지 결정한다
- 핵심 3축 중 첫 번째 축을 명확히 실행 가능한 표로 만든다

작업:

- donor 후보를 4개 레이어로 재분류
  - market-data
  - execution
  - risk / flatness
  - accounting / telemetry
- commit family 기준 donor 추출표 작성
  - 2DEX alternate
  - Backpack/GRVT historical hardening
  - Nado WS line
  - EdgeX donor
- 각 donor를 `keep / adapt / reject` 로 분류

Acceptance Criteria:

- donor matrix 가 기능 단위로 완성된다
- 각 기능마다 source commit / rationale / rejection 이유가 적힌다
- "메인라인에 반드시 필요한 것"과 "실험적 donor"가 분리된다
- 3축 중 `기존 커밋 장점 추출` 항목이 독립 deliverable로 보인다

### 3. Pillar 2: WebSocket + Market Microstructure Exploitation Plan

목적:

- WebSocket 기능과 시장 미시구조를 어디까지 활용해야 `PnL > 0`로 갈 수 있는지 명확히 한다

작업:

- 거래소별 공식 capability와 현재 구현을 다시 매핑
  - BBO
  - BookDepth
  - order updates
  - fills
  - positions
  - funding / liquidation / account state
- DEX별 capability를 공통 기능 + venue-specific 기능으로 나눠 inventory 작성
- 각 DEX의 SDK / GitHub / 공식문서를 통해 사전 capability 조사 후, 최대 구현 후보군을 확보한다
- microstructure workstream 정의
  - maker queue position proxy
  - slippage-aware skip
  - dynamic spread filter
  - fill-probability-aware reprice
  - BookDepth 기반 exit capacity
  - BBO 기반 venue-role assignment
- 공통 benchmark schema 정의
  - gross pnl
  - fees
  - net pnl
  - pnl per notional
  - cycles/hour
  - one-leg fill rate
  - flat-close success rate
  - residual position frequency

메모:

- 2DEX 진입 로직에서는 `더 싼 venue long / 더 비싼 venue short` 와 `spread 구조상 더 유리한 side 배치` 가 상충할 수 있다
- 이 부분은 지금 당장 고정 doctrine으로 박지 않고, WebSocket capability / execution quality / benchmark / PnL 결과를 함께 보며 후속 최적화 대상으로 둔다

Acceptance Criteria:

- 각 거래소에서 "활용 중인 WS"와 "활용해야 할 WS" 차이가 명확히 적힌다
- microstructure 개선축이 구체적인 기능 목록으로 나온다
- 동일한 형식으로 1DEX/2DEX 결과를 비교할 수 있는 benchmark checklist가 생긴다

### 4. Pillar 3: Wintermute-Style Target Architecture

목적:

- 단순 "어떤 커밋이 좋다"가 아니라, 목표 달성용 메인라인 구조를 정의한다
- 핵심 3축 중 세 번째 축인 `quant lead 수준 아키텍처`를 독립 deliverable로 만든다

작업:

- target architecture를 다음 원칙으로 정의
  - WS-first market data
  - deterministic position truth
  - maker-first when economical, taker fallback when risk critical
  - event-driven order state
  - strict flatness guard
  - fee/slippage-aware entry gating
- lane 분리
  - mainline execution lane
  - donor research lane
  - subaccount feasibility lane
- 1DEX + subaccount / 2DEX fallback 아키텍처 비교

Acceptance Criteria:

- 메인라인 아키텍처 다이어그램 또는 서술이 완성된다
- 각 컴포넌트의 truth source 와 fallback source 가 정의된다
- one-leg fill, drift, stale BBO, close failure 에 대한 대응 경로가 명시된다
- `institutional-grade execution architecture` 라는 표현이 문서상 추상어가 아니라 구체 설계로 내려온다

### 5. Donor Extraction Sequencing: Infra First, PnL Second

목적:

- donor extraction을 한 덩어리로 보지 않고, 실제 개발 순서에 맞게 **2 phase**로 나눈다
- 초기 목표는 전략 알파보다 먼저 **매매 인프라 안정화**를 달성하는 것이다

Phase 1 — Trading Infrastructure Hardening:

- execution continuity
- order lifecycle correctness
- flat-close reliability
- safety / verification gate
- WS / REST truth handling
- emergency flatten path
- telemetry / accounting integrity
- one-leg fill 방지
- residual position 방지
- 기존 포지션 / 기존 주문 상태 확인
- position truth reconciliation

Phase 1 평가 기준:

- practical baseline 은 우선 `-0.05%` 로 둔다
- `-0.1%` 이하 손실 로직은 direct strategy 용도로는 즉시 폐기한다
- 다만 PnL 이 나쁜 로직이라도 execution / safety / telemetry donor 가치는 별도로 평가한다
- 인프라 안정성을 악화시키는 donor 는 PnL 개선이 있어도 reject 또는 donor-only 로 본다
- market order heavy logic 는 기본적으로 메인라인 execution doctrine에서 불채택한다
- baseline 테스트는 `$100 notional`, `3 iter` 로 시작하고 개선 시 `5 -> 10 -> 15 -> 20` iter 로 확장한다

Phase 2 — PnL Hardening / Optimization:

- strategy quality 개선
- maker ratio 개선
- spread / slippage / fee 정책 고도화
- venue-specific economics 최적화
- 최종 목표인 `PnL >= 0` 로 수렴
- `alternate` 는 현재 임시 baseline 전략으로만 두고, mean reversion / breakout 및 spread-based venue-role assignment 와 함께 후속 최적화 대상으로 유지한다

Phase 2 메모:

- 인프라가 안정화되면 `0.02%` 수준의 tighter baseline 도 현실적인 기준으로 볼 수 있다
- 일부 historical test 에서는 plus-PnL 사례도 있었으므로, tighter economics 는 장기 목표로 취급한다

Acceptance Criteria:

- donor extraction 이 infra phase 와 pnl phase 로 명시적으로 분리된다
- 초기 donor 평가는 PnL 하나로만 하지 않는다
- `-0.05%` / `-0.1%` / `PnL >= 0` 의 계층적 의미가 문서에 남는다
- infra 안정화 이후에만 tighter economics 기준을 적용한다

### 6. PnL > 0 Workstreams

목적:

- 위 3축을 실제 경제성 목표에 연결한다
- "가능할 것 같다" 수준을 넘어서, 실제로 `PnL >= 0` 를 만들기 위한 연구/개선 축을 고정한다

작업:

- 필수 workstream 정의
  - maker fill rate 개선
  - entry spread filter calibration
  - BookDepth-based slippage skip
  - fee tier / rebate utilization
  - trade cadence와 anti-sybil pattern 설계
  - accounting 정합성 (realized vs fee-inclusive)
- 각 workstream 별 go/no-go threshold 정의
  - 예: one-leg fill rate < X면 lane 중단
  - net pnl% 가 Y 이하이면 strategy 폐기

Acceptance Criteria:

- 각 개선축마다 measurable KPI가 존재한다
- `continue / pause / kill` 기준이 숫자로 적힌다
- "노답이면 거래소 버린다"가 실제 decision rule로 문서화된다
- 3축이 `PnL > 0` 이라는 단일 목표 아래 연결되어 보인다

## Risks

- 지나친 donor 혼합으로 branch가 다시 미궁화될 수 있음
- 1DEX subaccount가 문서상 가능해도 live에서 health / leverage / signer 문제가 남을 수 있음
- low-fee economics와 anti-sybil behavior가 충돌할 수 있음
- WS 기능을 많이 붙여도 position truth가 불안하면 오히려 위험해질 수 있음

## Test Coverage Requirements

- smoke: 3 cycles
- stability: 10 cycles
- baseline validation: 20+ cycles
- no-go gate:
  - one-leg fill with residual position
  - flat close failure
  - fee-inclusive pnl materially negative

## Deliverables

1. economics-updated decision docs
2. donor extraction matrix
3. benchmark harness checklist
4. target architecture memo
5. PnL>0 workstream / kill criteria
