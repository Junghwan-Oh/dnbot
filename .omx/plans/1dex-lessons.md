# 1DEX Lessons

작성일: 2026-04-11

## 목적

이 문서는 `1DEX` 작업에서 이미 얻은 steering lesson 을 짧게 고정하는 파일이다.

이 문서가 소유하는 것:

- 테스트/실행에서 얻은 교훈
- 방향 수정 규칙
- 반복 실수를 줄이는 steering note

이 문서는 requirements 나 gate 나 source-map count 를 소유하지 않는다.

## Lessons

1. `UNWIND` 가 더 위험하지만, 실행 순서상 `BUILD` 가 먼저 gate 다.
   - 따라서 `BUILD` 는 먼저 선별해야 하고
   - `UNWIND` 는 먼저 강화해야 한다
   - 둘은 모순이 아니라 다른 우선순위다

2. `forced liquidation` 은 메인 방법이 아니다.
   - baseline 이 여기에 자주 의존하면 이미 실패다
   - 본체는 건강한 `BUILD / UNWIND` 여야 한다

3. `POST_ONLY` 가 경제적으로 예뻐 보여도, live 에서 fill 이 안 나오면 baseline 후보가 아니다.
   - current verdict: `POST_ONLY = reject`

4. `source map` 은 코드 감상문이 아니다.
   - `keep / reject / extract / compare-next`를 빠르게 내리기 위한 판정판이다

5. `Stage2` 는 current branch 기준 legacy drift 다.
   - `Stage3`가 canonical smoke
   - `Stage4`가 cycle behavior verification

6. real mainnet read-only sanity는 live run 전에 반드시 필요하다.
   - 그래야 env 문제와 로직 문제를 분리할 수 있다

7. `min-spread-bps 6` 에서 skip, `0` 에서도 no fill 이면
   - 지금 blocker 는 cleanup 이 아니라 `entry viability` 다

8. 문서는 로컬에 오래 묵히지 말고 GitHub visible 상태로 자주 올리는 편이 steering correction 에 더 좋다.

## Current takeaway

지금 `1DEX`는:

- `UNWIND / close / flatness` 는 계속 추출 가치가 있고
- `POST_ONLY BUILD entry` 는 더 이상 오래 붙잡고 미시 최적화할 가치가 적다
- 다음 비교 후보는 `IOC` 다
