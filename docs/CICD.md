# CI/CD (GitHub Actions)

`.github/workflows/ci.yml` — main 푸시와 모든 PR에서 자동 실행되는 검증 파이프라인.
`.github/workflows/deploy.yml` — main에 병합되면 EC2로 자동 배포.

## 무엇을 검증하나 (ci.yml)

| Job | 대상 | 하는 일 |
|-----|------|---------|
| **ai-service** | FastAPI | 의존성 설치 → 앱 기동 검증 → pytest(mock/stub 기반) → 엔드포인트 스모크 테스트 |
| **backend** | Spring Boot | `gradle test` (컴파일 + 테스트 전체 실행) |
| **frontend** | Next.js | 의존성 설치 → 타입 체크 → 빌드 |
| **docker-build** | 전체 | 위 3개 통과 후 compose 문법 검증 + 이미지 빌드 |
| **push-to-ecr** | 전체 | `docker-build` 통과 후, main 브랜치 push이고 `ENABLE_ECR_PUSH` 변수가 켜져 있을 때만 OIDC로 AWS 자격증명을 받아 3개 이미지를 ECR에 푸시 |

## 흐름

```
PR 생성/푸시
   ├─ ai-service ┐
   ├─ backend    ┤ (병렬 실행)
   └─ frontend   ┘
        ↓ 모두 통과하면
     docker-build (통합 빌드 검증)
        ↓ main 머지 시
     deploy.yml → EC2 자동 배포
```

세 서비스를 병렬로 검증하고, 다 통과해야 Docker 통합 빌드를 돌립니다.
하나라도 깨지면 PR에 ❌가 표시되어 머지 전에 문제를 잡습니다.

## 스모크 테스트 (ai-service)

단순 빌드만이 아니라 **실제로 동작하는지** 확인합니다:
- `/health`가 200과 `{"status": "ok"}`를 반환하는가
- `/api/ai/validate`에 원문·생성문을 보내면 실제로 `passed: true`를 반환하는가

즉 헬스체크와 검증 엔드포인트의 핵심 응답 형식이 깨지지 않았는지 매 PR마다 자동 확인합니다.

## 배포 (deploy.yml)

main 브랜치에 push(PR 머지)되면 SSH로 EC2에 접속해 `git fetch` → `git reset --hard origin/main` → 시크릿 갱신 → `docker compose -f docker-compose.prod.yml up -d --build`를 실행합니다. 배포 실패 시 직전 커밋으로 자동 롤백하는 로직이 포함되어 있습니다. Actions 탭에서 수동 실행(`workflow_dispatch`)도 가능합니다.

## 팀 협업

- PR을 올리면 CI가 자동으로 돌아 초록/빨강으로 결과 표시
- README 상단 배지로 main의 현재 상태를 한눈에 확인

## 확장 아이디어 (향후)

- 실제 모델 추론 테스트 추가 (현재 스모크 테스트는 mock 경로 기준)
- 커버리지 리포트, lint 추가
