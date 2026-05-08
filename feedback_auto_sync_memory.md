---
name: Auto-sync memory to git
description: 메모리 변경은 Claude 가 turn 안에서 diff 기반 메시지 제안 + 컨펌 후 직접 commit+push (1차), Stop hook 은 깜빡한 경우 fallback
type: feedback
originSessionId: 3f2c3f1e-54e8-4e89-b366-494e814c2a7c
---
현재 PC 의 메모리 디렉토리 (`~/.claude/projects/<project-id>/memory/`, 예: 리눅스 `-home-crypto`, Mac `-Users-brandon`) 변경은 saugjong0521/claude_memory.git 으로 commit + push.

**Why:** 사용자가 여러 PC 에서 동일한 메모리 컨텍스트를 사용하기 위해 GitHub 동기화. commit 메시지는 변경 내용을 반영해야 함 ([feedback_commit_message_format.md](feedback_commit_message_format.md) §"컨펌 정책") — 따라서 generic 자동 메시지가 아니라 Claude 가 직접 의미 있는 메시지로 commit + 사용자 컨펌 받는 게 1차 플로우.

**How to apply (1차 플로우 — Claude 직접 commit):**
- 메모리 파일 (MEMORY.md / `feedback_*.md` / `user_*.md` / `project_*.md` / `reference_*.md`) Write/Edit 후 turn 끝나기 전에:
  1. `git -C <memory-dir> status --porcelain` + `git -C <memory-dir> diff --name-status` 로 변경 파악.
  2. 컨벤션 메시지 제안: `(yyyymmdd) 동사_파일내용, 동사_파일내용, ...` (`A` → `add`, `M` → `update`, `D` → `remove`).
  3. 사용자에게 메시지 보여주고 **컨펌**.
  4. 컨펌 후 `git add -A` + `git commit -m "..."` + `git push origin main`.
- commit author: `saugjong0521 <saugjong0521@gmail.com>` (memory repo 로컬 git config).
- `Co-Authored-By: Claude` 미포함 ([feedback_commit_no_claude_coauthor.md](feedback_commit_no_claude_coauthor.md)).

**Stop hook (2차 — fallback):**
- `~/.claude/settings.json` 의 Stop hook 이 turn 종료 시 staged 변경분이 남아 있으면 generic 메시지 (`(yyyymmdd) update_memory`) 로 자동 commit + push.
- 1차 플로우가 정상이면 turn 끝에 staged 변경이 없어 hook 은 no-op.
- Claude 가 메모리 변경을 깜빡하고 commit 안 한 경우만 fallback 으로 작동.
- push 실패 시 `~/.claude/memory-sync.log` (Windows: `%USERPROFILE%\.claude\memory-sync.log`) 에 기록.

**새 PC 셋업:**
- 메모리 디렉토리를 saugjong0521/claude_memory 로 clone + 인증 + 그 PC `~/.claude/settings.json` 에 hook 추가.
- 상세는 [reference_stop_hook_memory_sync.md](reference_stop_hook_memory_sync.md) 참조.
- settings.json 은 PC 마다 별도 (sync 대상 아님).
