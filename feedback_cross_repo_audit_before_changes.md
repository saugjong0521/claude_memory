---
name: cross-repo-audit-before-changes
description: frontend / backend 양쪽 영향 변경 시작 전에 endpoint method+path 매트릭스 + ADMIN_PATH 사용 매트릭스 + helper/api wrapper method coverage + 직접 호출 (= adminApi 우회) 점검을 한 번에 cross-check 매트릭스로 작성. trial-and-error 금지
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d46e4ac-70e3-4d2c-ab0d-104af852a354
---

frontend / backend 양쪽 영향 받는 변경 (= 새 endpoint 도입, schema 변경, body 형식 변경, helper wrapper 추가/변경) 시작 전에 다음 4 차원 cross-check 매트릭스를 작성. 매트릭스에서 매칭/누락/dead 발견 후 변경 시작. **사용자가 prod 에서 시도하며 새 버그 발견하는 trial-and-error 패턴 금지**.

**Why:** 2026-05-27 docs/013 + docs/014 + VIP 통합 endpoint 작업에서, frontend 변경 시작 시 backend 와 cross-check 안 한 채 진행 → 사용자가 prod 에서 시도하며 4 건의 fix 발견 (= [nulls_last MySQL syntax 미지원](https://referral.kstadium.io/admin) 500 / Input onChange double dereference / `adminApi.put is not a function` / share-calc.csv `referralApi` 직접 호출로 인한 401). 사용자 답답함 표명 ("뭣하는꼬라지임 frontend, backend 비교해서 frontend 전체 점검해"). 본 4 건 모두 cross-check 매트릭스 한 번 작성하면 사전 발견 가능했음.

**How to apply (= 4 차원 매트릭스):**

### 1. Backend endpoint method+path 매트릭스 (= grep + prefix 합성)

```bash
# router prefix 추출
grep -rn "APIRouter(prefix=" app/routers/ --include="*.py"

# 각 router 의 @router.<method>("path") + prefix 합성 → fully-qualified path
python3 -c "
import re
from pathlib import Path
for f in sorted(Path('app/routers').rglob('*.py')):
    if '__pycache__' in str(f): continue
    text = f.read_text()
    m = re.search(r'APIRouter\(prefix=\"([^\"]+)\"', text)
    if not m: continue
    prefix = m.group(1)
    for m2 in re.finditer(r'@router\.(get|post|put|patch|delete)\(\s*\"([^\"]*)\"', text, re.DOTALL):
        print(f'{m2.group(1).upper()} {prefix}{m2.group(2)}')
"
```

### 2. Frontend ADMIN_PATH 사용 매트릭스 (= 호출 method + PATH key)

```bash
grep -rn "adminApi\.\(get\|post\|put\|patch\|delete\)" src/ --include="*.jsx" --include="*.js" | \
  sed -E "s|.*adminApi\.(get|post|put|patch|delete)\(ADMIN_PATH\.([A-Z_]+).*|\1 \2|" | sort -u
```

### 3. Helper / api wrapper method coverage 점검

```bash
# adminApi 가 지원하는 method 목록 (= helper 의 return object 키)
grep -E "^\s+(get|post|put|patch|delete):" src/api/adminApi.js

# 모든 helper component 의 inner wrapping (= onChange / onSubmit 등) 점검
grep -rnE "onChange=\{[^}]+\}" src/components/admin/adminUtils.jsx
```

→ frontend 가 호출하는 method 가 helper 에 모두 있는지 확인. e.g. backend 가 PUT endpoint 추가했는데 helper 에 `put` 없으면 `n.put is not a function`.

### 4. 직접 호출 (= helper 우회) 점검

```bash
# referralApi / 기본 axios 직접 호출 — admin auth header 누락 위험
grep -rn "referralApi\.\(get\|post\|put\|patch\|delete\)" src/components/admin/ --include="*.jsx"
```

→ admin 측에서 `referralApi.<method>` 직접 호출 발견 시 → `adminApi.<method>` 로 변경 (= Authorization 자동 추가).

### Cross-check 결과 분류

| 결과 | 조치 |
|------|------|
| frontend method+path → backend 매칭 ✅ | OK |
| frontend → backend 미존재 | backend endpoint 추가 또는 frontend 호출 제거 |
| backend → frontend 미사용 | 운영 도구 의도 (= keep) 또는 dead code (= remove) 결정 |
| helper method 누락 | helper 에 method 추가 |
| 직접 호출 (= 우회) | helper 로 변경 |

본 매트릭스 결과를 변경 plan doc 의 §X (= "Cross-check 결과" sub-section) 에 보존. 이후 변경 commit 의 reference 로 활용.

**적용 트리거:**
- 새 admin endpoint 도입 시 (= 양쪽 변경 plan)
- adminApi helper 변경 시 (= method 추가/제거)
- frontend 의 admin component 신규 작성 / 큰 변경 시
- schema (= request body / response shape) 변경 시
- prod deploy 직전 마지막 sanity check

관련: [[no_guess_in_docs]] (= docs 표 작성 시 grep 기반), [[docs_as_guide_code_as_truth]] (= 검증은 코드 직접 read), [[full_columns_in_classification_sql]] (= 분류 SQL 전체 컬럼 한 번에).
