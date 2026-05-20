---
name: ""
metadata: 
  node_type: memory
  originSessionId: bd781d25-8a41-4e59-bf23-afcb25e5fb66
---

UPSERT 패턴은 AUTO_INCREMENT counter 를 INSERT 시도 시점에 발급 + UPDATE 분기에서도 rollback 안 함 → 빈도 높은 sync / polling 코드에서 실 row 수의 수십~수백 배 id 누적.

**Why:** 2026-05-20 발견 — explorer_holder_transfers 의 60초 polling × 600 row UPSERT → 30일 만에 id 47,455 (= 실 row 616, 평균 77배). transaction_valuations 6.1%, available_reward_contribution 0% (= 호출 빈도 낮음) 도 같은 패턴 잠재 위험.

**How to apply:**
- UPSERT 가 idempotent + atomic 이라 직관적이나 AUTO_INC overflow 부작용 인지
- 빈도 높은 sync / polling 코드 = 변경 감지 분기 적용:
  - 신규 row → INSERT (= AUTO_INC +1 정상)
  - 기존 row + 값 변경 → UPDATE (= id 안 건드림)
  - 기존 row + 값 동일 → skip
- 변경 감지용 batch SELECT 가 추가 query 1회 비용이지만 id overflow 회피 + 일반적으로 더 빠름 (= 대부분의 row 가 skip)
- 1회성 / 빈도 낮은 INSERT 는 UPSERT OK (= 누적 영향 적음)

**Related:**
- 본 case: kstadium-referral-backend docs/019 (= 새벽 작업 예정) + app/services/common/explorer_sync_service.py
- MySQL / PostgreSQL 일반 RDBMS 패턴 — 다른 프로젝트에도 동일 적용 가능
- 코드 작성 시 빈도 높은 path 면 UPSERT 대신 변경 감지 분기 고려
