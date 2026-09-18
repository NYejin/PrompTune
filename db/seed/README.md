# DB 시드

시드 데이터는 **Flyway 마이그레이션**으로 관리됩니다:
`backend/src/main/resources/db/migration/V2__seed.sql`

이유: Flyway가 스키마(V1)를 만든 뒤 순서대로 시드(V2)를 넣어야
"테이블 없음" 에러가 안 납니다. DB 컨테이너의 initdb로 넣으면 순서가 꼬입니다.

시드 내용: 샘플 사용자(id=1), 선호설정, 내부문서 3건(pgvector).

> **참고**: 이후 추가된 `V27__remove_mock_data.sql`, `V28__cleanup_legacy_sample_documents.sql`이
> 이 V2 시드의 목업 사용자·샘플 문서를 정리한다. 즉 전체 마이그레이션을 끝까지
> 적용한 환경에는 V2가 원래 넣었던 목업 데이터가 더 이상 남아있지 않다.
