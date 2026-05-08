---
name: 코드 식별자 참조 시 동작 설명 동반
description: 함수명·변수명·테이블명·컬럼명 등 코드 식별자 언급 시 그것이 무슨 동작/책임을 하는지 짧은 설명 함께. docs/000 작성도 동일 원칙
type: feedback
originSessionId: 3f2c3f1e-54e8-4e89-b366-494e814c2a7c
---
코드 식별자 (함수명, 변수명, 테이블명, 컬럼명, endpoint 경로 등) 를 언급할 때 항상 **그것이 무슨 동작/책임을 하는지 짧은 설명**을 함께 제공.

**Why:** 사용자가 여러 프로젝트 (`kstadium-poker256-captain-campain`, `kstadium-poker256-deposit`, `kstadium-referral-develop-backend`, `kstadium-referral-develop-frontend` 등) 를 동시에 관리. 비슷한 이름인데 동작 다른 함수도 많음 (예: `sync_member_wallets` vs `sync_members`). 함수명만 던지면 빠른 컨텍스트 회수 어려움 — 사용자가 "그 함수 뭐하는 거였지?" 다시 물어봐야 하는 비효율 발생.

**How to apply:**
- 답변에서 함수/변수/테이블/컬럼 언급 시 `식별자 (= 동작/책임 한 줄)` 형식.
  - 예: `_post_ticket_once` (= poker256 `/api/addTicket` 으로 1건 전송)
  - 예: `dispatch_attempts` (= 그 dispatch 가 POST 시도된 누적 횟수)
- docs/000 작성 시에도 동일 — 단순 함수 / 테이블 list 가 아니라 "harness + 비즈니스 로직" 통합. 데이터가 어떻게 흐르고, 어떤 결정이 어디서 일어나는지까지 풀어 작성.
- "추가 검토 사항" / "결정 항목" 제시할 때도 항목명만 던지지 말고 풀어서: ❌ `dispatch_attempts 보존 여부` → ✓ `dispatch_attempts (POST 시도 횟수 누적 카운터) — 보존? 또는 새 시즌이라 0 reset?`
- 코드 위치가 의미 있으면 `[filename.py:line](path)` 링크 포함.
- 본 규칙은 답변 / 메모리 / docs 본문 / docstring 에 적용.
- **커밋 메시지는 예외** — `(yyyymmdd) 동사_내용` 형식 ([feedback_commit_message_format](feedback_commit_message_format.md)) 으로 짧게 유지하므로 식별자 동작 설명 생략 가능. 단 commit body (본문) 에 자세한 설명 적을 때는 본 룰 적용.
