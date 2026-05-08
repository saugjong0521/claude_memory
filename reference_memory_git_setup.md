---
name: 메모리 git sync 셋업 (새 PC)
description: 새 PC 에서 메모리 디렉토리를 saugjong0521/claude_memory.git 과 sync 시키기 위한 git 셋업 절차 (clone + config + 인증). Stop hook 사용 X
type: reference
originSessionId: 469e673c-f305-4b08-9202-16e3811f4952
---
# 목적

새 PC 의 Claude 메모리 디렉토리를 saugjong0521/claude_memory.git 에 연결해서 다중 PC 메모리 공유 가능하게 셋업.

commit + push 는 Stop hook 자동화 없이 Claude 가 turn 안에서 직접 처리 (컨펌 받고) — 룰: [feedback_auto_sync_memory.md](feedback_auto_sync_memory.md).

---

# 사전 조건 (PC 1대당 1회)

## 1. 메모리 디렉토리 식별

그 PC 의 `~/.claude/projects/` (Windows: `%USERPROFILE%\.claude\projects\`) 안의 디렉토리 목록 확인 → 현재 사용 중인 프로젝트 폴더 (예: 리눅스 `-home-crypto`, Mac `-Users-brandon`, Windows `C--Users-<Name>`) 안에 `memory/` 가 있어야 함.

## 2. git clone

메모리 디렉토리에서:

```bash
cd ~/.claude/projects/<PROJECT_ID>/memory
git clone https://github.com/saugjong0521/claude_memory.git .
```

또는 디렉토리에 이미 파일이 있다면:

```bash
git init
git remote add origin https://github.com/saugjong0521/claude_memory.git
git pull origin main --rebase
```

## 3. git config (메모리 repo 로컬)

```bash
git -C ~/.claude/projects/<PROJECT_ID>/memory config user.name "saugjong0521"
git -C ~/.claude/projects/<PROJECT_ID>/memory config user.email "saugjong0521@gmail.com"
```

(글로벌 git config 와 별도로 메모리 repo 로컬 config 설정 — author 일관성 유지)

## 4. 인증

push 가능해야 함. 둘 중 하나:

- **SSH key**: GitHub 에 SSH key 등록 + remote 를 SSH URL 로 변경
  ```bash
  git -C <memory-dir> remote set-url origin git@github.com:saugjong0521/claude_memory.git
  ```
- **HTTPS + PAT**: GitHub Personal Access Token 발급 → credential helper 사용 또는 remote URL 에 embed

## 5. settings.json

`~/.claude/settings.json` (Windows: `%USERPROFILE%\.claude\settings.json`) 에 **Stop hook 추가하지 말 것**. 자동 commit 은 컨펌 룰 ([feedback_commit_message_format.md](feedback_commit_message_format.md)) 과 충돌해서 2026-05-08 폐기됨.

---

# 적용 후 검증

1. 메모리 파일 임시 수정 → Claude 한테 commit 처리 요청 → diff 기반 메시지 제안 + 컨펌 → push
2. GitHub saugjong0521/claude_memory main 에 새 commit 도착 확인
3. 다른 PC 에서 `git -C <memory-dir> pull` → 변경사항 도착 확인

---

# 트러블슈팅

- **push 실패 (auth)**: `Permission denied` / `403` → SSH key 또는 PAT 재확인
- **commit author 가 다른 이름**: `git -C <memory-dir> config user.name/email` 재설정 (글로벌과 별도)
- **git not found (Windows)**: PATH 에 `git.exe` 등록 필요 (Git for Windows 설치)
