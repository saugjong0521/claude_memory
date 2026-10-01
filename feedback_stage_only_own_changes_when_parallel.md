---
name: 병렬 세션 중엔 내 세션 변경만 경로 지정해 커밋
description: 같은 repo 에 다른 Claude 세션이 동시에 작업 중이면 git add -A / commit -a 금지. 내 세션이 만든 파일만 경로로 명시 stage 하고, 기능 단위로 나눠 커밋
metadata:
  type: feedback
---
같은 저장소에서 **다른 세션이 병렬로 작업**하고 있을 때, 커밋은 **내 세션이 만든/고친 경로만 명시적으로 stage** 한다. `git add -A`, `git add .`, `git commit -a` 는 쓰지 않는다.

**Why:** 사용자 (2026-09-29, scope-validate): "병렬적으로 작업 돌아갈 것임 다른 세션과 일할것을 잘 분리해서 현재 세션에서 작업하는 내용에 대해서만 관리하도록". `git add -A` 는 다른 세션이 편집 중인 파일을 **반쯤 쓰인 상태로 남의 커밋에 끌어넣는다** — 그 세션은 자기 변경이 이미 커밋된 줄 모르고, 되돌리기도 서로 얽힌다.

**How to apply:**
- 커밋 전 `git status --short` 로 전체를 보고, **그중 내가 만든 것만** 골라 `git add <경로> <경로>` 로 stage.
- 내 것인지 판단 기준 = 이 세션에서 내가 직접 편집했거나, 내가 띄운 작업자 에이전트에게 지시한 파일. 그 외는 남긴다.
- 한 파일에 두 기능의 변경이 섞였으면 (예: 공용 `styles.css`) hunk 를 나눠 각 커밋에 넣는다. `git add -p` 는 이 환경에서 대화형이 안 되니, **임시로 한쪽 블록을 뺀 상태로 저장 → commit → 원본 복원** 으로 처리한다.
- 기능이 둘이면 커밋도 둘. 한 커밋에 섞지 않는다.
- 커밋 메시지는 [[feedback_commit_message_format]] 을 따르고 **커밋 전 컨펌**을 받는다. push 는 [[feedback_git_push_confirm]].
- 병렬 작업 중엔 `git checkout` / `reset --hard` 를 더욱 피한다 — [[feedback_git_branch_switch_destroys_orphan_tracked_files]] 의 위험이 남의 작업물까지 날린다.
- 작업자 에이전트를 띄울 때는 **파일이 겹치지 않게** 쪼개고, 프롬프트에 "이 파일만 수정, 다른 파일 금지" 와 "커밋 안 된 남의 변경을 되돌리지 마라" 를 명시한다.
