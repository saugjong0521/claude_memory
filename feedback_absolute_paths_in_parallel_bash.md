---
name: absolute_paths_in_parallel_bash
description: Bash 도구를 한 응답에서 여러 개 병렬로 부르면 작업 디렉터리가 섞인다 — `cd` 에 의존하지 말고 모든 경로·git 은 절대 경로(`git -C`, `cd /abs && …` 한 줄 안) 로. npm install 이 엉뚱한 repo 에 package.json 을 만든 일
metadata:
  type: feedback
---

**한 응답에서 Bash 호출을 여러 개 병렬로 보내면 호출 사이에 작업 디렉터리가 바뀐다("Shell cwd was reset" / 다른 호출의 `cd` 가 반영됨).** 상대 경로로 쓴 명령이 다른 repo 에서 실행된다.

**Why (2026-10-12 boomerang):** 손님앱 `npm install @capacitor/push-notifications` 가 **백엔드 repo** 에 `package.json`·`node_modules` 를 만들었고(빌드가 "모듈을 못 찾음" 으로 실패해서 발견), `git add docs/…` 가 "pathspec did not match" 로 두 번 실패했고, 검수 스크립트가 `android/` 안에서 실행돼 파일을 못 찾았다. 전부 병렬 호출 중 cwd 가 섞인 것.

**How to apply:**
- 병렬 Bash 에서는 **모든 명령을 절대 경로로**: `git -C /abs/repo …`, `cd /abs/repo && npm …` 를 **같은 명령 줄 안에서** 쓴다. 호출 간에 cwd 가 유지된다고 가정하지 않는다.
- 스크립트를 `subprocess` 로 띄울 때도 `cwd=` 를 명시한다.
- 설치·커밋처럼 위치가 중요한 명령 뒤에는 `grep -n … package.json` / `git -C … status --short` 로 **그 repo 에 들어갔는지** 바로 확인한다.
- 이상한 "파일 없음" 오류가 나면 코드보다 먼저 `pwd` 를 의심한다.

관련: [[verify_edit_applied_before_reporting]], [[device_only_bug_instrument_before_guessing]]

**재발 (2026-10-21 boomerang manage 앱)**: 병렬 Bash 두 개 중 하나가 `cd $M && cat > capacitor.config.json` 을 돌리는 사이 cwd 가 **frontend 체크아웃으로 리셋**돼 손님앱의 `capacitor.config.json` 을 매장 앱 값(appId town.boomerang.manage)으로 덮어썼고 `npx cap add android` 는 "이미 있음" 으로 실패해 들켰다. 커밋 전이라 `git checkout -- capacitor.config.json` 으로 복구. 같은 날 `scripts/build_android.sh` 도 상대 경로라 다른 repo 에서 "No such file" 로 실패.
**추가 규칙**: 파일을 **쓰는** 명령(리다이렉션 `>`·heredoc·`cp`·스캐폴드 도구)은 병렬 호출에 넣지 않거나, 경로를 전부 절대 경로로 쓰고 **쓴 직후 `git status` 로 다른 repo 에 변화가 없는지** 본다. 스크립트 실행도 `/abs/path/script.sh` 로.
