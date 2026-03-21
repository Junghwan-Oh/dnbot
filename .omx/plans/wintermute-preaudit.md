# Wintermute-Style Pre-Audit

작성일: 2026-03-20

## Audit Objective

현재 planning set이 아래 목표에 대해 **실행 전 기준으로 충분히 엄격한지** 사전감사한다.

- `$2k` seed
- `30일 / $30M 거래량`
- `fee-inclusive PnL > 0`

감사 기준:

1. donor extraction rigor
2. WS / market microstructure realism
3. architecture quality
4. TDD / SDD discipline
5. promotion / kill governance

## Executive Verdict

현재 계획은 **방향성은 강하고, 실행 전 계획으로는 유효**하다.
하지만 원본 상태 그대로는 `통짜 donor mixing` 위험이 남아 있어, strict TDD/SDD 계획으로 보기엔 부족했다.

이번 보강 이후 verdict:

- **Direction:** pass
- **Architecture intent:** pass
- **Donor methodology:** conditional pass
- **TDD / SDD rigor:** conditional pass
- **Promotion governance:** pass

즉, **조건부 합격**이다.

조건:

- donor를 반드시 함수/기능 단위로만 추출할 것
- donor 1개마다 테스트와 benchmark를 선행할 것
- architecture contract 고정 전에 donor grafting을 시작하지 말 것

## Strengths

### 1. Goal realism is fixed

- `$200/day`도 위험하다는 economics constraint가 반영됐다
- 목표가 "낮은 비용"이 아니라 `PnL > 0` 로 다시 정의됐다

### 2. Candidate lanes are separated correctly

- `mainline-now`
- `mainline-target`
- `donor-research`
- `fallback`

이 분리가 없으면 donor와 mainline이 다시 섞여 branch가 미궁화될 가능성이 높다.

### 3. Kill criteria are numeric

- `3 cycles`, `10 cycles`, `20+ cycles`, `100+ cycles`
- residual position
- one-leg fill
- fee-inclusive negative pnl

이 수치 기준은 전략 집착을 줄이고 decision quality를 높인다.

### 4. WS is treated as performance edge, not truth oracle

- WS-first market data
- REST-verified position truth

이 구분은 실거래 시스템에서 맞는 방향이다.

## Findings

### Finding 1. Original plan was too macro

문제:

- 초기 plan은 donor family, WS workstream, architecture layer를 잘 잡았지만
- donor 1개를 어떤 단위로 뽑고 어떤 테스트를 통과해야 하는지까지는 강제하지 않았다

영향:

- 실행 단계에서 여러 donor를 한 번에 섞을 위험
- regression origin 추적 어려움

조치:

- `donor-ws-architecture-execution-plan.md`
- `donor-matrix.md`
- `execution-handoff.md`

에 함수/기능 단위 extraction loop와 TDD gate를 추가했다.

### Finding 2. TDD was implicit, not explicit

문제:

- smoke / stability / baseline / mainline tier는 있었지만
- donor 1개 통합 전에 필요한 `unit / regression / benchmark` 3단 구조가 분리돼 있지 않았다

영향:

- lane-level 실패가 donor-level 문제를 가릴 수 있음

조치:

- donor별 테스트 요구사항을 명시적으로 계획에 포함했다

### Finding 3. SDD was present in spirit, weak in sequence

문제:

- target architecture가 있었지만
- "contract 먼저, donor graft 나중" 순서가 강한 규율로 적혀 있지 않았다

영향:

- 좋은 donor를 먼저 붙이고 나중에 구조를 맞추려는 반대 순서가 나올 수 있음

조치:

- `SDD-first`
- `No batch donor grafting`
- `target contract before integration`

을 execution discipline으로 추가했다.

## Required Non-Negotiables

아래 셋 중 하나라도 깨지면 Wintermute-style 기준에서 plan quality가 급락한다.

1. **Donor granularity**
   - commit 단위가 아니라 함수/기능 단위
2. **Evidence-driven promotion**
   - test + benchmark 없는 donor 승격 금지
3. **Truth-source discipline**
   - WS market data와 position truth를 분리

## What This Plan Can Likely Achieve

### Can it make `30M/month` technically possible?

- 가능성 높음
- throughput 자체는 달성 가능한 설계 방향이다

### Can it guarantee `PnL > 0`?

- 보장 불가
- 현재는 가능성을 높이는 plan이지, economics를 증명한 plan은 아니다

### Can it reduce false starts?

- 가능
- 특히 donor extraction과 kill criteria를 엄격히 쓰면 branch 미궁화를 줄일 수 있다

## Audit Recommendation

추천 실행 순서:

1. full-history donor universe 고정
2. donor function queue 작성
3. donor 1개당 tests-first extraction
4. lane benchmark 후 promotion
5. `100+ cycles fee-inclusive >= 0` 전까지 mainline 확정 금지

## Final Audit Judgment

이 planning set은 **실행할 가치가 있다.**
다만 아래처럼 읽어야 한다.

- **"이대로 하면 무조건 된다"** 는 아니다
- **"이 순서로 하면 될 가능성이 가장 높은 쪽으로 탐색할 수 있다"** 가 맞다

따라서 최종 판정:

- **Execution-worthy:** yes
- **Institutional-grade planning quality:** near-pass
- **Strict TDD/SDD readiness:** yes, after current guardrail additions
- **PnL > 0 probability uplift:** meaningful but unproven
