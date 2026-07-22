# PinLog ERD

```mermaid
erDiagram
    USERS ||--o{ SOCIAL_ACCOUNTS : authenticates_with
    USERS ||--o{ RECORDS : owns
    PLACES ||--o{ RECORDS : referenced_by
    RECORDS ||--|{ CONTEXTS : contains
    RECORDS ||--o{ RECORD_KEYWORDS : tagged_with
    KEYWORDS ||--o{ RECORD_KEYWORDS : maps
    USERS ||--|| SHELVES : owns
    USERS ||--o{ COLLECTIONS : creates
    SHELVES ||--o{ COLLECTIONS : displays
    COLLECTIONS ||--|{ COLLECTION_RECORDS : contains
    RECORDS ||--o{ COLLECTION_RECORDS : included_in
    USERS ||--o{ FOLLOWS : follows
    SHELVES ||--o{ FOLLOWS : followed_by
    RECORDS ||--o{ EMBEDDINGS : indexed_by
    CONTEXTS ||--o{ EMBEDDINGS : represented_by

    USERS {
        bigint id PK
        string status
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    SOCIAL_ACCOUNTS {
        bigint id PK
        bigint user_id FK
        string provider
        string provider_user_id
        timestamp created_at
        timestamp deleted_at
    }

    PLACES {
        bigint id PK
        string kakao_place_id UK
        string name
        string address
        decimal latitude
        decimal longitude
        string category
        timestamp created_at
        timestamp updated_at
    }

    RECORDS {
        bigint id PK
        bigint user_id FK
        bigint place_id FK
        string status
        bigint previous_record_id FK
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    CONTEXTS {
        bigint id PK
        bigint record_id FK
        text body
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    KEYWORDS {
        bigint id PK
        string code UK
        string label
    }

    RECORD_KEYWORDS {
        bigint record_id FK
        bigint keyword_id FK
        timestamp created_at
        timestamp deleted_at
    }

    SHELVES {
        bigint id PK
        bigint user_id FK
        timestamp created_at
        timestamp deleted_at
    }

    COLLECTIONS {
        bigint id PK
        bigint user_id FK
        bigint shelf_id FK
        string title
        timestamp published_at
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    COLLECTION_RECORDS {
        bigint id PK
        bigint collection_id FK
        bigint record_id FK
        timestamp created_at
        timestamp deleted_at
    }

    FOLLOWS {
        bigint id PK
        bigint follower_user_id FK
        bigint shelf_id FK
        string alias
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    EMBEDDINGS {
        bigint id PK
        bigint record_id FK
        bigint context_id FK
        string source_type
        vector vector
        string model
        timestamp created_at
        timestamp deleted_at
    }
```

## 핵심 제약

- `social_accounts`: `UNIQUE(provider, provider_user_id)`
- `places`: `UNIQUE(kakao_place_id)`
- `records`: 활성 상태 기준 `UNIQUE(user_id, place_id)`
- `shelves`: `UNIQUE(user_id)`
- `collection_records`: 활성 상태 기준 `UNIQUE(collection_id, record_id)`
- `follows`: 활성 상태 기준 `UNIQUE(follower_user_id, shelf_id)`
- Record는 Context를 1개 이상, Collection은 Record를 1개 이상 가져야 합니다.
