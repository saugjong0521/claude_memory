---
name: vip-event-ordering-not-max-id
description: "kstadium-referral-backend 의 vip_membership_event 최신 행 조회 시 MAX(id) 금지 — application 은 (effective_at, block_number, log_index, id) DESC 정렬 사용"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d46e4ac-70e3-4d2c-ab0d-104af852a354
---

`vip_membership_event` 테이블에서 wallet 별 "현재 VIP 상태" 를 SQL 로 구할 때 `MAX(id)` 또는 `id DESC LIMIT 1` 만 사용하면 안 됨. 반드시 application 과 동일한 정렬을 써야 함:

```sql
ORDER BY effective_at DESC,
         COALESCE(block_number, -1) DESC,
         COALESCE(log_index, -1) DESC,
         id DESC
LIMIT 1
```

또는 window function `ROW_NUMBER() OVER (PARTITION BY wallet_address ORDER BY ... )` 사용.

**Why:** [vip_membership_service.py:78-87](../../kstadium-referral-backend/app/services/referral/vip_membership_service.py#L78-L87) 의 `_current_vip_status` 가 위 4 컬럼 ordering 사용. id 와 effective_at 이 불일치하는 케이스 존재 (예: manual grant 가 과거 effective_at 으로 나중에 INSERT 되면 id 는 크지만 effective_at 은 옛날). 2026-05-27 0x3635e701... wallet 조회 시 MAX(id) 로 manual 행 (id=63, effective_at=2025-12-01) 을 잡아서 더 최신 auto 행 (id=35, effective_at=2026-05-13) 을 놓치고 "VIP 아님" 으로 잘못 판단한 사례 발생.

**How to apply:** VIP 현황 / "현재 source 가 auto 인지 manual 인지" / "현재 grant 인지 revoke 인지" 조회 SQL 작성 시 항상 위 4-tuple ordering 사용. 단일 컬럼 ORDER BY 금지. dual-write 패턴이 아닌 audit-log 패턴 테이블 (이력 누적) 은 일반적으로 같은 원칙 적용.

관련: [[docs_as_guide_code_as_truth]] (docs 의 status 표기 맹신 X, 코드의 ordering 으로 검증).
