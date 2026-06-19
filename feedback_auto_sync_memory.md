---
name: Sync memory to git (manual, with confirmation)
description: 메모리 변경은 Claude 가 turn 안에서 diff 기반 메시지 제안 + 사용자 컨펌 후 직접 commit + push. Stop hook 은 사용 안 함 (컨펌 룰과 충돌)
type: feedback
originSessionId: 3f2c3f1e-54e8-4e89-b366-494e814c2a7c
---
현재 PC 의 장기 메모리 디렉토리 (`~/.claude/claude_memory/`) 변경은 saugjong0521/claude_memory.git 으로 commit + push. (구조: [reference_memory_git_setup.md](reference_memory_git_setup.md) — 장기는 프로젝트 폴더 `projects/<id>/memory/` 가 아니라 `~/.claude/claude_memory/` 에 둠)

**Why:** 사용자가 여러 PC 에서 동일한 메모리 컨텍스트를 사용하기 위해 GitHub 동기화. commit 메시지는 변경 내용을 반영해야 하고 컨펌이 필요함 ([feedback_commit_message_format.md](feedback_commit_message_format.md) §"컨펌 정책"). 자동 hook 은 turn 종료 시점에 fire 하는데 그 시점이 곧 사용자 응답 대기 시점이라 컨펌 받을 기회가 없음 → **hook 비활성, Claude 가 매번 직접 처리**.

**How to apply (Claude 직접 commit):**
- 메모리 파일 (MEMORY.md / `feedback_*.md` / `user_*.md` / `project_*.md` / `reference_*.md`) Write/Edit 한 turn 안에서:
  1. `git -C <memory-dir> status --porcelain` + `git -C <memory-dir> diff --name-status` 로 변경 파악.
  2. 컨벤션 메시지 제안: `(yyyymmdd) 동사_파일내용, 동사_파일내용, ...` (`A` → `add`, `M` → `update`, `D` → `remove`).
  3. 사용자에게 메시지 보여주고 **컨펌**.
  4. 컨펌 후 `git add -A` + `git commit -m "..."` + `git push origin main`.
- commit author: `saugjong0521 <saugjong0521@gmail.com>` (memory repo 로컬 git config).
- `Co-Authored-By: Claude` 미포함 ([feedback_commit_no_claude_coauthor.md](feedback_commit_no_claude_coauthor.md)).

**누락 안전망 (Claude 의식):**
- 새 turn 시작 시 `git -C <memory-dir> status --porcelain` 으로 uncommitted memory 변경 있는지 먼저 체크.
- 있으면 직전 turn 에 commit 누락된 것 — 본 turn 의 1차 작업으로 메시지 제안 + 컨펌 + commit + push 처리 후 사용자 요청에 응답.
- 즉 누락 방지를 hook 자동화 → Claude 의식으로 이관.

**Stop hook 사용 X:**
- 과거 (2026-05-06 ~ 2026-05-08) `~/.claude/settings.json` 의 Stop hook 이 turn 종료마다 generic 메시지 (`auto: memory update <iso>`, 이후 `(yyyymmdd) update_memory`) 로 자동 commit 했음 — 컨펌 룰과 구조적 충돌로 2026-05-08 제거.
- 새 PC 셋업 시에도 hook 설정하지 말 것 (settings.json 의 hooks.Stop 비워두기).

**새 PC 셋업:**
- 메모리 디렉토리를 saugjong0521/claude_memory 로 clone + git config (saugjong0521 / saugjong0521@gmail.com) + 인증 (SSH key 또는 PAT).
- settings.json 은 PC 마다 별도 (sync 대상 아님). hook 항목 추가 X.
