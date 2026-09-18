# 진행 현황

🟢 **AI 파이프라인 핵심 단계 실제 구현 완료** — 4개 서비스 통합·전체 흐름 동작.
진단·문구생성·답변생성·검증·내부검색이 실제 모델로 연동되어 있다.

---

## 인프라 — 완료 ✅
- ✅ 폴더 구조 · docs
- ✅ CI/CD (GitHub Actions — 3서비스 빌드·테스트, 통과)
- ✅ docker-compose 통합 (헬스체크·기동 순서 보장)
- ✅ PostgreSQL + pgvector 시드 (Flyway)
- ⬜ AWS RDS 배포 (예정 — 팀 공용 DB)

## Frontend — 1,2,9,10 — 완료 ✅
- ✅ 프롬프트 입력 화면
- ✅ 입력중단 감지 (debounce + AbortController)
- ✅ 인라인 진단 · Ghost text
- ✅ 키보드 선택 (Tab/Esc/↑↓/Enter)

## Backend — 0,3,4,6,11,12,16
- ✅ 게이트 검사 (3, 금칙어·PII, 실제 규칙)
- ✅ 요청 분류 (12, 실제 구현)
- ✅ AI 오케스트레이션 (HTTP 연동)
- ✅ Flyway 스키마
- ✅ 소셜 로그인 구조 (Google·Kakao·Naver·Microsoft OAuth2, 환경변수 기반) — 단, Google·Kakao·Naver 중 **최소 하나는 반드시 설정**해야 하며 셋 다 비어있으면 백엔드가 기동 시점에 `IllegalStateException`으로 실패함 (`docs/SOCIAL_LOGIN_SETUP.md` 참고)
- 🟢 행동 저장 (16) · 개인화 점수(6) — 실제 구현 진행 중
- 🟡 업무맥락 조회 (4, MS Graph) — 코드상 `GraphMockService`가 여전히 기본 사용 중, 실제 연동 대기
- 🟡 소셜 로그인 실연결·테스트 — 구조는 완료, 배포 도메인 확정 후 redirect URI 실테스트 필요

## AI Service — 5,7,8,13,14,15
- ✅ 5 통합진단 — KcELECTRA 실제 구현 (`AutoModelForSequenceClassification`)
- ✅ 7 추천문구 생성 — HyperCLOVA X 로컬 추론 (`hcx_runtime.py` 기반)
- ✅ 8 안전검사 — 규칙 기반, 실제 동작
- ✅ 13 내부검색 — BGE-M3(`FlagEmbedding`) + pgvector 실제 연동 (`rag_retriever.py`)
- ✅ 14 최종 답변 생성 — HyperCLOVA X 로컬 추론 + Tavily 실제 API 연동
- ✅ 15 최종검증 — 규칙 검증(`rule_validator.py`) 실제 동작, 최종 pass/fail을 결정. BGE-M3 임베딩 기반 의미 유사도 검증(`semantic_validator.py`)도 구현되어 있으나 `ENABLE_SEMANTIC_VALIDATION_TELEMETRY` 플래그가 기본 `false`라 지금은 텔레메트리 수집에만 쓰이고 pass/fail에는 반영되지 않음

## ML — 완료 ✅
- ✅ KoELECTRA vs KcELECTRA 5-fold 교차검증 → KoELECTRA 선정
- ✅ 라벨링 킷 (kappa 측정)
- ✅ 라벨링 기준 개정

---

## 범례
✅ 완료 / 🟢 실제 구현 진행 중 / 🟡 mock (교체 대기) / ⬜ 미착수

## 남은 큰 작업
1. AWS RDS 배포 — 팀 공용 DB
2. 업무맥락 조회(MS Graph) 실제 연동 — 현재 `GraphMockService` 사용 중
3. 소셜 로그인 실연결 테스트 — 배포 도메인 확정 후 redirect URI 등록
4. 배포 시 환경변수·CORS·redirect URI 정리
