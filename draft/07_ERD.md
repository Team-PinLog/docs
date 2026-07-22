# PinLog ERD

DBMS는 PostgreSQL이며, 스키마는 `core`(백엔드 소유)와 `ai`(AI 파트 소유)로 분리합니다. 동일 인스턴스이므로 크로스 스키마 조인이 가능합니다.

상세 정의와 근거는 데이터 모델 및 무결성 문서를 따릅니다.

## 1. core 스키마

```mermaid
erDiagram
    MEMBER ||--o{ SOCIAL_ACCOUNT : authenticates_with
    MEMBER ||--o{ RECORD : owns
    MEMBER ||--o{ COLLECTION : creates
    MEMBER ||--o{ FOLLOW : follows
    MEMBER ||--o{ FOLLOW : followed_by
    PLACE  ||--o{ RECORD : referenced_by
    RECORD ||--|{ CONTEXT : contains
    RECORD ||--o{ COLLECTION_RECORD : included_in
    COLLECTION ||--|{ COLLECTION_RECORD : contains

    MEMBER {
        bigint id PK "외부 노출 금지"
        timestamptz created_at
        timestamptz deleted_at "소프트 삭제"
    }

    SOCIAL_ACCOUNT {
        bigint id PK
        bigint member_id FK
        varchar provider "GOOGLE/KAKAO/NAVER"
        varchar provider_user_id "탈퇴 시 마스킹"
        timestamptz created_at
        timestamptz deleted_at
    }

    PLACE {
        bigint id PK
        varchar kakao_place_id UK "식별 기준"
        varchar name
        varchar address "도로명 우선, 없으면 지번"
        varchar phone "nullable"
        varchar place_url "nullable"
        decimal lat "카카오 y"
        decimal lng "카카오 x"
        timestamptz created_at
        timestamptz updated_at
    }

    RECORD {
        bigint id PK
        bigint member_id FK
        bigint place_id FK
        timestamptz created_at
        timestamptz updated_at
        timestamptz deleted_at
    }

    CONTEXT {
        bigint id PK
        bigint record_id FK
        bigint member_id FK "비정규화: 검색 범위 필터"
        text body "개인정보, 타인 미공개"
        timestamptz created_at
        timestamptz deleted_at
    }

    COLLECTION {
        bigint id PK
        bigint member_id FK
        varchar title "20자"
        boolean is_published
        timestamptz published_at
        int record_count "활성 연결 수 비정규화"
        timestamptz created_at
        timestamptz updated_at
        timestamptz deleted_at
    }

    COLLECTION_RECORD {
        bigint id PK
        bigint collection_id FK
        bigint record_id FK
        timestamptz created_at
        timestamptz deleted_at
    }

    FOLLOW {
        bigint id PK
        bigint followee_member_id FK "선반 주인"
        bigint follower_member_id FK
        varchar display_name "팔로워가 지정, nullable"
        timestamptz created_at
        timestamptz deleted_at
    }
```

## 2. ai 스키마

AI 파트 소유입니다. 백엔드는 `context_keyword`를 읽기 조인하고, 삭제 시에만 `context_embedding`과 `context_keyword`를 DELETE합니다.

```mermaid
erDiagram
    KEYWORD_PRESET ||--o{ CONTEXT_KEYWORD : mapped_by
    CONTEXT_REF ||--o{ CONTEXT_KEYWORD : tagged_with
    CONTEXT_REF ||--o| CONTEXT_EMBEDDING : indexed_by

    CONTEXT_REF {
        bigint id PK "core.context 참조"
    }

    KEYWORD_PRESET {
        bigint id PK
        varchar code UK "enum 상수와 1:1, 불변"
        varchar label "표시 문구, 변경 가능"
        text description
        text examples
        vector embedding
        varchar visibility
        int version
        varchar embedding_profile "모델+차원"
        boolean active "폐기는 비활성화"
    }

    CONTEXT_EMBEDDING {
        bigint id PK
        bigint context_id FK "UNIQUE"
        vector embedding "HNSW 인덱스"
        varchar model
        varchar status "PENDING/DONE/FAILED"
        timestamptz created_at
    }

    CONTEXT_KEYWORD {
        bigint context_id FK
        bigint keyword_id FK
        decimal confidence
        int preset_version
    }
```

`CONTEXT_REF`는 다이어그램 표현을 위한 `core.context` 참조이며 실제 테이블이 아닙니다.

파이프라인 처리 단위는 Context입니다. Context가 불변이고 작업이 Context 단위로 격리되므로, 완료 순서가 뒤바뀌어도 서로의 결과를 덮어쓰지 않습니다. 따라서 버전 컬럼을 두지 않습니다. 유일한 경합은 처리 중 Context가 삭제되는 경우이며, 워커가 쓰기 직전 생존을 확인하는 것으로 방어합니다.

## 3. 물리 테이블이 아닌 개념

| 도메인 용어 | 물리 표현 |
|---|---|
| Shelf(선반) | `collection.member_id`로 그룹핑한 조회 결과 |
| Library(라이브러리) | 내 Collection + 팔로우한 Shelf의 Collection을 모은 조회 결과 |
| Record Keyword | `ai.context_keyword`를 Record 단위로 집계한 파생값 |
| Collection Keyword | Record Keyword를 Collection 단위로 집계한 파생값 |

Shelf는 User와 1:1이면서 고유 속성이 없으므로 테이블로 두지 않습니다. 선반 이름은 선반이 아니라 팔로우 관계(`follow.display_name`)에 속합니다.

Keyword는 Context 단위로만 저장합니다. Record 단위로 저장하면 Context 수정·삭제 시 어느 Keyword를 제거할지 판별할 수 없습니다.

## 4. 핵심 제약

### 4.1 유니크

삭제 정책이 "소프트 삭제 + 복구 없이 새 INSERT"이므로, `place`를 제외한 모든 유니크는 **활성행 기준 부분 유니크 인덱스**입니다.

| 테이블 | 제약 | 범위 |
|---|---|---|
| `place` | `UNIQUE(kakao_place_id)` | 전체 |
| `social_account` | `UNIQUE(provider, provider_user_id)` | `WHERE deleted_at IS NULL` |
| `record` | `UNIQUE(member_id, place_id)` | `WHERE deleted_at IS NULL` |
| `collection_record` | `UNIQUE(collection_id, record_id)` | `WHERE deleted_at IS NULL` |
| `follow` | `UNIQUE(followee_member_id, follower_member_id)` | `WHERE deleted_at IS NULL` |

전체 유니크로 정의하면 삭제 후 재저장이 실패합니다.

### 4.2 CHECK

| 테이블 | 제약 |
|---|---|
| `follow` | `followee_member_id <> follower_member_id` (자기 팔로우 금지) |
| `context` | `length(btrim(body)) > 0` |
| `collection` | `length(btrim(title)) > 0`, `record_count >= 0` |
| `place` | `lat BETWEEN -90 AND 90`, `lng BETWEEN -180 AND 180` |

### 4.3 애플리케이션 보장

DB 제약으로 표현할 수 없어 서비스 트랜잭션과 행 잠금으로 보장합니다.

- 활성 Record는 활성 Context를 1개 이상 가집니다. 마지막 Context 삭제 요청은 Record 삭제로 승격합니다.
- 활성 Collection은 활성 Record를 1개 이상 가집니다. 마지막 연결 제거 요청은 Collection 삭제로 승격합니다.
- Record 삭제 시, 해당 Record가 마지막이던 Collection은 자동 삭제합니다.
- Context 수정은 추가를 먼저 하고 삭제를 나중에 수행하여 빈 Record 상태를 만들지 않습니다.
- `collection.record_count`는 연결 추가·제거와 동일 트랜잭션에서 갱신합니다.

활성 수를 세고 그 결과로 분기하는 작업은 부모 행(`record`, `collection`)을 잠근 뒤 수행합니다. 자식 행 잠금은 서로 다른 자식을 대상으로 할 때 겹치지 않아 경합을 막지 못합니다.

## 5. 삭제 방식 요약

| 대상 | 방식 |
|---|---|
| member, social_account, record, context, collection, collection_record, follow | 소프트 삭제 (`deleted_at`) |
| `ai.context_embedding`, `ai.context_keyword` | 즉시 파기 (DELETE) |
| place | 삭제하지 않음 |
| keyword_preset | 삭제하지 않음 (`active` 플래그로 비활성화) |

임베딩을 소프트 삭제하면 HNSW 인덱스에 죽은 벡터가 쌓여 검색 품질이 저하되므로 즉시 파기합니다.
