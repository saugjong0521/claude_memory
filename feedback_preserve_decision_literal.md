---
name: preserve-decision-literal
description: 사용자가 명시한 결정·의도·정책 발언은 자연어 원문 그대로 plan doc 의 § 안에 quote. AskUserQuestion 의 선택지 label 만 의존하면 본질 의도 추적 부족
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d46e4ac-70e3-4d2c-ab0d-104af852a354
---

사용자가 결정 / 의도 / 정책 발언을 했을 때, 그 발언의 자연어 원문을 plan doc (= docs/012, docs/013 같은) 의 § 안에 quote 로 보존. AskUserQuestion 의 선택지 label 만 보고 후속 작업 진행하지 말 것.

**Why:** 2026-05-27 docs/013 작성 흐름에서 사용자가 (1) "pre_paid_block은 패스되야함" + AskUserQuestion → "v1 입금자도 분모 포함" 선택 → 211 분포 (2) 잠시 후 "왜 211, 209여야 되는거 아니야?" 정정 (3) → 그 다음 docs/013 §2.3 작성 시 사용자가 "1번은 아니라고 말했지? 해당 블록 아래도 넣으라고 한 3번 말한 것 같은데" 답답함 표명. 두 발언 (= step 1 의 "패스" + step 2 의 "209") 의 본질을 자연어로 보존 안 하고 자체 해석으로 진행 → 사용자가 동일 결정 여러 번 강조하게 됨.

**How to apply:** plan doc 작성 시 § "사용자 결정 사항" 같은 sub-section 에 사용자 원문 quote 보존. e.g. `> 사용자 (2026-05-27): "pre_paid_block은 패스되야함 / v1 입금자도 분모에 포함"`. AskUserQuestion 의 선택지 label 은 보조 정보로만 사용. 의문 시 quote 와 cross-check.

관련: [[no_repeat_decided_questions]] (= 이미 결정된 사항 반복 질문 금지), [[no_guess_in_docs]] (= docs 표 작성 시 grep + 본문 read).
