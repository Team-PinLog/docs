# PinLog AI 자연어 검색 및 RAG 설계 — 개정본

## 1. 문서 상태

GraphRAG 사용 여부는 미확정이다.

현재 검토 대상:

- Java AI 기반 일반 RAG
- 관계형 DB와 벡터 검색 결합
- GraphRAG
- PostgreSQL·pgvector 기반 구현

기술 방식이 확정되기 전까지 기능 명칭은 `AI 자연어 검색`으로 사용한다.

## 2. 검색 목적

사용자가 정확한 Place 이름을 기억하지 못해도 저장 당시의 Context와 Keyword로 Record를 찾게 한다.

## 3. 검색 데이터

Record별 검색 대상:

- Place 이름
- Place 주소
- Place 카테고리
- Context 원문
- Keyword
- 필요한 경우 Place의 지역 정보

다른 사용자의 데이터는 검색하지 않는다.

## 4. 검색 결과

```text
자연어 질의
→ Place·Context·Keyword 후보 검색
→ 각 후보의 Record 식별
→ Record 단위 점수 통합
→ Record 중복 제거
→ Record 목록 반환
```

검색 결과에는 다음을 표시할 수 있다.

- Place
- Context
- Keyword
- Record 생성일

검색 선택 이유 문구는 제공하지 않는다.

## 5. 임베딩

### Context 생성

새 Context 임베딩을 생성한다.

### Context 수정

수정된 Context 임베딩을 다시 생성한다.

### Record 재생성

Record 재생성 시 기존 Collection 연결을 새 Record로 승계한다.

AI 처리:

```text
새 Record 생성
→ 모든 활성 Context 조회
→ Context 전체 재임베딩
→ Keyword 검색 표현 재생성
→ 새 Record ID 기준 벡터 연결
```

재임베딩 이유:

- 임베딩과 Record 식별자의 연결을 새로 구성해야 함
- 삭제된 Record를 검색 결과에서 제외해야 함
- Collection 연결과 검색 결과가 동일한 활성 Record를 가리켜야 함

## 6. 권장 트랜잭션 상태

AI 처리 실패 시 불완전한 새 Record가 노출되지 않도록 상태값을 둘 수 있다.

```text
CREATING
EMBEDDING
ACTIVE
FAILED
DELETED
```

권장 흐름:

```text
새 Record CREATING
→ Collection 연결 재생성
→ 임베딩 생성
→ 성공 시 ACTIVE
→ 기존 Record DELETED
```

기존 Record를 먼저 삭제하는 구현을 선택한다면 전체 작업을 단일 트랜잭션 또는 보상 트랜잭션으로 보호해야 한다.

## 7. 미결정 사항

- GraphRAG 여부
- Java AI 라이브러리
- 임베딩 모델
- Place·Context·Keyword 점수 통합 방식
- 동기·비동기 임베딩
- 실패 재시도 방식
- 검색 랭킹 알고리즘
