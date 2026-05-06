---
name: Auto-sync memory to git
description: 메모리 디렉토리 변경은 Stop hook 이 자동 commit+push 로 saugjong0521/claude_memory 동기화
type: feedback
originSessionId: 3f2c3f1e-54e8-4e89-b366-494e814c2a7c
---
메모리 디렉토리 (`~/.claude/projects/-home-crypto/memory/`) 의 변경은 자동으로 git commit + push 됨.

**Why:** 사용자가 다른 PC 에서도 동일한 메모리 컨텍스트를 사용하기 위해 https://github.com/saugjong0521/claude_memory.git 에 동기화. 매 turn 마다 수동 commit 은 누락 위험.

**How to apply:**
- 메모리 변경 (MEMORY.md / `feedback_*.md` / `user_*.md` / `project_*.md` / `reference_*.md` 의 Write/Edit) 후 별도 commit 명령 불필요 — `~/.claude/settings.json` 의 Stop hook 이 자동 처리.
- commit author: `saugjong0521 <saugjong0521@gmail.com>` (memory repo 의 local git config).
- commit message 에 `Co-Authored-By: Claude` 미포함 (별도 memory rule 준수).
- push 실패 시 `~/.claude/memory-sync.log` 에 기록 — 주기적 확인 또는 수동 해결.
- 다른 PC 에 동일 동기화 적용하려면 그 PC 의 `~/.claude/settings.json` 에도 같은 Stop hook 추가 필요 (settings.json 자체는 sync 대상 아님).
