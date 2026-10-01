---
name: config_not_env_for_addresses
description: 환경별 주소·URL·도메인 같은 파생 가능한 상수는 코드 config(APP_ENV 분기)에 두고, env 파일에는 API 키·비밀값·DB 접속 정보 같은 비밀/외부 의존값만 둔다
metadata:
  type: feedback
---

**env 는 비밀값·외부 의존값 전용, 주소는 config 코드.** 콘솔 URL·도메인·콜백 주소처럼 환경(dev/prod)에서 규칙적으로 파생되는 값은 `settings` 의 `APP_ENV` 분기(property)로 두고, `.env.*` 에는 API 키·시크릿·DB URL·지갑 키처럼 코드에 넣을 수 없는 값만 둔다.

**Why:** 2026-08-25 kstadium-shop — 가맹점 관리 앱 주소를 `MANAGE_CONSOLE_URL` env 로 추가하자 사용자: "주소같은건 env로 둘필요는 없고 config로 두는게 날것같은데 env는 api key등만". env 에 주소가 흩어지면 dev/prd 두 파일을 따로 맞춰야 하고, gitignore 라 코드 리뷰·이력에 남지 않는다.

**How to apply:**
- 새 설정값을 만들 때 자문: "이 값이 환경 이름만 알면 정해지나?" → yes 면 config 파생. "비밀이거나 외부에서 발급된 값인가?" → env.
- 기존 env 에 같은 성격의 키가 이미 있어도 관행을 따르지 말고 config 로 옮기며 정리한다 (같은 날 `ADMIN/RECRUIT/MANAGE_CONSOLE_URL` 3종을 `APP_ENV` 파생 property 로 통합).
- env 키는 예외 override 용으로만 남길 수 있으나 기본 운용은 미기재.
