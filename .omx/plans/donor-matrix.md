# Donor Matrix Plan

작성일: 2026-03-20

## 목적

관련 히스토리 전체를 기능 축으로 재배열해서, 어떤 donor를 메인라인 후보로 살리고 무엇을 버릴지 결정하기 위한 **실행 가능한 donor matrix** 를 정의한다.

이 문서는 아직 코드 변경 계획이 아니라 **donor extraction plan** 이다.

## Functional Axes

모든 donor는 아래 6개 축으로 분해한다.

1. `market-data`
2. `execution`
3. `risk-flatness`
4. `subaccount-leverage`
5. `accounting-economics`
6. `ops-observability`

## Extraction Granularity Rules

donor는 commit 통째로 가져오지 않는다.

기본 추출 단위:

- 단일 함수
- 작은 클래스 메서드 묶음
- 하나의 상태기계 feature
- 하나의 WS handler
- 하나의 risk guard 또는 accounting unit

금지:

- "이 커밋이 좋아 보이니 전체를 베이스로 채택"
- 테스트 없이 donor 여러 개 동시 graft
- source behavior와 target contract를 분리하지 않은 채 이식

즉 donor matrix의 목적은 **좋은 커밋 선정**이 아니라 **함수/기능 단위 추출 순서 결정**이다.

## Known Family Landscape

현재까지 확보한 family 분류:

- `2DEX`
- `alternate`
- `mean-reversion-backpack`
- `mean-reversion-paradex`
- `nado-pair`
- `test-results`
- `historical-hardening`

## First-Pass Donor Candidates

아래는 full-history 조사 전에 이미 높은 우선순위로 봐야 하는 donor 후보군이다.

### A. Risk / Flatness Donors

후보:

- `b027c98` family
- `2DEX restoration` family
- `7e11a6f` family

이유:

- clean start
- hedge verification
- stale order cleanup
- post-fill verification
- close order / orphaned position / subaccount mismatch 수정

현재 판정:

- `keep` 우선

### B. Execution Donors

후보:

- `3c89fcd` family
- `2DEX restoration` family
- `853ef51` family

이유:

- POST_ONLY attempt + MARKET fallback
- active cancel-and-replace
- 실제 small live 체결 / 청산 성공
- Nado working baseline 존재

현재 판정:

- `adapt` 우선

### C. Market-Data / WS Donors

후보:

- `23d89e2` family
- `5bc077d` family
- `30471ab` family
- `61e825a` family
- `566bd89` / `481b57b` family
- `hedge_mode_grvt_v2` family

이유:

- Nado public WS client
- WebSocket BBO first
- BookDepth handler
- BBO routing
- GRVT WS RPC order path
- order book WS usage

현재 판정:

- `keep/adapt` 혼합

### D. Accounting / Economics Donors

후보:

- `4231674` family
- `aca55d2` family
- `8f0a6e8` family
- `Complete test summary / test-results` family

이유:

- spread filter
- fee-aware thresholds
- partial fill accounting
- fee-inclusive thinking
- target notional logic

현재 판정:

- `adapt`

### E. Subaccount / Leverage Donors

후보:

- `7e11a6f` family
- `Nado working baseline` family
- `ONEDEX subaccount research` docs

이유:

- subaccount address truth
- isolated_margin handling
- leverage verification paths
- linked signer / subaccount viability

현재 판정:

- `research-first`

### F. Ops / Observability Donors

후보:

- `b027c98` family
- `2DEX metrics/export` family
- `30471ab` enhanced csv logging

이유:

- CSV
- trade metrics
- phase tracking
- rebate tracker
- momentum/spread/liquidity logging

현재 판정:

- `keep`

## Explicit Reject Buckets

아래는 donor 가치가 낮거나, 그대로 메인라인에 들어가면 위험한 항목이다.

- `1DEX pair` 자체를 mainline 전략으로 채택
  - 이유: pair basis risk
- pure `REST-only BBO` 를 최종형으로 유지
  - 이유: stale risk
- market fallback-heavy execution을 기본 economics engine으로 채택
  - 이유: `PnL > 0` 달성 난도 상승
- "expected positive" 만 있는 economics commit을 profitable baseline으로 승격
  - 이유: 증빙 부족

## Extraction Order

### 1. Risk / Flatness first

목적:

- `PnL > 0` 이전에 포지션 완결성과 residual risk를 먼저 고정

산출:

- flatness donor shortlist
- donor-by-donor validation queue

### 2. Market-data / WS second

목적:

- Nado / GRVT / EdgeX donor에서 WS-first pricing과 BookDepth 계층을 흡수

산출:

- WS donor shortlist

### 3. Execution third

목적:

- active cancel-replace, maker attempt, taker fallback 논리를 donor로 재조합

산출:

- execution donor shortlist

### 4. Economics fourth

목적:

- fee-aware threshold, spread filter, pnl accounting donor를 마지막에 얹는다

산출:

- economics donor shortlist

### 5. Subaccount lane separately

목적:

- 1DEX + subaccount 는 mainline에 바로 합치지 않고 feasibility lane으로 분리

산출:

- subaccount go/no-go checklist

## Per-Donor Validation Loop

각 donor 후보는 아래 검증 루프를 통과해야 한다.

1. donor function/feature 식별
2. source artifact / source behavior 확인
3. target architecture contract 연결
4. 해당 donor의 테스트 정의
   - unit
   - regression
   - lane benchmark
5. 검증 통과 시에만 `keep/adapt`
6. 실패 시 `donor-only` 또는 `reject`

## Acceptance Criteria

- donor matrix가 기능 축 기준으로 정리된다
- 각 축마다 최소 1개 이상의 1차 donor family가 지정된다
- `keep / adapt / reject / research-first` 판단이 있다
- 메인라인과 donor-only가 분리된다
- donor별 테스트 요구사항과 promotion 조건이 존재한다
- 다음 단계인 mainline candidate comparison에 바로 넘길 수 있다

## Next

- `.omx/plans/mainline-candidate-comparison.md`
- `.omx/plans/target-architecture.md`
