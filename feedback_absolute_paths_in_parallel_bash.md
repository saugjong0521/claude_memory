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
