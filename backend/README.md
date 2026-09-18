# Backend (Spring Boot)

PrompTune 파이프라인의 **백엔드 단계 0,3,4,6,11,12,16** 담당 + AI 오케스트레이션.
흐름도의 "사용자 입력/조작" 백엔드 처리 + AI 서비스 호출.

## 실행

```bash
# DB 먼저 (docker compose up db)
./gradlew bootRun

# Docker
docker build -t promptune-backend . && docker run -p 8080:8080 promptune-backend
```

## 엔드포인트

| 단계 | 메서드 | 경로 | 하는 일 |
|------|--------|------|---------|
| 2 | POST | `/api/analyze` | 게이트(3)→진단(5,AI)→추천선정(6) |
| 11 | POST | `/api/execute` | 분류(12)→생성(14,AI)→저장(16) |
| 0 | GET | `/api/context/{userId}` | 로그인 후 사용자 맥락(4) |

## 구조

```
service/
├── GateService.java        # 3번 게이트 (실제 규칙)
├── RecommendService.java   # 6번 추천 선정 — PersonalizationScoreRepository 실점수 기반
├── AiServiceClient.java    # ai-service HTTP 호출
└── GraphMockService.java   # 0,4번 인증·Graph — 현재 mock 유지
controller/
└── PipelineController.java # 오케스트레이터
resources/
├── application.yml
└── db/migration/           # Flyway — V1~V36 (스키마+시드+누적 변경)
```

## 남은 mock

| 파일 | 상태 |
|------|------|
| GraphMockService | 0,4번(인증·업무맥락) — 실제 MS Graph 연동(`MicrosoftGraphService`)은 별도로 존재하지만, 파이프라인 오케스트레이터는 아직 Mock 쪽을 기본으로 사용 |

## DB (PostgreSQL + pgvector)

Flyway가 스키마 관리 (`db/migration/V1~V36`). 초기 스키마(V1)·시드(V2) 이후로도
개인화 점수·행동 로그·MS 연동·문서 컨텍스트 등 기능이 늘면서 마이그레이션이
계속 누적되어 왔다. V27·V28은 초기 목업 시드/샘플 데이터를 정리하는
마이그레이션이다.

## 로컬에서 AWS(S3) 기능 테스트하기

파일 업로드(`/api/documents`)처럼 S3를 쓰는 기능을 로컬 Docker 환경에서 테스트하려면, AWS 자격증명을 로컬에 등록해야 합니다.
배포 환경(EC2)은 IAM 역할로 자동 인증되지만, 로컬 환경은 별도로 액세스 키가 필요합니다.

### 1. AWS 액세스 키 발급
1. AWS 콘솔 → IAM → Users → 본인 계정 클릭
2. **Security credentials** 탭 → **Create access key**
3. 사용 목적 "Command Line Interface (CLI)" 또는 "Local code" 선택
4. Access Key ID / Secret Access Key 저장 (Secret Key는 이 화면에서 한 번만 보여줌)

⚠️ S3(promptune-document 버킷) 권한이 없으면 인프라 담당자에게 먼저 확인하세요.

### 2. 프로젝트 루트에 `.env` 파일 생성
```
AWS_ACCESS_KEY_ID=발급받은_액세스키
AWS_SECRET_ACCESS_KEY=발급받은_시크릿키
```
⚠️ `.env`는 `.gitignore`에 반드시 포함되어 있어야 합니다 (`cat .gitignore | grep .env`로 확인).

### 3. 재빌드
```bash
docker compose up --build
```
`docker-compose.yml`의 `backend` 서비스 `environment`에 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`가 이미 연결되어 있어서, `.env`만 채우면 자동으로 반영됩니다.

### 자주 겪는 에러
`SdkClientException: Unable to load credentials from any of the providers in the chain` 이 뜨면, 위 1~2단계가 아직 안 된 상태입니다.
