---
name: verify_against_code_and_runtime
description: docs·주석·내 계획서·한 파일의 관찰은 지도일 뿐 — 단정·구현·보고 전에 코드 grep/본문 read, 동작하는 형제와 대조, 런타임 호출로 확인. 표 작성도 행마다 grep. build 통과≠런타임, 워크플로 파일≠인프라, 검증 도구의 불일치·전부 통과도 샘플 원문 대조로 도구 버그부터 배제
metadata:
  type: feedback
---

**docs 는 어디를 읽을지 알려주는 지도, 코드와 런타임이 진실.** 읽는 쪽·쓰는 쪽·단정하는 쪽 모두 같은 원리.

**1. 읽을 때 — status 표기를 믿지 않는다.** docs/000 의 ✅/🔴/"구현 완료"/"gap", 코드 주석, 그리고 **내가 같은 세션에 쓴 계획 문서**까지 전부 참고용. "구현됐나?" 는 service/router grep + 본문 read, "가드가 도나?" 는 코드 흐름 추적, "테스트 통과" 는 실제 실행. drift 발견 시 코드를 신뢰하고 docs 갱신.
(2026-05-08 §15.16 "🔴 code gap" 이 실은 구현 완료 → docs/016 오분류 / 2026-08-12 내 스펙이 착수 시점에 6곳 틀림 — 항목 착수 직전 심볼·파일·개수 재grep, 어긋나면 코드를 따르고 차이를 문서에 남김)

**2. 쓸 때 — 표는 행마다 grep.** 매트릭스·endpoint 인덱스·컴포넌트 카탈로그·라우트 매핑은 docs 끼리 cross-match 로 채우지 않는다.
- 컴포넌트가 부르는 endpoint: `grep -E "PATH\.[A-Z_]+|fetch|axios" <file>`
- 함수 dead 여부: 정의 제외 호출처 1곳 이상이어야 "사용 중"
- BE 라우트 부재 단정: 풀 경로 literal / suffix-only (`@Get('x')`) / controller prefix 합성 — **3종 grep 모두 비어야**
- 동작 단정: service 본문 5~20줄 (await/Promise.all/큐/cron 같은 동기·비동기 결정 요소)
단정 강도: "호출됨"=grep 1+ / "미사용"=0+검색했음 표기 / "BE 미존재"=3종 전부 빈 뒤 / "버그"=실행 경로+재현 뒤. 부족하면 "추정/확인 필요".

**3. 단정할 때 — 한 소스로 일반화하지 않는다.** "여기 없으니 auto-load 겠지 / 주석이 맞겠지" 금지. **동작하는 형제(sibling)가 실제로 어디에 등록·배선됐는지 grep** 해서 같은 지점 전부에 맞춘다.
- 새 엔티티/프로바이더/모듈: entity + migration + module `forFeature` + **`app.module.ts` forRoot `entities`** + (있으면) data-source. (2026-06-26 Round 엔티티 누락 → prod 500, build·부팅 전부 통과)
- **build/typecheck 통과 ≠ 런타임 정상** — DI/ORM 메타데이터는 첫 호출에 lazy 로 터진다. 엔드포인트를 실제로 한 번 태우거나 wiring 을 형제와 대조.
- **워크플로 파일 존재 ≠ 인프라 존재** — 배포 대상은 DNS(`curl -sI`/`host`) + repo secrets(`gh secret list`) + 실행 이력(`gh run list`) 3종 probe 뒤에 "구성돼 있다" 고 말한다. (2026-08-21 recruit prd 오안내 — NXDOMAIN + 시크릿 전무)
- 검증 안 된 메커니즘을 사용자에게 자신 있게 단언하지 않는다.

**4. 검증 도구의 결과도 도구부터 의심한다.** 내가 만든(또는 기존) 판정 규칙·대조 스크립트가 낸 "불일치/오류" 는 보고 전에 **샘플 1건을 원문 값으로 직접 대조**해 도구 쪽 가정(필드 형태·인코딩·마크업)을 먼저 배제한다. 반대로 "전부 통과" 도 규칙이 실데이터 형태를 실제로 잡는지 양성 샘플로 확인한다 — 한 번도 걸린 적 없는 규칙은 미탐 의심.
(2026-09-28 scope-validate: 새 단계 판정이 "답변 텍스트 원장≠코어" error 를 냈는데 원장 answer 가 문자열이 아니라 `{text, modelSlug, contentHash}` 객체였음 / 스탠스 "근거 없음" 4건 = 원문 `**` 강조 차이 / `citation-consistency` 정규식이 실제 마커 `citeturn…` 를 못 잡아 **항상 통과** — 셋 다 보고 전 원문 diff 로 발견)

**5. 답하기 전 확인 순서 (2026-11-06 추가)**
- 「A인가 B인가」 질문은 둘 다 틀릴 수 있다. 구성요소로 나눠 각각 확인한 뒤 답한다. (이미지·영상 단계를 「코드는 있고 데이터가 없다」고 답했다가 번복. 실제로는 수집 코드 없음 · 후보 추리기 있음 · 고르기 없음)
- 같은 세션에서 이미 읽은 파일과 다른 말을 하기 전에 그 파일을 다시 본다. (`round_snapshot.py` 머리 주석의 「최선 선택」을 읽고도 「최선 고르기는 코드에 없다」고 답함)
- 문서·주석에 「초안 · 미적용 · 예정」 표기가 있으면 적용 여부를 확인하기 전에 「있다」고 말하지 않는다. (머리가 「초안, 미적용」인 마이그레이션의 표를 「Supabase에서 볼 수 있다」고 함)
- 「추정 · 확인 필요」 같은 단서는 내 답변에 쓴다. 사용자가 보낼 문서에는 판단을 바꿀 때만 넣는다.

관련: [[verify_runtime_supervisor_before_restart]] (실행 중 서비스는 supervisor 가 진실), [[verify_edit_applied_before_reporting]] (편집 적용도 재조회), [[update_docs_000_after_changes]]
