---
name: git-checkout-reset-tracked
description: "branch 전환 (`git checkout`/`git switch`) 또는 `git reset --hard` 시, 옛 ref 에서 tracked 였던 파일이 새 ref 에 tracked 안 되면 working tree 에서 자동 삭제. .gitignore 보호 무관. checkout/reset 전 untracked sensitive 파일 (= .env*) 백업 + Claude 의 일반화 단정 금지"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 29dec362-8e5f-46b6-822b-167254262f4e
---

branch 전환 (`git checkout`, `git switch`) 또는 `git reset --hard` 시, **옛 ref 에서 tracked 였던 파일이 새 ref 에 tracked 안 되면 working tree 에서 자동 삭제**된다. `.gitignore` 에 추가됐어도 옛 commit 의 tracked 상태는 무관.

**Why:** 2026-05-22 사고. PR rebase merge 후 다음 흐름:
1. `git checkout main` (= local main HEAD=59c301f, 2026-04-08 — 그 시점 .env.dev tracked) → 옛 main 의 .env.dev 가 silent 하게 working tree 덮어쓰기 (= 사용자 vi 편집 내용 소실)
2. `git reset --hard origin/main` (= 새 origin/main, .env.dev ignored / untracked) → 옛 tracked → 새 untracked 전환이라 working tree 에서 **.env.dev 삭제**

결과: 사용자 `.env.dev` 가 사라짐. Claude 가 두 번 잘못 일반화:
- 1차: "git operations 는 ignored 파일 못 건드림" — 옛 ref 의 tracked 상태 무시
- 2차: "본 세션 시작 시점에 이미 없었음" — pytest fail 을 곧 파일 없음으로 단정. 사용자가 shell history (= `vi .env.dev` 다수) 로 직접 증거 제공해도 인정 안 함

**How to apply:**

### A. `git checkout` / `git switch` / `git reset --hard` 전 untracked sensitive 파일 백업
운영 환경의 `.env*` / secret 파일은 사라지면 복구 어려움. branch 전환 전 강제 백업:
```bash
cp .env.dev .env.prod /tmp/.env-backup-$(date +%Y%m%d-%H%M%S)/  2>/dev/null || true
```

### B. 옛 ref ↔ 새 ref 의 tracked 파일 diff 확인
checkout/reset 전 차이 인지:
```bash
git diff --name-status <old-ref>..<new-ref>
# D 상태 (= delete) 행에 .env/secret 있으면 working tree 의 그 파일이 삭제됨
```

### C. 본인 working tree 의 untracked 파일이 다른 ref 에 tracked 였는지 의식
```bash
git log --all --diff-filter=DA --name-only -- .env.dev
# 결과 있으면 → branch 전환 시 silent overwrite + 삭제 위험
```

### D. Claude 가 일반화 단정 금지
다음 형식의 단정 위험:
- ❌ "git operations 는 ignored 파일 못 건드림" — 옛 ref 의 tracked 상태 무관 아님
- ❌ "untracked 면 git 영향 없음" — 옛 ref tracked 시 영향
- ❌ "pytest fail = 파일 없음" — partial 파일 / cwd mismatch / env loader path 등 다양

검증 안 된 일반화는 사용자 직접 증거 (= shell history, IDE 흔적, 사용자 기억) 와 충돌 시 사용자 우선시. 즉답 단정 전 git/file system 의 실제 상태 명령으로 확인 ([[feedback_docs_as_guide_code_as_truth]]).

### E. 다른 branch 의 옛 commit HEAD checkout 시 특히 주의
local branch 가 오래 stale 한 경우 (= 본 사고 시 local main 이 v0.9 옛 commit 가리킴) checkout 만 해도 옛 working tree 적용. 사용자 환경의 untracked 변경이 silent 하게 덮여쓰기 가능. 그 후 force-pull 또는 reset --hard 시 한 번 더 위험.

대안 패턴:
```bash
# 위험: 옛 local main 으로 checkout 후 reset --hard
git checkout main && git reset --hard origin/main

# 안전: working tree 변경 없이 local main 만 새 commit 으로 forward
git fetch origin
git update-ref refs/heads/main origin/main   # working tree 영향 0
# 그 후 필요시 checkout
git checkout main
```
또는:
```bash
# fast-forward only (= local main 이 origin/main 의 ancestor 일 때만 진행, 그 외 abort)
git checkout main && git merge --ff-only origin/main
```
