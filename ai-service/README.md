# AI Service (FastAPI)

PrompTune 파이프라인의 **AI 단계 5,7,8,13,14,15** 담당.
흐름도의 "프롬프트 분석/수정 추천" + "결과 생성" 영역.

> **2026-09-18 코드 재확인:** 아래 엔드포인트는 대부분 실제 모델로 이미 연동되어
> 있다. 최신 구현 상태는 [`../docs/STATUS.md`](../docs/STATUS.md) 참고.

## 실행

```bash
# 로컬
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Docker
docker build -t promptune-ai . && docker run -p 8000:8000 promptune-ai
```

- API 문서(자동): http://localhost:8000/docs
- 상태 확인: http://localhost:8000/health

## 엔드포인트

| 단계 | 메서드 | 경로 | 실제 구현 |
|------|--------|------|-----------|
| 5 | POST | `/api/ai/diagnose` | KcELECTRA(`diagnose_real.py`) + 규칙(`diagnose_rules.py`) |
| 7 | POST | `/api/ai/suggest` | HyperCLOVA X 로컬 추론(`suggest_hcx.py` → `hcx_runtime.py`) |
| 8 | POST | `/api/ai/safety-check` | 규칙 |
| 13 | POST | `/api/ai/retrieve` | BGE-M3(`FlagEmbedding`) + pgvector(`rag_retriever.py`) |
| 14 | POST | `/api/ai/generate` | HyperCLOVA X 로컬 추론(`generate_hcx.py`) + Tavily(최신정보 시) |
| 15 | POST | `/api/ai/validate` | 규칙 검증(실제 pass/fail 결정) + BGE-M3 의미 유사도 검증(구현됨, 기본 비활성 — 텔레메트리 전용) |

## 구조

```
app/
├── main.py                      # FastAPI 앱, /health
├── routers/pipeline.py          # 엔드포인트 정의
├── schemas/models.py            # 입출력 형식(계약서)
└── services/
    ├── diagnose_real.py         # 5번 — KcELECTRA
    ├── diagnose_rules.py        # 5번 — 규칙(맞춤법·업무유형)
    ├── suggest_hcx.py           # 7번 — HyperCLOVA X
    ├── generate_hcx.py          # 14번 — HyperCLOVA X
    ├── hcx_runtime.py           # HyperCLOVA X 모델 로딩(공용)
    ├── safety_rule.py           # 8번
    ├── validation/
    │   ├── rule_validator.py    # 15번 — 규칙 검증
    │   └── semantic_validator.py # 15번 — BGE-M3 의미 유사도(현재 비활성)
    └── retrieval/
        ├── bge_m3.py            # 임베딩 생성 스크립트
        └── rag_retriever.py     # 13번 — 실시간 검색(BGE-M3+pgvector)
```

## 흐름도 분기 반영

- **통합 진단(5)** → 8요소/오탈자/업무유형 3갈래를 한 번에 판정
- **업무유형 → 내부문서 필요 여부**: `_internal` 또는 `application`이면
  `needs_internal_docs=True` → 13번(RAG) 실행 (흐름도 분기 그대로)
- **답변 생성(14)** → `use_web_search`면 Tavily 결과 반영 (흐름도 "최신정보" 갈래)
