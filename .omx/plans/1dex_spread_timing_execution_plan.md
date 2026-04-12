# 1DEX Spread/Timing Gate Execution Plan

작성일: 2026-04-12

## 목적

이 문서는 `spread / timing gate family`의 첫 실제 테스트 배치를
donor 코드 기준으로 바로 실행할 수 있게 적는 runbook 이다.

이 문서가 소유하는 것:

- 실행 순서
- 실제 커맨드
- 각 테스트의 목적
- 관찰할 로그/CSV 포인트
- 통과 / 실패 / 보류 판정 규칙

이 문서는 용어 정의를 소유하지 않는다.
용어 정의는 `1dex_glossary.md`,
배경 해석은 `1dex_execution_insights.md`,
상위 우선순위는 `1dex-plan.md`,
판정판은 `1dex_source_map.md`가 소유한다.

## 왜 이 테스트를 먼저 하나

지금 `1DEX`는 `POST_ONLY`, `IOC` 둘 다 baseline 후보에서 탈락했다.

하지만 그 다음 단계가
"새 entry mode 찾기"인지
"현재 BUILD gate semantics 다시 자르기"인지가 중요하다.

현재 donor 코드 독해 기준으로는 후자가 맞다.

핵심 이유:

1. `_check_spread_profitability()`는 이름과 달리 profitability filter보다 sanity check 성격이 강하다.
2. `_wait_for_optimal_entry()`는 threshold 미달이어도 timeout 후 진입을 허용한다.
3. `No BookDepth data`여도 sizing path가 target quantity로 계속 진행한다.

즉 지금 첫 실전 테스트의 목적은:

- `min-spread-bps` 숫자를 예쁘게 찾는 것

이 아니라

- 현재 gate가 실제로 무슨 역할을 하고 있는지 고정
- timeout entry가 얼마나 자주 release path로 작동하는지 확인
- `No BookDepth data`를 blocker로 올려야 하는지 확인

이다.

## 작업 디렉토리와 donor 기준점

donor 실행 기준:

- repo root:
  - `/Users/bot_trader/2dex`
- main target script:
  - `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py`

핵심 donor 코드 표면:

- `_check_spread_profitability()`
  - `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py:1851`
- `_wait_for_optimal_entry()`
  - `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py:2541`
- `calculate_order_size_with_slippage()` 의 `No BookDepth data`
  - `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py:910`
- `execute_build_cycle()`
  - `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py:3403`
- CLI args
  - `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py:4380`

## 고정 원칙

### 1. 이 배치는 BUILD gate 해석 배치다

이번 배치에서 바로 해결하려는 것은:

- spread/timing gate semantics
- timeout entry 해석
- BookDepth 부재 시 BUILD 차단 여부

이번 배치에서 바로 해결하지 않는 것:

- per-leg pricing-mode
- UNWIND contract 최종 마감
- economics tuning

### 2. 테스트는 한 줄씩 쪼갠다

이번 문서에서는 `spread / timing gate family`를 한 덩어리로 두지 않는다.

쪼개는 순서는 아래다.

1. `semantic split`
2. `timeout release`
3. `BookDepth blocker`
4. `policy lock`

### 3. 한 번에 여러 가설을 섞지 않는다

이번 배치에서는:

- `use_ioc`
- `per-leg pricing`
- `subaccount`
- `UNWIND fallback redesign`

같은 다른 축은 같이 섞지 않는다.

이유:

- 지금은 gate semantics를 먼저 자르는 단계이기 때문이다
- 여러 축을 같이 건드리면 실패 원인 분리가 다시 안 된다

## Preflight

### P0. read-only sanity

목적:

- 환경 문제와 gate 문제를 분리

확인:

- 키/계정/포지션 조회가 정상인지
- 이미 잔여 포지션이 없는지
- donor runtime 자체가 죽지 않는지

실행:

```bash
cd /Users/bot_trader/2dex
python hedge/DN_pair_eth_sol_nado.py --size 100 --iter 1 --env-file .env
```

주의:

- 이 커맨드는 실제 진입까지 갈 수 있으므로
  `P0`에서는 바로 실행하지 말고
  먼저 `.env`, 서브계정, 현재 포지션 상태를 사람 눈으로 확인한 뒤 진행한다.

실패 시 해석:

- 이 단계에서 막히면 spread/timing 실험으로 가지 않는다
- 환경/계정 문제를 먼저 분리한다

### P1. evidence snapshot

목적:

- 이번 배치의 로그와 CSV를 이전 러닝과 섞지 않기

실행:

```bash
cd /Users/bot_trader/2dex
mkdir -p logs/1dex-spread-timing-batch
cp logs/DN_pair_eth_sol_nado_log.txt logs/1dex-spread-timing-batch/pre_batch.log 2>/dev/null || true
cp logs/DN_pair_eth_sol_nado_trades.csv logs/1dex-spread-timing-batch/pre_batch.csv 2>/dev/null || true
```

## Test 1. Semantic Split

### 질문

`_check_spread_profitability()`는 지금

- profitability hard gate 인가
- sanity check 인가

### 왜 이 테스트를 하나

이 정의가 안 서면

- `min-spread-bps`를 올릴지 내릴지
- timeout을 줄일지 늘릴지

같은 조정이 전부 헛수고가 되기 때문이다.

### donor 근거

- 함수 주석은 "sanity check, NOT profitability filter"라고 적고 있다
- 그런데 상위 문맥에서는 여전히 `spread profitability`라는 이름과 skip path가 있다

### 실행

이 테스트는 첫 단계에서는 live 진입보다 로그/코드 정합성 확인이 중심이다.

관찰 대상:

- `_check_spread_profitability()` 주석과 반환값
- `execute_build_cycle()`에서 `[BUILD] SPREAD FILTER SKIP`
- CSV의 `cycle_skipped`, `skip_reason`

로그 grep:

```bash
cd /Users/bot_trader/2dex
rg -n "\\[BUILD\\] SPREAD FILTER SKIP|SPREAD CHECK PASS|cycle_skipped|skip_reason" logs/DN_pair_eth_sol_nado_log.txt logs/DN_pair_eth_sol_nado_trades.csv
```

판정:

- `profitability hard gate`로 읽지 않기:
  - donor 주석이 sanity check이고
  - 실전 skip reason도 "수익이 안 난다"보다 "entry를 열 수 없다"에 가까우면
- 이 테스트의 목표는 코드 수정이 아니라
  문서 기준을 "sanity/entry-release gate"로 잠그는 것이다

## Test 2. Timeout Release

### 질문

`_wait_for_optimal_entry()`는

- 실제로 optimal entry를 찾는가
- 아니면 timeout 후 entry release를 하는가

### 왜 이 테스트를 하나

현재 `IOC` 해석의 핵심 중 하나가
"좋아서 들어간 게 아니라 timeout 후 들어갔다"는 점이기 때문이다.

이게 맞다면

- timeout path는 alpha가 아니라 release semantics로 봐야 한다.

### donor 근거

- `_wait_for_optimal_entry()`는 threshold 미달이어도 timeout 후
  `reason: "timeout"`으로 반환한다
- `execute_build_cycle()`는 그 결과를 받아 BUILD를 계속 진행한다

### 실행 커맨드

기본 donor baseline 1-cycle:

```bash
cd /Users/bot_trader/2dex
python hedge/DN_pair_eth_sol_nado.py --size 100 --iter 1 --min-spread-bps 0 --env-file .env
```

관찰 로그:

```bash
cd /Users/bot_trader/2dex
rg -n "\\[ENTRY\\] Dynamic threshold|\\[ENTRY\\] Monitoring BBO|\\[ENTRY\\] Optimal spread detected|\\[ENTRY\\] Timeout after|\\[BUILD\\] Entry timing" logs/DN_pair_eth_sol_nado_log.txt
```

관찰 포인트:

- `reason=optimal_spread` 인지
- `reason=timeout` 인지
- `waited_seconds`
- `threshold_bps`
- `entry_spread_bps`

판정:

- `timeout`이 자주 뜨면:
  - 현재 timing layer는 optimality search가 아니라 release gate다
- `optimal_spread`가 안정적으로 뜨면:
  - timing layer는 부분적으로 의미가 있다
- 둘 중 어느 쪽이든
  - timeout과 optimal을 같은 의미로 취급하면 안 된다

## Test 3. BookDepth Blocker Candidate

### 질문

`No BookDepth data`는

- 단순 warning 인가
- BUILD blocker 인가

### 왜 이 테스트를 하나

지금 donor 코드에서 가장 큰 모순이 여기 있기 때문이다.

- `estimate_slippage()`는 handler 부재 시 `999999`
- 하지만 sizing path는 `No BookDepth data`여도 target quantity로 진행

즉 코드 표면상으로는 "중요"하다고 해놓고,
실행 경로상으로는 아직 blocker가 아니다.

### 실행 커맨드

동일한 1-cycle 커맨드를 쓰되, 이 테스트는 `ENTRY`보다 `SLIPPAGE` 로그가 중심이다.

```bash
cd /Users/bot_trader/2dex
python hedge/DN_pair_eth_sol_nado.py --size 100 --iter 1 --min-spread-bps 0 --env-file .env
```

관찰 로그:

```bash
cd /Users/bot_trader/2dex
rg -n "\\[SLIPPAGE\\]|No BookDepth data|SKIPPING TRADE|target quantity|BookDepth unavailable" logs/DN_pair_eth_sol_nado_log.txt
```

관찰 포인트:

- `No BookDepth data for ETH/SOL, using target quantity`
- 이후에도 BUILD가 계속 진행되는지
- 결국 no-fill / one-leg / skip 중 무엇으로 끝나는지

판정:

- `No BookDepth data`가 나왔는데도 BUILD가 계속 열리면:
  - 현재 baseline 에서 BookDepth는 blocker가 아니다
- 만약 이 상태가 반복되고
  paired fill / sizing authority가 계속 약하면:
  - 다음 policy에서 hard blocker 승격 후보로 올린다

## Test 4. Gate Policy Lock

### 질문

spread/timing/BookDepth 부재를
최종적으로 어떤 정책으로 잠글 것인가

### 선택지

1. `hard block`
   - 조건 미달이면 BUILD 금지
2. `soft advisory`
   - 경고만 찍고 BUILD 허용
3. `composite gate`
   - spread + timing + WS/BBO/BookDepth availability를 묶어서 release 결정

### 왜 이 테스트를 하나

이 결정을 안 하면

- 다음 실험도 똑같이
  - timeout entry
  - BookDepth 없음
  - weak authority
  가 섞인 채로 반복될 수 있다

### 현재 권고

잠정 권고는 `composite gate` 쪽이다.

이유:

- spread 하나로 BUILD를 자르는 건 너무 거칠고
- soft advisory만 두면 현재처럼 계속 entry가 열릴 수 있고
- 실제 blocker는 `spread`, `timing`, `BookDepth`, `BBO authority`가 함께 섞여 있기 때문이다

단, 이 권고는 아직 live 검증 전이므로
이번 배치의 실제 목적은 "composite gate를 채택"이 아니라
"왜 composite gate 후보가 되었는지 근거를 확보"하는 데 있다.

## 한 번에 실행할 최소 배치

가장 작은 실전형 배치는 아래다.

1. preflight snapshot
2. `--min-spread-bps 0`, `--iter 1` 기준 1-cycle 실행
3. `ENTRY` 로그 grep
4. `SLIPPAGE` 로그 grep
5. `cycle_skipped / skip_reason` 확인
6. 아래 3개 질문에 yes/no 판정

질문:

1. timeout entry가 실제로 발생했는가
2. `No BookDepth data` 상태에서 BUILD가 계속 진행됐는가
3. 현재 spread gate를 profitability filter라고 부르는 게 여전히 맞는가

이 3개 중 2개 이상이

- `yes`
- `yes`
- `no`

에 가깝다면,
이번 배치 결론은 아래다.

- `spread/timing family = semantics retune 필요`
- `BookDepth unavailable = blocker 승격 후보`
- 다음 우선순위는 `websocket-first BBO / BookDepth family`

## 이번 배치의 산출물

이번 테스트 배치가 끝나면 반드시 남겨야 할 산출물은 아래다.

1. 사용 커맨드 1줄
2. 실행 시각
3. 로그 파일 경로
4. `ENTRY` grep 발췌
5. `SLIPPAGE` grep 발췌
6. `cycle_skipped / skip_reason`
7. 최종 verdict:
   - `keep but retune`
   - `hard blocker candidate`
   - `compare-next`
   중 무엇인지

## 다음 분기

### A. timeout entry가 핵심이면

- `spread/timing`을 release semantics 중심으로 다시 고정
- 다음 배치에서 timeout path를 더 세밀하게 자름

### B. `No BookDepth data`가 핵심이면

- `websocket-first BBO / BookDepth family`로 즉시 넘어감
- BookDepth availability를 blocker 승격 후보로 다룸

### C. 둘 다 동시에 보이면

- `spread/timing`과 `websocket-first`를 분리하되
- 다음 배치 순서는:
  1. `spread/timing semantics lock`
  2. `BookDepth blocker promotion`
  로 둔다
