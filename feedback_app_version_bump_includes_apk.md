---
name: app_version_bump_includes_apk
description: boomerang 손님앱 "버전 X.Y.Z 로" 지시는 package.json 올리기 + 웹 배포 + Android APK 빌드·S3 업로드(dev·prod 각 체크아웃)까지 한 세트. 웹 번들만 올리고 "완료" 보고 금지
metadata:
  type: feedback
---

**"앱 버전 0.1.7 ㄱㄱ" = package.json 올림 → develop/main push(웹 배포) → `scripts/build_android.sh dev`(boomerang-dev, develop) → `scripts/build_android.sh prod`(boomerang-prod, main) → `downloads/app.json` 의 version 으로 확인.** 셋 중 하나라도 빠지면 버전 작업이 끝난 게 아니다.

**Why:** 2026-09-15 — "0.1.7 ㄱㄱ" 에 package.json 만 올리고 prod 웹 배포까지 확인한 뒤 "dev·prod 모두 0.1.7" 로 보고. admin 앱 버전 패널의 Android 앱은 어제 빌드한 v0.1.6 그대로였고, 사용자: "0.1.7만들면 배포까지 되야지 정상이지 뭐하는짓?". APK 는 별도 산출물(Capacitor 빌드 → S3 `downloads/boomerang(-dev).apk` + `app.json`)이라 package.json 만으론 안 바뀐다. 강제 업데이트 설정(최소 0.1.6 / 안내 0.1.7)까지 APK 없이 걸면 사용자가 옛 APK 를 다시 받는다.

**How to apply:**
- 버전 지시를 받으면 체크리스트 3항목(웹 dev·prod, APK dev·prod, app.json 확인)을 전부 돌리고 각 결과를 보고. APK 빌드는 브랜치 가드(dev=develop, prod=main) 때문에 각 클론에서 돌린다.
- 보고 문구에 "웹 번들 0.1.7 / APK 0.1.7(app.json)" 처럼 두 산출물을 따로 적는다.
- 관련: [[prod_deploy_needs_explicit_instruction]] (prod 지시 범위 안에 APK 도 포함), [[read_user_words_literally]]
