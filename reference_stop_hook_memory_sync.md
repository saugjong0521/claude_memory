---
name: Stop hook 메모리 자동 sync 셋업 자료
description: 새 PC 에 메모리 자동 commit+push (saugjong0521/claude_memory) 적용할 때 참조하는 OS 별 Stop hook 스니펫 + 사전 조건
type: reference
originSessionId: 469e673c-f305-4b08-9202-16e3811f4952
---
# 목적
새 PC 에서 Claude 메모리 변경을 자동으로 saugjong0521/claude_memory.git 에 push 하도록 Stop hook 셋업. settings.json 은 PC 별 로컬이라 PC 마다 직접 적용 필요.

행동 규칙은 [feedback_auto_sync_memory.md](feedback_auto_sync_memory.md) 참조. 본 파일은 **셋업 절차 + 스니펫 템플릿**.

---

# 사전 조건 (PC 1대당 1회)

1. **메모리 디렉토리 식별**: 그 PC 의 `~/.claude/projects/` (Windows 는 `%USERPROFILE%\.claude\projects\`) 안의 디렉토리 목록 확인 → 현재 사용 중인 프로젝트 폴더 (예: `-home-crypto`, `-Users-brandon`, `C--Users-Brandon`) 안에 `memory/` 가 있어야 함.
2. **git clone**: 메모리 디렉토리에서 `git clone https://github.com/saugjong0521/claude_memory.git .` (또는 `git init` + `git remote add origin` + `git pull`).
3. **git config (메모리 repo 로컬)**:
   ```bash
   git -C <memory-dir> config user.name "saugjong0521"
   git -C <memory-dir> config user.email "saugjong0521@gmail.com"
   ```
4. **인증**: push 가 가능해야 함. 둘 중 하나:
   - SSH key 등록 + remote 를 SSH URL (`git@github.com:saugjong0521/claude_memory.git`) 로 변경
   - HTTPS + GitHub PAT 를 remote URL 에 embed 또는 credential helper 사용

---

# Linux / Mac (bash)

`~/.claude/settings.json` 의 `hooks.Stop` 에 추가. `<PROJECT_ID>` 는 그 PC 의 실제 프로젝트 폴더명으로 치환 (예: `-home-crypto`, `-Users-brandon`).

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "MDIR=$HOME/.claude/projects/<PROJECT_ID>/memory; if [ -d \"$MDIR/.git\" ] && [ -n \"$(cd \"$MDIR\" && git status --porcelain)\" ]; then cd \"$MDIR\" && git add -A && git commit -m \"($(date +%Y%m%d)) update_memory\" >> $HOME/.claude/memory-sync.log 2>&1 && git push origin main >> $HOME/.claude/memory-sync.log 2>&1; fi"
          }
        ]
      }
    ]
  }
}
```

동작: turn 종료 시 → 메모리 dir 변경 있으면 add+commit+push, 로그는 `~/.claude/memory-sync.log`.

---

# Windows (PowerShell)

`%USERPROFILE%\.claude\settings.json` 의 `hooks.Stop` 에 추가. `<PROJECT_ID>` 는 실제 프로젝트 폴더명 (Windows 는 보통 `C--Users-<Name>` 형태).

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell -NoProfile -Command \"$MDIR = \\\"$env:USERPROFILE\\.claude\\projects\\<PROJECT_ID>\\memory\\\"; if ((Test-Path \\\"$MDIR\\.git\\\") -and (git -C $MDIR status --porcelain)) { git -C $MDIR add -A; git -C $MDIR commit -m \\\"($((Get-Date).ToString('yyyyMMdd'))) update_memory\\\" *>> \\\"$env:USERPROFILE\\.claude\\memory-sync.log\\\"; git -C $MDIR push origin main *>> \\\"$env:USERPROFILE\\.claude\\memory-sync.log\\\" }\""
          }
        ]
      }
    ]
  }
}
```

대안 — **Git Bash / WSL 사용 가능 시** Linux/Mac 스니펫 그대로 사용 (단 hook command 앞에 `bash -c '...'` 래핑 또는 Git Bash 를 default shell 로 설정).

---

# 적용 후 검증

1. 메모리 파일 하나 임시 수정 → 저장 → Claude 에 임의 메시지 → turn 종료 후 확인:
   - `~/.claude/memory-sync.log` (Windows: `%USERPROFILE%\.claude\memory-sync.log`) 에 commit + push 로그
   - GitHub 의 saugjong0521/claude_memory main 브랜치에 새 commit
2. 다른 PC 에서 `git pull` → 변경사항 도착 확인
3. 임시 수정 되돌리기 (또 자동 commit 됨)

---

# 트러블슈팅

- **push 실패 (auth)**: log 파일에 `Permission denied` 또는 `403` → 인증 (SSH key / PAT) 재확인
- **hook 자체 안 돌아감**: settings.json 문법 오류 (Claude Code 시작 시 무시될 수 있음) → JSON 검증
- **git not found (Windows)**: PATH 에 `git.exe` 등록 필요 (Git for Windows 설치)
- **commit author 가 Claude / 다른 이름**: `git -C <memory-dir> config user.name/email` 재설정 (글로벌 config 와 별도)
