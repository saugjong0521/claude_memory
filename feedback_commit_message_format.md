---
name: Commit message 포맷 컨벤션
description: 모든 commit 메시지는 `(yyyymmdd) 동사_내용` 형식 + - / _ 규칙 + 다중 변경은 콤마 구분 + Claude co-author 미포함 + **반드시 커밋 전 사용자 컨펌 받기**
type: feedback
originSessionId: 3f2c3f1e-54e8-4e89-b366-494e814c2a7c
---
모든 git commit 메시지는 다음 컨벤션을 따름.

## 포맷

```
(yyyymmdd) 동사_변경내용
```

예: `(20260506) update_member-db_full-sync-logic`, `(20260506) add_harness.md`

## 규칙

1. **날짜 prefix**: `(yyyymmdd)` — 8자리 (예: `(20260506)`).
2. **동사**: `update`, `add`, `fix`, `remove` 등 변경 의도 동사로 시작.
3. **구분자**:
   - `-` (하이픈): 의미가 **이어지는** 단어 사이 — 한 개념을 표현
     - 예: `mail-address`, `vip-username-column`, `from-db`, `member-db_full-sync-logic`
   - `_` (언더스코어): 의미가 **안 이어지는** 단어 사이 — 다른 영역 구분
     - 예: `update_mail-address` (동사 ↔ 대상)
     - 예: `remove_vip-username-column_from-db` (동사 ↔ 대상 ↔ 위치)
4. **수준**: 적당히 — "어떤 내용이 변경됐는지" 알 수 있는 정도. 너무 짧거나 너무 세세하지 않게.
5. **다중 변경**: 한 commit 에 2개 이상의 변경이 묶이면 콤마(`,`) 로 구분
   - 예: `(20260506) update_alert-logic, fix_deposit-sync-multiple_error`
   - 예: `(20260506) add_commit-message-convention, update_auto-sync-hook`
6. **Co-Authored-By: Claude 미포함** (별도 메모리 룰: feedback_commit_no_claude_coauthor).

## 컨펌 정책 (= 의무)

- **모든 commit 전에 commit 메시지 컨펌 받기**. Claude 가 자발적 commit 진행 금지.
- 흐름:
  1. 작업 완료
  2. Claude 가 commit 메시지 안 제안 (= 본문 포함)
  3. 사용자 컨펌 (= "ㅇㅋ" 또는 메시지 수정 지시)
  4. commit 실행
- 예외: 사용자가 명시적으로 "커밋 메시지 X 로 커밋해줘" 처럼 메시지 직접 지정 시.
- `git push` 는 별도 룰 ([feedback_git_push_confirm.md](feedback_git_push_confirm.md)) — commit 컨펌과 별개로 push 도 컨펌 필수.

**Why (강화 이유):** 2026-05-20 session 에서 Claude 가 commit 메시지 + push 모두 자발 진행 → 사용자 두 차례 지적 ("commit명도 내 허락 받으라 했는데?"). "가능하면" 표현으로는 부족 → "반드시" 로 의무화.

## 메시지 결정 시점 (중요)

- commit 메시지는 **작업 완료 후 결과를 보고** 결정. plan / 계획 단계에서 미리 메시지 안을 정하지 X.
- 흐름:
  1. 작업 필요 내용 정리 (plan)
  2. 작업 진행 (코드 변경)
  3. 작업 완료 + 결과 확인
  4. 그 시점에 실제 변경 내용 기준으로 메시지 결정 → 컨펌 → commit
- 이유: plan 단계에 미리 정하면 작업 도중 scope 변경 / 추가 발견 시 메시지가 부정확. 결과 본 후 정확한 메시지 작성 가능.

## 자동 hook commit 사용 X

- 과거 메모리 sync 용 Stop hook 이 자동 commit 했으나 컨펌 룰과 구조적 충돌로 2026-05-08 폐기.
- 메모리 변경 포함 모든 commit 은 Claude 가 turn 안에서 메시지 제안 + 컨펌 + 직접 처리 ([feedback_auto_sync_memory.md](feedback_auto_sync_memory.md)).

## Why

- commit log 가 시간 정렬 + 한눈에 무엇이 바뀌었는지 가독.
- `-` / `_` 구분으로 같은 영역 vs 다른 영역 시각적 구분.
- 사용자가 4개 이상 프로젝트 동시 관리 — 일관 컨벤션이 cross-project log 비교에 유리.

## How to apply

- 모든 git commit 명령어 실행 전 메시지 안 사용자에게 보여주고 컨펌 받기 (= 의무).
- 본 포맷 (= `(yyyymmdd) 동사_내용`) 검증.
- `Co-Authored-By: Claude` trailer 절대 포함 X.
- 명시적 지시 시에만 컨펌 생략.
