---
name: docs는 지침서, 코드가 진실
description: docs/하네스는 어디를 읽을지 안내하는 지침서로만 사용. 모든 검증·테스트·status 확인은 코드 직접 read 로 진행. docs 의 ✅/🔴/구현 완료/gap 등 status 표기 맹신 금지
type: feedback
originSessionId: 469e673c-f305-4b08-9202-16e3811f4952
---
프로젝트의 `docs/000.harness_guide.md` 와 그 외 docs 는 **참고용 지침서** — 프로젝트 구조 이해 + 어느 파일/섹션을 읽어야 하는지 안내하는 역할. **진실의 출처가 아님**.

**Why:** 2026-05-08 사고 — backend `docs/000 §15.16` 의 S26 검증 매트릭스가 "🔴 code gap (구현 필요)" 로 표기돼 있었지만 실제로는 같은 날 (2026-05-03) Phase 3 commit 으로 이미 구현 완료된 상태. §11 changelog 와 §15.16 의 drift. Claude 가 §15.16 을 그대로 신뢰해 docs/016 초안에서 "구현 안 됨" 으로 잘못 분류 → 사용자 지적으로 정정. 모든 docs 는 작성 시점 snapshot 이고 코드는 그 후 변경될 수 있음.

**보강 (2026-08-12): 내가 방금 쓴 계획 문서도 예외가 아니다.**

같은 세션 안에서 내가 작성한 스펙(docs/016·023 의 항목별 "작업 내용")이 **착수 시점에 이미 6곳 틀려** 있었다:
잔존 자체구현이 6곳→**4곳**(그 사이 다른 항목이 처리) / 스펙이 치환 대상이라 지목한 2곳은 **SQL 식이라 파이썬 헬퍼로 치환 불가** /
canonical 이라던 함수가 실제로는 필요한 5-tuple 을 **안 돌려줌** / 지시대로 옮기면 **순환 import** 2건 /
"canonical 로 승격" 하라던 함수는 이미 **호출처 0 인 dead code**. 매번 착수 재조사로 잡았지만,
스펙대로 밀었으면 기동이 깨지거나 자금 경로 동작이 바뀔 건이 섞여 있었다.

→ **계획 문서는 "무엇을 왜 하려 했는지" 의 기록이지 실행 지시가 아니다.** 항목 착수 **직전**에
대상 심볼·파일·개수를 다시 grep 해서 스펙과 대조하고, 어긋나면 **스펙이 아니라 코드를 따르고 그 차이를 문서에 남긴다**.

**How to apply:**

1. **docs 의 역할** = "어디를 읽어야 할지" 알려주는 지도:
   - `docs/000` 의 §디렉터리 / §라우터 인덱스 / §모델 책임 등 → 해당 파일·함수 위치 파악용
   - `docs/0NN` 도메인 문서 → 정책 / 시나리오 / 의도 파악용
   - status 표기 (✅/🔴/⚠️/구현 완료/code gap/policy 미정 등) → **참고만**, 단정 X

2. **검증·확인 작업은 항상 코드 직접 read**:
   - "X 가 구현됐는가?" → 해당 service / router 파일 직접 grep + 함수 본문 read
   - "Y 가드가 작동하는가?" → 가드 함수 코드 + 테스트 코드 직접 read
   - "Z 시나리오 정합 여부" → 코드 흐름 추적 (process_v2_deposit → cutoff_fix 등 invocation chain) 직접 검증
   - **테스트 실행도 코드 기반** — docs 가 "테스트 통과" 라 적혀 있어도 실제로 `pytest` 돌려서 확인

3. **docs ↔ 코드 drift 발견 시**:
   - 코드를 신뢰. docs 갱신.
   - 갱신 후 §변경이력 / §관련 매트릭스 양쪽 동기 ([feedback_update_docs_000_after_changes.md](feedback_update_docs_000_after_changes.md)).

4. **새 docs 작성 시**:
   - 매트릭스 / 인덱스 / 카탈로그 행마다 코드 grep + 본문 read 로 채움 ([feedback_no_guess_in_docs.md](feedback_no_guess_in_docs.md)).
   - 다른 docs 의 status 표기를 카피하지 말고 코드로 재검증.

**구체 적용 예 (2026-05-08 사고 분석)**:

| 단계 | 잘못한 행동 | 옳은 행동 |
|------|-----------|----------|
| 1 | docs/000 §15.16 의 "🔴 S26-A-1 code gap" 표기 신뢰 | `grep -n "_cancel_requested_claims\|add_exclusion" app/services/v2/` + 본문 read 로 실제 상태 확인 |
| 2 | §11 changelog 의 "구현 완료" 와 §15.16 의 "🔴" 모순을 알아차리지 못함 | docs 내부 cross-section drift 감지 시 코드로 어느 쪽이 맞는지 검증 |
| 3 | 사용자 지적 받기 전까지 잘못된 정보 그대로 docs/016 작성 | 매트릭스 작성 전에 행마다 코드 검증 (§4 룰) |

**핵심**: docs 는 **출발점** (어디 읽을지), 코드는 **종착점** (실제 동작). 이 순서를 뒤집지 말 것.

**런타임 확장**: 실행 중 서비스 (재시작/중지/리로드/기동) 는 repo 의 스크립트/Makefile/Dockerfile 이 아니라 라이브 supervisor (systemd 등) 가 진실 → [feedback_verify_runtime_supervisor_before_restart](feedback_verify_runtime_supervisor_before_restart.md).
