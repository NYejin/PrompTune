# PrompTune (프롬프튠)


한국어 프롬프트 개선 AI 코파일럿. 사용자가 입력한 거친 업무 지시문에서
**부족한 요소(8요소)를 감지해 되묻고**, 보완된 프롬프트로 최종 결과물을 생성한다.

---

## 아키텍처

```
[frontend]  Next.js + React + TypeScript      단계 1,2,9,10 (입력·표시·선택)
     │  HTTP
[backend]   Spring Boot + JPA + Flyway         단계 0,3,4,6,11,12,16 (게이트·맥락·점수·저장)
     │  HTTP
[ai-service] FastAPI + Python                  단계 5,7,8,13,14,15 (진단·생성·검증·검색)
     │
[db]        PostgreSQL + pgvector              온보딩·개인화·문서 임베딩
```

전체 16단계 상세는 [`docs/PIPELINE.md`](docs/PIPELINE.md) 참고.

---

## 빠른 시작

```bash
# 전체 스택 한 번에 실행 (Docker)
docker compose up --build

# 접속
# 프론트:      http://localhost:3000
# 백엔드 API:  http://localhost:8080
# AI 서비스:   http://localhost:8000/docs  (FastAPI 자동 문서)
```

개별 실행·개발 방법은 각 폴더의 README 참고:
- [`frontend/README.md`](frontend/README.md)
- [`backend/README.md`](backend/README.md)
- [`ai-service/README.md`](ai-service/README.md)

---

## 기술 스택

- **Frontend**: Next.js 14 (App Router), React, TypeScript
- **Backend**: Spring Boot 3, JPA, Flyway, Spring Security (OAuth2 — 로컬/Google/Kakao/Naver/Microsoft)
- **AI Service**: FastAPI, Python 3.11, HuggingFace transformers(HyperCLOVA X 로컬 추론), FlagEmbedding(BGE-M3), Tavily
- **DB**: PostgreSQL 16 + pgvector
- **Infra**: Docker Compose

## 프로젝트 상태

진단(5)·문구생성(7)·답변생성(14)·최종검증(15)·내부검색(13) 실제 모델 연동 완료.
최신 구현 상태는 [`docs/STATUS.md`](docs/STATUS.md) 참고.
