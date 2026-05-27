---
name: no-repeat-decided-questions
description: 사용자가 이미 답한 사항을 다시 옵션으로 묻지 않음. 동일 사안의 새 옵션은 이전 답변과 본질적 차이를 명시 가능할 때만 발행
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d46e4ac-70e3-4d2c-ab0d-104af852a354
---

사용자가 이미 결정한 정책 / 옵션 / 분류 기준을 다시 AskUserQuestion 으로 묻지 않는다. "안전한 척" 모든 분기를 묻는 패턴 금지.

**Why:** 2026-05-27 docs/013 / docs/014 작업 중 (1) 사용자가 lifetime > 0 + floor 채택 후 (2) 그 결정 이미 명시했는데 0x260c1f9b 의 별도 backfill 결정 필요하다고 또 옵션 제시 → 사용자가 "이런 것도 물어봐야 되는 거 아님? exclusion 됐으니까 당연히 빠지는 게 맞지 왜 자꾸 얼토당토 않는 실수들을 하는 거임" 답답함 표명. docs/012 의 exclusion 정책이 이미 있는데 그 정책의 자동 적용 여부를 자체 판단하지 않고 사용자에게 재확인 요청 → 비효율.

**How to apply:**
1. AskUserQuestion 발행 전 자체 점검: "사용자가 이 사안에 이미 답했나? 이전 답으로 자동 추론 가능한가?"
2. 이미 명시된 정책 (= docs 의 정책 sub-section 또는 사용자 quote) 의 자동 적용은 묻지 않음. e.g. "deposit_exclusion_tx 등록 → 분모 제외" 가 정책이면 신규 wallet 도 자동 적용.
3. 새 옵션은 이전 답변과 본질적 차이가 있을 때만. 차이가 무엇인지 사용자에게 명시 후 발행.
4. 의문 발생 시 옵션 제시 대신 quote + 자체 결정으로 진행 + "위 정책에 따라 자동 적용했습니다. 다른 결정 필요하시면 말씀" 한 줄 안내.

관련: [[preserve_decision_literal]] (= 사용자 발언 원문 quote), [[read_user_words_literally]] (= 사용자 발언을 IDE 정황 등으로 치환 금지).
