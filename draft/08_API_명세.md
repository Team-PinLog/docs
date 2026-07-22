# PinLog API 명세

이 문서는 MVP 기능을 Endpoint 수준으로만 정의합니다. 요청·응답 DTO와 상태 코드는 포함하지 않습니다.

기본 경로:

```text
/api/core/v1
```

이 경로는 두 조각으로 구성됩니다.

| 조각 | 값 | 소유 |
|---|---|---|
| 서비스 prefix | `/api/core` | `server.servlet.context-path` |
| API 버전 | `/v1` | 컨트롤러 매핑 |

버전을 `context-path`가 아니라 컨트롤러 매핑에 두는 이유는 운영 엔드포인트를 버전에서 분리하기 위해서입니다. `context-path`에 버전을 넣으면 actuator도 `/api/core/v1/actuator/*`로 내려가, API 버전을 올릴 때마다 프로브와 메트릭 수집 경로까지 바뀝니다.

따라서 운영 엔드포인트는 버전 없이 유지합니다.

```text
/api/core/actuator/health
/api/core/actuator/prometheus
```

서비스 prefix는 Ingress 경로와 일치해야 하며 StripPrefix를 사용하지 않습니다. 아래 표의 Endpoint는 모두 기본 경로에 이어붙인 상대 경로입니다.

이 문서가 다루는 범위는 Client가 호출하는 외부 API입니다. Spring과 FastAPI 사이의 내부 API(`/internal/v1/*`)는 여기에 정의하지 않으며, 논리 계약은 [AI 설계](../static/05_AI_설계.md)의 내부 API 계약을 따릅니다. Client는 FastAPI를 직접 호출하지 않습니다.

## 1. 인증과 계정

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/auth/{provider}/login` | Google·Kakao·Naver 소셜 로그인 시작 |
| GET | `/auth/{provider}/callback` | 소셜 로그인 콜백 처리 |
| POST | `/auth/logout` | 로그아웃 |
| POST | `/users/me/agreements` | 최초 가입 필수 약관 동의 |
| GET | `/users/me` | 내 계정 조회 |
| GET | `/users/me/social-accounts` | 연결된 소셜 계정 조회 |
| DELETE | `/users/me` | 회원 탈퇴 |

## 2. Place

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/places/search` | 카카오맵 Place 검색 |
| GET | `/places/{placeId}` | 내부 Place 기본 정보 조회 |
| GET | `/places/map` | 내 Record 기반 지도 마커 조회 |

## 3. Record와 Context

| Method | Endpoint | 설명 |
|---|---|---|
| POST | `/records` | Place와 첫 Context로 Record 생성 |
| GET | `/records` | 내 활성 Record 목록 조회 |
| GET | `/records/{recordId}` | 내 Record 상세 조회 |
| PUT | `/records/{recordId}` | Record 재생성 방식 수정 |
| DELETE | `/records/{recordId}` | Record 소프트 삭제 |
| POST | `/records/{recordId}/contexts` | Context 추가 |
| GET | `/records/{recordId}/contexts` | Context 목록 조회 |
| PUT | `/records/{recordId}/contexts/{contextId}` | Context 수정. 내부적으로 구 Context 삭제 + 신 Context 생성이며 **새 contextId를 반환** |
| DELETE | `/records/{recordId}/contexts/{contextId}` | Context 삭제 |

Context는 불변입니다. 수정 Endpoint는 형태를 유지하지만 서버 내부에서는 구 Context를 소프트 삭제하고 새 Context를 생성합니다.

```text
PUT /records/{recordId}/contexts/{contextId}
→ 구 Context 삭제 및 AI 취소
→ 신 Context 생성
→ 응답에 새 contextId 반환
```

**구 `contextId`를 그대로 반환하는 응답 계약은 허용하지 않습니다.** Client는 응답의 새 `contextId`를 반영해야 하며, 구 `contextId`로 재조회하거나 이를 캐시 키로 계속 사용해서는 안 됩니다. 자세한 내용은 [AI 설계](../static/05_AI_설계.md)의 Context 불변성을 따릅니다.
| GET | `/records/{recordId}/collections` | 연결된 Collection 조회 |

## 4. AI 자연어 검색

| Method | Endpoint | 설명 |
|---|---|---|
| POST | `/search/records` | 내 Context 기반 Record 자연어 검색 |

검색은 본인 Context만 대상으로 하며 결과는 Record 단위로 중복 제거합니다. 처리 방식은 [AI 설계](../static/05_AI_설계.md)의 개인 자연어 검색을 따릅니다.

## 5. Collection

| Method | Endpoint | 설명 |
|---|---|---|
| POST | `/collections` | Record 1개 이상으로 Collection 생성 및 자동 발행 |
| GET | `/collections` | 내 Collection 목록 조회 |
| GET | `/collections/{collectionId}` | Collection 상세 조회 |
| PATCH | `/collections/{collectionId}` | Collection 제목 수정 |
| DELETE | `/collections/{collectionId}` | Collection 소프트 삭제 |
| POST | `/collections/{collectionId}/records` | Record 추가 |
| DELETE | `/collections/{collectionId}/records/{recordId}` | Record 제거 |

## 6. Shelf, Library, Follow

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/shelves/me` | 내 Shelf 조회 |
| GET | `/shelves/{shelfId}` | 공개 Shelf 조회 |
| GET | `/shelves/{shelfId}/collections` | Shelf의 발행 Collection 조회 |
| GET | `/library` | 내 Shelf와 팔로우한 Shelf 조회 |
| GET | `/library/follow-counts` | 본인 팔로워·팔로잉 수 조회 |
| POST | `/shelves/{shelfId}/follow` | Shelf Follow |
| DELETE | `/shelves/{shelfId}/follow` | Shelf Follow 해제 |
| PATCH | `/shelves/{shelfId}/follow` | Follow 별칭 수정 또는 제거 |

## 7. Feed

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/feed/collections` | 발행 Collection 추천 목록 조회 |
| GET | `/feed/collections/{collectionId}` | Feed Collection 상세 조회 |
| POST | `/feed/places/{placeId}/records` | 공개 Place에 내 Context를 작성해 Record 생성 |
| POST | `/feed/places/{placeId}/contexts` | 기존 내 Record에 Context 추가 |
| POST | `/feed/events` | Feed 추천 결과의 CLICK·SAVE 이벤트 수집 |

`IMPRESSION`은 Feed 응답을 만들 때 서버가 직접 기록하므로 Client가 전송하지 않습니다. `CLICK`과 `SAVE`는 Client가 전송하며 `requestId`와 `position`을 함께 보냅니다. 수집 항목과 저장 구조는 back 파트의 Feed 문서에서 정의합니다.

## 8. 공통 접근 규칙

- 개인 Record, Context, Library, 계정 Endpoint는 본인 데이터만 다룹니다.
- 공개 Endpoint는 Context 원문과 신원 정보를 반환하지 않습니다.
- 모든 조회는 소프트 삭제 데이터를 제외합니다.
- Collection 생성은 자동 발행을 포함합니다.
- Record 수정 Endpoint는 기존 행 직접 수정이 아니라 재생성 정책을 따릅니다.
- AI 처리는 저장 이후 비동기로 진행되므로 Record·Context 생성·수정 응답은 AI 완료를 기다리지 않습니다. 응답의 Keyword가 비어 있는 것은 오류가 아닙니다.
- 타인에게 반환하는 Keyword는 `PUBLIC` 등급으로 한정합니다.
