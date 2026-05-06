---
name: Update docs/000 after code changes
description: 코드 / 정책 / 구조 변경 시 작업 대상 프로젝트의 docs/000 하네스를 함께 갱신. 변경 이력 + 영향받은 섹션 (디렉터리/라우터/서비스 인덱스/가드/Gotchas) 동기화.
type: feedback
originSessionId: f0b3d85e-85ce-4fce-b620-5fc990ab1f6e
---
코드 변경 (편집/추가/삭제), 정책 변경, 구조 변경 후에는 작업 대상 프로젝트의 `docs/000.harness_guide.md` 를 함께 갱신한다.

**Why:** docs/000 은 Claude 가 작업 시작 전 1차 참조하는 harness. 코드는 바뀌었는데 docs/000 이 stale 하면 다음 세션에서 잘못된 정보를 신뢰하고 작업하게 됨. UI 라벨 / 가드 / 라우터 / 서비스 함수 / 정책 등은 코드와 docs 가 같이 움직여야 함.

**How to apply:**
- 작업 완료 직전 (commit 직전 권장) 에 docs/000 의 어느 섹션이 영향받았는지 점검:
  - 새 파일 / 모듈 추가 → §2 디렉터리 트리 갱신
  - 새 endpoint → §5 라우터 인덱스
  - 새 서비스 함수 → §6 서비스 인덱스
  - UI 라벨 / 컴포넌트 변경 → frontend §3 카드 매트릭스 + §4 라벨 매핑 (해당 있을 때만)
  - 새 가드 / 정책 → §10 Gotchas + (backend) §13 v2 라우팅 표
  - 새 mock 시나리오 → frontend §6 + 03.mock_scenarios.md
  - 모든 변경 → §11 이력에 1 줄 append (날짜·요약·사유)
- backend 정책이 frontend UI 에 영향 (예: 새 필드 노출, 라벨 변경) → 양쪽 docs/000 동시 갱신.
- docs/000 의 라우팅 표 (다른 docs 로의 링크) 도 stale 안 되도록, 새 docs 추가 시 §0 읽기 순서 표에도 entry 추가.
- 트리비얼한 변경 (오타 수정, 주석만 변경) 은 docs/000 갱신 안 해도 됨. 동작/구조/정책 변경에 한정.
- docs/000 갱신 후 본문에 명시된 파일 경로·함수명·라벨이 실제 코드와 일치하는지 grep 으로 1 회 확인.
