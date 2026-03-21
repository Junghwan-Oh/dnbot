# Evidence Grading Plan

작성일: 2026-03-20

## 목적

relevant commit universe 전체를 **증빙 강도** 기준으로 분류한다.

이 단계의 목적은 donor matrix 이전에 아래를 엄격히 분리하는 것이다.

- 실제로 잘 돌았던 것
- 실제로 돈을 벌었던 것
- 이론상 개선만 제시한 것
- 구조 donor이지만 결과 증빙은 없는 것

즉 "좋아 보이는 커밋"과 "증명된 커밋"을 섞지 않게 하는 단계다.

## Grading Dimensions

각 commit은 아래 두 축으로 본다.

1. **Evidence Strength**
2. **Functional Role**

### 1. Evidence Strength

#### Grade A

정의:

- realized live result 있음
- committed artifact 있음 (`md/csv/txt/json`)
- flatness 또는 포지션 안정성도 같이 확인됨

예시:

- live test 결과가 문서나 csv로 같이 커밋됨
- commit message와 artifact가 서로 일치함
- 결과가 positive 또는 stable baseline 으로 읽힘

의미:

- mainline baseline 또는 economics baseline 후보

#### Grade B

정의:

- realized result는 있음
- 하지만 artifact가 incomplete 하거나
- commit message 의존성이 크거나
- flatness / fees / residual position 정보가 불완전함

의미:

- strong donor 후보
- 단, baseline 확정에는 추가 검증 필요

#### Grade C

정의:

- expected improvement / target improvement / planned improvement 만 있음
- 테스트는 unit / TDD / simulation 중심
- 실제 live realized evidence 없음

의미:

- economics donor / architecture donor
- 결과를 약속하는 baseline 으로 쓰면 안 됨

#### Grade D

정의:

- runtime 결과보다 구조 / infra / cleanup / refactor 위주
- execution path donor로는 유효할 수 있으나 outcome 증빙은 약함

의미:

- architecture donor
- risk/ops donor

## 2. Functional Role Tags

각 commit은 아래 역할 태그를 1개 이상 가진다.

- `baseline-working`
- `baseline-profitable`
- `economics`
- `ws-infra`
- `risk-flatness`
- `execution`
- `accounting`
- `subaccount-leverage`
- `donor-only`

## Artifact Priority Rules

증빙 우선순위는 아래 순서다.

1. committed `csv`
2. committed `md/txt/json` with explicit result summary
3. commit message with explicit test result
4. branch README / strategy description
5. inferred behavior from code only

규칙:

- 상위 증빙이 하위 증빙과 충돌하면 상위 증빙을 우선한다
- commit message 와 md가 충돌하면 artifact 쪽을 우선 본다
- code only inference는 절대 `Grade A` 또는 `baseline-profitable`로 올리지 않는다

## Positive PnL Classification Rules

positive 관련 판정은 아래처럼 나눈다.

- `realized-positive`
  - 실제 테스트 결과가 `> 0`
- `near-flat`
  - 결과는 음수지만 fee/slippage 포함 손실이 매우 작음
- `expected-positive`
  - 문서/메시지상 목표치만 양수
- `unknown`
  - 판단 근거 부족

규칙:

- `expected-positive` 는 절대 `realized-positive` 로 승격하지 않는다
- `near-flat` 은 donor 가치가 높을 수 있으나, profitable baseline 으로 분류하지 않는다

## Flatness / Stability Classification Rules

flatness는 별도 표식으로 남긴다.

- `flat-confirmed`
- `flat-unclear`
- `residual-risk`
- `one-leg-fill`

규칙:

- `one-leg-fill` 이 한 번이라도 강하게 관찰되면 `baseline-working` 태그를 제한한다
- `flat-confirmed` 는 positive 여부와 별개로 안정성 donor로 분류 가능하다

## Plan

### 1. Candidate grading

작업:

- universe에 포함된 각 commit에 grade(A/B/C/D)를 붙인다
- 동시에 role tag를 1개 이상 붙인다

Acceptance Criteria:

- 각 candidate commit에 grade와 role이 모두 있다
- ambiguous한 commit은 `unknown` 또는 `artifact incomplete`로 남긴다

### 2. Conflicting evidence resolution

작업:

- message / md / csv / code inference가 충돌하는 커밋을 따로 표시
- override rule을 적용해 최종 판정을 내린다

Acceptance Criteria:

- "message는 plus인데 artifact는 minus" 같은 케이스가 명시적으로 표시된다
- 나중에 donor matrix에서 다시 같은 논쟁이 반복되지 않는다

### 3. Baseline shortlist extraction

작업:

- grade와 role을 바탕으로 shortlist를 뽑는다
  - working baselines
  - profitable baselines
  - WS baselines
  - subaccount feasibility baselines

Acceptance Criteria:

- shortlist가 branch family별로 분리된다
- 2DEX / Nado / alternate / mean-reversion line이 같은 잣대로 비교된다

## Output

- `.omx/plans/evidence-grading.md` (this file)
- next:
  - `.omx/plans/donor-matrix.md`
  - `.omx/plans/mainline-candidate-comparison.md`
