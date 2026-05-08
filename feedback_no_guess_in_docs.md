---
name: docs 매트릭스/인덱스 작성 시 추측 금지, grep + 코드 본문으로만 채움
description: harness 의 카드 매트릭스 / 엔드포인트 인덱스 / 컴포넌트 카탈로그 / BE 라우트 매핑 등 표 형태 정보를 작성·갱신할 때, 각 행을 추측이나 docs cross-match 로 채우지 말고 코드 grep + 본문 read 결과로만 채울 것
type: feedback
originSessionId: f3af43d0-b662-4e0d-a30c-fee4d49d5f94
---
harness / 인덱스성 docs 의 표 형태 항목 (= 매트릭스, endpoint 인덱스, 컴포넌트 카탈로그, 라우트 매핑) 작성 시 각 행을 직접 grep / Read 결과에 기반해서만 채움.

**Why:** docs 끼리 cross-match (= FE docs ↔ BE docs / agent 요약 ↔ 응답 shape) 만 하면 둘 다 추상이라 코드 실 동작과 어긋날 수 있음. 한 번 잘못 적힌 매트릭스는 다음 작업자에게 misinformation 으로 작용. "import 만 보고 컴포넌트가 X endpoint 호출하는 듯" 추측은 호출 사이트 0 인 dead code 일 수 있고, "BE 응답이 processing 이면 비동기" 추측은 service 가 `await` 한 결과일 수 있고, "라우트 grep 안 보이면 BE 미존재" 단정은 `@Controller(prefix)` + `@Get(suffix)` 합성 못 잡았을 수 있음.

**관련 룰**: 본 룰은 docs **작성 (write)** 측 — 매트릭스 / 인덱스 / status 표기 채울 때 코드 grep 기반. 반대 방향 — docs **사용 (read)** 시 status 표기 맹신 금지 룰은 [feedback_docs_as_guide_code_as_truth](feedback_docs_as_guide_code_as_truth.md). 두 룰이 "코드가 진실" 같은 원리의 write/read 양면.

**How to apply (작성·갱신 직전):**

1. **컴포넌트 / 모듈이 호출하는 endpoint 확인**: `grep -E "PATH\.[A-Z_]+|fetch[A-Z]\w+|axios\.|reveal[A-Z]\w+" <file>` — 결과 그대로 표에 옮김.
2. **함수 / 함수의 dead 여부 확인**: `grep -rn "<함수명>" src/ --include="*.js" --include="*.jsx"` — 정의 위치 제외 1군데 이상이어야 "사용 중", 0 이면 **dead code** 명시.
3. **BE 라우트 존재 여부 확인** — 3종 grep 모두 비어야 "미존재" 단정 가능:
   - 풀 경로 literal (`grep -rn "/full/path"`)
   - suffix-only (`grep -rn "@Get('suffix')"`, `@Post`, etc.)
   - controller prefix 와 합성 (`@Controller('prefix')` 파일의 `@Get('suffix')` 매칭)
4. **비즈니스 룰 / 동작 단정 시 service 본문 1회 읽기** — controller route handler + service method body 5–20줄. 특히 `await` / `Promise.all` / `setTimeout` / 큐잉 / cron / event-emit 같은 동기·비동기 결정 요소 직접 확인. agent 요약 / FRONTEND_API.md 같은 docs 만 보고 단정 X.

**단정 강도 매칭:**
- "노출됨 / 호출됨" — grep hit 1 군데 이상
- "사용 중" — import + call site grep hit
- "미사용 / 미호출" — 호출 사이트 grep 0 + 직접 검색했음 표기
- "BE 미존재" — 3종 grep 모두 비어야 함
- "버그" — 실 실행 경로 추적 + 재현 시나리오 확보 후만

증거 부족 시 "추정" / "확인 필요" / "FE 호출 X" 같이 약화된 표현 사용. "버그" / "BE 미존재" 같은 강한 단정은 검증 다 끝난 뒤에만.

**적용 트리거:**
- 새 endpoint 를 인덱스 표에 추가할 때
- 새 컴포넌트 / 서비스 를 카탈로그·매트릭스에 추가할 때
- Gotchas 에 ⚠️ 강도 항목 추가할 때
- 다른 repo (BE/FE) 동작을 docs 에 인용할 때
- harness 의 BE 또는 외부 시스템 관련 단정 갱신 시
