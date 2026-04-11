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

## Methodology To Reuse For 2DEX Team

이건 `1DEX`만의 결과가 아니라, 다음에 `2DEX` 팀도 따라 할 수 있는 작업 방식이다.

1. 먼저 canonical 문서를 적게 고정한다.
   - `plan`
   - `operating contract`
   - `source map`
   - `glossary`

2. `plan`을 기준점으로 두고, `source map`은 판정판으로 쓴다.
   - `plan`은 방향 / phase / 우선순위 소유
   - `source map`은 `keep / reject / extract / compare-next`
   - `operating contract`는 truth / gate / kill criteria 소유

3. code-only archaeology 를 금지한다.
   - `plan + code + tests + live result` 같이 본다

4. 먼저 source map을 만든다.
   - commit 수
   - 그룹 수
   - 현재 verdict 수
   - 남은 comparison 후보 수
   를 숫자로 가시화한다

5. 초반에는 미시 최적화 대신 `reject`를 빨리 내린다.
   - live 에서 막히는 front gate는 빨리 버린다
   - 약한 후보에 오래 매달리지 않는다

6. 테스트 ladder를 현재 코드 구조 기준으로 다시 고정한다.
   - drift 난 테스트는 과감히 legacy 로 내린다
   - 현재 branch와 맞는 smoke / cycle test를 canonical 로 잡는다

7. real read-only sanity 를 먼저 한다.
   - env / key / account / position 조회가 되는지 먼저 본다
   - 그래야 로직 문제와 환경 문제를 분리할 수 있다

8. live run 결과는 바로 verdict 로 연결한다.
   - 예:
     - spread skip
     - build 진입 but no fill
     - one-leg fill
     - post position 0
   - 이런 결과를 바로 `reject / pending / keep` 으로 연결한다

9. 문서는 작은 배치로 자주 GitHub main 에 올린다.
   - steering correction 을 빠르게 받기 위해서다

10. 본체와 fallback 을 분리해서 본다.
   - healthy baseline = 본체가 99%+ 건강하게 작동
   - forced liquidation = 2차 fallback

## 2DEX Team Hand-off Hint

`2DEX` 팀에 전달할 때는 아래 링크 4개만 먼저 보면 된다.

- `1dex-plan.md`
- `1dex-operating-contract.md`
- `1dex_source_map.md`
- `1dex_glossary.md`

이 4개를 보면:

- 기준점이 뭔지
- 판정판이 뭔지
- 어떤 후보를 빨리 버려야 하는지
- 어떻게 live result 를 verdict 로 바꾸는지

바로 따라갈 수 있다

## Current takeaway

지금 `1DEX`는:

- `UNWIND / close / flatness` 는 계속 추출 가치가 있고
- `POST_ONLY BUILD entry` 는 더 이상 오래 붙잡고 미시 최적화할 가치가 적다
- 다음 비교 후보는 `IOC` 다
