# Full-History Donor Matrix Plan

작성일: 2026-03-20

## Goal

대표 커밋 몇 개만 뽑는 수준이 아니라, **관련 히스토리 전체를 대상으로 donor 후보를 전수 조사**해서 메인라인 후보와 donor-only 후보를 분리한다.

현재 확인된 규모:

- 전체 reachable commit 수: `287`
- 현재 직접 관련성이 높은 remote/branch family:
  - `origin/DN-mean-reversion-GRVT-backpack-v1`
  - `origin/DN_pair_eth_sol_nado`
  - `origin/hedge-alternative-GRVT-paradex-v1`
  - `origin/hedge-mean-reversion-GRVT-paradex-v1`
  - `origin/test-results-20260129`

즉 이 계획은 "샘플 몇 개"가 아니라 **relevant commit universe 전체**를 donor 관점에서 분류하는 계획이다.

## Scope Definition

전수 조사 대상은 아래 기준을 만족하는 commit 이다.

1. strategy / hedge / exchange runtime에 영향을 주는 코드
2. 실행 결과를 설명하는 md/csv/txt/json 아티팩트
3. 수익성 / 안정성 / WS 활용도 / 포지션 관리에 영향을 주는 설정/실험 변경

제외 대상:

- 단순 문체 수정
- 무관한 HUD/OMX/도구 설정
- 기능/경제성과 직접 관계 없는 잡다한 정리

## Plan

### 1. Commit Universe 인덱싱

작업:

- 관련 브랜치/remote 전체에서 donor candidate commit을 수집
- commit마다 아래 metadata를 붙인다
  - date
  - branch family
  - touched areas
  - strategy class
  - exchange set
  - artifact presence

산출물:

- `.omx/plans/commit-universe-index.md`

Acceptance Criteria:

- relevant commit universe가 빠짐없이 인덱싱된다
- 각 commit이 어떤 family에 속하는지 식별 가능하다
- branch / strategy / exchange 기준으로 필터링할 수 있다

### 2. Evidence Strength 분류

작업:

- 각 commit을 evidence strength 기준으로 등급화
  - `A`: realized live result + committed artifact + flatness evidence
  - `B`: realized result but artifact incomplete or manual evidence
  - `C`: expected / planned improvement only
  - `D`: structural/refactor donor only
- 특별히 분리할 것
  - working baseline commits
  - plus-economics commits
  - WS-infrastructure commits
  - risk/flatness commits

산출물:

- `.omx/plans/evidence-grading.md`

Acceptance Criteria:

- 각 candidate commit이 증빙 강도에 따라 분류된다
- “실제 플러스”와 “예상 플러스”가 엄격히 분리된다
- “working but not profitable” baseline 이 따로 표시된다

### 3. Full Donor Matrix 작성

작업:

- 기능 축으로 donor를 전부 재조합
  - market data
  - execution
  - risk / flatness
  - leverage / subaccount
  - accounting / fee / pnl
  - strategy filters
- 각 기능마다 source를 1차/2차 후보로 기록
- `keep / adapt / reject` 판정을 붙인다

산출물:

- `.omx/plans/donor-matrix.md`

Acceptance Criteria:

- donor matrix가 representative가 아니라 full-history 기반으로 작성된다
- 각 기능에 최소 하나 이상의 source commit family가 연결된다
- reject 이유도 명시된다

### 4. Mainline Candidate 비교

작업:

- 아래 candidate lane를 같은 기준으로 비교
  - `2DEX mainline`
  - `1DEX Nado pair`
  - `1DEX + subaccount`
- 비교축
  - flatness
  - one-leg fill risk
  - WS capability usage
  - speed
  - fee economics
  - sybil / wash risk
  - implementation complexity

산출물:

- `.omx/plans/mainline-candidate-comparison.md`

Acceptance Criteria:

- 3개 lane를 같은 표준 축으로 비교할 수 있다
- 어느 lane를 mainline으로 둘지, 어느 lane를 donor로 둘지 논리적으로 설명 가능하다
- fallback path가 정의된다

### 5. Wintermute-Style Target Architecture & Roadmap

작업:

- full donor matrix를 바탕으로 target architecture를 확정
- 구현 순서를 다음처럼 정한다
  - baseline stabilization
  - WS / microstructure graft
  - accounting correctness
  - subaccount feasibility
  - PnL > 0 validation
- kill criteria 포함

산출물:

- `.omx/plans/target-architecture.md`
- `.omx/plans/pnl-positive-kill-criteria.md`

Acceptance Criteria:

- architecture가 donor 추출 결과와 직접 연결된다
- roadmap가 실행 순서대로 정리된다
- `PnL > 0`가 안 나올 때 중단/전환 기준이 숫자로 정의된다

## Notes

- 이 계획은 "몇 개의 좋아 보이는 커밋 선정"이 아니라, **전체 donor landscape를 만든 뒤 메인라인을 설계**하는 접근이다
- 따라서 다음 단계에서 commit 수가 많더라도 family clustering과 evidence grading이 먼저 필요하다
