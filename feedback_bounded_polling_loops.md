---
name: bounded_polling_loops
description: 배포·CI 대기 폴링은 반복 상한 + 빈 조회값 즉시 실패. 검증된 명령을 변형(필드 추가 등)할 땐 단독 실행으로 출력을 먼저 확인
metadata:
  type: feedback
---

**폴링 루프는 반드시 상한을 두고, 조회 값이 비면 즉시 실패로 끝낸다.** `until [ "$(gh run view $ID ...)" = completed ]` 꼴은 `$ID` 가 비면 영원히 돈다.

**Why:** 2026-09-15 boomerang prod 승격 — dev 대기에서 잘 돌던 `gh run list --json databaseId` 에 존재하지 않는 `workflowName` 필드를 붙여 변형 → gh 가 "Unknown JSON field" 로 빈 값 반환 → run id 없이 `until … completed` 무한 루프 → 도구 타임아웃 10분을 꽉 채우고도 정리 안 돼 사용자가 약 20분 뒤 중단. 실제 배포는 36초 만에 끝나 있었다. 사용자: "그 단계에서 20분정도 걸렸던 이유 확인".

**How to apply:**
- 폴링 전 `[ -n "$ID" ] || { echo "no run id"; exit 1; }`. 루프는 `for i in $(seq 1 N)` 처럼 상한, 초과 시 실패 종료.
- 검증된 명령을 변형하면(옵션·JSON 필드 추가) 루프에 넣기 전에 **단독 실행으로 출력을 눈으로 확인**.
- 긴 대기는 별도 백그라운드 명령으로 분리하고, 정리·검증 같은 짧은 작업과 한 명령에 묶지 않는다 (묶으면 어느 단계가 막혔는지 사용자가 알 수 없다).
- **`pgrep -f`/`pkill -f` 로 완료를 기다리지 말 것 — 패턴이 자기 명령줄에 그대로 들어 있어 자신을 매칭한다.** 대상이 이미 끝났는데도 루프가 계속 돌아 "멈춘 듯" 보인다 (2026-09-23 APK 빌드: 실제 32초 만에 끝났는데 `while pgrep -f "scripts/build_android.sh dev"` 가 자기 자신을 잡아 8분 대기 → 사용자가 중단하고 "멈춘듯? 확인해봐"). 같은 뿌리로 `pkill -f` 가 제 셸을 죽여 exit 144 가 난 적도 있다.
- **`[u]vicorn` 같은 자기 제외 패턴도 같은 명령줄에 그 문자열이 또 있으면 소용없다** — `pkill -f "[v]ite … 4181"` 뒤에 `nohup vite … 4181` 을 이어 쓰면 셸 자신이 매칭돼 exit 144 로 죽는다 (2026-09-23 두 번). **kill 과 재기동은 반드시 다른 명령으로 나눈다.**
- `pgrep -f "dist-mock"` 처럼 **조회**도 자기 자신을 잡는다 — pid 가 매번 바뀌면 되살아나는 게 아니라 내 pgrep 을 보고 있는 것. 조회도 `[d]ist-mock` 로.
- **확인용 대기를 foreground 로 길게 잡지 않는다.** 상한은 안전장치인데 사용자에게는 "그만큼 기다리는 것"으로 보이고 대화가 막힌다. 예상 소요가 1~2분을 넘으면 background 로 돌려 알림을 받거나, 예상 시각에 한 번 조회하고, 그 사이에는 답을 먼저 준다 (2026-10-13 ECS 작업 재기동 뒤 시작 로그 확인을 상한 15분 foreground 루프로 → 사용자 중단 "동작 확인하는게 이렇게 오래 걸림? 15분 두는 이유 뭐임". 실제 시작은 4.5분).
- 대신 **결과물을 본다**: 로그 마지막 줄·산출 파일 mtime·완료 표식. 꼭 프로세스를 봐야 하면 `pgrep -f` 대신 시작할 때 받은 PID(`$!`) 를 쓰거나 패턴에 `[b]uild_android` 처럼 자기 제외 장치를 넣는다.

관련: [[verify_edit_applied_before_reporting]] (cherry-pick 체인도 커밋별 종료코드), [[verify_against_code_and_runtime]]
