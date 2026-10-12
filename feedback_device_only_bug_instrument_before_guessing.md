---
name: device_only_bug_instrument_before_guessing
description: 폰(네이티브 앱)에서만 재현되고 서버에 흔적이 없는 문제는 추측 수정 빌드를 반복하지 말고, 첫 빌드부터 단계별 진단을 서버 로그로 보내는 코드를 넣는다. 같은 파일명 APK 는 CDN 캐시로 옛 빌드가 깔린다 — 빌드마다 바뀌는 쿼리로 받게. Capacitor 플러그인 객체(Proxy)는 async 함수에서 return/await 하면 thenable 로 오인돼 영원히 멈춘다
metadata:
  type: feedback
---

**폰에서만 재현되는 문제(푸시 토큰 미등록 등)는 "추측 → 수정 → APK 빌드 → 사용자 설치 → 확인" 을 돌리기 전에, 첫 빌드에 단계별 진단을 심는다.** 한 바퀴가 10분 이상이고 사용자 손을 타기 때문에 추측 한 번이 비싸다.

**Why (2026-10-12 boomerang 손님앱 푸시):** 폰이 FCM 토큰을 서버에 등록하지 않았다. 서버에는 요청 흔적이 0이라 원인이 안 보였는데, 동적 import 의심 → 정적 import 로 교체 → 타임아웃 추가 … 추측 빌드를 **4번** 사용자에게 설치시킨 뒤에야 "단계마다 서버로 진단을 보내는" 빌드를 넣었고, 그 즉시 원인이 잡혔다(아래 Capacitor 항목). 중간에 폰이 **이전 APK 를 받는** 일(CloudFront 캐시)까지 겹쳐 두 바퀴가 더 낭비됐다. 사용자 "오늘 내역 메모리에 잘 정리 뭐가 문제였는지(추후 APP빌드시에도 같은 내용 있을 수 있기에)".

**How to apply:**
1. **첫 수정 빌드 = 진단 빌드.** 서버에 흔적이 없으면 먼저 `POST /api/.../diag {stage, detail}` 같은 엔드포인트(저장 없이 로그만)를 만들고, 의심 경로의 **모든 단계 앞뒤**에 진단을 넣는다. 멈춤이 의심되면 진단 전송을 `await` 하고 `fetch(..., {keepalive:true})` 로 보내 다음 호출이 앱을 죽여도 직전 단계가 닿게 한다. 삼키던 `.catch(() => {})` 는 진단으로 바꾼다. 로그에 **빌드 꼬리표**(semver 가 아니라 빌드마다 바뀌는 값)를 같이 찍어 "새 빌드가 돌고 있는가" 부터 가린다.
2. **같은 파일명으로 덮어쓰는 APK 는 캐시된 옛 파일이 깔린다.** CloudFront `max-age` + 브라우저 캐시. 다운로드 링크에 빌드마다 바뀌는 값(`?v=<sha256 앞 12자>`)을 붙이고, 급하면 그 직접 링크를 준다. 사용자가 "설치했다" 고 해도 **빌드 꼬리표로 확인**한 뒤 로그를 해석한다(설정 탭 앱 버전 옆에 꼬리표 표시).
3. **Capacitor 플러그인 객체는 Promise 에 태우지 않는다.** `registerPlugin()` 이 돌려주는 Proxy 는 어떤 속성이든 네이티브 메서드로 만들어 `.then` 도 함수가 된다 → `async function load(){ return PushNotifications }` 처럼 **async 함수에서 return 하거나 await 하면** JS 가 thenable 로 보고 네이티브의 없는 `then` 을 영원히 기다린다(예외도 없음). 로더는 동기 함수로, `const p = load()` 로 받는다. 증상 = 진단이 `await load()` 직전까지만 오고 그 뒤가 없음.
4. **클라이언트 가드(ref·localStorage)는 로그아웃에 리셋**한다 — "손님당 1회" 가드가 세션을 넘어 남으면 재로그인 때 토큰 재등록이 안 된다.
5. 폰을 PC 에 USB 로 붙일 수 있으면 `adb logcat` / `chrome://inspect` 가 가장 빠르다 — 처음에 물어본다.

관련: [[verify_against_code_and_runtime]] (런타임이 진실 — 폰도 런타임), [[bounded_polling_loops]], [[absolute_paths_in_parallel_bash]] (같은 날 발견한 도구 쪽 함정)
