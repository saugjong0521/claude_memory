---
name: Phase 별 진행은 협업 (혼자 자동 진행 금지)
description: 다단계 작업/phase 가 정의된 프로젝트에서 사용자가 단계별로 데이터·결정 입력 — 자동으로 다음 phase 로 진행 금지
type: feedback
originSessionId: d01e2a5a-7724-414e-bba0-1d970683cd50
---
다단계 phase 작업 (예: kstadium-referral 의 013 doc Phase A→G 운영 흐름) 에서는 **각 phase 마다 사용자가 데이터·결정을 직접 입력**.

**Why**: phase B (운영 그룹 시드), C (부트스트랩 데이터), E (마이그레이션 트리거) 등은 운영자만 알 수 있는 데이터·시점에 의존. 사용자가 "phase 대로 진행" 이라고 말해도 자동 진행 금지.

**How to apply**:
- 각 phase 시작 시 멈추고 사용자에게 입력 요청 ("Phase B: receiver_auto_groups 어떤 값으로?", "Phase C: 유저 JSON 어디 있나?").
- 다음 phase 진행 전 사용자 응답 대기.
- 사용자가 "ㅇㅇ 진행" 같은 단순 동의만 하면 → 다음 phase 의 첫 단계 (예: 명령 실행 가이드 또는 데이터 입력 요청) 만 제시하고 멈춤.
- 단계 자동 점프 금지 (P6-1 → P6-3 → P6-5 같은 연속 코드 변경 X).

이전 사고 (2026-04-27): 사용자가 "ㅇㅇ 이제 페이즈대로 업데이트 시작" 이라고 했을 때 P6-1, P6-3, P6-5 를 모두 자동으로 코드 작성 + commit 시도 → 사용자가 "내가 작동을 해야 하잖아" 라고 정정. 미commit 변경 모두 revert.
