## 프로세스별 참조 가이드

작업 시작 시점에 해당 process 의 feedback 파일을 read 후 진행. 한 룰이 여러 process 에 속할 수 있음.

| Process | Read 필요 파일 |
|---------|----------------|
| **코드 작업 시작** (편집/추가/삭제 / 정책 질문) | check_docs_000_first, docs_as_guide_code_as_truth, explain_code_references, answer_existence_questions_literally |
| **코드 검증 / 테스트** | docs_as_guide_code_as_truth, no_guess_in_docs, no_single_source_generalization, no_handtyped_dates_echo_inputs_on_empty |
| **날짜/연도 들어가는 쿼리·스크립트 작성** | no_handtyped_dates_echo_inputs_on_empty |
| **코드 변경 완료 → docs 갱신** | update_docs_000_after_changes, no_guess_in_docs, harness_doc_structure, explain_code_references |
| **새 docs/000.* 작성** | harness_doc_structure, no_guess_in_docs, explain_code_references |
| **사용자 응답 (질문/응답 형식)** | answer_existence_questions_literally, read_user_words_literally, phase_progression_collaborative, explain_code_references, no_repeat_decided_questions |
| **"없다/불가능" 답변** (기능·방법 부재 단정) | verify_ecosystem_before_saying_impossible, no_single_source_generalization |
| **손해/낭비/실패 원인 질문 응답** (= "왜 이만큼 들었/느렸/깨졌냐") | answer_the_loss_not_the_accounting, explain_with_business_logic |
| **데이터 추출 / 스냅샷 요청 처리** | read_user_words_literally, docs_as_guide_code_as_truth |
| **다단계 phase 작업** | phase_progression_collaborative, preserve_decision_literal |
| **카테고리/분류 표 작성** (= docs 매트릭스 / 분포 표) | no_guess_in_docs, full_columns_in_classification_sql |
| **plan doc 작성** (= docs/012~014 같은) | preserve_decision_literal, harness_doc_structure, no_guess_in_docs |
| **Frontend / Backend 양쪽 영향 변경** (= 새 endpoint / schema / helper) | cross_repo_audit_before_changes, no_guess_in_docs |
| **새 엔티티/모듈/프로바이더 등록** (DI/ORM wiring 추가) | no_single_source_generalization, cross_repo_audit_before_changes |
| **Git commit (코드/docs)** | commit_message_format, commit_no_claude_coauthor |
| **Git push** | git_push_confirm, commit_message_format |
| **Git checkout / switch / reset --hard** (branch 전환 또는 옛 ref 적용) | git_branch_switch_destroys_orphan_tracked_files |
| **RDBMS sync 코드 작성** (UPSERT / INSERT) | rdbms_upsert_autoinc |
| **실행 중 서비스 운영** (재시작/중지/리로드/배포·프로세스 상태 확인) | verify_runtime_supervisor_before_restart, docs_as_guide_code_as_truth |
| **멀티에이전트 워크플로 실행** (= Workflow / 대량 Agent fan-out) | confirm_before_large_agent_fanout |
| **메모리 변경 commit** | auto_sync_memory, commit_message_format, commit_no_claude_coauthor |
| **새 PC memory 셋업** | reference_memory_git_setup, auto_sync_memory |

## 전체 인덱스

- [No Claude co-author in commits](feedback_commit_no_claude_coauthor.md) — omit `Co-Authored-By: Claude` trailer from all git commit messages
- [Answer existence questions literally](feedback_answer_existence_questions_literally.md) — "있어?" 류에는 Yes/No 만 먼저, 구현은 명시적 지시 이후
- [Read user words literally](feedback_read_user_words_literally.md) — "지금/현재/fresh/최신" 같은 시점·범위 키워드를 IDE 정황·기존 파일로 치환 금지 + "필요하면 별도로 말씀" 류 책임 회피 단서 금지
- [Phase 별 진행은 협업](feedback_phase_progression_collaborative.md) — 다단계 phase 에서 사용자가 각 phase 마다 데이터·결정 입력, 자동 진행 금지
- [Check docs/000 before code tasks](feedback_check_docs_000_first.md) — 코드 작업 시작 전 프로젝트 `docs/000.*` (3자리 prefix) 확인 후 진행 (UI 라벨 의역 등 가드 정책 1차 참조)
- [Update docs/000 after code changes](feedback_update_docs_000_after_changes.md) — 코드/정책/구조 변경 시 docs/000 의 영향 섹션 (§5/§6/§10/§11 + §13~§15 deep-dive 매트릭스) cross-section sync 갱신 (drift 방지)
- [Sync memory to git (manual)](feedback_auto_sync_memory.md) — 메모리 변경은 Claude 가 diff 기반 메시지 제안+컨펌+직접 commit+push. Stop hook 사용 X (컨펌 룰과 충돌)
- [메모리 구조 + git sync 셋업 (리눅스/Mac/Windows)](reference_memory_git_setup.md) — 장기/중기/단기 3계층 구조 + 장기 메모리를 `~/.claude/claude_memory/` + `~/.claude/CLAUDE.md`(@import) 로 셋업·sync. **3 OS 공통** (`~/.claude/` 동일, Git Bash 권장 / Windows 네이티브는 `%USERPROFILE%`). 옛 `projects/<id>/memory/` 방식 폐기 — 마이그레이션 시 중복 로드 주의. hook 사용 X
- [Explain code references](feedback_explain_code_references.md) — 함수/변수/테이블/컬럼 언급 시 항상 "X (= Y 하는 것)" 형식. commit message 는 예외 (짧은 형식 유지)
- [Commit message 포맷 컨벤션](feedback_commit_message_format.md) — `(yyyymmdd) 동사_내용` **영어 문구** + `-`/`_` 구분 + 다중은 `,` + Claude co-author 미포함 + **반드시 커밋 전 메시지 컨펌**
- [Git push 시 사용자 confirm 받기](feedback_git_push_confirm.md) — commit 까지는 자동 OK, push 전엔 반드시 컨펌. 자발적 push 금지 (= remote 영향 + revert 어려움)
- [RDBMS UPSERT 시 AUTO_INCREMENT 폭발 주의](feedback_rdbms_upsert_autoinc.md) — `INSERT ... ON DUPLICATE KEY UPDATE` 또는 `ON CONFLICT` 가 UPDATE 분기에서도 AUTO_INC +1 소비 → 빈도 높은 sync 코드면 변경 감지 분기 권장
- [Harness doc 권장 구조](feedback_harness_doc_structure.md) — `docs/000.*.md` 작성 시 §1 역할 → §2 한눈에 보기 → §3 시스템 흐름 → §4 시나리오 → §5+ 인덱스 → Gotchas 골격. 변경이력 섹션은 권장 O 이되 git commit 시점에만 1 entry (표 1행) append (작업 중간 X)
- [docs 추측 금지, grep 으로만 채움](feedback_no_guess_in_docs.md) — 매트릭스/인덱스/카탈로그 표 작성 시 행마다 코드 grep + service 본문 read, docs cross-match 만으로 단정 X. docs 사용 측 짝꿍 = docs_as_guide_code_as_truth
- [docs는 지침서, 코드가 진실](feedback_docs_as_guide_code_as_truth.md) — docs/하네스는 어디를 읽을지 안내하는 지침서로만 사용. 검증/테스트/status 확인은 코드 직접 read. docs 의 ✅/🔴 등 status 표기 맹신 금지
- [git checkout/reset 가 옛 tracked 파일 삭제](feedback_git_branch_switch_destroys_orphan_tracked_files.md) — branch 전환 / `reset --hard` 가 옛 ref 의 tracked 였던 파일을 working tree 에서 자동 삭제. `.gitignore` 보호 무관. checkout/reset 전 `.env*`/secret 백업 + Claude 의 일반화 단정 금지
- [vip_membership_event 최신 행은 MAX(id) 금지](feedback_vip_event_ordering_not_max_id.md) — kstadium-referral-backend 의 VIP 현황 SQL 은 `(effective_at, block_number, log_index, id) DESC` 정렬 필수. MAX(id) 만 쓰면 manual/auto 행 순서 뒤집힘
- [사용자 결정은 자연어 quote 로 보존](feedback_preserve_decision_literal.md) — AskUserQuestion 선택지 label 만으로 본질 의도 추적 부족. plan doc § 안에 사용자 원문 quote
- [이미 결정된 사항 반복 질문 금지](feedback_no_repeat_decided_questions.md) — "안전한 척" 모든 분기 묻기 금지. 기존 정책의 자동 적용은 자체 판단 + 한 줄 안내
- [분류 SQL 은 모든 sanity check 컬럼 한 query 에](feedback_full_columns_in_classification_sql.md) — 카테고리 분류 시 exclusion / pre_genesis / balance_row / 등 모든 영향 컬럼 한 SELECT 에. 부분 query 단정 금지
- [Frontend / Backend cross-check 매트릭스 (변경 전)](feedback_cross_repo_audit_before_changes.md) — 양쪽 영향 변경 시 endpoint method+path 매트릭스 + ADMIN_PATH 사용 + helper method coverage + 직접 호출 4 차원 점검. trial-and-error 금지
- [프로젝트별 메모리는 프로젝트 안에](feedback_project_memory_stays_in_project.md) — 글로벌 memory dir 는 Claude 의 일반 process/생각/선호(feedback 룰)만. 프로젝트 고유 사실(테스트 실행법·구조·정책)은 해당 repo(docs/000 하네스·README 등)에 기록
- [발견·코드 설명은 비즈니스 로직과 연계](feedback_explain_with_business_logic.md) — file:line 인용에 그치지 말고 비즈니스 의미 + 시나리오/숫자 트레이스 + 유저·사측 영향으로 설명. 코드 변경 제안도 동일
- [손해 질문엔 계측 아닌 정당성으로 답하라](feedback_answer_the_loss_not_the_accounting.md) — "왜 낭비/손해/실패했냐"엔 "숫자는 맞으니 정상"으로 시스템 변호 금지. 그 결과가 정당했는지를 먼저. "재현됨≠올바름". (2026-06-12 토큰폭주 조사서 내가 범한 오판)
- [실행 중 서비스는 런타임 supervisor가 진실](feedback_verify_runtime_supervisor_before_restart.md) — 재시작/중지/리로드는 repo 스크립트 아닌 라이브 supervisor(systemd 등) 확인. negative("X가 안 띄움")는 positive 식별 probe 강제, PPID=1은 분기. mutation 전 restart 정책 확인. **수동 프로세스 재기동 시 cmdline 만 복제 금지 — env 격리 (unset 등) 포함한 repo 기동 스크립트로** (2026-07-23 KSTA_RPC_URL mainnet 오염 재발). **sudo 필요한 재기동은 kill-PID 우회 금지 — 배포까지만 하고 명령 제시 후 사용자 핸드오프** (2026-08-05). docs_as_guide_code_as_truth 의 런타임 확장
- ["없다/불가능" 단정 전 생태계 최신 검색](feedback_verify_ecosystem_before_saying_impossible.md) — 설치된 버전의 한계 ≠ 세상에 없음. "~방법 없니?" 질문엔 공식 docs·후속 라이브러리 웹검색 먼저 (2026-07-22 metamask connect-evm 놓친 사건)
- [단일 소스 일반화 단정 금지 — 형제와 cross-check, build≠런타임](feedback_no_single_source_generalization.md) — 한 파일/관찰로 동작 단정 금지, **동작하는 형제와 cross-check**. 새 엔티티는 module `forFeature` + `app.module.ts` forRoot `entities` **둘 다** 등록(data-source 부재로 auto-load 단정 X). **build/typecheck 통과 ≠ 런타임 정상**(DI/ORM 메타데이터 lazy → 첫 호출에 터짐) → 엔드포인트 실제 호출 또는 wiring 대조. (2026-06-26 Round 엔티티 미등록 → closed-tournaments 500)
- [날짜 리터럴 직접 타이핑 금지 + 빈 결과는 입력 echo 먼저](feedback_no_handtyped_dates_echo_inputs_on_empty.md) — 연도/월은 시스템 날짜에서 파생 (학습 분포 탓 2025 미끄러짐 반복). "성공+빈 결과"면 외부 원인 가설 전에 내가 보낸 파라미터 echo 검증 (2026-08-07 NCP 202507 오타 → 오진 사건)
- [에이전트 fan-out 은 토큰 3구간 + 작업 체인 누적 판정](feedback_confirm_before_large_agent_fanout.md) — 판정은 개수 아닌 **예상 토큰 3구간 (≤20만 자유 / ≤60만 한줄고지 / >60만 컨펌)** + **판정 단위 = 배치 1개가 아니라 작업 체인 누적** (워크플로 끝날 때마다 실측 totalTokens 합산해 재판정). 무거운 에이전트 ≈4.5~5만/개, **검증류는 ~9만/개** (휴리스틱 2배). 1:1 검증은 상한 안에서만, 무상한 fan-out 구조 금지, 필요성 선판단(한 컨텍스트면 main loop). 사고 2건: 2026-06-12 (52ag=234만 크래시) + **2026-07-02 (≤10개짜리 워크플로 3연속 = 누적 127.5만 = 개수 룰 전부 통과하고도 세션 한도 소진 — 배치 단위 판정이 구멍)**. "재현됨≠올바름", "정상이라 괜찮다" 금지 — 사전 고지+누적 상한으로 능동 방어
