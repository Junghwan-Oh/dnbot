# 1DEX Glossary

작성일: 2026-04-11

## 목적

이 문서는 `1DEX` 문맥에서 반복 등장하는 용어를 빠르게 읽고 맞추기 위한 glossary 다.

포함 원칙:

- 단순 사전 정의가 아니라
- 우리 대화 / 현재 1DEX 문맥에서 어떤 의미로 쓰는지까지 같이 적는다

## POST_ONLY

- 의미:
  - 즉시 체결되지 않도록 resting order 로 올리려는 주문 방식
  - 보통 maker 쪽을 노리는 limit-style 주문으로 이해하면 된다
- 왜 헷갈리나:
  - 이름만 보면 "수수료가 싸고 예쁜 주문 방식"처럼 보이기 쉽다
  - 하지만 실제 문제는 비용이 아니라 "체결이 되느냐"다
- 우리 문맥:
  - entry fee 를 줄이기 위해 BUILD 에서 썼다
  - 하지만 real run 에서는 `min-spread-bps 0`이어도 no fill 이 나왔다
  - 그래서 현재 live baseline entry 로는 부적합하다고 본다
- 실전에서 이렇게 읽는다:
  - `POST_ONLY`는 "더 고급" 주문이 아니라
  - "체결이 안 나와도 괜찮을 때만 쓸 수 있는" 주문이다
  - 지금 `1DEX`에서는 BUILD 자체가 시작되어야 하므로, no fill이 반복되면 장점보다 단점이 더 크다

## IOC

- 의미:
  - Immediate-Or-Cancel
  - 즉시 체결 가능한 만큼만 체결하고, 나머지는 취소하는 주문 방식
- 왜 헷갈리나:
  - `POST_ONLY`가 안 되면 그다음 정답처럼 보이기 쉽다
  - 하지만 "체결이 더 잘 된다"와 "pair가 건강하게 체결된다"는 다른 문제다
- 우리 문맥:
  - POST_ONLY 보다 체결 가능성은 높고, 수수료는 더 불리할 수 있다
  - 하지만 현재 live run 에서는 BUILD 단계에서 one-leg fill 을 반복적으로 만들었다
  - 그래서 현재 baseline entry mode 로는 `reject` 상태다
- 실전에서 이렇게 읽는다:
  - `IOC`는 진입을 억지로 만들 수는 있어도
  - paired fill 을 보장하지는 않는다
  - 그래서 지금 `1DEX`에서는 "POST_ONLY 다음 후보"가 아니라 "이미 탈락한 현재 entry mode"로 읽는다

## maker-first

- 의미:
  - 수수료와 rebate 를 우선해서 resting order 중심으로 진입하려는 주문 성향
- 우리 문맥:
  - 현재 `POST_ONLY maker-first family` 를 가리킨다
  - 비용 구조는 예쁘지만, live 에서 fill 이 안 나오면 baseline 탈락이다

## taker-immediate

- 의미:
  - 즉시 체결 가능성을 높이기 위해 공격적으로 진입하는 주문 성향
- 우리 문맥:
  - 현재 `IOC taker-immediate family` 를 가리킨다
  - 진입성은 올라가지만 paired fill 을 못 만들면 baseline 탈락이다

## fallback

- 의미:
  - 본체 로직이 원하는 경로로 안 될 때 쓰는 대체 경로
- 우리 문맥:
  - POST_ONLY 실패 후 IOC 로 넘어가는 것
  - forced liquidation / forced flatten 같이 긴급 safety path 로 넘어가는 것

## BBO

- 의미:
  - Best Bid / Best Offer
  - 최우선 매수호가 / 매도호가
- 우리 문맥:
  - 진입 가격 판단의 최전선 데이터
  - BBO truth 가 틀리면 BUILD 가격 판단이 틀어진다

## spread filter

- 의미:
  - 현재 spread 가 기준 이상일 때만 진입하게 막는 필터
- 왜 헷갈리나:
  - 이름만 보면 수익성 계산기처럼 들린다
  - 하지만 donor 코드 주석상 현재 구현은 strict profitability filter 보다 sanity check 성격이 더 강하다
- 우리 문맥:
  - 원래는 PnL 보호용
  - 지금은 진입 자체를 막는지 검증 대상
  - `6bps` 에서는 skip, `0bps` 에서는 BUILD 시도까지 갔음
- 실전에서 이렇게 읽는다:
  - 지금 논점은 "몇 bps가 맞는가"보다
  - "이 필터가 수익성 판단인지, 진입 허가 장치인지"를 다시 구분하는 것이다

## profitability filter

- 의미:
  - "이 진입이 경제적으로 말이 되느냐"를 먼저 거르는 필터
- 왜 헷갈리나:
  - spread filter와 자주 섞여 쓰이기 때문이다
- 우리 문맥:
  - 현재 donor 구현의 `_check_spread_profitability()` 이름은 profitability filter처럼 보이지만
  - 실제 주석은 sanity check에 더 가깝다
- 실전에서 이렇게 읽는다:
  - profitability filter라면
  - "수익이 안 나면 진입 금지"가 중심이어야 한다
  - 지금 `1DEX`는 아직 그 단계가 아니라 infra/build viability 단계다

## sanity check

- 의미:
  - 수익 계산을 하려는 것이 아니라
  - 데이터가 너무 이상하거나 잘못된 상태는 아닌지 보는 최소 정상성 체크
- 왜 헷갈리나:
  - 이름이 약해 보여서 중요하지 않은 보조 로직처럼 느껴질 수 있다
  - 하지만 잘못된 sanity check는 BUILD를 쓸데없이 막거나 반대로 이상한 진입을 열 수 있다
- 우리 문맥:
  - `_check_spread_profitability()`는 현재 profitability hard gate보다 sanity check로 읽는 편이 맞다
- 실전에서 이렇게 읽는다:
  - "이 조건이면 돈 버나?"가 아니라
  - "이 정도면 진입 로직을 돌려도 될 만큼 데이터가 정상인가?"를 본다

## entry-release gate

- 의미:
  - 진입을 완전히 결정하는 엔진이 아니라
  - 지금 entry를 풀어줄지 보류할지를 결정하는 허가 장치
- 왜 헷갈리나:
  - filter, timing, optimal entry 같은 이름과 섞이면
  - 마치 이 gate가 곧 수익성 증명인 것처럼 보이기 쉽다
- 우리 문맥:
  - `spread`, `timing`, `BookDepth availability`는 지금 모두 BUILD release gate 후보들이다
- 실전에서 이렇게 읽는다:
  - "좋은 진입"을 찾는 게 아니라
  - "지금 들어가면 안 되는 상황을 막는가"를 먼저 본다

## spread / timing gate family

- 의미:
  - spread threshold 와 entry timing wait 를 함께 묶은 BUILD front gate family
- 왜 헷갈리나:
  - spread는 economics처럼 들리고
  - timing은 alpha처럼 들리고
  - 실제 코드는 timeout 진입도 허용해서 둘이 서로 다른 역할이 섞여 있다
- 우리 문맥:
  - `_wait_for_optimal_entry()`
  - `_check_spread_profitability()`
  - `--min-spread-bps`
  를 같이 본다
  - 원래는 economics gate 였지만, 지금은 entry viability gate 역할까지 같이 하고 있다
- 실전에서 이렇게 읽는다:
  - 이 family의 질문은 "몇 bps가 맞나?" 하나가 아니다
  - 실제 질문은:
  - 이 gate가 hard block 인가
  - soft advisory 인가
  - timeout 후 진입을 허용하는 release 장치인가
  - WS/BBO/BookDepth를 같이 보는 composite gate인가

## timeout entry

- 의미:
  - 일정 시간 동안 더 좋은 entry를 기다리다가
  - 끝까지 안 오면 현재 조건으로 그냥 들어가는 것
- 왜 헷갈리나:
  - 함수 이름에 `optimal`이 들어가면
  - timeout 후 진입도 마치 "좋은 타이밍을 찾은 결과"처럼 읽히기 쉽다
- 우리 문맥:
  - 현재 donor 코드는 threshold 미달이어도 timeout 후 진입을 허용한다
- 실전에서 이렇게 읽는다:
  - timeout entry는 optimality proof가 아니다
  - 오히려 gate가 끝내 entry를 풀어버렸다는 뜻에 가깝다

## websocket-first BBO / BookDepth family

- 의미:
  - REST 조회보다 WebSocket 기반 BBO / BookDepth 를 먼저 권위 경로로 삼는 build family
- 왜 헷갈리나:
  - "WS를 붙였다"와 "WS가 진짜 판단 기준이 됐다"는 전혀 다른 문제다
  - BBO와 BookDepth도 이름은 같이 나오지만 역할이 다르다
- 우리 문맥:
  - order mode 비교보다 먼저 봐야 할 next candidate family
  - 목적은 더 좋은 주문 타입 찾기가 아니라
  - build 직전 paired fill viability 와 sizing authority 를 올리는 것이다
- 실전에서 이렇게 읽는다:
  - BBO는 "지금 앞 가격이 얼마냐"
  - BookDepth는 "그 가격 근처에 실제로 얼마나 두께가 있느냐"
  - 즉, 이 family는 quote 보기용이 아니라
  - BUILD를 실제 체결 가능성 기준으로 다시 자르는 family다

## paired fill viability

- 의미:
  - ETH와 SOL 두 다리가 함께 건강하게 체결될 가능성
- 왜 헷갈리나:
  - 한쪽 체결만 보고 "진입은 됐다"고 착각하기 쉽기 때문이다
- 우리 문맥:
  - `IOC`는 진입성은 있었지만 paired fill viability가 낮아서 탈락했다
- 실전에서 이렇게 읽는다:
  - 한 leg만 들어가면 성공이 아니라
  - BUILD가 스스로 one-leg risk를 만든 것이다

## market-data authority

- 의미:
  - 봇이 "지금 시장 상태를 뭘 기준으로 믿을지" 정하는 권위 경로
- 왜 헷갈리나:
  - BBO, BookDepth, REST fallback, cached state가 한꺼번에 섞여 있기 때문이다
- 우리 문맥:
  - 현재 쟁점은 order mode보다 먼저 market-data authority를 WS 쪽으로 올릴 수 있느냐다
- 실전에서 이렇게 읽는다:
  - 단순히 데이터가 있느냐가 아니라
  - BUILD release와 sizing 판단에 실제로 쓰이느냐를 본다

## sizing authority

- 의미:
  - 주문 수량을 얼마로 둘지 결정할 때 무엇을 믿을지에 대한 기준
- 왜 헷갈리나:
  - notional 계산, min size, slippage, BookDepth가 서로 다른 층인데 한 함수 안에서 섞일 수 있다
- 우리 문맥:
  - `calculate_order_size_with_slippage()`는 sizing authority의 핵심 표면이다
- 실전에서 이렇게 읽는다:
  - 수량이 "계산되었다"보다
  - "그 수량이 실제 depth 기준으로 감당 가능한가"가 더 중요하다

## build blocker

- 의미:
  - BUILD를 진행하면 안 될 정도로 핵심적인 장애 상태
- 왜 헷갈리나:
  - warning과 blocker를 로그에서 비슷하게 처리하면
  - 사람은 위험을 봤다고 생각하지만 로직은 계속 진행할 수 있다
- 우리 문맥:
  - `No BookDepth data`를 단순 warning으로 둘지
  - hard blocker로 승격할지가 현재 쟁점이다
- 실전에서 이렇게 읽는다:
  - blocker는 "나중에 고치자"가 아니라
  - "이 상태면 BUILD를 열면 안 된다"는 뜻이다

## per-leg pricing-mode family

- 의미:
  - ETH 와 SOL 두 leg 를 같은 가격모드로 묶지 않고 leg 별로 다르게 주는 family
- 왜 헷갈리나:
  - 지금 문제의 중심처럼 보일 수 있지만
  - 실제로는 WS/BBO/BookDepth 정리 전에는 아직 spec-only에 가깝다
- 우리 문맥:
  - `eth_mode`, `sol_mode`
  - `bbo_minus_1`, `bbo_plus_1`, `bbo`, `aggressive`, `market`
  를 조합하는 설계 family 다
  - 아직은 `dormant / spec-only` 상태다
- 실전에서 이렇게 읽는다:
  - 이 family는 "주문을 얼마나 공격적으로 둘지"의 미세조정 층이다
  - 지금은 gate/data authority가 먼저고, pricing-mode는 그 다음이다

## flatness

- 의미:
  - 포지션이 0 또는 허용 오차 이내인 상태
- 우리 문맥:
  - `UNWIND 성공`의 최종 기준
  - stop 여부보다 flatness 가 더 중요하다

## baseline

- 의미:
  - 현재 시점에서 비교의 기준으로 삼는 기본 구현 / 기본 경로
- 우리 문맥:
  - 아무 예쁜 아이디어가 아니라
  - 실제로 계속 돌려보며 다음 후보들을 비교할 기준선
  - 체결이 안 되거나 forced liquidation 에 자주 의존하면 baseline 탈락

## truth

- 의미:
  - 봇이 실제 상태라고 믿는 최종 기준
- 우리 문맥:
  - 예쁘거나 빨라 보이는 데이터가 아니라
  - 의사결정에 권한을 주는 상태값

## truth source

- 의미:
  - 봇이 무엇을 실제 상태로 믿는지, 그 출처
- 우리 문맥:
  - BBO truth
  - fill truth
  - position truth
  - close truth

## BBO truth

- 의미:
  - 현재 얼마에 사고팔 수 있는지를 봇이 믿는 기준
- 우리 문맥:
  - ETH / SOL 양 다리의 `fetch_bbo_prices(...)` 경로

## fill truth

- 의미:
  - 주문이 실제로 체결됐는지, 얼마나 체결됐는지에 대한 기준
- 우리 문맥:
  - `accepted` 와 `actually filled` 를 구분해야 한다
  - 지금 pair baseline 에서는 POST_ONLY `OPEN` 처리 규칙이 핵심 약점이다

## position truth

- 의미:
  - 현재 실제 보유 포지션이 얼마인지에 대한 기준
- 우리 문맥:
  - `get_account_positions()` 기반 recheck 가 authority 다

## close truth

- 의미:
  - 포지션이 진짜 닫혔는지에 대한 기준
- 우리 문맥:
  - 주문 성공이 아니라
  - ETH / SOL 양 다리 모두 verified flat 인지로 본다

## plane

- 의미:
  - 책임 층 / 역할 층
- 우리 문맥:
  - Market Data Plane
  - Execution Plane
  - Position Truth Plane
  - Risk / Flatness Plane

## source map

- 의미:
  - 어떤 truth 를 어디서 읽고, 어디서 검증하는지 적는 지도
- 우리 문맥:
  - 미시 최적화용 목록이 아니다
  - 현재 로직을 비교해서 `keep / reject / extract` 를 빠르게 결정하는 도구다

## compare-next

- 의미:
  - 지금 당장 baseline 후보로 승격하지는 않지만
  - 다음 비교/검토 순서로 올려둔 후보
- 왜 헷갈리나:
  - keep과 비슷해 보이지만 의미가 다르다
- 우리 문맥:
  - `websocket-first BBO / BookDepth family`는 keep이면서 compare-next다
  - 즉 버리지는 않지만 아직 baseline으로 확정한 것도 아니다
- 실전에서 이렇게 읽는다:
  - "다음 타자"이지 "이미 통과한 후보"가 아니다

## data gap snapshot

- 의미:
  - 현재 구현에서 데이터 경로가 어디서 비거나 약한지 요약한 스냅샷
- 우리 문맥:
  - 연결은 됐는데 decision path 에 안 들어간 경우
  - query path 의존이 과한 경우
  - close truth 가 약한 경우를 잡는 데 쓴다

## connected

- 의미:
  - 채널이 붙어 있다는 뜻
- 우리 문맥:
  - WS connected 로그가 있다고 해서 충분하지 않다

## tmux leader precondition

- 의미:
  - `omx team` 류 런타임이 현재 세션을 tmux 안의 leader pane 으로 보고 시작한다는 전제
- 우리 문맥:
  - tmux 바깥에서 실행하면 worker pane startup / readiness 가 꼬일 수 있다
  - dirty workspace blocker 를 고쳐도, tmux leader 조건이 안 맞으면 team runtime 이 실패할 수 있다
- 자동화 가능성:
  - 일부는 가능하다
  - 예: preflight 에서 `$TMUX` 확인, 자동 안내, runtime state ignore
  - 하지만 `AGENTS.md` 에 글을 적는 것만으로 런타임 자체가 자동 수정되지는 않는다
  - 실제론 wrapper / preflight / config 쪽 보강이 필요하다

## operationally integrated

- 의미:
  - 연결된 데이터를 실제 decision / truth / risk path 에 쓰고 있다는 뜻
- 우리 문맥:
  - 그냥 연결됐다는 것보다 훨씬 강한 상태다

## REST fallback

- 의미:
  - 원래는 웹소켓 기반으로 읽고 싶었지만
  - 웹소켓이 안 붙거나 기능이 빠져서 REST 조회로 내려가는 것
- 왜 헷갈리나:
  - "연결은 됐다"와 "실시간으로 제대로 읽고 있다"를 구분 안 하면 놓치기 쉽다
- 우리 문맥:
  - WS import 실패 시 `REST fallback`으로 내려갔고
  - 그 상태에서는 BookDepth가 실제 gate 역할을 못 했다
- 실전에서 이렇게 읽는다:
  - REST fallback은 임시 대체 경로이지
  - WS-first가 이미 살아 있다는 뜻이 아니다

## 웹소켓 값과 REST 값이 안 맞음

- 의미:
  - 봇 내부의 웹소켓 포지션 값과
  - 다시 조회한 REST 포지션 값이 서로 다르게 보이는 상태
- 왜 헷갈리나:
  - 얼핏 보면 "실제 포지션이 남았다"로 바로 읽기 쉽다
  - 하지만 실제로는 값 동기화가 늦었거나, 이벤트 해석이 잘못됐을 수도 있다
- 우리 문맥:
  - WS 켠 런에서 SOL 값이 웹소켓 쪽엔 `-1.2`처럼 남아 있었지만
  - REST 재조회는 `0`이었다
- 실전에서 이렇게 읽는다:
  - 이런 상황에서는 먼저 실제 포지션을 다시 조회한다
  - 실제 포지션이 남아 있으면 정리
  - 실제 포지션이 0이면 그 다음 원인분석

## 잔여 포지션 의심

- 의미:
  - 봇이 "포지션이 아직 남았을 수 있다"고 판단하는 상태
- 우리 문맥:
  - one-leg fill
  - close 이후 값 불일치
  - WS/REST 불일치
  에서 자주 나온다
- 실전에서 이렇게 읽는다:
  - 의심이 생기면 먼저 실제 포지션 재조회
  - 실제 잔여가 있으면 강제정리
  - 의심만 있고 실제 잔여가 없으면 그때부터 분석

## risk priority

- 의미:
  - 어떤 failure가 더 위험한지의 우선순위
- 우리 문맥:
  - 여전히 `UNWIND` 쪽 리스크가 더 크다
- 실전에서 이렇게 읽는다:
  - "더 위험하다"와 "이번 배치에서 먼저 테스트한다"는 같은 말이 아니다

## batch execution order

- 의미:
  - 이번 배치에서 실제로 어떤 순서로 파고들지 정한 작업 순서
- 우리 문맥:
  - 현재는 `BUILD-first`
  - 즉 `spread/timing -> websocket-first BBO/BookDepth -> per-leg pricing -> 그 다음 UNWIND contract`
- 실전에서 이렇게 읽는다:
  - risk priority와 batch order를 섞어 읽으면 계속 헷갈린다
  - 이번 `1DEX` 문서에서는 둘을 분리해서 본다

## forced liquidation / forced flatten

- 의미:
  - 본체가 망가졌을 때 노출을 빨리 제거하는 긴급 청산 경로
- 우리 문맥:
  - 메인 방법이 아니라 2차 fallback
  - 수수료 손실 가능성이 크므로 여기에 의존하면 baseline 탈락

## alternate / alternating controller

- 의미:
  - cycle 마다 방향을 번갈아 가는 상위 execution controller
- 우리 문맥:
  - `BUY_FIRST`
    - ETH long / SOL short BUILD
  - `SELL_FIRST`
    - ETH short / SOL long BUILD
  - 현재 build family 비교는 이 controller 를 유지한 채, 그 위의 gate/data/pricing family 를 비교하는 방식이다

## BBO / BookDepth

- 의미:
  - BBO 는 최우선 bid/ask
  - BookDepth 는 여러 depth level 의 order book
- 우리 문맥:
  - BBO 는 빠른 가격 판단
  - BookDepth 는 sizing, slippage, paired fill viability 판단
  - 둘은 경쟁이 아니라 역할 분담 관계다

## bbo_minus_1

- 의미:
  - 현재 top-of-book 보다 한 tick 덜 공격적으로 가격을 두는 모드
- 우리 문맥:
  - fill 가능성은 낮아질 수 있지만 maker 성격은 강해진다

## bbo_plus_1

- 의미:
  - 현재 top-of-book 보다 한 tick 더 공격적으로 가격을 두는 모드
- 우리 문맥:
  - fill 가능성은 올라갈 수 있지만 비용/one-leg risk 도 같이 봐야 한다

## bbo

- 의미:
  - 현재 BBO 자체를 기준 가격으로 쓰는 모드
- 우리 문맥:
  - 가장 중립적인 기준선 후보로 읽을 수 있다

## aggressive

- 의미:
  - 더 빠른 체결을 위해 가격을 더 공격적으로 잡는 모드
- 우리 문맥:
  - `market` 보다는 덜 극단적이지만
  - one-leg 를 줄이는지 늘리는지는 live 검증이 필요하다

## market

- 의미:
  - 즉시 체결을 최우선으로 하는 시장가 또는 시장가 성격에 가까운 모드
- 우리 문맥:
  - fill 자체는 유리할 수 있지만
  - fee/slippage 비용이 크고 baseline 본체보다 fallback 성격으로 읽는 편이 맞다
