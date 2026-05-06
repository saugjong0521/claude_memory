---
name: Answer existence/yes-no questions literally before implementing
description: When user asks whether something exists ("있어?", "있냐"), answer the factual question first; do not preemptively implement it
type: feedback
originSessionId: c36c37e7-44d1-455b-8d6b-f1790104d445
---
사용자가 "X 있어?" / "X 있냐?" 처럼 **존재 여부**를 묻는 질문을 할 때는, 먼저 "있다/없다" 사실만 답한다. 바로 구현으로 뛰어들지 않는다.

**Why:** 2026-04-23 세션에서 "프론트에 992~999 보존 리셋 버튼 있냐" 는 질문에 대해 사실 확인 대신 엔드포인트·서비스 함수·UI 버튼·docs 를 한 커밋으로 만들어버려 되돌려야 했다 (커밋 291d447 → reset --hard). 사용자는 **필요한지 아직 결정하지 않았음** — 존재 여부만 확인하고 있던 것.

**How to apply:**
- "있어?" / "있냐" / "있는지" / "하나?" / "가능해?" 같은 질문에는 **먼저 Yes/No + 근거**로만 답한다.
- 없다는 걸 확인한 뒤 사용자가 "만들어줘" / "넣어줘" / "추가해" 등 명시적 구현 지시를 내리면 그때 구현한다.
- 기존 CLI 스크립트/파일만 있고 UI 가 없다 같은 상황에서는 "CLI 는 있고 UI 는 없다" 수준에서 멈추고, 추가 지시를 기다린다.
- 구현이 사소해 보여도 먼저 확인. 되돌리는 비용이 질문 비용보다 크다.
