---
name: Check docs/000 before code tasks
description: 코드 작업 시작 전 작업 대상 프로젝트의 docs/000 (또는 docs/00) 존재 여부 확인 + 있으면 먼저 읽기. UI 라벨 의역 등 가드 정책을 1차 참조.
type: feedback
originSessionId: f0b3d85e-85ce-4fce-b620-5fc990ab1f6e
---
코드 작업이나 프로젝트 관련 질문을 받으면, 작업 시작 전에 작업 대상 프로젝트 폴더에 `docs/000.*` (또는 일반화: `docs/00*` 첫 번호) 파일이 있는지 확인하고, 있으면 먼저 읽고 진행한다.

**Why:** 직전 라운드 (2026-04-30) 에 사용자가 frontend "지급 예정 리워드" 라는 UI 라벨로 요청했는데, Claude 가 backend 필드 `pending_reward` 로 자의적 의역해 두 라운드를 잘못 작업. 원인 = "UI 라벨 ↔ field 매핑" 이 docs 에 분산되어 있어 작업 전 참조 안 된 것. docs/000 이 프로젝트 별로 가드 정책 + 라우팅 + 의역 금지 룰을 1차 인덱싱하는 harness 문서 역할.

**How to apply:**
- 사용자가 코드 작업 (편집/추가/삭제) 또는 프로젝트 정책 관련 질문을 하면, 작업 대상 폴더의 `docs/` 디렉토리 확인.
- 우선순위: `docs/000.harness_guide.md` → `docs/000.*` → `docs/00.*` → `docs/00*`.
- 있으면 먼저 Read. 작업 시작 전 §10 (Gotchas) + §3~5 (가드/라벨 매핑) 1 회 확인.
- 알려진 프로젝트 매핑:
  - `/home/crypto/kstadium-referral-develop-backend` → `docs/000.harness_guide.md` (v2 정책 §13 + 가드 §10 18~24)
  - `/home/crypto/kstadium-referral-develop-frontend` → `docs/000.harness_guide.md` (UI 라벨 매핑 §4 + 카드 가시 조건 §3)
- 트리비얼한 질문 (단순 명령어, 시각, 일반 정보) 에는 적용 안 해도 됨. 코드/정책 영향이 있는 작업에 한정.
- 시간이 지나 docs/000 이 stale 할 수 있으니, 코드와 docs 가 충돌 시 코드를 신뢰하고 docs 를 갱신.
