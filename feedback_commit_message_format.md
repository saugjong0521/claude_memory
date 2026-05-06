---
name: Commit message 포맷 컨벤션
description: 모든 commit 메시지는 `(yyyymmdd) 동사_내용` 형식 + - / _ 규칙 + 다중 변경은 콤마 구분 + Claude co-author 미포함 + 가능하면 커밋 전 컨펌
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

## 컨펌 정책

- **가능하면 커밋 전에 커밋명 컨펌받기** — 사용자에게 제안 후 승인 받고 commit.
- 단, 사용자가 명시적으로 "커밋해줘" 또는 메시지 직접 지정한 경우 컨펌 생략 가능.

## 메시지 결정 시점 (중요)

- commit 메시지는 **작업 완료 후 결과를 보고** 결정. plan / 계획 단계에서 미리 메시지 안을 정하지 X.
- 흐름:
  1. 작업 필요 내용 정리 (plan)
  2. 작업 진행 (코드 변경)
  3. 작업 완료 + 결과 확인
  4. 그 시점에 실제 변경 내용 기준으로 메시지 결정 → 컨펌 → commit
- 이유: plan 단계에 미리 정하면 작업 도중 scope 변경 / 추가 발견 시 메시지가 부정확. 결과 본 후 정확한 메시지 작성 가능.

## 자동화된 commit 도 동일 컨벤션 적용

- Stop hook (메모리 자동 sync) 의 commit 메시지도 본 포맷 따름:
  - `($(date +%Y%m%d)) update_memory` 형태로 (날짜 동적 + 일반 의미).

## Why

- commit log 가 시간 정렬 + 한눈에 무엇이 바뀌었는지 가독.
- `-` / `_` 구분으로 같은 영역 vs 다른 영역 시각적 구분.
- 사용자가 4개 이상 프로젝트 동시 관리 — 일관 컨벤션이 cross-project log 비교에 유리.

## How to apply

- 모든 git commit 명령어 작성 시 본 포맷 검증.
- `Co-Authored-By: Claude` trailer 절대 포함 X.
- 커밋 전 사용자에게 메시지 보여주고 컨펌 받기 (시간 효율 / 명시 지시 시 생략 가능).
