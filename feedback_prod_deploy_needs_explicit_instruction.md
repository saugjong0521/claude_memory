# prod 반영은 "prod 올려" 명시 지시가 있을 때만

**Rule:** `main` push · prd clone cherry-pick · prd migration · prd 재기동 요청은 **사용자가 그 시점에 prod 를 지목한 말**("prod 올려줘", "prd 반영")이 있을 때만 한다. plan doc 의 A-N 단계에 "prd 포팅" 이 적혀 있어도, "남은 결정들 ㄱㄱ"·"계속 진행"·"다음 작업" 은 **dev 범위 안의 진행**이지 prod 지시가 아니다. 같은 흐름의 후속이라도 prod 는 "고지만" 이 아니라 **컨펌**이다 ([[feedback_git_conventions]] 의 후속-고지 예외는 dev/develop 에만 적용).

**Why:** 2026-09-03 현장 미션(docs/030) — "업데이트 하고 남은결정들도 ㄱㄱ" 를 plan 의 A-4(prd 포팅)까지로 읽고 손님앱·admin main 에 push → deploy-prd 가 그대로 나가 약 20분간 미승인 UI(3km 필터·새 매장 탭)가 라이브됨. 사용자: "아닌데? prod는 올리면 안됐는데?". revert 커밋으로 원복.

**How to apply:**
- prod 를 건드리는 액션 직전에 "이 turn 의 사용자 말에 prod 가 있는가" 를 확인. 없으면 dev 까지 마치고 "prd 반영은 지시 주시면" 한 줄로 멈춘다.
- plan doc 의 단계표에 prd 포팅이 있어도 그 행은 **별도 착수 신호** 가 필요한 단계로 취급 ([[feedback_plan_stage_no_implementation]] 와 같은 게이트).
- 잘못 올렸으면 revert 커밋(강제 push X)으로 포팅 전 트리와 diff 0 을 assert 하고 재배포, DB migration 실행 여부·서비스 재기동 여부를 먼저 확인해 런타임 영향을 사실대로 보고.
