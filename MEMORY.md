## 프로세스별 참조 가이드

**지시를 받을 때마다** 그 지시의 성격을 아래 표에서 분류하고 해당 행의 `feedback_*` 파일을 read 한 뒤 진행한다. 맞는 행이 없으면 가장 가까운 룰을 적용하고 행 신설을 제안. 한 룰이 여러 행에 속할 수 있다.

| Process | Read 필요 파일 |
|---------|----------------|
| **사용자 응답 전반** (질문·확인·컨펌 해석) | read_user_words_literally, no_repeat_decided_questions, explain_code_references |
| **코드 작업 시작** | plan_stage_no_implementation (착수 신호 게이트), check_docs_000_first, verify_against_code_and_runtime, check_design_history_before_changes, verify_edit_applied_before_reporting |
| **UI 화면·문구·목록·폼** (신규 화면, 라벨, 명단, 신청 폼) | ui_design_principles, check_docs_000_first |
| **유저 플로우/기능 변경** | ui_design_principles, sync_test_tools_with_feature_changes, cross_repo_audit_before_changes |
| **설정값 추가** (env / config) | config_not_env_for_addresses |
| **코드 검증 / 테스트 / "구현됐나" 확인** | verify_against_code_and_runtime, no_handtyped_dates_echo_inputs_on_empty |
| **날짜/연도 들어가는 쿼리·스크립트** | no_handtyped_dates_echo_inputs_on_empty |
| **코드 변경 완료 → docs 갱신** | update_docs_000_after_changes, harness_doc_structure, verify_against_code_and_runtime, verify_edit_applied_before_reporting |
| **대량 기계적 변경** (일괄 삭제·정규식 치환) | verify_edit_applied_before_reporting, verify_against_code_and_runtime |
| **계획 문서대로 항목 착수** | verify_against_code_and_runtime |
| **새 docs/000.* 작성 · 매트릭스/분류 표** | harness_doc_structure, verify_against_code_and_runtime, full_columns_in_classification_sql |
| **발견·이상 보고** | no_repeat_decided_questions, answer_the_loss_not_the_accounting, explain_with_business_logic, reward_abuse_scenario_check |
| **리워드/지급 구조 설계·구현·검증** | reward_abuse_scenario_check, answer_the_loss_not_the_accounting, preserve_decision_literal |
| **"없다/불가능" 답변** | verify_ecosystem_before_saying_impossible, verify_against_code_and_runtime |
| **손해/낭비/실패 원인 질문** | answer_the_loss_not_the_accounting, explain_with_business_logic |
| **다단계 phase 작업** | phase_progression_collaborative, preserve_decision_literal |
| **플랜(설계) 단계 · plan doc 작성** | plan_stage_no_implementation, plan_review_artifact_before_work, preserve_decision_literal, harness_doc_structure, phase_progression_collaborative |
| **Frontend / Backend 양쪽 영향 변경 · 새 엔티티/모듈 등록** | cross_repo_audit_before_changes, new_entity_crud_completeness, verify_against_code_and_runtime |
| **Git commit / push / 메모리 commit** | git_conventions |
| **Git checkout / switch / reset --hard** | git_branch_switch_destroys_orphan_tracked_files |
| **RDBMS sync 코드 (UPSERT)** | rdbms_upsert_autoinc |
| **실행 중 서비스 운영** (재시작·배포·프로세스 확인) | verify_runtime_supervisor_before_restart, verify_against_code_and_runtime |
| **배포/승격** (dev→prd, "배포 가능" 전제 답변) | verify_against_code_and_runtime (파일≠인프라 3종 probe), verify_runtime_supervisor_before_restart, cross_repo_audit_before_changes, git_conventions |
| **멀티에이전트 워크플로 / 대량 Agent fan-out** | confirm_before_large_agent_fanout |
| **새 PC memory 셋업** | reference_memory_git_setup, git_conventions |

## 전체 인덱스

- [사용자 말은 그대로 읽는다](feedback_read_user_words_literally.md) — "있어?/확인해봐" 는 조사 보고까지, "지금/최신" 은 라이브 소스, 항목 N개면 답 N개(사용자 라벨 그대로), 책임 회피 단서 금지, "사용자 몫" 은 진짜 할 일만
- [이미 결정된 사항 반복 질문·재보고 금지](feedback_no_repeat_decided_questions.md) — 기존 정책 자동 적용은 자체 판단 + 한 줄 안내. 종결된 건을 "새 발견" 으로 보고하지 않기
- [플랜 단계와 작업 단계 분리](feedback_plan_stage_no_implementation.md) — 플랜 중 "ㄱㄱ" 는 plan doc 반영까지, 구현은 명시 착수 신호 뒤. 단계 전환은 사용자만
- [큰 개발은 착수 전 아티팩트 확인 문서](feedback_plan_review_artifact_before_work.md) — 흐름·돈·결정·분담 요약을 아티팩트로 최종 확인 (2026-08-19 템플릿)
- [Phase 별 진행은 협업](feedback_phase_progression_collaborative.md) — 각 phase 마다 사용자 입력, 자동 진행 금지
- [사용자 결정은 원문 quote 로 보존](feedback_preserve_decision_literal.md) — plan doc § 안에 자연어 원문
- [UI 설계 원칙](feedback_ui_design_principles.md) — 한 화면 한 판단(상태별), 소개↔입력 분리·긴 폼은 스텝 위저드, 목록은 표, 문구는 형제 양식(이모지 X), 버튼은 바탕과 구분. 지적받고 고치면 실패
- [설정: 주소는 config, env 는 키만](feedback_config_not_env_for_addresses.md) — 환경 파생 상수는 APP_ENV 분기 property, env 는 시크릿·외부 발급값만
- [코드·런타임으로 검증](feedback_verify_against_code_and_runtime.md) — docs·주석·내 계획서·한 파일은 지도일 뿐. 표는 행마다 grep, 형제와 cross-check, build≠런타임, 워크플로 파일≠인프라(DNS·secrets·실행이력 probe)
- [Check docs/000 before code tasks](feedback_check_docs_000_first.md) — 작업 전 프로젝트 `docs/000.*` 확인
- [Update docs/000 after code changes](feedback_update_docs_000_after_changes.md) — 영향 섹션 cross-section sync
- [Harness doc 권장 구조](feedback_harness_doc_structure.md) — §1 역할→§2 한눈에→§3 흐름→§4 시나리오→인덱스→Gotchas. 변경이력은 commit 시점에 1행
- [기존 로직 변경 전 결정 이력 탐독·충돌 고지](feedback_check_design_history_before_changes.md) — 어긋나면 "기존엔 ~로 설계 — 바꾸는 건데 괜찮냐" 먼저
- [편집 적용 확인 후 보고 + 대량 삭제는 파서로](feedback_verify_edit_applied_before_reporting.md) — 재조회로 확인, `assert` 앵커, 린터·빌드·런타임 3중
- [Explain code references](feedback_explain_code_references.md) — 식별자 언급 시 "X (= Y 하는 것)"
- [발견·설명은 비즈니스 로직과 연계](feedback_explain_with_business_logic.md) — 의미 + 시나리오/숫자 + 유저·사측 영향
- [손해 질문엔 정당성으로 답하라](feedback_answer_the_loss_not_the_accounting.md) — "숫자는 맞으니 정상" 금지, 재현됨≠올바름
- [리워드 구조는 어뷰즈 시나리오 검증](feedback_reward_abuse_scenario_check.md) — 총 지급 상한·다계정·트리거 주체-비용 정렬, fan-out 지급 경보
- [새 엔티티는 CRUD 수명주기 완결](feedback_new_entity_crud_completeness.md) — 생성 경로 전수 grep, 운영 CRUD 매트릭스, 미구현 명시, 기존 운영 화면 통합 검토
- [테스트 도구는 기능 변경과 함께 갱신](feedback_sync_test_tools_with_feature_changes.md) — 유저와 같은 진입점, 내부 함수 직행 금지
- [FE/BE cross-check 매트릭스](feedback_cross_repo_audit_before_changes.md) — endpoint 매트릭스·PATH 사용·helper 커버리지·직접 호출 4차원
- [Git 컨벤션](feedback_git_conventions.md) — `(yyyymmdd) 동사_내용` 영어, co-author 금지, commit 메시지·push 컨펌 (같은 흐름의 후속은 고지만), 메모리 repo 동일
- [git checkout/reset 가 옛 tracked 파일 삭제](feedback_git_branch_switch_destroys_orphan_tracked_files.md) — 전환 전 `.env*` 백업
- [실행 중 서비스는 supervisor 가 진실](feedback_verify_runtime_supervisor_before_restart.md) — systemd 확인, env 격리 기동 스크립트, sudo 재기동은 사용자 핸드오프
- ["없다/불가능" 단정 전 생태계 검색](feedback_verify_ecosystem_before_saying_impossible.md) — 설치 버전 한계 ≠ 세상에 없음
- [날짜 리터럴 금지 + 빈 결과는 입력 echo](feedback_no_handtyped_dates_echo_inputs_on_empty.md) — 연도는 시스템 날짜에서, "성공+빈 결과" 면 파라미터 먼저
- [에이전트 fan-out 토큰 3구간 + 체인 누적](feedback_confirm_before_large_agent_fanout.md) — ≤20만 자유 / ≤60만 고지 / >60만 컨펌, 검증류 ~9만/개
- [분류 SQL 은 모든 컬럼 한 query 에](feedback_full_columns_in_classification_sql.md)
- [RDBMS UPSERT AUTO_INCREMENT 주의](feedback_rdbms_upsert_autoinc.md)
- [프로젝트별 메모리는 프로젝트 안에](feedback_project_memory_stays_in_project.md) — 여기는 Claude 의 일반 process/선호만
- [vip_membership_event 최신 행 정렬](feedback_vip_event_ordering_not_max_id.md) — (referral 프로젝트 고유 — repo 가 이 PC 에 있을 때 docs/000 로 이관)
- [메모리 구조 + git sync 셋업](reference_memory_git_setup.md) — 3계층, `~/.claude/claude_memory/` + `CLAUDE.md` @import, hook 사용 X
