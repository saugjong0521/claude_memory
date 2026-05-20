## 프로세스별 참조 가이드

작업 시작 시점에 해당 process 의 feedback 파일을 read 후 진행. 한 룰이 여러 process 에 속할 수 있음.

| Process | Read 필요 파일 |
|---------|----------------|
| **코드 작업 시작** (편집/추가/삭제 / 정책 질문) | check_docs_000_first, docs_as_guide_code_as_truth, explain_code_references, answer_existence_questions_literally |
| **코드 검증 / 테스트** | docs_as_guide_code_as_truth, no_guess_in_docs |
| **코드 변경 완료 → docs 갱신** | update_docs_000_after_changes, no_guess_in_docs, harness_doc_structure, explain_code_references |
| **새 docs/000.* 작성** | harness_doc_structure, no_guess_in_docs, explain_code_references |
| **사용자 응답 (질문/응답 형식)** | answer_existence_questions_literally, phase_progression_collaborative, explain_code_references |
| **다단계 phase 작업** | phase_progression_collaborative |
| **Git commit (코드/docs)** | commit_message_format, commit_no_claude_coauthor |
| **Git push** | git_push_confirm, commit_message_format |
| **메모리 변경 commit** | auto_sync_memory, commit_message_format, commit_no_claude_coauthor |
| **새 PC memory 셋업** | reference_memory_git_setup, auto_sync_memory |

## 전체 인덱스

- [No Claude co-author in commits](feedback_commit_no_claude_coauthor.md) — omit `Co-Authored-By: Claude` trailer from all git commit messages
- [Answer existence questions literally](feedback_answer_existence_questions_literally.md) — "있어?" 류에는 Yes/No 만 먼저, 구현은 명시적 지시 이후
- [Phase 별 진행은 협업](feedback_phase_progression_collaborative.md) — 다단계 phase 에서 사용자가 각 phase 마다 데이터·결정 입력, 자동 진행 금지
- [Check docs/000 before code tasks](feedback_check_docs_000_first.md) — 코드 작업 시작 전 프로젝트 `docs/000.*` (3자리 prefix) 확인 후 진행 (UI 라벨 의역 등 가드 정책 1차 참조)
- [Update docs/000 after code changes](feedback_update_docs_000_after_changes.md) — 코드/정책/구조 변경 시 docs/000 의 영향 섹션 (§5/§6/§10/§11 + §13~§15 deep-dive 매트릭스) cross-section sync 갱신 (drift 방지)
- [Sync memory to git (manual)](feedback_auto_sync_memory.md) — 메모리 변경은 Claude 가 diff 기반 메시지 제안+컨펌+직접 commit+push. Stop hook 사용 X (컨펌 룰과 충돌)
- [메모리 git sync 셋업 (새 PC)](reference_memory_git_setup.md) — 새 PC 셋업용 git clone + config + 인증 절차. hook 사용 X
- [Explain code references](feedback_explain_code_references.md) — 함수/변수/테이블/컬럼 언급 시 항상 "X (= Y 하는 것)" 형식. commit message 는 예외 (짧은 형식 유지)
- [Commit message 포맷 컨벤션](feedback_commit_message_format.md) — `(yyyymmdd) 동사_내용` + `-`/`_` 구분 + 다중은 `,` + Claude co-author 미포함 + **반드시 커밋 전 메시지 컨펌**
- [Git push 시 사용자 confirm 받기](feedback_git_push_confirm.md) — commit 까지는 자동 OK, push 전엔 반드시 컨펌. 자발적 push 금지 (= remote 영향 + revert 어려움)
- [Harness doc 권장 구조](feedback_harness_doc_structure.md) — `docs/000.*.md` 작성 시 §1 역할 → §2 한눈에 보기 → §3 시스템 흐름 → §4 시나리오 → §5+ 인덱스 → Gotchas 골격. 변경이력 섹션은 권장 O 이되 git commit 시점에만 1 entry (표 1행) append (작업 중간 X)
- [docs 추측 금지, grep 으로만 채움](feedback_no_guess_in_docs.md) — 매트릭스/인덱스/카탈로그 표 작성 시 행마다 코드 grep + service 본문 read, docs cross-match 만으로 단정 X. docs 사용 측 짝꿍 = docs_as_guide_code_as_truth
- [docs는 지침서, 코드가 진실](feedback_docs_as_guide_code_as_truth.md) — docs/하네스는 어디를 읽을지 안내하는 지침서로만 사용. 검증/테스트/status 확인은 코드 직접 read. docs 의 ✅/🔴 등 status 표기 맹신 금지
