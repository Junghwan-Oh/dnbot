# 1DEX WebSocket Execution Plan

작성일: 2026-04-12

## 목적

이 문서는 `websocket-first BBO / BookDepth family`를
하나의 source map 항목으로 실제로 닫기 위한 실전 런북이다.

이 문서가 소유하는 것:

- WS transport 체크
- stream별 truth ownership
- 3-cycle / 10-cycle 실전 실행 순서
- 합격 / 실패 기준

이 문서는 용어 사전을 소유하지 않는다.
용어는 `1dex_glossary.md`,
상위 우선순위는 `1dex-plan.md`,
판정은 `1dex_source_map.md`,
배경 해석은 `1dex_execution_insights.md`가 소유한다.

## 현재 결론 요약

지금까지 확인된 사실:

1. `best_bid_offer` 수신됨
2. `book_depth` 수신됨
3. `fill` 수신됨
4. `position_change.amount`는 봇 수량 단위와 바로 맞지 않음

즉:

- `WS transport` 자체는 살아 있다
- 문제는 `position_change`를 수량 truth로 쓴 것
- 그래서 현재 방향은:
  - `fill` = 수량 truth
  - `position_change` = raw signal / 진단용

## 현재 로컬 2DEX 코드 상태

현재 로컬에서 바뀐 파일:

- `/Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py`
- `/Users/bot_trader/2dex/hedge/exchanges/nado.py`

현재 반영된 것:

1. `sortedcontainers` 설치 후 WS 활성화
2. `fill` stream 구독 추가
3. `position_change`는 진단용 raw signal로 격하
4. `fill` 기준 `_ws_positions` 추적 시작
5. order result ↔ fill WS handoff bridge 추가
6. retry 판단에 `WS fill` 반영
7. close 검증에서 REST는 bounded fallback으로만 사용

현재 아직 남은 것:

- `phase handoff` 완전 안정화
- `3-cycle` 반복 확인
- `10-cycle` 반복 확인

## 실전 전제

이 런북은 아래 전제가 충족돼야 한다.

1. actual position `ETH=0`, `SOL=0`
2. `NADO_PRIVATE_KEY`, `NADO_MODE`, `NADO_SUBACCOUNT_NAME` 준비
3. `sortedcontainers` 설치 완료
4. `/tmp/1dex-dn-pair-venv/bin/python` 사용 가능

## 공통 실행 환경

```bash
set -a
source /Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/.env
set +a
export NADO_PRIVATE_KEY="$EVM_wallet_Private_Key"
export NADO_MODE=MAINNET
export NADO_SUBACCOUNT_NAME=default
export PYTHONPATH="/Users/bot_trader/2dex:/Users/bot_trader/2dex/perp-dex-tools-original"
```

## Step 0. read-only preflight

목적:

- 실전 전에 actual flat 상태 확인

실행:

```bash
/tmp/1dex-dn-pair-venv/bin/python /Users/bot_trader/2dex/test_nado_connection.py
```

합격:

- ETH position `0`
- SOL position `0`

실패:

- 잔여 포지션 있으면 실전 중단
- 먼저 정리

## Step 1. WS transport check

목적:

- WS가 진짜 살아 있는지 확인

확인 포인트:

- `WEBSOCKET_AVAILABLE: True`
- `ETH: CONNECTED`
- `SOL: CONNECTED`
- `Subscribed to ETH fill stream`
- `Subscribed to SOL fill stream`
- `Subscribed to ETH position_change stream`
- `Subscribed to SOL position_change stream`

실패:

- 이 단계 실패 시 이후 실전은 의미 없다
- 먼저 transport 복구

## Step 2. 1-cycle smoke

실행:

```bash
/tmp/1dex-dn-pair-venv/bin/python /Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py \
  --size 100 \
  --iter 1 \
  --min-spread-bps 0 \
  --env-file /Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/.env
```

관찰 포인트:

1. `FILL WS`가 찍히는가
2. `position_change`는 raw diagnostic으로만 남는가
3. 한 leg가 fill된 뒤 같은 leg를 다시 주문하지 않는가
4. close 검증에서 false residual stop이 줄었는가
5. actual position `0/0`로 끝나는가

합격:

- `FILL WS` 확인
- `BookDepth` usable
- actual position `0/0`
- false residual stop 없음

부분 합격:

- `FILL WS`는 잘 오고
- actual position도 `0/0`이지만
- phase handoff 때문에 false stop 또는 과잉 retry가 남아 있음

실패:

- transport 실패
- `fill` 안 옴
- actual position non-flat

## Step 3. 3-cycle smoke

실행:

```bash
/tmp/1dex-dn-pair-venv/bin/python /Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py \
  --size 100 \
  --iter 3 \
  --min-spread-bps 0 \
  --env-file /Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/.env
```

목적:

- 단발 개선인지 확인하지 않고
- 최소 반복 런에서 WS path가 버티는지 확인

합격:

- 3 cycle 모두 actual position `0/0`
- false residual stop 없음
- 같은 leg 중복재주문 없음
- `No BookDepth data` 없음

실패:

- cycle 하나라도 non-flat
- false residual stop 반복
- stale WS 때문에 종료 실패

## Step 4. 10-cycle stability

실행:

```bash
/tmp/1dex-dn-pair-venv/bin/python /Users/bot_trader/2dex/hedge/DN_pair_eth_sol_nado.py \
  --size 100 \
  --iter 10 \
  --min-spread-bps 0 \
  --env-file /Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/.env
```

목적:

- source map 항목을 실제로 닫을 수 있는지 확인

합격:

- actual position `0/0` 반복 유지
- false residual stop 없음
- one-leg 발생 시에도 phase handoff가 안정적
- REST fallback 과호출 징후 없음

실패:

- repeated false residual stop
- same-leg duplicate retry
- final actual position non-flat

## 완료 판정

`websocket-first BBO / BookDepth family`를 source map에서 하나의 완료된 항목으로 올리려면:

1. WS transport 안정
2. `fill`을 수량 truth로 고정
3. `position_change`는 진단용으로만 유지
4. 3-cycle smoke 통과
5. 10-cycle stability 통과

이 다섯 개가 필요하다.

그 전까지는:

- `keep`
- `in progress`
- `actual improvement confirmed, not closed`

정도로 읽는 것이 맞다.
