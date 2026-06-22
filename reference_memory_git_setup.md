---
name: 메모리 구조 + git sync 셋업 (리눅스/macOS/Windows)
description: Claude 메모리 3계층(장기/중기/단기) 구조와, 장기 메모리를 ~/.claude/claude_memory/ + ~/.claude/CLAUDE.md(@import) 로 셋업해 saugjong0521/claude_memory.git 과 sync. 리눅스·macOS·Windows 공통. Stop hook 사용 X
type: reference
originSessionId: 469e673c-f305-4b08-9202-16e3811f4952
---
# 메모리 3계층 구조

Claude 의 메모리는 수명·범위에 따라 3계층으로 나뉜다. **장기 메모리(이 repo)는 프로젝트 폴더 안에 넣지 않는다.**

| 계층 | 무엇 | 위치 | 성격 / 로드 시점 |
|------|------|------|------------------|
| **장기 (long-term)** | 코드·분석·작업 전반을 어떻게 할지 지침하는 글로벌 룰 (= 이 repo `saugjong0521/claude_memory`) | `~/.claude/claude_memory/` (repo clone) + `~/.claude/CLAUDE.md` 가 `@import` | 프로젝트 무관. **모든 세션·모든 작업 시작 전에 자동 로드·적용** |
| **중기 (mid-term)** | 특정 프로젝트의 내역·구조·정책·특이사항 | 그 프로젝트 repo 안 `docs/000.harness_guide` (1차) — 또는 필요 시 `~/.claude/projects/<id>/memory/` | 그 프로젝트 한정. 해당 프로젝트 작업 시 참조 |
| **단기 (short-term)** | 세션 중 사용자가 내리는 일반 명령 | 세션 메모리 (휘발) | 특별 지시 없으면 그 세션 안에서만 유지 |

판단 기준:
- "다른 프로젝트에서도 적용되는 나의 행동 규칙인가?" → **장기** (이 repo)
- "이 프로젝트에만 해당하는 사실인가?" → **중기** (그 프로젝트 repo `docs/000`)
- "이번 세션 한정 지시인가?" → **단기** (저장 안 함, 명시 지시 시에만 승격)

관련 룰: [feedback_project_memory_stays_in_project.md](feedback_project_memory_stays_in_project.md), [feedback_check_docs_000_first.md](feedback_check_docs_000_first.md)

---

# OS별 경로 — 핵심: `~/.claude/` 는 3 OS 공통

Claude Code 는 리눅스·macOS·Windows 모두에서 홈의 `.claude/` 를 인식한다. 경로 표기만 다르고 구조는 동일.

| OS | 홈 디렉토리 | 장기 메모리 clone 위치 | 전역 CLAUDE.md |
|----|-----------|----------------------|----------------|
| **Linux** | `/home/<user>` | `~/.claude/claude_memory/` | `~/.claude/CLAUDE.md` |
| **macOS** | `/Users/<user>` | `~/.claude/claude_memory/` | `~/.claude/CLAUDE.md` |
| **Windows** | `C:\Users\<user>` | `%USERPROFILE%\.claude\claude_memory\`<br>(Git Bash: `~/.claude/claude_memory/`) | `%USERPROFILE%\.claude\CLAUDE.md` |

- **Git Bash 를 쓰면 3 OS 모두 `~/.claude/...` 동일 명령** 이 된다 (권장).
- Windows PowerShell/cmd 네이티브는 `%USERPROFILE%` 또는 `$HOME` 사용.
- CLAUDE.md 안의 `@claude_memory/MEMORY.md` 는 **OS 무관** — Claude Code 가 **CLAUDE.md 파일 위치 기준 상대경로**로 해석하므로 `~/.claude/claude_memory/MEMORY.md` 로 풀린다 (import 최대 depth 4, 우리는 1단계).

> ⚠️ **옛 방식 폐기**: 과거에는 repo 를 `~/.claude/projects/<PROJECT_ID>/memory/` (프로젝트 폴더) 안에 clone 했으나, 그건 장기(글로벌) 메모리를 프로젝트 폴더에 중복으로 박아 넣는 구조라 폐기. 이제 **글로벌 위치 1곳(`~/.claude/claude_memory/`)** 에만 두고 `CLAUDE.md` 로 로드한다.

---

# 셋업 절차 (PC 1대당 1회)

## A. Linux / macOS / Windows(Git Bash) — 공통

```bash
# 1) 글로벌 위치로 clone
git clone https://github.com/saugjong0521/claude_memory.git ~/.claude/claude_memory

# 2) 메모리 repo 로컬 config (author 일관성 — 글로벌 git config 와 별도)
git -C ~/.claude/claude_memory config user.name  "saugjong0521"
git -C ~/.claude/claude_memory config user.email "saugjong0521@gmail.com"

# 3) ~/.claude/CLAUDE.md 작성 (아래 § "CLAUDE.md 내용")
# 4) 인증 설정 (아래 § 인증)
# 5) 새 세션에서 /memory 로 검증
```

## B. Windows PowerShell (네이티브) — 경로만 다름

```powershell
git clone https://github.com/saugjong0521/claude_memory.git "$env:USERPROFILE\.claude\claude_memory"
git -C "$env:USERPROFILE\.claude\claude_memory" config user.name  "saugjong0521"
git -C "$env:USERPROFILE\.claude\claude_memory" config user.email "saugjong0521@gmail.com"
# 편집 대상: "$env:USERPROFILE\.claude\CLAUDE.md"
```

## CLAUDE.md 내용 (3 OS 동일)

`~/.claude/CLAUDE.md` (Windows: `%USERPROFILE%\.claude\CLAUDE.md`) — Claude Code 가 모든 프로젝트·세션에서 자동 로드:

```markdown
# 장기 메모리 (전역 지침)

모든 작업을 시작하기 전에 아래 메모리 인덱스를 따른다.
인덱스 상단의 프로세스별 참조 가이드 표에 맞춰, 해당 작업 유형의 `feedback_*` 파일을 먼저 read 한 뒤 진행한다.

@claude_memory/MEMORY.md
```

(`@claude_memory/MEMORY.md` 는 `CLAUDE.md` 기준 상대경로 → `~/.claude/claude_memory/MEMORY.md` 로 해석됨. MEMORY.md 가 인덱스이고, 개별 `feedback_*` 룰은 필요 시 read.)

## 인증 (push 가능해야 함)

둘 중 하나:
- **SSH key**: GitHub 에 SSH key 등록 + remote 를 SSH URL 로 변경
  ```bash
  git -C ~/.claude/claude_memory remote set-url origin git@github.com:saugjong0521/claude_memory.git
  ```
- **HTTPS + PAT**: GitHub Personal Access Token 발급 → credential helper 사용 또는 remote URL 에 embed

## settings.json

`~/.claude/settings.json` 에 **Stop hook 추가하지 말 것**. 자동 commit 은 컨펌 룰 ([feedback_commit_message_format.md](feedback_commit_message_format.md)) 과 충돌해서 2026-05-08 폐기됨. (settings.json 자체는 PC 마다 별도 — sync 대상 아님)

---

# ⚠️ 옛 방식에서 마이그레이션 — 중복 로드 주의

`~/.claude/CLAUDE.md`(@import) 와 옛 `~/.claude/projects/<id>/memory/`(auto-memory) 는 **별개 메커니즘이라 둘 다 로드된다**. 옛 방식에서 넘어올 땐 **옛 위치를 비워야** 같은 룰이 2번 안 실린다.

```bash
# 현재 repo(최신 상태)를 글로벌 위치로 이동 → 옛 위치 자동으로 비움
mv ~/.claude/projects/<PROJECT_ID>/memory ~/.claude/claude_memory
# 이후 git config 재확인 + ~/.claude/CLAUDE.md 작성
```
(이미 로드된 현재 세션은 영향 없고, **다음 세션부터** 새 방식 적용.)

---

# 적용 후 검증

1. `~/.claude/claude_memory/MEMORY.md` 존재 + `~/.claude/CLAUDE.md` 에 `@claude_memory/MEMORY.md` import 라인 확인
2. **새 세션에서 `/memory` 실행** → "User instructions" 에 `~/.claude/CLAUDE.md` + import 된 `claude_memory/MEMORY.md` 가 보이면 성공. (안 보이면 그 하네스가 글로벌 CLAUDE.md 로드를 안 하는 것 → 대체 방식 필요)
3. 메모리 파일 임시 수정 → Claude 한테 commit 처리 요청 → diff 기반 메시지 제안 + 컨펌 → push
4. GitHub `saugjong0521/claude_memory` main 에 새 commit 도착 확인
5. 다른 PC 에서 `git -C ~/.claude/claude_memory pull` → 변경사항 도착 확인

---

# 다중 PC sync

- **저장(변경)**: Claude 가 turn 안에서 diff 기반 commit 메시지 제안 + 컨펌 + 직접 commit + push (Stop hook 자동화 X) — 룰: [feedback_auto_sync_memory.md](feedback_auto_sync_memory.md).
- **가져오기**: `git -C ~/.claude/claude_memory pull` (각 PC).

---

# 트러블슈팅

- **push 실패 (auth)**: `Permission denied` / `403` → SSH key 또는 PAT 재확인
- **commit author 가 다른 이름**: `git -C ~/.claude/claude_memory config user.name/email` 재설정 (글로벌과 별도)
- **git not found (Windows)**: PATH 에 `git.exe` 등록 필요 (Git for Windows 설치 → Git Bash 사용 권장)
- **장기 룰이 세션에 안 실림**: `~/.claude/CLAUDE.md` 가 그 하네스의 글로벌 메모리 파일로 로드되는지 `/memory` 로 확인. 프로젝트 폴더(`projects/<id>/memory/`)에 다시 넣는 옛 방식으로 회귀하지 말 것 — 중복·drift 원인.
- **중복 로드**: 마이그레이션 후 옛 `projects/<id>/memory/` 를 안 비우면 같은 룰 2번 로드 → 옛 위치 정리.
