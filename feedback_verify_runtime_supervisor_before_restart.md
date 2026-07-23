---
name: 실행 중 서비스는 런타임 supervisor가 진실 (negative→positive 식별)
description: 실행 중 서비스의 재시작/중지/리로드/기동 질문은 repo 스크립트가 아니라 라이브 supervisor(systemd/docker 등)가 진실. negative 결론("X가 안 띄움")은 positive 식별 probe를 강제, PPID=1은 결론 아닌 분기. mutation 처방 전 supervisor+restart 정책 확인
type: feedback
originSessionId: 552687bc-ce8e-4bce-96e7-4e732ad1fbd9
---
실행 중인 서비스/프로세스를 재시작·중지·리로드·"어떻게 띄우나" 다룰 때, **진실의 출처는 repo 의 스크립트가 아니라 라이브 supervisor (systemd / docker / k8s / supervisord / 수동)** 다. repo 산출물 (script / Makefile / Dockerfile / README 실행법 / CI) 은 "의도" 를 기술할 뿐 실제 감독 방식이 아니다. [feedback_docs_as_guide_code_as_truth](feedback_docs_as_guide_code_as_truth.md) 의 런타임 확장.

**Why:** 2026-06-10 사고 — `kstadium-referral-backend` 텔레그램 토큰 교체 후 "재시작 코드" 요청. `scripts/restart_gunicorn.sh` 를 읽고 "이게 서비스 도는 방식" 으로 진실 승격. 라이브 master 가 `PPID=1` + `.server.pid` 없음 + argv 가 스크립트 (`--daemon --pid`) 와 불일치 = "스크립트가 안 띄웠다" 는 falsification 이었는데, 이를 "사람이 손으로 disown 한 데몬" 이라는 **미관측 행위자** 로 메워 프레임 유지. `PPID=1` (= 고아 데몬 OR PID-1 supervisor 둘 다 가능한 모호 신호) 을 systemd 가능성 버리고 (a) 에 끼워맞춤. 결과: (1) 올바른 `sudo systemctl restart` 를 놓침, (2) `kill -TERM` + `run_gunicorn.sh` 처방 → unit 의 `Restart=always` (RestartSec=3) 가 3초 내 자기 gunicorn 을 8000 에 재기동 → 이중 인스턴스 / 포트 충돌 / worker_leaders split-brain 가능한 **prod 사고 절차**. 사용자가 "systemd restart 하면 되지 않음?" 으로 교정. 결정적 tell = /proc/PID/**environ** (확증) 은 읽고 sibling /proc/PID/**cgroup** (반증) 은 안 읽은 비대칭 증거 수집.

**How to apply:**

1. **트리거**:
   - 좁게: 실행 중 서비스/프로세스의 재시작·중지·리로드·기동 방법 질문.
   - 넓게: repo 산출물에서 얻은 "X 동작 모델" 로 행동하려는데 런타임 관측이 모델과 안 맞을 때.
   - 자기탐지: 내가 "이건 [내가 읽은 그것] 이 안 한 거네" 라고 말하는 순간 = 방아쇠.

2. **negative → positive (핵심)**:
   - negative 결론 ("스크립트가 안 띄움") 을 특정 positive ("사람이 손으로") 로 비약 금지. negative 는 후보집합만 줄일 뿐 하나를 고르지 않음.
   - positive 식별을 추론 아닌 **probe 로 실행**: `cat /proc/PID/cgroup`, `systemctl status <PID>`, ps PPID 체인, `docker inspect`, supervisorctl.
   - `PPID=1` 은 결론 아닌 **분기** (고아 데몬 OR systemd) → cgroup/systemctl 로 즉시 해소. (`0::/system.slice/<unit>.service` 한 줄이면 끝)

3. **멈춤 기준**: "일관된 스토리 생김" 이 아니라 **"내 권고를 바꿀 경쟁가설을 배제함"**. 다음 검증이 read-only 로 싸고 + 권하려는 행동이 mutation/비가역 (특히 prod) 이면 → 무조건 probe 먼저.

4. **anti-pattern 자기탐지**:
   - 미관측 행위자 발명 ("누가 ~했겠지") 으로 설명 구멍 메우기 → 설명이 관측 안 된 행동을 가정해야 성립하면 프레임이 틀렸다는 깃발.
   - 비대칭 증거 수집 = 확증 파일은 읽고 인접한 싼 반증 파일은 건너뛰기.

5. **mutation 전 supervisor 정책 확인**: 재시작/중지 처방 전 supervisor 의 restart 정책 (`Restart=always` 등) 확인. 수동 stop/start 는 supervisor 가 없다고 **positive 확인된 경우에만**; 있으면 supervisor-native 명령 (`systemctl restart` 등) 사용.

6. **수동(비-supervisor) 프로세스를 재기동할 때는 명령뿐 아니라 env 까지 복제**: cmdline 만 복제해 다시 띄우면 원 기동 절차의 **환경 정리 (unset/export) 가 누락**될 수 있다. `/proc/PID/environ` 이 안 읽히면 repo 의 기동/배포 스크립트 (`scripts/deploy_*.sh` 등) 에 env 격리 절차가 내장돼 있는지 확인하고 **그 스크립트로 재기동**하는 게 기본값.
   - **Why (2026-07-23 사고)**: kstadium-shop dev 서버를 수동 `nohup uvicorn` 으로 3회 재시작 — 정식 절차 `scripts/deploy_dev.sh` 의 `unset KSTA_RPC_URL` (= `~/.bashrc` 전역 mainnet export 가 `.env.dev` 를 덮는 것 차단) 을 우회 → 잔액 조회가 mainnet 으로 가서 "집금지갑 0 KSTA" 오판, 사용자가 지급 실패로 발견. **07-22 에 이미 문서화된 사고의 재발** — docs/000 §7 을 읽지 않고 재기동한 것이 원인. rule 1 (docs 는 supervisor 가 아니다) 과 모순 아님: supervisor 식별은 라이브가 진실이되, **기동 "절차" (env 정리 포함) 는 repo 스크립트가 담고 있을 수 있다** — 둘 다 확인.
