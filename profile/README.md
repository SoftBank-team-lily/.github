# 🌷 Team Lily

**GitHub 레포 URL 하나로 빌드부터 배포, 트래픽 전환, 모니터링까지 자동으로 처리하는 배포 플랫폼**  
클라우드와 내 PC를 오가도, 넘치는 요청만 나눠 받아도 **공개 주소는 그대로**  
SoftBank Hackathon 2026 in Korea 예선 (Term1)

![](https://img.shields.io/badge/k3s-FFC61C?style=for-the-badge&logo=k3s&logoColor=black)![](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)![](https://img.shields.io/badge/AWS_EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)![](https://img.shields.io/badge/Amazon_ECR-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)![](https://img.shields.io/badge/Nginx_Ingress-009639?style=for-the-badge&logo=nginx&logoColor=white)  
![](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)![](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)![](https://img.shields.io/badge/Kaniko-4285F4?style=for-the-badge)![](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)![](https://img.shields.io/badge/Amazon_RDS_(PostgreSQL)-527FFF?style=for-the-badge&logo=amazonrds&logoColor=white)![](https://img.shields.io/badge/DynamoDB-4053D6?style=for-the-badge&logo=amazondynamodb&logoColor=white)![](https://img.shields.io/badge/Flyway-CC0200?style=for-the-badge&logo=flyway&logoColor=white)  
![](https://img.shields.io/badge/Cloudflare_Tunnel-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)![](https://img.shields.io/badge/CloudWatch-FF4F8B?style=for-the-badge&logo=amazoncloudwatch&logoColor=white)![](https://img.shields.io/badge/Fluent_Bit-49BDA5?style=for-the-badge&logo=fluentbit&logoColor=white)![](https://img.shields.io/badge/Let's_Encrypt-003A70?style=for-the-badge&logo=letsencrypt&logoColor=white)

---

Vercel처럼 GitHub 주소와 인증 정보만 넣으면, 플랫폼이 레포를 클론해 배포하고 **HTTPS가 적용된 접속 URL**을 돌려줌. 배포 위치는 k3s 클러스터(클라우드)와 사용자 PC(온프레미스) 중에서 선택 가능함.

```
GitHub URL + 인증 정보 입력
→ 레포 클론 · 이미지 빌드 (CI)
→ DB 준비 · 마이그레이션 자동 적용
→ 블루-그린 / 카나리 트래픽 전환 (CD · LB)
→ 카나리 판정 통과 시 전환 완료, 에러율 · 응답 시간 악화 시 자동 롤백
→ HTTPS 접속 URL 반환 (https://{app}.apps.lilycloud.kr)
→ 로그 수집 · 메트릭 대시보드
```

핵심은 **안정적인 무중단 배포**임. 기본 모듈을 먼저 완성한 뒤 AI 기능을 단계적으로 붙임.

## 핵심 차별화


|                       | 유지                                                            | 전환                                                                                                            |
| --------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **무중단 재배포**           | 공개 주소. 판정이 끝나기 전까지는 이전 슬롯이 계속 트래픽을 받음                         | Ready 이후 30초간 새 슬롯과 이전 슬롯의 에러율·p95를 비교하고, 통과하면 Service selector만 새 색으로 바꿈. 실패하면 새 버전을 지우고 이번 배포에서 바꾼 스키마를 되돌림 |
| **클라우드 버스팅**          | 공개 주소와 터널                                                     | 로컬 동시 처리 한도를 넘긴 요청만 클라우드로 넘어감. 실측 1,714건 모두 200, 그중 40%를 클라우드가 처리, 부하 종료 20초 뒤 PC로 복귀                         |
| **클라우드와 온프레미스 거점 전환** | 공개 주소. 목적지가 준비되기 전까지는 출발 거점이 트래픽을 받음                          | 주소가 가리키는 거점 전체. CNAME 대상만 바꿈                                                                                  |
| **온프레미스**             | 인바운드 포트는 닫힌 상태로 둠. 인증서 개인키와 AI 키는 PC에 두지 않음. 빌더 서버는 소스를 받지 않음 | 에이전트가 WebSocket으로 먼저 연결하고, clone·빌드는 특권 권한 없이 worker 노드의 Kaniko가 수행함                                          |
| **AI**                | 배포와 롤백은 규칙이 결정함                                               | 규칙으로 분류되지 않는 실패와 설정 키만 모델에 물음. 호출이 실패하면 규칙 결과를 유지하고, 사용자 값과 DB 비밀번호는 보내지 않음                                   |




## 열어 보기


| 열어 볼 곳     | 주소                                                                               | 여기서 확인할 것                                       |
| ---------- | -------------------------------------------------------------------------------- | ----------------------------------------------- |
| Lily 콘솔    |                                                                                  | 레포 URL 하나로 클라우드 / 내 PC 배포 시작, 버스팅 비율 · 거점 전환 조작 |
| 샘플 앱       | [https://blog.apps.lilycloud.kr/version](https://blog.apps.lilycloud.kr/version) | 지금 트래픽을 받는 슬롯 색(blue / green)과 버전               |
| 샘플 앱 반복 호출 | [https://blog.apps.lilycloud.kr/whoami](https://blog.apps.lilycloud.kr/whoami)   | 카나리 비중대로 응답 인스턴스가 나뉘는지                          |
| 모니터링       |                                                                                  | 요청 수 · 5xx 비율 · p95 · 파드 상태 · 실행 로그             |




## 레포에 준비할 것

**없음.** 설정 파일을 따로 쓰지 않아도 `lily-builder`가 레포를 읽고 정함.


| 레포에 없으면            | Lily가 하는 일                                                                                                            |
| ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| `Dockerfile`       | `pom.xml` · `build.gradle` · `package.json` · `requirements.txt` · `pyproject.toml` · `go.mod` · `index.html`을 보고 생성함 |
| 포트 · 헬스 경로 · DB 종류 | 빌드 파일과 설정을 읽어 결정함. PostgreSQL / MySQL 흔적을 찾으면 테넌트 DB를 만들어 접속 정보를 환경변수로 주입함                                            |
| 프론트 · 백엔드 분리 구성    | `backend/` + `frontend/`처럼 서버 하나와 정적 프론트 하나면 한 이미지로 묶어 주소 하나로 띄움. 같은 출처라 CORS 설정이 필요 없음                               |
| 공개 레포가 아님          | 토큰을 넣으면 빌드하는 동안만 k3s Secret으로 Kaniko에 넘기고, 끝나면 삭제함                                                                    |




## 팀 전체 그림

공개 주소는 배포를 반복해도 그대로 유지함. 바뀌는 것은 그 주소가 가리키는 대상임.


|              | 하는 일                                                                                                                                                                           |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **카나리 · 롤백** | 새 버전이 Ready가 되어도 바로 100%로 넘기지 않음. 30초 동안 새 슬롯과 이전 슬롯의 에러율 · p95를 비교하고, 통과하면 Service selector만 새 색으로 변경함. 실패하면 새 버전을 지우고 스키마를 되돌리며, 트래픽은 이전 색에 남김. 이미 넘긴 뒤에는 이전 슬롯을 다시 띄워 복귀시킴. |
| **버스팅**      | 거점이 사용자 PC일 때, 로컬이 동시에 처리하는 요청이 한도를 넘으면 그 요청만 클라우드로 넘김. 공개 주소는 터널에 그대로 있고, 부하가 끝나면 다시 PC가 받음.                                                                                  |
| **거점 전환**    | 같은 주소의 트래픽 전체를 PC와 클라우드 중 한쪽으로 옮김. 버스팅(요청 일부)이나 블루-그린(한 거점 안의 버전 교체)과는 다름. CNAME 내용물만 바꾸고, 목적지가 준비되기 전에는 출발 거점을 유지함.                                                           |
| **모니터링**     | 앱 코드를 고치지 않고 입구에서 관측함. 요청 지표와 슬롯 · 버전이 붙은 로그를 모아, 앱마다 지금 괜찮은지를 색과 한 줄로 알림.                                                                                                     |




## 아키텍처

```mermaid
flowchart LR
    FE["lily-frontend<br/>계정 · 프로젝트"] --> BU["lily-builder<br/>레포 확인 · 빌드 · 배포 요청"]

    BU -->|클라우드| CD["lily-cicd<br/>블루-그린 / 카나리 · 롤백 · DB"]
    CD --> K8S(["k3s · Nginx Ingress<br/>HTTPS 접속 URL"])

    BU -->|"내 PC<br/>(WebSocket)"| ON["lily-on-premise<br/>로컬 Docker 블루-그린"]
    ON -.->|"버스팅: 넘친 요청만"| K8S
    ON <-.->|"거점 전환: 주소 전체"| K8S

    K8S --> OB["lily-observer<br/>지표 · 로그 · 위험도 판정"]
    ON --> OB
```





## 어디서 실패해도 주소는 살아 있음


| 실패 지점    | 사용자가 보는 것       | Lily가 하는 일                                    |
| -------- | --------------- | --------------------------------------------- |
| 레포 확인    | 이유가 적힌 `FAILED` | 빌드를 시작하지 않음. 브랜치나 앱을 못 찾으면 폴더 목록과 함께 알림       |
| 이미지 빌드   | 단계별 빌드 로그와 원인   | `AiAdvisor`가 로그를 읽고 원인을 기록함. 기존 버전은 계속 서비스 중임 |
| 새 슬롯 기동  | 기존 버전 그대로       | 비활성 색 슬롯에만 띄우고, Ready 전에는 트래픽을 넘기지 않음         |
| 카나리 판정   | 기존 버전 그대로       | 새 버전을 지우고 스키마를 되돌림. 트래픽은 이전 색에 남음             |
| 전환 이후 장애 | 잠깐의 오류 뒤 이전 버전  | 이전 슬롯을 다시 띄워 트래픽을 복귀시킴                        |
| 내 PC 과부하 | 응답은 계속 200      | 넘친 요청만 클라우드로 보내고, 부하가 끝나면 PC로 복귀함             |




## 직접 확인한 결과


| 시점         | 확인한 것                                                                                   | 결과                                                  |
| ---------- | --------------------------------------------------------------------------------------- | --------------------------------------------------- |
| 2026-09-30 | k3s(server 1 + worker 2)에서 `lily-builder → lily-cicd → lily-db-provisioner → RDS` 전체 흐름 | 샘플 앱이 RDS 테넌트 DB에 붙어 Flyway 적용, 글 작성이 RDS에 저장       |
| 2026-09-30 | 재배포 (blue → green)                                                                      | 같은 DB 재사용, 데이터 유지                                   |
| 2026-09-30 | 처음 보는 앱 이름(`blog2`) 배포                                                                  | ECR 저장소 자동 생성부터 접속까지 한 번에 성공                        |
|            | 내 PC 거점 버스팅                                                                             | 1,714건 모두 200, 그중 40%를 클라우드가 처리, 부하 종료 20초 뒤 PC로 복귀 |




## Infra


| 구성       | 내용                                                                   |
| -------- | -------------------------------------------------------------------- |
| 클러스터     | k3s — server 1대(관리) + worker 2대(앱 실행), AWS EC2 t3.medium             |
| 트래픽      | Nginx Ingress — 카나리 · 블루-그린 트래픽 전환                                   |
| 이미지      | Kaniko 빌드 → Amazon ECR (내 PC는 로컬 Docker 빌드, 레지스트리 없음)                |
| 온프레미스 노출 | Cloudflare Tunnel — 인바운드 포트 없이 공개 주소 연결, 인증서는 Cloudflare 엣지          |
| 데이터      | 플랫폼 DB: DynamoDB / 사용자 앱 DB: Amazon RDS (PostgreSQL)                 |
| 비밀 정보    | 테넌트 DB 비밀번호는 SSM Parameter Store(SecureString), 사용자 PC에는 AI 키를 두지 않음 |
| 관측       | CloudWatch Agent(지표) · Fluent Bit(로그) → Amazon CloudWatch            |




## 핵심 모듈


| 모듈                            | 내용                                    | 담당       |
| ----------------------------- | ------------------------------------- | -------- |
| **CI / CD Module**            | 블루-그린 / 카나리 배포 · 카나리 판정 · 롤백          | 이현수, 박준석 |
| **Database Migration Module** | 무중단 스키마 변경 · 판정 실패 시 스키마 되돌리기         | 이현수, 박준석 |
| **On-Premise Module**         | 사용자 PC 로컬 블루-그린 · 버스팅 · 거점 전환         | 이현수, 박준석 |
| **Logging Module**            | 로그 기록 · 위험도 판정 · 저장 · 롤백              | 심형규, 최도일 |
| **Monitoring System**         | 요청 지표 · 슬롯 / 버전 로그 기반 앱 상태 시각화        | 심형규, 최도일 |
| **Load Balancing**            | Nginx 기반 트래픽 분산 (다수 컨테이너 운영으로 비용 효율화) | 이도현, 차주혜 |




## AI가 하는 일

규칙을 먼저 적용함. 규칙으로 정하기 어려운 것만 모델에 묻고, 키가 없거나 호출이 실패하면 배포를 막지 않고 규칙 결과를 유지함.

- **설정 분류 · 실패 진단** — `lily-builder`의 `AiAdvisor`. Claude 키가 있으면 Claude, 없고 OpenAI 키가 있으면 OpenAI. 사용자 값과 DB 비밀번호는 보내지 않음. 코드 수정이 필요하면 고칠 목록은 비우고 원인만 기록함.
- **애매한 선택만** — `lily-jev`는 확신도가 높을 때만 답을 돌려주고, 실패하면 빈 값을 반환함. 요청마다 나누는 버스팅, 슬롯 비율, DB 생성, 화면 전달에는 넣지 않음.

롤백은 직전 슬롯으로 트래픽을 되돌리는 호출이며, 그 호출이 레포에 수정 PR을 만들지는 않음.

기록은 [결정 기록](결정-기록.md), 모듈이 주고받는 경로는 [모듈 계약](모듈-계약.md)에 있음.

## 다음에 붙일 것


| 항목            | 지금                      | 다음                                                  |
| ------------- | ----------------------- | --------------------------------------------------- |
| `lily-jev` 연결 | builder의 코드 결함 판정에만 연결됨 | 빌드 파일이 여럿일 때 대상 고르기, DB 엔진이 둘 다 보일 때 고르기, 롤백 직전 재확인 |
| 공개 랜딩의 배포 시연  | 6단계 흐름을 보여주는 연출임        | 실제 배포 API와 연결                                       |




## Repositories


| 레포                                                                                             | 설명                                                                                                                                                            |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[lily-builder](https://github.com/SoftBank-team-lily/lily-builder)`                           | **배포의 입구이자 컨트롤 플레인.** GitHub 주소를 받아 이미지를 빌드하고, 클라우드면 lily-cicd에 배포를 요청함. 내 PC면 붙어 있는 에이전트로 잡을 보냄. Dockerfile이 없으면 빌드 파일을 보고 만들고, 포트 · 헬스 경로 · DB도 레포를 읽어 결정함. |
| `[lily-cicd](https://github.com/SoftBank-team-lily/lily-cicd)`                                 | 레지스트리에 있는 이미지를 k3s에 올림. 빌드는 하지 않음. 블루-그린 / 카나리 판정, 트래픽 전환, 롤백, 스키마 적용을 담당함. DB가 필요하면 lily-db-provisioner로 테넌트 DB를 만들고 접속 정보를 앱 환경변수로 주입함.                     |
| `[lily-frontend](https://github.com/SoftBank-team-lily/lily-frontend)`                         | 로그인 · 프로젝트 · 클라우드 / 내 PC 배포 화면. 버스팅 비율과 거점 전환도 여기서 조작함.                                                                                                       |
| `[lily-monitoring-dashboard](https://github.com/SoftBank-team-lily/lily-monitoring-dashboard)` | 앱별 관측 화면. 요청 수 · 5xx 비율 · p95 · 파드 상태 · 카나리 비중 · 실행 로그.                                                                                                       |
| `[lily-on-premise](https://github.com/SoftBank-team-lily/lily-on-premise)`                     | 사용자 PC 에이전트. 로컬 블루-그린, 버스팅, 거점 전환. 인바운드 포트를 열지 않고 WebSocket으로 먼저 연결함.                                                                                         |
| lily-db-provisioner                                                                            | 프로젝트마다 RDS database와 계정을 생성함. 공개 페이지가 없어 링크는 제외함.                                                                                                             |
| lily-loadbalancer                                                                              | Nginx Ingress 매니페스트. Host로 앱을 나누고, selector로 블루-그린을 선택함. 공개 페이지가 없어 링크는 제외함.                                                                                  |
| lily-observer                                                                                  | 요청 지표 · 버전 붙은 로그 · 위험도 판정 API. 공개 페이지가 없어 링크는 제외함.                                                                                                            |
| `[lily-jev](https://github.com/SoftBank-team-lily/lily-jev)`                                   | 애매한 선택만 짧게 묻는 클라이언트. 실패하거나 확신도가 낮으면 빈 값을 돌려주고, 호출한 쪽은 규칙을 유지함.                                                                                                |
| `[lily-blog-sample](https://github.com/SoftBank-team-lily/lily-blog-sample)`                   | 배포 대상 샘플 앱 — 플랫폼 검증용 블로그 CRUD (Spring Boot), 장애 주입 엔드포인트 포함                                                                                                   |
| lily-load-test                                                                                 | 시연 확인. 배포가 공개 주소까지 열리는지, 버스팅이 클라우드로 넘겼다가 돌아오는지. 공개 페이지가 없어 링크는 제외함.                                                                                           |
| `[.github](https://github.com/SoftBank-team-lily/.github)`                                     | 이 소개 페이지                                                                                                                                                      |




## Team


| 이름  | Role | 담당 모듈                               |
| --- | ---- | ----------------------------------- |
| 최도일 | 팀장   | Logging · Monitoring · Infra        |
| 심형규 | 팀원   | Logging · Monitoring · Frontend     |
| 박준석 | 팀원   | CI / CD · DB Migration · On-Premise |
| 이현수 | 팀원   | CI / CD · DB Migration · On-Premise |
| 이도현 | 팀원   | Load Balancing · AI                 |
| 차주혜 | 팀원   | Load Balancing · AI                 |


