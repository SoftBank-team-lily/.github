# 로그에서 수정 PR까지

다섯 단계의 방향은 코드와 같다. 시작하는 쪽, jev를 부르는 쪽, PR을 여는 쪽은 아래가 맞다.

Fluent Bit이 컨테이너 로그를 CloudWatch Logs(`/lily/apps`)에 넣는다. 시작하는 쪽은 lily-observer의 `RemediateWatch`다. `WATCH_INTERVAL`(기본 30초)마다 `default` 네임스페이스 앱을 상태 조회와 같은 방식으로 판정한다. 지표는 Prometheus ingress이고, 5xx 비율이 critical(기본 5%)인 틱이 `JUDGE_CONSECUTIVE`(기본 2)번 연속이면 `RemediateSender`가 CloudWatch에서 최근 15분, 최대 200줄을 읽는다. 한 번 보낸 앱은 등급이 내려갔다가 다시 그 연속 기준에 닿아야 다시 보낸다. 이 감시는 롤백을 부르지 않는다. `POST /api/apps/{app}/remediate` 도 같은 sender를 쓴다.

기본값은 꺼져 있다. 아래가 켜져 있어야 한 건이 PR까지 간다.

| 스위치 | 기본 | 꺼지면 |
|---|---|---|
| `CLOUDWATCH_LOGS_ENABLED` | false | observer가 로그를 읽지 못한다 |
| `observer.remediate.enabled` | false | 연속 CRITICAL은 세되, 로그를 읽거나 프론트에 보내지 않는다 |
| 프론트 `REMEDIATE_ENABLED` | 그 값이 `true`가 아니면 | 사건 본문을 읽지 않는다 |
| 프로젝트 `remediate` | 코드상 동의 플래그 | PR을 열지 않는다 |
| `lily.remediate.enabled` | false | builder가 diff를 만들지 않는다 |
| `JEV_API_KEY` | 비어 있음 | jev를 묻지 않고 코드 결함으로 본다 |
| `GROQ_API_KEY` | 비어 있음 | Groq 대신 Claude, 그것도 없으면 OpenAI |

## 1. observer가 로그에서 사건을 만든다

`RemediateWatch`가 연속 CRITICAL에서 `RemediateSender.send`를 부른다. 수동 `POST /api/apps/{app}/remediate`도 그 메서드다. 예외가 있고, 그 스택의 첫 프레임이 레포 안 소스일 때만 사건을 만든다. `java.`, Spring, 사이트 패키지처럼 라이브러리 프레임만 있으면 프론트에 보내지 않는다.

사건에는 앱 이름, 서명(`예외 파일:줄`), 가린 로그, 파일 경로가 있다. Java 프레임은 `src/main/java/...` 로 바꾼다.

준비가 되면 observer가 프론트 `POST /api/internal/remediate` 로 그 사건을 보낸다. `Authorization: Bearer` 는 프론트의 `DEPLOYMENT_API_KEY`와 같은 값이어야 한다.

## 2. 프론트가 앱 이름으로 프로젝트를 찾는다

`builder_runs.app_name`이 그 앱 이름이고, 배포 상태가 `succeeded`인 가장 최근 행을 찾는다. 없으면 `배포된 프로젝트가 없다`. 배포 실행기는 `GET /api/builds/{id}`의 `commit`이 40자면 `builder_runs.commit_sha`에 남긴다. 그 값이 없으면 PR을 열지 않는다.

그 다음에 프론트가 직접 거절한다. 이 검사는 jev보다 앞이다.

- 프로젝트 동의가 꺼져 있다
- GitHub App 설치가 없다
- 배포 커밋(40자)이 없다
- 같은 서명의 PR이 이미 `opened`다
- 레포 안 파일이 없다

## 3. builder의 jev가 코드 결함인지 본다

프론트는 builder `POST /api/remediate/drafts` 를 부른다. 배포 커밋의 해당 파일, 가린 로그, 설치 토큰을 넘긴다.

jev는 프론트가 아니라 builder가 묻는다. 질문은 "이 로그가 레포 소스의 결함인가"이다. 설정, 외부 장애, 순간 오류면 아니오다.

| 예일 확률 | 결과 |
|---|---|
| 0.8 이상 | 코드 결함. diff를 만든다 |
| 0.2 이하 | 아니다. diff를 만들지 않는다 |
| 그 사이, 또는 호출 실패 | 빈 값. diff를 만들지 않는다 |

`JEV_API_KEY`가 없으면 이 질문을 하지 않고 코드 결함으로 둔다. 0.8 문턱은 키가 있을 때만 적용된다.

## 4. Groq가 그 파일의 diff를 만든다

jev가 예라고 한 뒤에만 모델을 부른다. `GROQ_API_KEY`가 있으면 Groq `https://api.groq.com/openai/v1/chat/completions`, 모델 `openai/gpt-oss-20b`다. 그 키가 없으면 Claude, 그것도 없으면 OpenAI다.

모델은 사건 파일만 고친 diff를 낸다. `.env`와 `.github/workflows`는 넣지 않는다. 고칠 수 없으면 diff는 빈 문자열이고, 그러면 PR은 열리지 않는다. builder는 GitHub를 호출하지 않는다.

## 5. 프론트가 GitHub PR을 연다

diff가 사건 파일 안에만 있으면, 프론트가 배포된 커밋 위에 패치 커밋을 올리고 PR을 연다. 브랜치는 `lily/fix-{서명}`이고 기본 브랜치에는 머지하지 않는다. PR 본문에 로그를 붙이고 "자동으로 머지하지 않는다"고 적는다.

`.env`, `.pem`, `.key`, `.github/workflows`와 사건 밖의 경로는 거절한다.

```
RemediateWatch  30초 · Prometheus 판정     lily-observer
  CRITICAL 2회 연속, remediate.enabled
  (수동 POST /api/apps/{app}/remediate 도 같은 길)
        │
        ▼
CloudWatch Logs 최근 15분
  레포 안 프레임이 있을 때만
        │
        ▼
POST /api/internal/remediate            lily-frontend
  app_name 으로 성공한 배포의 프로젝트
  commit_sha · 동의 · GitHub App · 같은 서명
        │
        ▼
POST /api/remediate/drafts              lily-builder
  jev 0.8 이상 (키 없으면 질문 생략)
  GROQ_API_KEY 있으면 Groq diff
        │
        ▼
GitHub PR                               lily-frontend
  배포 커밋 위 lily/fix-* , 머지하지 않음
```
