---
name: memory-dir
description: ~/.claude/.../memory/ 글로벌 dir 는 Claude 의 일반 작업 process/생각 절차/사용자 선호(=feedback 룰)만. 특정 프로젝트에만 해당하는 사실(테스트 실행법·구조·정책·gotcha)은 그 프로젝트 repo 안(docs/000 하네스·README·CLAUDE.md 등)에 기록
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 15b16684-6cc8-4e06-817e-6df385c62690
---

글로벌(장기) 메모리 디렉토리 `~/.claude/claude_memory/` (모든 세션에서 `~/.claude/CLAUDE.md` 가 import) 는 **Claude 의 일반 작업 process / 생각 절차 / 사용자 선호 (= feedback 룰)** 전용. 특정 프로젝트에만 해당하는 사실은 여기 저장하지 않는다.

**Why:** 2026-05-29, kstadium-referral-backend 의 pytest 실행법(`DATABASE_URL` env)을 글로벌 memory 에 `reference` 로 저장했더니 사용자가 정정 — "프로젝트에 대한 메모리는 프로젝트로 가야 함. 해당 memory dir 는 claude 에 대한 기본 생각/절차 내용이라 거기 업데이트되면 안 됨." 글로벌 memory 는 모든 작업에 공통 로드되는 일반 지침 인덱스(MEMORY.md)라, 프로젝트 고유 사실이 섞이면 노이즈가 되고 다른 프로젝트 작업 시 무관한 내용이 끼어든다.

**How to apply:**
- **글로벌 memory dir 에 저장하는 것**: 프로젝트 무관 + 나(Claude)의 행동 규칙 — 코드/docs/commit 작업 process, 사용자 응답 선호, 일반 컨벤션.
- **프로젝트 repo 안에 기록하는 것**: 그 프로젝트에만 해당하는 사실 — 테스트/실행 명령, 구조·정책·gotcha, 배포 절차. 위치는 `docs/000.harness_guide.md`(1차 참조 하네스), README, 프로젝트 `.claude/` 또는 CLAUDE.md.
- 판단 기준: "다른 프로젝트에서도 적용되는 나의 행동 규칙인가?" → yes 면 글로벌, no(이 프로젝트 한정) 면 프로젝트.
- 예외: 프로젝트 **간 관계**(FE↔BE 매핑 같은 cross-repo 사실)나 여러 프로젝트에 걸친 사용자 선호는 글로벌 가능. 단일 프로젝트 내부 사실은 프로젝트로.
