# 1DEX + Subaccount Decision Memo

작성일: 2026-03-20

## 1. 목적

현재 선택지는 크게 3개였다.

1. `2DEX`로 단일 자산을 서로 다른 거래소에서 델타 뉴트럴
2. `1DEX`에서 서로 다른 코인 노셔널을 맞춰 델타 뉴트럴처럼 운영
3. `1DEX + subaccount`로 같은 거래소 안에서 단일 자산을 롱/숏 분리

이 문서는 왜 `1DEX + subaccount`가 가장 자연스러운 다음 방향인지, 그리고 왜 아직 바로 메인라인으로 확정할 수는 없는지 정리한다.

## 2. 지금까지 테스트에서 확인된 사실

### 2.1 2DEX

작은 live test에서는 거래가 잘 발생했다.
flat 종료도 어느 정도 확인됐다.

하지만 확장 테스트에서는:

- market fallback 비중이 높았고
- maker 체결 유지율이 낮았고
- 시간이 지나며 net delta drift가 누적됐다
- 10회 확장에서는 완주 전 포지션 불일치가 다시 드러났다

즉:

- 실행 흐름은 비교적 예측 가능
- 경제성은 약함
- 오래 굴리면 포지션 완결성이 깨질 수 있음

### 2.2 1DEX Nado pair

최신 Nado pair 계열은 기술적으로 훨씬 세련됐다.

- WebSocket BBO
- BookDepth
- liquidity-aware skip
- momentum / spread-state

같은 시장 미시구조 계층이 들어갔다.

하지만 실제 small live test 결과는 둘로 갈렸다.

- 어떤 후보는 3회 스모크는 통과했지만 10회에서 one-leg fill
- 어떤 후보는 WebSocket이 붙어도 no-trade
- 즉 기술은 진화했지만, 연속 운용 안정성은 아직 부족했다

### 2.3 Nado working baseline

가장 의미 있었던 Nado baseline은 3회 기준:

- WebSocket 실제 연결
- 실제 체결
- 실제 청산
- flat 종료

까지 성공했다.

그러나 10회로 늘리면:

- 한쪽 leg만 fill
- safety stop
- 연속 운용 실패

가 다시 나타났다.

즉 Nado는 "아예 안 되는 거래소"는 아니지만, 아직 기준 베이스라인으로 삼기엔 불안정하다.

## 3. 왜 1DEX + subaccount 가 더 자연스러운가

`1DEX pair (ETH/SOL)` 방식은 엄밀한 의미의 델타 뉴트럴이 아니다.
그건 결국 상관 기반 pair trade에 가깝다.

문제:

- ETH/SOL basis risk 존재
- 상관관계가 깨지면 델타 중립 해석이 무너짐
- 같은 노셔널이라도 진짜 hedge라고 보기 어렵다

반면 `1DEX + subaccount`는:

- 같은 거래소
- 같은 API
- 같은 자산
- 단일 방향 long / short 분리

로 갈 수 있기 때문에 전략 구조가 훨씬 단순하다.

이 방식의 장점:

- pair basis risk 제거
- 같은 시장 미세구조 위에서 체결
- 구현 난이도 하락
- 2DEX보다 exchange mismatch 감소

즉 "진짜 델타 뉴트럴"에 더 가깝다.

## 4. Nado 문서 리서치 핵심

### 4.1 subaccount / margin / leverage

공식 문서:

- [Subaccounts & Health](https://docs.nado.xyz/subaccounts-and-health)
- [Margin Types](https://docs.nado.xyz/margin-types)
- [Linked Signers](https://docs.nado.xyz/developer-resources/get-started/linked-signers)

확인된 내용:

- subaccount는 지갑 주소 아래 여러 개 만들 수 있다
- 프론트는 4개 제한이 있지만 API 사용자는 더 유연하다
- health는 실시간 risk engine 기준으로 계속 바뀐다
- isolated / unified margin이 섞이므로 표시 leverage / health가 체감상 자주 변할 수 있다
- linked signer를 써서 메인 지갑 대신 거래 전용 키를 분리할 수 있다

의미:

- 사용자 체감처럼 "isolated 5x를 줬는데 자꾸 상태가 변한다"는 건 구조상 이상한 현상은 아니다
- 다만 bot 관점에서는 이 동적 health / subaccount 구조가 포지션 truth를 더 어렵게 만든다

### 4.2 fee 구조

공식 문서:

- [Fees & Rebates](https://docs.nado.xyz/fees-and-rebates)
- [Fee Rates API](https://docs.nado.xyz/developer-resources/api/gateway/queries/fee-rates)

확인된 내용:

- base taker: 약 `3.5 bps`
- 상위 tier taker: `1.5 bps`까지 감소 가능
- base maker: 약 `1 bp`
- 상위 tier에선 maker rebate 가능
- taker에는 minimum fee 개념이 있음

의미:

- 이론적으로는 `$1M` 거래량에 총 fee를 `$200` 안팎으로 넣는 건 maker 성격이 강해야 가능하다
- pure taker 위주면 `$350+` 쪽이 더 현실적이다

### 4.3 points / sybil / wash trading 리스크

공식 문서:

- [Season 1](https://docs.nado.xyz/points/season-1)
- [Referrals](https://docs.nado.xyz/points/referrals)

확인된 내용:

- points는 raw volume만이 아니라 "genuine market participation" 기준
- wash trading / self-matching / non-organic behavior 는 금지
- 비유기적 행동은 weekly epoch 기준으로 `reduced or zero allocation` 가능

의미:

- "오픈하자마자 닫기" 전략은 기술적으로는 가능해도, 포인트 프로그램 관점에선 위험하다
- 특히 같은 계정/같은 거래소/너무 기계적인 왕복은 sybil / wash trading으로 해석될 여지가 있다

## 5. 1M 거래량 / $200 비용 가능성

### 5.1 이론적 하한

`$1M` 거래량에서 총 비용 `$200`이면 비용률은 `2 bps`다.

가능 조건:

- maker 체결 유지율이 매우 높다
- 혹은 maker rebate tier에 진입한다
- spread / slippage가 거의 없다

즉 "시장가에 가까운 IOC/taker" 중심으론 거의 불가능하다.

### 5.2 현재 baseline 관찰

지금까지 본 baseline들:

- 2DEX: gross 손실률은 작지만 fallback이 많고 net은 음수
- Nado: 어떤 baseline은 체결/청산은 빨랐지만, fee 포함 net은 여전히 음수

즉 현재 상태는:

- `$1M` 가능 여부보다
- `net >= 0`을 먼저 못 맞춘 상태

### 5.3 내 판단

사용자 감각처럼:

- WebSocket
- BookDepth
- 시장 미시구조 활용
- maker 유지율 관리

를 잘하면 `$1M` 거래량에 `$200` 이하, 잘하면 `0` 근처까지 가는 건 **이론상 가능**하다.

하지만 현재 구현 수준에선 아직 아니다.

이유:

- one-leg fill 리스크
- 포지션 truth 불안정
- 실제 maker 체결 유지 부족
- fee/PNL accounting 불완전

즉 지금은 "가능성은 있다"가 맞고, "현재 이미 된다"는 아니다.

## 6. pros / cons

### 6.1 1DEX + subaccount

장점:

- 같은 API
- 같은 거래소
- 같은 자산
- 구조적으로 가장 단순한 진짜 hedge
- 2DEX보다 체결 구조 해석이 쉬움

단점:

- 거래소 health / subaccount / leverage 체계가 까다로움
- wash/sybil 해석 리스크
- close / position truth가 아직 완전히 신뢰되지 않음

### 6.2 2DEX

장점:

- 구조적으로 직관적
- trade는 비교적 잘 발생
- historical reference가 더 많음

단점:

- 느림
- exchange mismatch
- market fallback 많음
- economics 약함

### 6.3 1DEX pair

장점:

- 단일 거래소라 구현은 가능
- 데이터 계층 실험엔 좋음

단점:

- 진짜 delta-neutral이 아님
- correlation trade에 가깝다
- 메인라인으로는 비추천

## 7. 최종 판단

현재 추천 순위:

1. `1DEX + subaccount` ETH 단일자산 델타 뉴트럴
2. `2DEX` ETH 델타 뉴트럴
3. `1DEX pair` ETH/SOL 노셔널 매칭

이유:

- 메인 목표가 `1M volume`, `PnL >= 0`라면
- pair trade보다 단일자산 hedge가 먼저다
- 2DEX는 fallback 메인 후보이고
- Nado는 아직 거래소 품질 리스크가 남지만, subaccount 방식이 제일 자연스럽다

## 8. 실행 제안

다음 단계는 이 순서가 맞다.

1. `1DEX + subaccount` 가 실제 API로 가능한지 확인
   - order
   - position query
   - fee query
   - linked signer
   - subaccount isolation
2. small live test
3. one-leg fill / close failure가 계속 나오는지 확인
4. 안 되면 즉시 `2DEX` fallback

## 9. 한 줄 결론

지금 메인라인 철학은 이렇다.

- `Nado`는 기술적으로 더 미래형이다
- 하지만 아직 거래소 리스크가 남아 있다
- 그래도 장기적으로는 `1DEX + subaccount`가 가장 맞는 방향이다
- 단, 그게 실제로 안 되면 미련 없이 `2DEX`로 돌아가야 한다
