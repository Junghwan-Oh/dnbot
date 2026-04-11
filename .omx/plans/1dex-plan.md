# 1DEX Plan

작성일: 2026-04-11

## 목적

이 문서는 `1DEX` 레인의 유일한 상위 진입점이다.

범위:

- `1DEX pair / Nado pair`
- `1DEX + subaccount`

비범위:

- `2DEX` 메인라인 논쟁
- DN 전체 donor universe
- cross-lane 통합 architecture

이 문서는 `1DEX`의 방향, 단계, 우선순위만 소유한다.
기술 규약, truth source, failure taxonomy, gate, kill criteria는
`1dex-operating-contract.md`가 소유한다.

canonical steering rule:

- `1dex-plan.md` 가 기준점이다
- `1dex_source_map.md` 는 기준점을 대체하지 않는다
- source map은 `keep / reject / extract / compare-next` 판정을 plan 에 공급하는 판정판이다

## Core 1DEX View

현재 `1DEX`의 첫 문제는 `PnL`이 아니다.
첫 문제는 `인프라 불안정성`이다.

특히 먼저 해결해야 하는 것은 아래다.

- position truth instability
- one-leg fill
- close/unwind incompleteness
- websocket/data path under-integration
- subaccount truth ambiguity

따라서 `1DEX`는 아래 순서로만 진행한다.

1. 흡수 단계
2. 최적화 단계
3. 그 다음 economics

## Phase 1. 흡수 단계

정의:

- 기존 `SOL-ETH Nado pair` 계열 구현을 버리지 않고 흡수
- 현재 있는 실험/구현/evidence를 기반으로 infra baseline을 세운다

핵심 목적:

- existing pair line의 운영 진실을 복구
- flatness / close / unwind / truth source를 먼저 고정
- Nado WS/BBO/BookDepth line을 donor가 아니라 실제 infra baseline 후보로 검증

이 단계의 주제:

- pair basis risk 논쟁이 아니다
- "기존 구현을 baseline 수준까지 안정화할 수 있느냐"다

### Phase 1 dashboard

- active items total: `4`
- completed / confirmed: `3 / 4`
- in progress: `1 / 4`
- remaining after current batch: `1`

세부:

1. `POST_ONLY verdict`
   - status: `complete`
   - result: `reject`

2. `source map`
   - status: `complete`
   - result:
     - build-stage `13 / 13` mapped
     - build candidate groups `7 / 7` mapped

3. `IOC comparison`
   - status: `complete`
   - result: `reject`

4. `UNWIND / one-leg baseline contract`
   - status: `in progress`
   - remaining:
     - current live entry mode 들이 다 탈락한 이후
     - 남은 build family 3개를 우선순위대로 재선정

현재 batch execution order:

1. `spread / timing gate family`
2. `websocket-first BBO / BookDepth family`
3. `per-leg pricing-mode family`
4. 그 다음 `UNWIND / one-leg baseline contract` 마감

필수 확인 항목:

- 3-cycle smoke에서 flat close
- 10-cycle stability에서 one-leg fill 재발 금지
- primary fill 후 hedge fail 시 deterministic unwind
- restart/reconnect 후 truth consistency
- WS connected 상태가 실제 execution-quality improvement로 이어지는지

### Phase 1 priority ordering

phase 1 은 `BUILD` 와 `UNWIND` 를 동등하게 보지 않는다.

우선순위는 아래다.

1. `UNWIND`
2. `BUILD one-leg / hedge-fail handling`
3. 그 다음 BUILD entry quality

이유:

- 실제 치명 사고는 `UNWIND 실패`, `잔여 포지션`, `flat 실패` 쪽에 더 많이 몰린다
- BUILD 도 중요하지만, 본체는 어디까지나 건강한 BUILD/UNWIND 흐름이어야 한다
- `forced liquidation` 은 메인 방법이 아니라, 본체가 무너졌을 때만 쓰는 2차 fallback 이어야 한다

### Forced liquidation / forced flatten family

지금 `1DEX`에서 봐야 할 강제 청산 장치는 적어도 아래 3개 family 다.

1. `partial / one-leg fill` 직후 강제 청산
2. `UNWIND 실패` 이후 강제 청산 또는 강제 flatten 경로
3. `BUILD 시작 전 잔여 포지션 존재` 시 진입 금지 + 필요 시 강제 flatten

참고:

- BUILD 단계에서도 hedge 가 안 되는 경우가 꽤 있었으므로, `BUILD` 역시 강제 청산 관점에서 봐야 한다
- 다만 priority 는 여전히 `UNWIND-first` 다
- 그리고 강제 청산은 `수수료 손실 확률이 매우 높은 긴급 장치` 이므로, baseline 이 여기에 의존하면 안 된다

### Build controller assumption

phase 1 의 BUILD 평가는 `alternate` cycle controller 를 전제로 한다.

- `BUY_FIRST`
  - ETH long / SOL short
- `SELL_FIRST`
  - ETH short / SOL long

즉 지금 비교하는 것은 controller 자체가 아니라, 같은 alternating 흐름 위에서 어떤 build family 가 살아남는지다.

이 단계에서 보는 lane:

- `1DEX pair / Nado pair`

현재 판정:

- donor/evidence value는 높다
- 아직 mainline-grade baseline은 아니다

### Current absorption baseline file

현재 authoritative pair baseline source는 아래다.

- `git ref: origin/DN_pair_eth_sol_nado`
- target file: `hedge/DN_pair_eth_sol_nado.py`

현재 읽기:

- ETH / SOL 양쪽이 모두 Nado 위에서 동작하는 pair 구조다
- phase 1 의 목표는 이 pair baseline 의 truth / flatness / close 경로를 먼저 고정하는 것이다

### 현재 pair baseline의 infra 사실

현재 active pair baseline 은 `ETH/SOL on Nado` 구조다.

지금 보이는 baseline 성격:

- active pair baseline 쪽은 `same-exchange pair` 구조를 목표로 한다
- pair baseline 구현은 simultaneous order / emergency unwind / verified flat close 방향을 이미 갖고 있다
- 다만 truth source 와 WS integration 기준은 아직 더 명시적으로 고정해야 한다

핵심 해석:

- phase 1 의 목적은 새 전략을 넣는 것이 아니라
- `ETH/SOL on Nado` baseline 의 truth / close / flatness contract 를 먼저 stable baseline 으로 바꾸는 것이다

### Current live judgment

실제 mainnet 1-cycle run 기준 현재 판단은 아래다.

- `POST_ONLY BUILD entry` 는 live baseline 후보로 보기 어렵다
- `min-spread-bps 0` 으로 내려도 실제 진입 체결이 안 났다
- 따라서 지금 이 라인에 더 미시적으로 매달려 최적화하는 것은 우선순위가 아니다

현재 verdict:

- `UNWIND / close / flatness` 관련 로직은 계속 추출 가치가 있다
- 하지만 현재 `POST_ONLY BUILD entry` 로직은 live baseline 으로는 `reject`
- `IOC BUILD entry` 도 real run 재현 기준 one-leg fill 이 반복되어 현재 live baseline 으로는 `reject`
- 즉 현재 `HEAD`의 두 entry mode 모두 baseline 에서 탈락했다
- 이 라인은 당분간 `infra lesson / unwind lesson` source 로 본다

### Next build candidate groups

현재 build family 총수는 `7개`고, 그중 verdict 가 끝난 것은 `4개`다.

- total build candidate groups: `7`
- verdict fixed: `4 / 7`
- remaining build families to classify: `3 / 7`

남은 `3개`는 아래다.

1. `spread / timing gate family`
   - 상태:
     - `keep but retune`
   - 의미:
     - 진입 수익성 필터이면서 동시에 front-line entry blocker
   - 현재 읽기:
     - hard profitability threshold 하나로 BUILD viability 를 대표시키면 안 된다
      - donor 코드상 `_check_spread_profitability()` 는 profitability filter 가 아니라 sanity check 로 적혀 있다
      - `_wait_for_optimal_entry()` 는 timeout 후에도 진입하므로, 현재는 "optimal entry"보다 "timeout-based entry release"에 가깝다

2. `websocket-first BBO / BookDepth family`
   - 상태:
     - `keep / compare-next`
   - 의미:
     - paired fill viability 와 sizing authority 를 올릴 가장 유력한 후보
   - 현재 읽기:
      - order mode 이전에 market-data authority 를 올리는 family 로 본다
      - donor 코드상 `No BookDepth data`일 때도 target quantity 로 진행하는 경로가 있어, WS 연결보다 BookDepth gate 강제 여부가 더 핵심이다

3. `per-leg pricing-mode family`
   - 상태:
      - `dormant / spec-only`
   - 의미:
      - `eth_mode / sol_mode` 를 다르게 주는 leg-level pricing family
   - 현재 읽기:
      - 바로 다음 후보라기보다는 WS/BBO/BookDepth 정리 뒤에 올릴 후보

### Current BUILD batch goal

이 batch 의 목표는 새 order mode를 더 찾는 것이 아니다.

- `spread`를 profitability gate가 아니라 sanity/entry-release gate로 다시 고정
- `timeout entry`를 optimality proof로 간주하지 않기
- `BookDepth unavailable` 상태에서는 BUILD가 그냥 진행되지 않도록 contract를 더 엄격히 고정

즉 이번 batch 는 `entry mode 비교`가 아니라
`BUILD gate / market-data authority / sizing authority`를 다시 자르는 작업이다.

### phase 1 에서 바로 흡수할 donor

`dn-build-unwind-fix` 에서 지금 맥락에 직접 필요한 donor는 아래다.

- primary fill 이후 hedge fail 을 cycle success 로 취급하지 않기
- unwind 판단은 intent 가 아니라 actual API position 기준으로 하기
- stop-only 로 끝내지 않고 flatten-required semantics 로 바꾸기
- residual accumulation 을 baseline failure 로 명시하기

즉 phase 1 의 close/unwind 목표는:

- `silent stop` 제거
- `false success` 제거
- `actual position-based unwind` 고정
- `forced liquidation / forced flatten` 은 2차 fallback 장치로만 고정

## Phase 2. 최적화 단계

정의:

- `1DEX + subaccount` 구조를 별도 최적화 레인으로 검증

핵심 목적:

- 같은 거래소 / 같은 자산 long-short 분리 구조로 진짜 hedge에 더 가까운 구조를 만든다
- pair 구조의 basis risk를 벗어난다

이 단계의 주제:

- subaccount feasibility
- subaccount truth
- health / leverage interpretation
- signer / account separation
- same-asset close path

필수 확인 항목:

- subaccount truth confirmed
- health / leverage 변화 해석 가능
- same-asset long/short close path validated
- UI 체감값과 bot truth가 충돌하지 않음

이 단계에서 보는 lane:

- `1DEX + subaccount`

현재 판정:

- 가장 자연스러운 mainline-target
- 아직 idea + feasibility lane
- baseline으로는 미확정

### 현재까지 확정된 Nado / subaccount hard facts

`ONEDEX_SUBACCOUNT_DECISION.md` 에서 지금 맥락에 직접 필요한 사실만 옮긴다.

#### 구조 사실

- `1DEX pair` 는 엄밀한 delta-neutral 이 아니라 correlation-based pair trade 에 가깝다
- `1DEX + subaccount` 는 같은 거래소 / 같은 API / 같은 자산 long-short 분리 구조라서 true hedge 에 더 가깝다
- 따라서 phase 2 의 목적은 단순 아이디어 검토가 아니라, pair basis risk 를 줄이는 구조적 전환 가능성 검증이다

#### Nado subaccount / leverage / health 사실

- subaccount 는 지갑 주소 아래 여러 개 둘 수 있다
- front 제한과 API 사용 가능 범위는 다를 수 있다
- health 는 risk engine 기준으로 동적으로 변한다
- isolated / unified margin 해석이 섞이면 표시 leverage / health 변화가 체감상 크게 보일 수 있다
- linked signer 를 써서 메인 지갑과 거래 전용 키를 분리할 수 있다

핵심 해석:

- `isolated 5x` 를 줬는데 상태가 계속 변하는 것이 곧바로 bug 라는 뜻은 아니다
- 하지만 bot 관점에서는 이 동적 health / subaccount 구조가 `position truth` 와 `risk truth` 를 더 어렵게 만든다

#### Points / sybil 리스크 사실

- points 는 raw volume 만이 아니라 genuine participation 기준이 들어간다
- wash trading / self-matching / non-organic behavior 해석 리스크가 있다

핵심 해석:

- subaccount 최적화가 가능하더라도, 지나치게 기계적인 왕복 구조는 운영 정책 리스크를 같이 본다

### Phase 2 open feasibility questions

현재 `1DEX + subaccount` 에서 아직 답이 필요한 질문:

1. subaccount state 를 bot 이 어떤 API / object 를 통해 authoritative 하게 읽을 수 있는가
2. health / leverage / margin mode 변화 중 무엇이 정상 동적 상태이고, 무엇이 incident 인가
3. linked signer / main wallet / trading key ownership 을 어떤 구조로 분리할 것인가
4. same-asset long/short 를 분리했을 때 close path 와 residual detection 은 어떻게 검증할 것인가
5. UI 표시값과 bot 내부 truth 가 어긋날 때 무엇을 authoritative truth 로 둘 것인가
6. subaccount 구조가 wash / sybil 해석 리스크를 높이지 않는 운영 cadence 를 만들 수 있는가

### Gate C 를 열기 위한 남은 질문

1. authoritative `subaccount truth` source 는 정확히 어느 API / SDK field 인가
2. health / leverage truth 를 bot 이 어떤 checkpoint 에서 재검증할 것인가
3. same-asset long/short aggregate exposure 는 어디서 계산할 것인가
4. one side success / other side fail 시 어떤 flatten path 를 탈 것인가
5. linked signer / wallet / trading key ownership 을 어떤 형식으로 고정할 것인가
6. subaccount mode 가 운영 정책상 wash / sybil 로 오해받지 않도록 cadence 를 어떻게 제한할 것인가

## Phase 3. Economics

이 단계는 앞의 두 인프라 단계가 통과한 뒤에만 시작한다.

확인 항목:

- fee-inclusive pnl
- maker ratio
- fallback-to-taker ratio
- cycle economics
- volume scaling
- anti-sybil-safe cadence

원칙:

- flatness가 약하면 economics 논의 승격 금지
- truth source가 흔들리면 volume 논의 금지

### 현재까지 확정된 fee/economics hard facts

`ONEDEX_SUBACCOUNT_DECISION.md` 에서 economics 판단에 직접 필요한 사실만 옮긴다.

- Nado base taker 는 대략 `3.5 bps` 수준으로 읽힌다
- 상위 tier taker 는 더 낮아질 수 있다
- base maker 는 대략 `1 bp` 수준으로 읽힌다
- 상위 tier 에서는 maker rebate 가능성이 있다

핵심 해석:

- `$1M/day` 에 `$200` 비용은 `2 bps` 수준이라서 pure taker 구조로는 매우 빡빡하다
- 따라서 `PnL >= 0` 를 노리려면 maker 성격, spread/slippage 통제, 구조적 execution quality 가 필수다
- 즉 economics 는 phase 1, phase 2 를 통과한 뒤에만 의미 있는 비교가 된다

## Current Lane Definition

### Lane A. `1DEX pair / Nado pair`

역할:

- 흡수 단계의 대상
- 현재 가장 풍부한 구현/evidence lane

얻을 것:

- WS BBO / BookDepth / liquidity-aware logic
- pair execution flow
- close/unwind evidence

한계:

- pair basis risk
- 10-cycle 확장 안정성 부족

### Lane B. `1DEX + subaccount`

역할:

- 최적화 단계의 대상
- 장기 target 구조

얻을 것:

- 같은 거래소
- 같은 자산
- 구조적으로 더 단순한 true hedge

한계:

- subaccount truth ambiguity
- health/leverage ambiguity
- close path 미확정

## Immediate Backlog

1. 현재 pair 라인에서 `UNWIND / close / flatness` 추출 포인트만 남기고 `POST_ONLY BUILD entry` 는 live baseline 에서 내리기
2. `BBO / fill / position / close` truth map 을 그룹 단위로 고정
3. `dn-build-unwind-fix` donor 를 absorption close/unwind hardening checklist 로 재연결
4. `Nado` 의 현재 WS under-integration gap 을 audit 하고, 어떤 stream 이 실제 decision/truth path 에 들어와야 하는지 고정
5. `3-cycle smoke` 와 `10-cycle stability` 를 1DEX infra gate 로 고정
6. `1DEX + subaccount` feasibility questions 를 pair absorption 과 분리해서 잠금
7. `subaccount truth model` 초안 작성
8. economics / volume workstream 은 Gate A/B, 필요 시 Gate C 전까지 보류

## Test Ladder

현재 `1DEX` 테스트는 아래처럼 읽는다.

### Stage 2

- 상태: `legacy / archive candidate`
- 이유:
  - single-order 인터페이스를 전제로 한다
  - 현재 pair baseline 의 simultaneous BUILD/UNWIND 구조와 어긋난다
  - 현재 본체 검증보다 과거 중간 단계 검증에 가깝다

### Stage 3

- 상태: `canonical smoke`
- 목적:
  - simultaneous order placement
  - partial fill / one-leg handling
  - position delta / flatness 기본 검증
- 현재 isolated smoke 결과:
  - `3 passed`

### Stage 4

- 상태: `cycle behavior verification`
- 목적:
  - alternating cycle
  - 반복 실행 흐름
  - CSV / history / iteration behavior
- 현재 isolated verification 결과:
  - core cycle tests pass
  - CSV header expectation 1건은 legacy test drift 성격이 강함

### Phase 1 immediate execution order

1. current local truth path 를 file-backed truth map 으로 고정
2. `dn-build-unwind-fix` donor 를 `close/unwind hardening checklist` 로 재작성
3. `Nado` WS gap 을 mandatory / strongly recommended stream 으로 분리
4. Gate A / Gate B 를 문서상 고정
5. 그 다음에만 subaccount feasibility 로 넘어간다

### Phase 1 mandatory WS gap closure

mandatory:

- `best_bid_offer`
- `order_update`
- `position_change`

strongly recommended:

- `fill`
- `book_depth`

## Source Documents

이 계획은 아래 문서에서 1DEX relevant 부분만 흡수해서 만든다.

- `docs/DEX_WS_INTEGRATION_STATUS_AND_PLAN.md`
- `docs/PROJECT_TEST_AND_ISSUES_REPORT.md`
- `.omx/plans/mainline-candidate-comparison.md`
- `.omx/plans/target-architecture.md`
- `.omx/plans/pnl-positive-kill-criteria.md`
- `../.omc/plans/dn-build-unwind-fix.md`

## Reading Order

1. 이 문서
2. `1dex-operating-contract.md`
3. `1dex_execution_insights.md`
4. 필요할 때만 source documents

## Final Stance

지금 `1DEX`는 `PnL` 문맥으로 읽으면 안 된다.

지금 `1DEX`는:

- Phase 1: `SOL-ETH Nado pair` 흡수 및 안정화
- Phase 2: `1DEX + subaccount` 최적화 및 feasibility 확인
- Phase 3: 그 다음 economics

이 순서로만 다뤄야 한다.

## Glossary

- 용어 정리는 별도 파일:
  - `.omx/plans/1dex_glossary.md`

## Source Map

- source map 별도 파일:
  - `.omx/plans/1dex_source_map.md`

## Execution Insights

- order / WS / REST 실전 해석 파일:
  - `.omx/plans/1dex_execution_insights.md`

## Lessons

- steering lesson 별도 파일:
  - `.omx/plans/1dex-lessons.md`
