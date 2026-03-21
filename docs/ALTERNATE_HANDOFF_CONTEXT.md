# Alternate Handoff Context

이 문서는 Codex app을 종료하고 `tmux` / `ghostty`에서 작업을 이어갈 때 필요한 최소 컨텍스트만 모아둔 handoff 메모입니다.

---

## 1. 먼저 읽을 파일

이 두 파일이 핵심 리서치 결과입니다.

1. [ALTERNATE_DIFF_REPORT_B027C98_VS_3C64AC3.md](/Users/bot_trader/Dropbox/dexbot/perpdex/2dex/docs/ALTERNATE_DIFF_REPORT_B027C98_VS_3C64AC3.md)
2. [ALTERNATE_BRANCH_TIMELINE_REPORT.md](/Users/bot_trader/Dropbox/dexbot/perpdex/2dex/docs/ALTERNATE_BRANCH_TIMELINE_REPORT.md)

둘만 읽어도:
- `b027c98`이 왜 안정성 기준 최고점인지
- `3c64ac3`가 왜 수익성 잠재력을 열어준 분기점인지
- alternate가 어디서 망했고 어디서 좋아졌는지
를 다시 복구할 수 있습니다.

---

## 2. 두 파일만으로 부족한 추가 컨텍스트

두 리포트는 충분히 강하지만, 아래 항목은 **리포트 밖 실제 증빙 원본**입니다.
다음 세 경로만 같이 기억하면 handoff가 훨씬 매끄럽습니다.

### A. `3c64ac3` 이후 positive runtime 로그

파일:
- [dn_grvt_primary_paradex_grvt_ETH_log.txt](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/dn_grvt_primary_paradex_grvt_ETH_log.txt)

중요 지점:
- 플러스 세션 1: [dn_grvt_primary_paradex_grvt_ETH_log.txt#L9571](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/dn_grvt_primary_paradex_grvt_ETH_log.txt#L9571)
  - `18 cycles`
  - `Total Gross PnL: $0.1505`
  - `Average Edge: +2.82 bps`
  - 단, `Net Delta: 0.0600`
- 플러스 세션 2: [dn_grvt_primary_paradex_grvt_ETH_log.txt#L15730](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/dn_grvt_primary_paradex_grvt_ETH_log.txt#L15730)
  - `7 cycles`
  - `Total Gross PnL: $0.0982`
  - `Average Edge: +4.77 bps`
  - 단, `Net Delta: 0.0600`

### B. alternate가 망했던 Backpack-GRVT deep dive

파일:
- [deep_dive_report.md](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/deep_dive_report.md)
- [pnl_report_phase2a6.txt](/Users/bot_trader/Library/CloudStorage/Dropbox/dexbot/perp-dex-tools-original/hedge/logs/pnl_report_phase2a6.txt)

핵심:
- `NET PnL: -0.1540 USDT`
- 고점매수/저점매도 구조
- scale-up 금지 결론

### C. `b027c98`의 branch-internal 문서 증빙

worktree 기준 파일:
- `/tmp/2dex-alt-wt/perpdex/hedge/docs/stories.md`
- `/tmp/2dex-alt-wt/perpdex/hedge/docs/POST_MORTEM_V4_MISDIAGNOSIS.md`

핵심:
- `STORY-V3` 진행 상태
- `STORY-V4` 오진단 / `STORY-V5` clean start 교훈

---

## 3. 현재 최종 판단

### 안정성 기준 최고
- `b027c98`

이유:
- clean start
- hedge verification
- progressive sizing
- rebate tracker
- stale order 문제를 문서/코드 양쪽에서 다룸

### 수익성 잠재력 기준 최고
- `3c64ac3`

이유:
- alternate를 독립 전략 엔진으로 분리
- `min_spread_bps`
- WS-first reconcile
- 이후 `GRVT-Paradex` 런타임에서 positive session 확인

### 엄격 기준 최종 우승 커밋
- 없음

이유:
- committed `md`
- committed `csv`
- positive pnl
- flat 종료
를 동시에 만족하는 단일 커밋이 아직 없음

---

## 4. OMX / HUD 상태

현재 환경 기준:
- `oh-my-codex v0.9.0`
- `omx doctor` 통과
- Rust 설치 완료 (`cargo`, `rustc` 인식)

HUD 관련 결론:
- Codex app 안에서는 HUD가 잘 안 보일 수 있음
- `tmux` / 일반 터미널에서 쓰는 게 맞음
- 필요할 때만 `omx hud --watch`
- 평소에는 `omx`만 써도 충분

---

## 5. 터미널로 옮긴 뒤 바로 할 일

### 최소 복구 순서

```bash
cd /Users/bot_trader/Dropbox/dexbot/perpdex/2dex
```

읽기:
- `docs/ALTERNATE_DIFF_REPORT_B027C98_VS_3C64AC3.md`
- `docs/ALTERNATE_BRANCH_TIMELINE_REPORT.md`
- 필요 시 `docs/ALTERNATE_HANDOFF_CONTEXT.md`

### tmux에서 바로 시작하는 3줄

```bash
cd /Users/bot_trader/Dropbox/dexbot/perpdex/2dex
tmux new -s alternate
codex
```

그 다음 필요하면 tmux 안에서 pane 하나 더 열고:

```bash
omx hud --watch
```

### tmux 추천

pane 1:
```bash
codex
```

pane 2:
```bash
omx hud --watch
```

HUD가 필요 없으면 pane 2는 생략해도 됩니다.

---

## 6. 다음 작업 추천

터미널로 옮긴 뒤 바로 이어서 하기 좋은 작업:

1. `b027c98`의 안정성 기능을 `3c64ac3` 계열에 이식할 patch 설계
2. `3c64ac3` 계열의 `Net Delta 0.0600` 원인 추적
3. alternate hybrid branch 설계

---

## 7. Ghostty / iTerm2 / Terminal / tmux 메모

### 관계 정리

- `Ghostty` = 터미널 앱
- `iTerm2` = 터미널 앱
- `Terminal.app` = macOS 기본 터미널 앱
- `tmux` = 위 터미널 앱 **안에서 실행하는 세션 관리자**

즉:
- `Ghostty vs iTerm2 vs Terminal`은 바깥 껍데기 비교
- `tmux`는 그 안에서 pane/session을 관리하는 도구

### 짧은 비교

#### Ghostty

- 빠르고 modern한 UI
- tabs / splits / native macOS 감각이 좋음
- `codex`, `omx`, `hud` 같은 CLI 작업과 잘 어울림
- `tmux`와 같이 쓰면 가장 기분 좋은 조합 쪽

#### iTerm2

- 기능이 가장 많고 power-user용으로 강함
- tmux integration, trigger, hotkey window 등 고급 기능 풍부
- 보수적으로는 가장 실패 없는 선택

#### Terminal.app

- 기본 내장이라 가장 무난
- 초기 셋업이 가장 간단
- 하지만 power-user 기능은 제일 약함

### 현재 워크플로 추천

이 프로젝트 기준 추천 순서:

1. `Ghostty + tmux`
2. `iTerm2 + tmux`
3. `Terminal.app + tmux`

이유:
- Codex app 안에서는 HUD가 잘 안 보일 수 있음
- 터미널 + tmux에서는 `codex`, `omx hud --watch`, long-running session 관리가 쉬움

### 바로 실행 예시

Ghostty 또는 iTerm2 또는 Terminal.app 중 하나를 연 뒤:

```bash
cd /Users/bot_trader/Dropbox/dexbot/perpdex/2dex
tmux new -s alternate
codex
```

필요하면 tmux 안에서 pane 하나 더 열고:

```bash
omx hud --watch
```
