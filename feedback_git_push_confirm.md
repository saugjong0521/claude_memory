---
name: git-push-confirm
description: "git commit 까지는 자동 진행 OK, push 전엔 반드시 사용자 confirm 받기. Claude 가 자발적 push 금지."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: bd781d25-8a41-4e59-bf23-afcb25e5fb66
---

git commit 후 origin 으로 push 진행 시 **반드시 사용자 confirm 받는다**. Claude 가 자발적 push 진행하지 않는다.

**Why:** 2026-05-20 session 에서 docs/019 plan commit + push 를 자동으로 진행 → 사용자가 "push 시에 내 허락 받으라 했는데?" 지적. push 는 remote 영향 (= 다른 협업자 / CI / 배포 자동화 trigger) + 외부 가시화 + revert 어려움. commit 은 local 만이라 안전, 단 push 는 사용자 결정.

**How to apply:**
- git add / commit 까지는 작업 흐름 따라 진행 OK
- `git push` 전에 명시적으로 "push 진행할까요?" 컨펌 받기
- 예외: 사용자가 명시적 "commit + push" 또는 "끝까지 진행" 또는 같은 의도 명시 시에만 자동 진행
- 메모리 룰 [[commit_message_format]] 과 짝 (= commit 자체도 가능하면 컨펌, push 는 의무 컨펌)
