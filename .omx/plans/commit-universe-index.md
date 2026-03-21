# Commit Universe Index Plan

작성일: 2026-03-20

## 목적

관련 히스토리 전체를 donor candidate universe로 만들기 위한 **인덱싱 단계**를 정의한다.

이 단계의 목표는 "어떤 커밋이 있는지"를 빠짐없이 목록화하고,
이후 `evidence grading` 과 `donor matrix` 단계가 바로 이어질 수 있게 metadata를 붙이는 것이다.

## 현재 확인된 branch family 규모

- `origin/DN_pair_eth_sol_nado`: `63 commits`
- `origin/DN-mean-reversion-GRVT-backpack-v1`: `144 commits`
- `origin/hedge-alternative-GRVT-paradex-v1`: `150 commits`
- `origin/hedge-mean-reversion-GRVT-paradex-v1`: `136 commits`
- `origin/test-results-20260129`: `38 commits`
- `origin/feature/alternate`: `22 commits`
- `origin/feature/2dex`: `52 commits`

즉 전수 조사 범위는 작지 않다.
따라서 먼저 `family clustering` 과 `path filtering` 을 걸어야 한다.

## Universe Rules

### 포함 기준

아래 중 하나 이상에 해당하면 universe에 포함한다.

1. 전략 실행 경로 수정
   - `hedge/**/*.py`
   - `perpdex/**/*.py`
   - `perp-dex-tools-original/hedge/**/*.py`
2. exchange adapter 수정
   - `**/exchanges/*.py`
3. 테스트/로그/분석 산출물 추가
   - `**/*.md`
   - `**/*.csv`
   - `**/*.txt`
   - `**/*.json`
4. 경제성 / 수수료 / WS / 포지션 안정성에 직접 영향 주는 설정 변경

### 제외 기준

아래는 universe에서 제외한다.

- HUD / OMX / tool 설정 전용 커밋
- README 문체 수정만 있는 커밋
- unrelated infra / editor / shell only changes
- 단순 rename / move 이지만 runtime 의미가 없는 변경

## Index Schema

각 commit에 아래 필드를 붙인다.

- `commit`
- `date`
- `branch_family`
- `strategy_family`
- `exchange_set`
- `touched_paths`
- `change_type`
  - runtime
  - exchange
  - docs/evidence
  - test
  - hybrid
- `keywords`
  - websocket
  - bbo
  - bookdepth
  - position
  - close
  - pnl
  - fee
  - leverage
  - subaccount
- `artifact_presence`
  - md
  - csv
  - txt
  - json
- `candidate_role`
  - baseline
  - economics
  - ws-infra
  - risk/flatness
  - donor-only

## Plan

### 1. Family clustering

작업:

- 관련 ref를 family 단위로 묶는다
  - `2DEX`
  - `alternate`
  - `mean-reversion-backpack`
  - `mean-reversion-paradex`
  - `nado pair`
  - `test-results`

Acceptance Criteria:

- 모든 relevant ref가 정확히 하나의 family에 속한다
- family 경계가 문서화된다

### 2. Path filtering

작업:

- 경로 기반 1차 필터를 적용해 donor irrelevant commit을 제거한다
- runtime / exchange / docs artifact path만 남긴다

Acceptance Criteria:

- universe 후보가 "전체 git history"가 아니라 "관련 commit history"로 축소된다
- 어떤 경로가 포함/제외되는지 명시된다

### 3. Metadata tagging

작업:

- commit마다 schema 필드를 채운다
- 특히 `strategy_family`, `exchange_set`, `candidate_role` 을 먼저 붙인다

Acceptance Criteria:

- 이후 evidence grading 단계에서 추가 질문 없이 바로 쓸 수 있다
- commit 단위 조회가 아니라 family 단위 aggregation이 가능하다

### 4. Artifact linkage

작업:

- 각 commit이 바꾼 `md/csv/txt/json` artifact를 연결한다
- realized 결과 증빙이 있는지 여부를 별도 표시한다

Acceptance Criteria:

- "실제 plus"와 "예상 plus"를 구분할 근거가 확보된다
- commit message만 강한 케이스와 committed artifact까지 있는 케이스가 분리된다

### 5. Handoff to evidence grading

작업:

- 인덱스 결과를 다음 문서로 넘길 준비를 한다
  - `evidence-grading.md`
  - `donor-matrix.md`

Acceptance Criteria:

- universe index만으로 다음 단계가 바로 시작 가능하다
- 빠진 branch family나 strategy family가 없는지 체크리스트가 있다

## Output Files

- `.omx/plans/commit-universe-index.md` (this file)
- next:
  - `.omx/plans/evidence-grading.md`
  - `.omx/plans/donor-matrix.md`
