# PinLog API 상세 명세 (초안)

이 문서는 **디자인 리뉴얼 화면**을 기준으로 화면 → 엔드포인트를 매핑하고 요청·응답 초안을 정리합니다. 엔드포인트 목록은 [API 명세](08_API_명세.md)를, 도메인 규칙은 [데이터 모델](06_데이터모델_및_무결성.md)·[정책](../static/02_정책_정의서.md)·[AI 설계](../static/05_AI_설계.md)를 원본으로 하며, 충돌 시 그 문서가 우선합니다.

- 기본 경로: `/api/core/v1` (구성은 08 문서 §기본 경로)
- 용어는 공식 용어(Record·Context·Collection·Shelf·Follow·Place)를 사용합니다. 디자인의 책/저자/구독 메타포는 각각 Collection·Shelf 소유자·Follow로 대응합니다.
- 요청·응답 필드는 **초안**입니다. 확정 DTO는 구현 시 코드가 원본입니다.

## 0. 공통 규약

- **인증**: 소셜 로그인 후 세션으로 본인 식별. 개인 엔드포인트는 인증 컨텍스트의 `memberId`를 사용하며 요청 본문으로 받지 않습니다. (세션 vs 토큰 방식은 §확인 필요)
- **식별자 노출**: `member.id`는 어떤 응답에도 포함하지 않습니다([06 §5.3](06_데이터모델_및_무결성.md)). 타인 접근 진입점은 **Collection id**입니다.
- **비동기 AI**: Record·Context 생성·수정 응답은 AI 완료를 기다리지 않습니다. `keywords`가 빈 배열인 것은 정상입니다.
- **소프트 삭제**: 모든 조회는 삭제 데이터를 제외합니다.
- **페이지네이션**: 목록·피드는 커서 기반을 기본으로 합니다(`cursor`, `limit`). (§확인 필요)
- **에러**: 권한·소유권 실패는 익명성 보호를 위해 존재를 드러내지 않는 방향(404)으로 응답합니다([06 §5.5](06_데이터모델_및_무결성.md)).

## 1. 로그인 · 온보딩

로그인 전 화면에서 소셜 로그인, 최초 가입 시 약관 동의.

| 화면 요소 | 액션 | Method · Endpoint | 요청 | 응답 |
|---|---|---|---|---|
| 소셜 로그인 버튼 | 로그인 시작 | `GET /auth/{provider}/login` | `provider`=google/kakao/naver | 리다이렉트 |
| (콜백) | 콜백 처리 | `GET /auth/{provider}/callback` | 공급자 code | 세션 수립. 신규면 약관 동의 필요 플래그 |
| 약관 동의 | 필수 동의 | `POST /users/me/agreements` | `agreed: true` | `204` |

신규 가입은 소셜 인증만으로 완료되지 않고 **약관 동의까지 마쳐야 계정이 생성**됩니다([09 §1](09_유저플로우.md)).

## 2. 홈 (home)

내 장소 지도, 저장한 장소(내 Record) 목록, 새 장소 저장, 기록 메모(Context) 수정·삭제.

| 화면 요소 | 액션 | Method · Endpoint | 비고 |
|---|---|---|---|
| 지도 마커 | 내 Record 지도 | `GET /places/map` | 내 활성 Record의 Place 좌표 목록 |
| 저장한 장소 목록 | 내 Record 목록 | `GET /records` | 커서 페이지네이션 |
| 장소 추가 → 검색 | Place 검색 | `GET /places/search?query=` | 카카오맵 검색 |
| 장소 저장(메모 입력) | Record 생성 | `POST /records` | 아래 예시 |
| 기록 메모 수정 | Context 수정 | `PUT /records/{recordId}/contexts/{contextId}` | **새 contextId 반환** |
| 기록 메모 삭제 | Context 삭제 | `DELETE /records/{recordId}/contexts/{contextId}` | 마지막 Context면 Record 삭제로 승격 |

**장소 저장 (Record 생성)**
```jsonc
// POST /records
{
  "kakaoPlaceId": "1234567",   // 검색 결과에서 받은 카카오 식별자
  "context": "비 오는 날 친구랑 오려고 저장"   // 첫 Context 본문(필수)
}
// 201
{
  "recordId": 8801,
  "place": { "placeId": 5501, "name": "앤트러사이트 성수", "address": "성동구 연무장길 47", "lat": 37.5447, "lng": 127.0557 },
  "context": { "contextId": 91001, "body": "비 오는 날 친구랑 오려고 저장", "createdAt": "2026-07-23T10:00:00Z" },
  "keywords": []   // 비동기 생성 전이라 비어 있을 수 있음(정상)
}
```

**기록 메모 수정 (Context 불변 — 삭제+생성)**
```jsonc
// PUT /records/8801/contexts/91001
{ "body": "비 오는 날 말고 주말 오후에" }
// 200 — 새 contextId 반환. 구 91001은 소프트 삭제·AI 취소됨
{ "contextId": 91002, "body": "비 오는 날 말고 주말 오후에", "createdAt": "2026-07-23T10:05:00Z", "keywords": [] }
```
Client는 새 `contextId`를 반영하고 구 id를 캐시 키로 쓰지 않습니다([08 §3](08_API_명세.md), [static/05 Context 불변성](../static/05_AI_설계.md)). 내부적으로 새 리소스를 만들지만, 엔드포인트가 수정(PUT)이고 URL이 구 리소스를 가리키므로 응답은 `200`으로 둡니다(`201 + Location`도 의미상 가능).

## 3. 검색 (search)

상단 검색은 장소·컬렉션 검색, "스마트 검색"은 AI 자연어 검색.

| 화면 요소 | 액션 | Method · Endpoint | 비고 |
|---|---|---|---|
| 상단 검색창 | 장소·컬렉션 검색 | `GET /search?query=` | 통합 검색(§확인 필요 — 08 미정의) |
| 스마트 검색 | AI 자연어 검색 | `POST /search/records` | 내 Context 대상, Record 단위 |

**AI 자연어 검색**
```jsonc
// POST /search/records
{ "query": "비 오는 날 친구랑 가려고 저장한 카페", "limit": 20 }
// 200 — 본인 Context만 대상, Record 단위 중복 제거
{
  "results": [
    { "recordId": 8801, "similarity": 0.82,
      "place": { "placeId": 5501, "name": "앤트러사이트 성수", "address": "…", "lat": 37.5447, "lng": 127.0557 },
      "createdAt": "2026-07-20T09:00:00Z" }
  ]
}
```
선택 이유 문구·Context 원문은 반환하지 않습니다([정책 §6](../static/02_정책_정의서.md), [MVP 제외](10_MVP_기능범위.md)). 결과 없음은 빈 `results`.

> 상단 "장소·컬렉션 검색"은 08 문서에 엔드포인트가 없습니다. AI 검색(`/search/records`)과 별개의 일반 검색이므로 `GET /search`(또는 `/places/search` + `/collections/search`)를 신설할지 §확인 필요.

## 4. 탐색 · 추천 (rec)

발행 Collection 추천 피드, Collection 상세, 소유자(Shelf) 패널·구독(Follow), 발견한 장소를 내 기록으로 담기.

| 화면 요소 | 액션 | Method · Endpoint | 비고 |
|---|---|---|---|
| 추천 목록 | Feed 조회 | `GET /feed/collections?cursor=` | `requestId`·`position` 포함해 반환. **이 응답 생성 시 서버가 IMPRESSION 기록** |
| Collection 열기 | 상세 조회 | `GET /feed/collections/{collectionId}` | 이벤트 기록 대상 아님. 클릭은 아래 `POST /feed/events`(CLICK) |
| 클릭·저장 | 이벤트 수집 | `POST /feed/events` | `CLICK`/`SAVE` + `requestId`·`position` |
| 소유자 책장 열기 | 공개 Shelf 조회 | `GET /collections/{collectionId}/shelf` | member.id 대신 collection 진입(§확인 필요) |
| 구독하기/구독중 | Shelf Follow 토글 | `POST` / `DELETE /collections/{collectionId}/follow` | 아래 참고 |
| 장소 담기(메모) | 발견 Place로 Record | `POST /feed/places/{placeId}/records` | 타인 기록 복제가 아니라 내 Context 신규 작성. **기존 활성 Record 유무를 서버가 판단** |

**Feed 응답 (이벤트 키 포함)**
```jsonc
// GET /feed/collections
{
  "requestId": "5b2c…uuid",
  "items": [
    { "position": 0, "collectionId": 7001, "title": "비 오는 날의 카페",
      "placeCount": 5, "createdAt": "2026-07-18T…", "keywords": ["조용한","커피챗"] }
  ],
  "nextCursor": "…"
}
```
`keywords`는 타인 Collection이므로 `PUBLIC` 등급만 반환하며([08 §8](08_API_명세.md)), **AI 미완료 Collection은 빈 배열입니다(정상)** — 미완료 Collection도 Feed 후보에 포함됩니다. Context 원문·소유자 신원은 반환하지 않습니다.

**IMPRESSION 기록 시점** — `IMPRESSION`은 "Spring이 Feed 응답에 Collection을 포함해 전달했다"는 의미이므로, **목록 응답(`GET /feed/collections`) 생성 시** 서버가 각 항목을 기록합니다. 상세 조회(`GET /feed/collections/{collectionId}`)는 이벤트 기록 대상이 아닙니다. 상세 조회마다 IMPRESSION을 기록하면 노출 패널티 계산이 왜곡됩니다. `CLICK`은 Client가 `POST /feed/events`로 전송합니다.

**클릭·저장 이벤트**
```jsonc
// POST /feed/events
{ "event": "CLICK", "collectionId": 7001, "placeId": null, "requestId": "5b2c…", "position": 0 }
// 204 — memberId는 인증 컨텍스트에서. IMPRESSION은 Client가 보내지 않음
```

**발견 Place를 내 기록으로**
```jsonc
// POST /feed/places/5501/records
{ "context": "탐색하다 발견, 다음 주말에" }
// 서버가 기존 활성 Record 유무를 판단한다("User당 Place별 활성 Record 1개" 제약과 맞물림):
//   기존 활성 Record 있음 → 해당 Record에 Context 추가 후 200
//   없음                 → 새 Record 생성 후 201
// Client는 사전에 Record 존재 여부를 알 필요가 없다.
```

## 5. 내 책장 (shelf)

내 발행 Collection 그리드, 새 컬렉션 생성.

| 화면 요소 | 액션 | Method · Endpoint | 비고 |
|---|---|---|---|
| 내 책장 | 내 Shelf 조회 | `GET /shelves/me` | 내 발행 Collection 목록 |
| 내 Collection 목록 | 목록 조회 | `GET /collections` | |
| 새 컬렉션 만들기 | Collection 생성 | `POST /collections` | 이름 + Record 1개 이상, 자동 발행 |
| Collection 제목 수정 | 제목 수정 | `PATCH /collections/{collectionId}` | |
| Collection 삭제 | 소프트 삭제 | `DELETE /collections/{collectionId}` | |
| Record 추가·제거 | 구성 편집 | `POST` / `DELETE /collections/{collectionId}/records[/{recordId}]` | 마지막 Record 제거 시 Collection 삭제 확인 |

**새 컬렉션 생성**
```jsonc
// POST /collections
{ "title": "비 오는 날의 카페", "recordIds": [8801, 8802, 8803] }
// 201 — 생성 즉시 자동 발행
{ "collectionId": 7050, "title": "비 오는 날의 카페", "recordCount": 3, "isPublished": true, "publishedAt": "2026-07-23T…" }
```
제목 필수·최대 20자, Record 최소 1개. 정렬은 담은 순서 기준([06 §2.7](06_데이터모델_및_무결성.md)).

> 디자인은 컬렉션 **커버 색상**을 고르고, 내 책장을 **이름 붙인 컬럼**으로 그룹핑합니다. 현재 `collection` 스키마에 색상 컬럼이 없고, 내 책장 그룹핑 개념도 도메인에 없습니다(Follow alias는 타인 Shelf 전용). §확인 필요.

## 6. 타인 책장 (author)

공개 Shelf와 그 발행 Collection 조회, Follow.

| 화면 요소 | 액션 | Method · Endpoint | 비고 |
|---|---|---|---|
| 저자 책장 | 공개 Shelf 조회 | `GET /collections/{collectionId}/shelf` | 같은 소유자의 다른 발행 Collection |
| 팔로우/해제 | Follow 토글 | `POST` / `DELETE /collections/{collectionId}/follow` | |
| 별칭 지정·수정 | Follow 별칭 | `PATCH …/follow` | `alias`, 최대 20자, 공백은 null |
| 라이브러리 | 내 + 팔로우 Shelf | `GET /library` | |
| 팔로워·팔로잉 수 | 본인만 | `GET /library/follow-counts` | 목록 미제공 |

공개 Collection 조회는 Place·`PUBLIC` Keyword·Record 생성일만 반환하고 Context 원문·신원은 제외합니다.

## 7. 설정

| 화면 요소 | 액션 | Method · Endpoint | 비고 |
|---|---|---|---|
| 내 계정 | 계정 조회 | `GET /users/me` | |
| 연결 소셜 계정 | 조회 | `GET /users/me/social-accounts` | |
| 로그아웃 | 세션 종료 | `POST /auth/logout` | |
| 탈퇴하기 | 회원 탈퇴 | `DELETE /users/me` | 소유 데이터 소프트 삭제. AI 파생 데이터는 State `CANCELLED` + `is_deleted=true` (물리 삭제 시점은 미결 — [static/05 잔여 미결](../static/05_AI_설계.md)) |

## 8. 설계 정합성 확인 필요

디자인과 현재 계약이 어긋나 결정이 필요한 항목입니다. 임의 확정하지 않고 표시만 합니다.

| # | 항목 | 내용 |
|---|---|---|
| 1 | **Shelf 식별자** | `08`은 `/shelves/{shelfId}`를 쓰지만 `06 §1.4`는 Shelf를 테이블로 두지 않고 `member.id` 노출을 금지합니다. 진입점을 **Collection id**(`/collections/{id}/shelf`·`/follow`)로 통일할지, Shelf용 공개 식별자(`public_id`)를 도입할지. 이 문서는 전자로 초안화. |
| 2 | **Collection 커버 색상** | 디자인은 생성 시 색상 팔레트 선택. 현재 `collection` 스키마에 색상 없음 → 컬럼 추가 vs 프론트 로컬 표현. |
| 3 | **내 책장 컬럼 그룹핑** | 디자인은 내 Collection을 "이름 붙인 컬럼"으로 묶음. 도메인에 내 Shelf 그룹핑 개념 없음(Follow alias는 타인용) → 신규 개념 도입 여부. |
| 4 | **통합 검색** | 상단 "장소·컬렉션 검색"이 `08`에 없음. AI 검색과 별도 일반 검색 엔드포인트 신설 필요. |
| 5 | **인증 방식** | 세션(쿠키) vs 토큰(JWT). 개인 엔드포인트의 `memberId` 취득 경로 확정 필요. |
| 6 | **페이지네이션** | 커서 vs 오프셋. Feed는 Feed Session(`requestId`) 안정성을 위해 커서 권장. |
| 7 | **탈퇴 시 AI 파생 데이터 물리 삭제** | MVP는 소프트 삭제(`CANCELLED`+`is_deleted`)로 착수, 최종 방식 미결. [static/05 잔여 미결](../static/05_AI_설계.md). |
| 8 | **Place별 활성 Record 중복 시 응답** | `POST /feed/places/{placeId}/records`에서 서버가 신규 생성(201)/Context 추가(200)를 판단할지. 이 초안은 서버 판단으로 정렬. |

> 이 문서는 디자인 화면 기준의 **초안**입니다. 확정 계약은 `08`(엔드포인트)·`06`(스키마)·`static/05`(AI)이며, 위 확인 필요 항목이 정해지면 그 문서에 반영한 뒤 이 문서를 갱신합니다.
