---
name: full-columns-in-classification-sql
description: 카테고리 분류용 SQL 은 sanity check 가능한 모든 컬럼 (= exclusion / pre_genesis / balance_row / 등) 을 한 query 에 collect 후 분류. 부분 query 결과로 단정 금지
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d46e4ac-70e3-4d2c-ab0d-104af852a354
---

SQL 로 wallet / row 카테고리 분류 (= docs 의 매트릭스 / 분포 표 작성 시) 진행할 때, **분류 결정에 영향 줄 수 있는 모든 컬럼**을 한 query 에 collect. 부분 query 후 카테고리 단정 금지.

**Why:** 2026-05-27 docs/013 §2.3 의 lifetime=0 wallet 카테고리 분류 시:
- 첫 query 가 `user_reward_balance` row + `lifetime` + `pre_genesis_cnt` 만 — `deposit_exclusion_tx` JOIN 컬럼 누락
- 결과: 0x260c1f9b... 의 exclusion 상태를 "exclusion 없음" 으로 잘못 분류
- 그 결과를 docs §2.3 카테고리에 반영 → 별도 backfill 결정 필요한 case 로 사용자에게 옵션 제시
- 사용자가 "둘 다 100이 안 돼서 exclusion 추가된 거 아닌가? 물어보는 이유가 뭐임?" 정정 + 답답함 표명
- 재확인 SQL 로 두 wallet 모두 운영자 수동 exclusion 등록 상태 발견 — 분류 처음부터 잘못

**How to apply:** 분류 SQL 작성 시 다음 sanity check 컬럼들을 모두 포함:
- `deposit_exclusion_tx` JOIN 결과 (= 정책상 제외 여부)
- `block_number <= GENESIS_BLOCK` (= pre-genesis 여부)
- `user_reward_balance` row 존재 + lifetime
- `referral_users` row 존재 + referral_code
- 관련 컬럼 (= context 별 추가)

→ 카테고리별 wallet 의 모든 sub-state 가 단일 SELECT 의 row 로 보이도록 작성. 그 후 카테고리 결정. 메모리 룰 [[no_guess_in_docs]] 의 SQL 본질화 버전.

관련: [[no_guess_in_docs]] (= docs 표 작성 시 grep + 본문 read), [[docs_as_guide_code_as_truth]] (= docs status 맹신 금지).
