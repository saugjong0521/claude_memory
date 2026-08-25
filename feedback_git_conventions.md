---
name: git_conventions
description: commit 메시지 `(yyyymmdd) 동사_내용` 영어 + `-`/`_` 규칙 + Claude co-author 금지 + commit 메시지·push 는 사용자 컨펌. 같은 작업 흐름에서 이미 허락받은 뒤의 후속 커밋·push 는 고지만
metadata:
  type: feedback
---

## 메시지 포맷

`(yyyymmdd) 동사_변경내용` — 예 `(20260506) update_member-db_full-sync-logic`, `(20260825) add_merchant_recruit_track`.
- 동사: `add` / `update` / `fix` / `remove` / `refine` … 영어만 (2026-07-22 "commit문구는 영어로").
- `-` = 한 개념 안의 단어 연결(`mail-address`), `_` = 영역 구분(`update_mail-address`, `remove_x_from-db`).
- 여러 변경이 한 커밋이면 `,` 로 나열. 수준은 "무엇이 바뀌었는지" 알 정도.
- **`Co-Authored-By: Claude` 트레일러 금지.**
- 메시지는 작업 **완료 후 결과를 보고** 정한다 (플랜 단계에서 미리 정하지 않음 — scope 가 바뀐다).

## 컨펌 (= 의무)

- **commit 메시지와 push 는 사용자 컨펌 후** — 자발적 commit·push 금지 (2026-05-20 "commit명도 내 허락 받으라 했는데?", "push 시에 내 허락 받으라 했는데?"). commit 은 로컬이라 되돌리기 쉽지만 push 는 remote·CI·배포를 건드린다.
- 흐름: 작업 완료 → 메시지 제안 → 컨펌("ㅇㅋ"/"ㄱㄱ"/수정 지시) → commit → "push 할까요?" → push.
- **같은 작업 흐름의 후속 수정**(리뷰 반영·문구 조정처럼 방금 허락받은 커밋의 연장)은 매번 다시 묻지 않고 **메시지·push 를 고지하며 진행**한다 — 단, 한 번이라도 "왜 push 했냐" 류 지적이 나오면 그 세션은 다시 매번 컨펌. 새 작업 흐름·prd/main 대상은 항상 컨펌. (2026-08-25 세 번 연속 "ㄱㄱ" 뒤 네 번째도 물어봄 — 과잉)
- 사용자가 메시지를 직접 지정하거나 "commit + push" / "끝까지" 라고 하면 그대로 진행.

## 메모리 repo (`~/.claude/claude_memory/`)

- 메모리 파일을 바꾼 turn 안에서 `git status` → 메시지 제안(`(yyyymmdd) add_x, update_y, remove_z`) → 컨펌 → commit + `push origin main`. author `saugjong0521 <saugjong0521@gmail.com>`.
- Stop hook 자동 커밋은 컨펌 룰과 충돌해 2026-05-08 폐기 — 새 PC 에서도 hook 넣지 않는다. 새 turn 시작 시 uncommitted 메모리 변경이 있으면 먼저 처리.
- 셋업 절차·3계층 구조는 [[reference_memory_git_setup]].

관련: [[git_branch_switch_destroys_orphan_tracked_files]] (checkout/reset 전 `.env*` 백업)
