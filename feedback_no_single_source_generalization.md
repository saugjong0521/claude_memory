---
name: 단일 소스 일반화 단정 금지 — 형제와 cross-check, build 통과 ≠ 런타임 정상
description: 한 파일/한 관찰로 "X는 이렇게 동작한다 / 여기만 고치면 된다"를 단정·구현·단언 금지. 실제 배선 또는 동작하는 형제(sibling)와 cross-check. typecheck/build 통과는 DI/ORM wiring·등록 정상을 보장 못 함(런타임 lazy 에러)
type: feedback
originSessionId: a774f72c-6235-43be-9fc7-4a54722432e2
---
한 파일·한 관찰에서 "X 는 이렇게 동작한다 / 여기만 고치면 된다" 를 추론했으면, **단정·구현·사용자에게 단언하기 전에** 그 메커니즘을 **실제 배선 또는 동작하는 형제(sibling)** 와 cross-check 한다. "일반화 단정 금지" 는 [feedback_git_branch_switch_destroys_orphan_tracked_files](feedback_git_branch_switch_destroys_orphan_tracked_files.md) 안에도 있었으나 그 맥락에 묻혀 독립 룰로 안 박혀 있었고, 같은 실패가 한 세션에 3번 반복됨(코드주석 맹신 → 페이지네이션 루프 과잉수정 → 엔티티 등록 누락).

**Why:** 2026-06-26 사고 — 새 TypeORM 엔티티 `TournamentLotteryRound` 를 `src/data-source.ts` 만 보고 (거기에 형제 Ticket/Issuance 도 없어서) "런타임은 module `forFeature` 로 auto-load 한다" 고 단정 → 런타임이 실제로 쓰는 `src/app.module.ts` 의 명시 `entities:[]`(autoLoadEntities 없음)에 등록 누락 → prod `GET /tournament-lottery/admin/closed-tournaments` 가 첫 호출에서 `EntityMetadataNotFoundError: No metadata for "TournamentLotteryRound"` 로 **500**. **typecheck·build·부팅 전부 통과**해서 못 잡음(메타데이터는 첫 `roundsRepo.find()` 때 lazy 조회 → 그 엔드포인트 첫 호출에만 터짐). 형제(Ticket/Issuance)가 `app.module.ts` entities 배열에 등록된 걸 `grep TournamentLotteryTicket` 한 번만 했으면 잡혔음. 게다가 사용자에게 "data-source 배열은 CLI 용이고 앱은 모듈로 로드한다" 고 **자신 있게 틀린 단언**까지 함.

**How to apply:**

1. **추론하면 곧장 cross-check** — "여기 없으니 auto-load 겠지 / 이 코드주석이 맞겠지" 식 단정 금지. **"그럼 형제는 실제로 어디에 등록/배선됐나?"** 를 grep 해서 그 위치 전부를 확인·매칭. 검증 안 된 메커니즘을 사용자에게 단언하지 말 것.

2. **새 엔티티/프로바이더/모듈 등록 체크리스트** — 형제를 grep 해 **모든 등록 지점**에 동일하게 추가:
   - entity 파일 + migration + module `forFeature` + **`app.module.ts` forRoot `entities` 배열** + (필요 시) `data-source.ts`.
   - `data-source.ts` 에 없다고 "auto-load" 로 단정 X — 런타임은 `forRootAsync` 의 명시 entities(autoLoad 없음)를 쓰고, `data-source.ts` 는 migration CLI 용이라 불완전할 수 있음.

3. **build 통과 ≠ 런타임 정상** — DI/ORM 메타데이터·등록·wiring 누락은 컴파일러가 못 잡고 **첫 호출에 lazy 하게** 터진다. 검증은:
   - **엔드포인트를 실제로 한 번 태운다**(앱 구동 + 그 경로 호출).
   - 못 태우면(로컬 DB/런타임 없음) **wiring 파일을 동작하는 형제와 대조**(app.module entities/providers/imports 등).
   - typecheck/build 성공을 "정답" 신호로 의지 금지.

| 단계 | 잘못한 행동 | 옳은 행동 |
|------|-----------|----------|
| 1 | `data-source.ts` 만 보고 "auto-load" 단정 | `grep TournamentLotteryTicket src/` → `app.module.ts` entities 발견 → Round 도 거기 추가 |
| 2 | build ✅ 를 정상으로 신뢰 | 엔티티 등록은 런타임 lazy → 엔드포인트 호출 또는 wiring 대조로 확인 |
| 3 | 사용자에게 메커니즘 자신 있게 단언 | 검증 통과 전엔 단언 금지 |

**핵심**: 한 소스의 부재/존재로 시스템 동작을 일반화하지 말고 **동작하는 형제와 대조**. build 는 wiring 에 눈멀었다 — 런타임으로 확인. ([feedback_docs_as_guide_code_as_truth](feedback_docs_as_guide_code_as_truth.md) 의 배선/런타임 확장, [feedback_answer_the_loss_not_the_accounting](feedback_answer_the_loss_not_the_accounting.md) 의 "재현됨≠올바름" 과 같은 결.)
