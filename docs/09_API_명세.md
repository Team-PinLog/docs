# PinLog API 명세

이 문서는 MVP 기능을 Endpoint 수준으로만 정의합니다. 요청·응답 DTO와 상태 코드는 포함하지 않습니다.

기본 경로:

```text
/api/v1
```

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
| PUT | `/records/{recordId}/contexts/{contextId}` | Context 수정 |
| DELETE | `/records/{recordId}/contexts/{contextId}` | Context 삭제 |
| GET | `/records/{recordId}/collections` | 연결된 Collection 조회 |

## 4. AI 자연어 검색

| Method | Endpoint | 설명 |
|---|---|---|
| POST | `/search/records` | Place·Context·Keyword 기반 내 Record 자연어 검색 |

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

## 8. 공통 접근 규칙

- 개인 Record, Context, Library, 계정 Endpoint는 본인 데이터만 다룹니다.
- 공개 Endpoint는 Context 원문과 신원 정보를 반환하지 않습니다.
- 모든 조회는 소프트 삭제 데이터를 제외합니다.
- Collection 생성은 자동 발행을 포함합니다.
- Record 수정 Endpoint는 기존 행 직접 수정이 아니라 재생성 정책을 따릅니다.
