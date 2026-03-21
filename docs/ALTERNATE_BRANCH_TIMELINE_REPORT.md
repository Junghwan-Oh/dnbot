# Alternate Branch Timeline Report

**Branch**: `feature/alternate`
**Analyzed On**: 2026-03-12
**Primary Comparison Anchors**:
- `b027c98` `feat(grvt): STORY-V3 Phase 2 complete - 10 iterations verified`
- `3c64ac3` `feat(alternate): Add DN_alternate_grvt_paradex strategy`

---

## 1. 핵심 결론

Alternate 브랜치는 크게 두 시기로 나뉜다.

1. **Backpack + Lighter / Backpack + GRVT 기반 안정화 시기**
   - 중심 커밋: `d5b1341`, `70f301f`, `b027c98`
   - 특징: 연결 인프라 정비, Progressive Sizing, Rebate Tracker, 10회 검증

2. **GRVT + Paradex 대안 전략 분리 시기**
   - 중심 커밋: `3c64ac3`, `6ccaa43`
   - 특징: 전략 독립, 이후 로컬 런타임에서 일부 플러스 PnL 세션 확인

엄격하게 보면, `feature/alternate` 안에서 **커밋된 CSV + 커밋된 MD + 플러스 PnL + flat 종료**를 동시에 만족하는 단일 커밋은 찾지 못했다.

그 대신 아래 두 개를 기준점으로 보는 것이 가장 정확하다.

- **브랜치 내 최고 품질 커밋**: `b027c98`
- **후기 alternate 계열의 최고 수익 런타임 출발점**: `3c64ac3`

---

## 2. 타임라인

| 날짜 | 커밋 | 구간 | 좋아진 점 | 망가진 점 / 한계 | 판단 |
|------|------|------|-----------|------------------|------|
| 2025-12-14 | `ebd2f95` | 초기 | Backpack + Lighter DN 봇 시작 | 전략은 point farming 중심, 수익성 검증 약함 | 출발점 |
| 2025-12-21 | `1e9063b` | 운영 | Telegram 제어, status file, graceful shutdown 추가 | 수익성 개선 없음 | 운영성 개선 |
| 2025-12-22 | `d5b1341` | WS 안정화 | Lighter WS auth deadline, market lookup, offset sequence 수정 | 전략 edge 자체는 그대로 | 인프라 핵심 수정 |
| 2025-12-22 | `1b83a8a` | sizing | Progressive Sizing 활성화 | sizing 전제는 연결 안정성이 필요 | 확장 준비 |
| 2025-12-24 | `70f301f` | GRVT 연결 | GRVT lazy import로 SDK hang 회피 | 거래 성능보다 초기화 안정성 중심 | GRVT 연결 핵심 수정 |
| 2025-12-24 | `d14dd22` | Backpack 검증 | Backpack import/ticker/balance 검증 완료 | 아직 hedge execution 검증 아님 | 기본 검증 |
| 2025-12-24 | `dd2cd54` | 설계 | production-ready TDD 프레임워크 정리 | 설계 문서 위주, 실전 성과 아님 | 구조화 |
| 2025-12-25 | `STORY-V4` 관련 변경군 | 실패 구간 | WS-only 아이디어 시도 | 오진단. 실제 원인은 stale open orders. over-engineering 발생 | 여기서 한번 망침 |
| 2025-12-25 이후 | `STORY-V5` 반영군 | 복구 | clean start, stale order 정리 방향 확인 | 커밋 메시지 단독보다 문서 증빙 중심 | 복구 시작 |
| 2025-12-26 | `b027c98` | 검증 정점 | 10/10 iterations verified, hedge verification 통과, rebate tracker 포함 | 브랜치 안에 커밋된 CSV는 없음 | 브랜치 최고점 |
| 2025-12-27 | `74998cd` 등 docs군 | 평가 | 경쟁력/성과/학습 리포트 정리 | 실행 로직 개선은 거의 없음 | 문서화 |
| 2026-01-21 | `3c64ac3` | 전략 전환 | `DN_alternate_grvt_paradex` 독립 전략 추가 | 새 전략이라 안정성 다시 검증 필요 | 후기 alternate 시작점 |
| 2026-01-22 | `6ccaa43` | 문서 | alternate 전략 README 정리 | 문서만 추가 | 운영 가이드 |

---

## 3. `b027c98` 기준점 분석

### 왜 기준점인가

`b027c98`은 alternate 브랜치에서 가장 균형이 좋다.

- 커밋 메시지 자체에 실전 검증 결과가 명시됨
  - `10/10 iterations successful at Phase 1 ($30)`
  - `All hedge verifications passed`
  - `GRVT rebate accumulated: $0.116 (20 trades)`
- 코드 변경도 단순 문서가 아니라 실행계층을 직접 건드림
  - `perpdex/hedge/exchanges/grvt.py`
  - `perpdex/hedge/hedge_mode_bp.py`
  - `perpdex/hedge/helpers/progressive_sizing.py`
  - `perpdex/hedge/helpers/rebate_tracker.py`

### 잘한 부분

1. **연결/실행 안정성**
   - GRVT order execution 경로 수정
   - tick size rounding 추가
   - position tracking field mapping 수정

2. **운영 안정성**
   - Progressive Sizing과 Rebate Tracker를 실제 hedge loop에 붙임
   - 문서상 `STORY-V3` 라이브 검증 흐름과 연결됨

3. **수익 구조 명시**
   - 단순 fill 성공이 아니라 maker rebate를 실제 추적 대상으로 끌어올림

### 한계

- 현재 브랜치 안에 `b027c98`에 직접 대응하는 committed CSV가 없다
- 따라서 강한 증빙은 `commit message + stories/post-mortem 문서`까지다
- "문서와 코드 기준 최고점"이지, "CSV까지 git에 남은 최고점"은 아니다

### 관련 증빙

- `STORY-V3` 진행 상황: [stories.md](/tmp/2dex-alt-wt/perpdex/hedge/docs/stories.md#L577)
- 오진단 롤백 및 clean start 필요성: [POST_MORTEM_V4_MISDIAGNOSIS.md](/tmp/2dex-alt-wt/perpdex/hedge/docs/POST_MORTEM_V4_MISDIAGNOSIS.md)

---

## 4. `3c64ac3` 계열 positive runtime 로그

`3c64ac3`은 `DN_alternate_grvt_paradex.py`가 브랜치에 정식 추가된 시점이다.

이후 로컬 런타임 로그에서는 **플러스 PnL 세션**이 실제로 확인된다.
다만 이 로그들은 `feature/alternate` 브랜치에 커밋된 산출물이 아니라, 별도 작업 디렉터리의 로컬 실행 결과다.

### 플러스 세션 예시 1

파일:
- `/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/dn_grvt_primary_paradex_grvt_ETH_log.txt`

요약:
- Completed Cycles: `18`
- Total Volume: `$534.09`
- Total Gross PnL: `$0.1505`
- Average Edge: `+2.82 bps`
- 단, Final Net Delta: `0.0600`

근거:
- [dn_grvt_primary_paradex_grvt_ETH_log.txt](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/dn_grvt_primary_paradex_grvt_ETH_log.txt#L9571)

### 플러스 세션 예시 2

요약:
- Completed Cycles: `7`
- Total Volume: `$205.99`
- Total Gross PnL: `$0.0982`
- Average Edge: `+4.77 bps`
- 단, Final Net Delta: `0.0600`

근거:
- [dn_grvt_primary_paradex_grvt_ETH_log.txt](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/dn_grvt_primary_paradex_grvt_ETH_log.txt#L15730)

### 왜 "좋아졌지만 아직 덜 끝난 상태"인가

좋아진 점:
- `Backpack-GRVT alternate`의 구조적 손실보다 낫다
- 일부 세션에서 명확한 플러스 PnL 달성
- 긴 세션(18 cycles)에서도 플러스 유지

아직 부족한 점:
- final `Net Delta`가 `0.0600`으로 남음
- 즉, "수익성은 좋아졌지만 flat 종료 안정성은 미완"

---

## 5. alternate가 어디서 망쳤고 어디서 좋아졌는지

### 망친 지점

#### 1. Backpack-GRVT 대안 전략의 방향 오류

실제 로컬 deep dive 결과:
- `NET PnL: -0.1540 USDT`
- 9 round trips 중 7 손실
- 원인: 고점매수/저점매도, adverse selection

근거:
- [deep_dive_report.md](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/deep_dive_report.md#L9)
- [pnl_report_phase2a6.txt](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/pnl_report_phase2a6.txt#L7)

이 시기 alternate는 "거래량 생성은 되지만, 구조적 손실 메커니즘이 있다"는 것이 증명됐다.

#### 2. STORY-V4 오진단

문제:
- Rate limit으로 오인하고 WS-only over-engineering
- 실제 원인은 이전 세션 stale open orders

근거:
- [POST_MORTEM_V4_MISDIAGNOSIS.md](/tmp/2dex-alt-wt/perpdex/hedge/docs/POST_MORTEM_V4_MISDIAGNOSIS.md)

이 구간은 alternate가 가장 크게 흔들린 지점이다.

### 좋아진 지점

#### 1. `d5b1341`

- Lighter WS auth deadline 수정
- market lookup 수정
- offset sequence check 수정

즉, 인프라 레벨에서 "아예 연결이 안 되던 상태"를 복구했다.

#### 2. `b027c98`

- order execution, rounding, position tracking 개선
- 10/10 검증
- rebate 추적

브랜치 품질이 가장 높아진 시점이다.

#### 3. `3c64ac3` 이후 runtime

- 전략을 `GRVT-Paradex`로 전환한 뒤 일부 세션에서 플러스 PnL 달성
- 즉, "수익 구조 없는 alternate"에서 "조건부 플러스가 가능한 alternate"로 이동

---

## 6. alternate 중 가장 좋은 커밋들

### 1위. `b027c98`

**왜 가장 좋은가**

- 브랜치 내부에서 가장 균형이 좋다
- 실전 검증 결과가 직접 명시된 마지막 강한 커밋이다
- 연결 안정성, execution 안정성, sizing, rebate tracking이 한 번에 들어 있다
- 후기 `3c64ac3`는 전략적으로 더 흥미롭지만, 안정성은 다시 흔들린다

**판정**
- 브랜치 내 "최고 품질 커밋"

### 2위. `3c64ac3`

**왜 좋은가**

- alternate를 진짜 alternate답게 만든 커밋이다
- `GRVT-Paradex` 독립 전략을 생성했고, 이후 positive runtime 로그의 출발점이다

**한계**
- 브랜치 안 committed 증빙보다 local runtime 증빙에 더 의존한다
- final net delta가 남아 안정성은 `b027c98`보다 낮다

**판정**
- 후기 alternate의 "최고 수익 잠재력 커밋"

### 3위. `d5b1341`

**왜 좋은가**

- WS auth와 market lookup이 안 되면 그 위 전략은 전부 무의미하다
- alternate 초기 실전 가능성을 열어준 인프라 핵심 수정이다

**판정**
- alternate 초기 인프라의 MVP 복구 커밋

### 4위. `70f301f`

**왜 좋은가**

- GRVT SDK hang 문제를 막아 연결 자체를 살렸다
- 이후 GRVT 기반 alternate 실험의 필수 기반이다

**판정**
- GRVT 사용 가능 상태를 만든 연결 커밋

### 5위. `1b83a8a`

**왜 좋은가**

- progressive sizing을 실제로 켠 시점이다
- 작은 성공을 누적해 scale-up하는 alternate 철학과 가장 잘 맞는다

**판정**
- 운영 확장성의 핵심 커밋

---

## 7. 2번에 추가할 다른 커밋들의 좋은 기능 리스트

### `1e9063b`

- Telegram 제어
- bot_status 상태 파일
- graceful shutdown
- 원격 운영성 향상

### `620fdd7`

- funding fee monitoring
- periodic funding log
- threshold warning
- 비용 관찰 가능성 향상

### `d14dd22`

- Backpack import/init 검증
- ticker fetch 검증
- balance fetch 검증
- CEX 측 연결 신뢰도 확보

### `dd2cd54`

- TDD/quality gate 프레임워크
- retry logic story
- WS reconnect story
- circuit breaker, position sync story 설계

### `6ccaa43`

- alternate 운영 문서
- rebate tracking 설명
- configuration/troubleshooting 정리
- branch 사용법 명확화

### `74998cd`, `8534a5f`, `7ed9184`, `b4151fb`

- 단기 실행 개선은 적지만,
- 전략 평가, 경쟁우위 재조정, 학습 방향 정리에 도움
- "무엇을 계속 밀고 무엇을 버릴지" 판단 자료 역할

---

## 8. 최종 판단

### 브랜치 안 최고 커밋

`b027c98`

이유:
- 실전 검증
- execution 안정화
- rebate/sizing 통합
- 문서 증빙 강함

### 후기 alternate 중 가장 promising한 커밋

`3c64ac3`

이유:
- 이후 로컬 실전 로그에서 플러스 PnL 세션이 확인됨
- 다만 flat 종료 안정성은 여전히 부족

### 엄격 기준에서 "최종 우승" 커밋이 없는 이유

다음 네 조건을 한 번에 만족하는 단일 커밋이 없다.

1. alternate branch 내부 커밋
2. committed `md` 증빙
3. committed `csv` 증빙
4. 플러스 PnL + flat 종료 안정성

따라서 현재 가장 정직한 평가는 다음과 같다.

- **안정성 우선 최고점**: `b027c98`
- **수익성 우선 최고점**: `3c64ac3` 이후 `GRVT-Paradex` 런타임 계열

