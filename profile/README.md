<h1 align="center">🌷 Team Lily</h1>

<p align="center">
  <b>GitHub 레포 URL 하나로 빌드부터 배포, 트래픽 전환, 모니터링까지 자동으로 처리하는 배포 플랫폼</b><br/>
  SoftBank Hackathon 2026 in Korea 예선 (Term1)
</p>

<p align="center">
  <img src="https://img.shields.io/badge/k3s-FFC61C?style=for-the-badge&logo=k3s&logoColor=black" />
  <img src="https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS_EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" />
  <img src="https://img.shields.io/badge/Amazon_ECR-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" />
  <img src="https://img.shields.io/badge/Nginx_Ingress-009639?style=for-the-badge&logo=nginx&logoColor=white" />
  <br/>
  <img src="https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white" />
  <img src="https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/Kaniko-4285F4?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Amazon_RDS_(PostgreSQL)-527FFF?style=for-the-badge&logo=amazonrds&logoColor=white" />
  <img src="https://img.shields.io/badge/DynamoDB-4053D6?style=for-the-badge&logo=amazondynamodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Flyway-CC0200?style=for-the-badge&logo=flyway&logoColor=white" />
  <br/>
  <img src="https://img.shields.io/badge/CloudWatch-FF4F8B?style=for-the-badge&logo=amazoncloudwatch&logoColor=white" />
  <img src="https://img.shields.io/badge/Fluent_Bit-49BDA5?style=for-the-badge&logo=fluentbit&logoColor=white" />
  <img src="https://img.shields.io/badge/Let's_Encrypt-003A70?style=for-the-badge&logo=letsencrypt&logoColor=white" />
</p>

---

Vercel처럼 GitHub 주소와 인증 정보만 넣으면, 플랫폼이 레포를 클론해 k3s 클러스터에 배포하고 **HTTPS가 적용된 접속 URL**을 돌려줘요.

```
GitHub URL + 인증 정보 입력
→ 레포 클론 · 이미지 빌드 · ECR 저장 (CI)
→ DB 마이그레이션 자동 적용
→ k3s 클러스터 배포 · 블루-그린 / 카나리 트래픽 전환 (CD · LB)
→ 헬스체크 통과 시 전환 완료, 에러율 급증 시 자동 롤백
→ Let's Encrypt HTTPS 적용 · 접속 URL 반환
→ 로그 수집 · 메트릭 대시보드
```

핵심은 **안정적인 무중단 배포**예요. 기본 모듈을 먼저 완성한 뒤 AI 기능을 단계적으로 붙여요.

## Infra

| 구성 | 내용 |
|---|---|
| 클러스터 | k3s — server 1대(관리) + worker 2대(앱 실행), AWS EC2 t3.medium |
| 트래픽 | Nginx Ingress — 카나리 · 블루-그린 트래픽 전환 |
| 이미지 | Kaniko 빌드 → Amazon ECR |
| 데이터 | 플랫폼 DB: DynamoDB / 사용자 앱 DB: Amazon RDS (PostgreSQL) |
| 관측 | CloudWatch Agent(지표) · Fluent Bit(로그) → Amazon CloudWatch |

## 핵심 모듈

| 모듈 | 내용 | 담당 |
|---|---|---|
| **CI / CD Module** | 블루-그린 / 카나리 배포 | 이현수, 박준석 |
| **Database Migration Module** | 무중단 스키마 변경 | 이현수, 박준석 |
| **Logging Module** | 로그 기록 · 위험도 판정 · 저장 · 롤백 | 심형규, 최도일 |
| **Monitoring System** | CloudWatch 기반 CPU / 메모리 지표 수집 및 시각화 | 심형규, 최도일 |
| **Load Balancing** | Nginx 기반 트래픽 분산 (다수 컨테이너 운영으로 비용 효율화) | 이도현, 차주혜 |

## 확장 계획 (AI)

기본 모듈이 동작한 뒤 여유가 되는 만큼 추가해요.

- **AI 장애 분석** — 오류 로그를 AI 에이전트에 넘겨 원인 범위를 좁히고, 비개발자도 이해할 수 있게 설명
- **빠른 분류 모델 결합** — 배포 파이프라인의 예/아니오 판단(에러 여부, 롤백 여부 등)을 경량 분류 모델로 빠르게 처리
- **AI 수정 PR** — 분석 결과를 바탕으로 AI 에이전트가 수정본을 GitHub PR로 제안
- **맞춤 대시보드** — 배포된 서비스의 도메인을 분석해 필요한 메트릭 패널을 자동 구성
- **권한 분리** — 서비스별 접근 권한, 운영 서버 배포 승인자 지정
- 서브도메인 제공, manifest 자동화

## Repositories

| 레포 | 설명 |
|---|---|
| [`lily-blog-sample`](https://github.com/SoftBank-team-lily/lily-blog-sample) | 배포 대상 샘플 앱 — 플랫폼 검증용 블로그 CRUD (Spring Boot) |
| [`.github`](https://github.com/SoftBank-team-lily/.github) | 이 소개 페이지 |

> 모듈별 레포는 개발을 시작하면서 추가돼요.

## Team

| 이름 | Role | 담당 모듈 |
|---|---|---|
| 최도일 | 팀장 | Logging · Monitoring |
| 심형규 | 팀원 | Logging · Monitoring |
| 박준석 | 팀원 | CI / CD · DB Migration |
| 이현수 | 팀원 | CI / CD · DB Migration |
| 이도현 | 팀원 | Load Balancing |
| 차주혜 | 팀원 | Load Balancing |
