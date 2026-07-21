# PinLog 정책 정의서

## 0. 문서 지위

- 본 문서는 팀 합의에 따른 최신 정책 기준서다.
- 기존 문서와 충돌할 경우 본 문서를 우선한다.
- 기존 공식 용어 `Memory`는 폐기한다.
- 개인 기록의 핵심 단위는 `Record`, 독립 서술 단위는 `Context`다.
- GraphRAG 도입 여부는 미확정이다. 현재 Java AI 기반 RAG를 포함해 검토 중이다.

---

# 1. 기본 원칙

1. 사용자가 장소를 저장한 이유와 개인적 맥락은 비공개 데이터다.
2. 다른 사용자에게는 개인 Record 원문이 아니라 발행된 Collection과 공개 가능한 정보만 제공한다.
3. 다른 사용자의 실명, 닉네임, 소셜 계정 등 신원 정보는 공개하지 않는다.
4. 사용자는 익명 사용자의 Shelf를 팔로우하여 해당 사용자의 발행 Collection을 확인한다.
5. Collection은 Record를 묶는 플레이리스트형 구조다.
6. Context가 없는 Record와 Record가 없는 Collection은 허용하지 않는다.
7. 사용자 소유 데이터의 삭제는 소프트 삭제한다.
8. 사용자에게 삭제 데이터 복구 기능은 제공하지 않는다.
9. Record 수정은 기존 행 직접 수정이 아니라 기존 Record 소프트 삭제 후 새 Record 생성 방식으로 처리한다.
10. Record 재생성 시 기존 Collection 연결은 새 Record로 승계하고, 활성 Context 전체의 임베딩을 다시 생성한다.

---

# 2. 공식 데이터 구조

```text
User + Place = 활성 Record 최대 1개
Record 1개 = Context 1개 이상
Collection 1개 = Record 1개 이상
Record 1개 = 여러 Collection에 포함 가능
User 1명 = Shelf 1개
Library = 본인 Shelf + 팔로우한 Shelf의 집합
```

## 2.1 핵심 관계

```text
User ── 1:N ── Record ── N:1 ── Place
Record ── 1:N ── Context
Record ── N:M ── Collection
User ── 1:1 ── Shelf
Shelf ── 1:N ── Collection
User ── N:M ── Shelf (Follow 관계)
```

---

# 3. 계정 및 인증 정책

## 3.1 로그인

MVP에서는 소셜 로그인만 제공한다.

- Google
- Kakao
- Naver

자체 아이디·비밀번호 로그인과 다음 기능은 제공하지 않는다.

- 자체 아이디 찾기
- 비밀번호 찾기
- 비밀번호 변경

최초 소셜 로그인 후 서비스 계정을 생성하고 필수 약관 동의를 완료하면 회원가입이 완료된다.

## 3.2 사용자와 인증 수단 분리

서비스 사용자와 소셜 로그인 수단은 분리하여 관리한다.

```text
User
- id

SocialAccount
- id
- user_id
- provider
- provider_user_id
- deleted_at
```

동일 소셜 계정은 다음 조합으로 식별한다.

```text
provider + provider_user_id
```

예:

```text
(KAKAO, 12345)
(NAVER, 12345)
```

두 계정은 식별자 문자열이 같아도 공급자가 다르므로 서로 다른 인증 수단이다.

권장 제약:

```text
UNIQUE(provider, provider_user_id)
```

## 3.3 가입 동의

최초 가입 시 약관 동의 화면에서 다음 내용을 안내하고 동의받는다.

- Collection 생성 시 자동 발행
- 개인 Context 원문은 공개되지 않음
- 공개 화면에는 Place, Keyword, Record 생성일 등이 노출될 수 있음
- Shelf 단위 익명 팔로우 구조

---

# 4. 익명성과 개인 영역

## 4.1 공개 프로필

일반적인 공개 프로필은 제공하지 않는다.

다른 사용자에게 공개되는 사용자 단위 정보는 Shelf뿐이다. Shelf 소유자의 실명, 닉네임, 소셜 계정은 공개하지 않는다.

## 4.2 Library

Library는 사용자의 개인 공간이다.

Library에서 관리하는 정보:

- 내 Record
- 내 Collection
- 내 Shelf
- 팔로우 중인 Shelf
- 팔로워 수
- 팔로잉 Shelf 수
- 연결된 소셜 계정
- 계정 설정
- 회원 탈퇴

Library는 공개 프로필이 아니다.

## 4.3 팔로워·팔로잉 정보

목록은 제공하지 않고 수치만 제공한다.

```text
팔로워 수 = 내 Shelf를 팔로우 중인 사용자 수
팔로잉 수 = 내가 팔로우 중인 Shelf 수
```

두 수치는 본인만 조회할 수 있다. 다른 사용자의 Shelf, Collection, Feed에는 노출하지 않는다.

---

# 5. Place 정책

## 5.1 지도 공급자

장소 검색과 지도 표시는 카카오맵 API를 사용한다.

장소 검색은 카카오맵 API 대상 검색이며, 개인 Record 자연어 검색과 별도 기능이다.

## 5.2 Place 소유권

Place는 실제 지점을 표현하는 공용 데이터다.

- 특정 사용자의 소유가 아니다.
- 여러 사용자의 Record가 동일 Place를 참조할 수 있다.
- 사용자가 Record를 삭제해도 Place는 삭제하지 않는다.

## 5.3 저장 시점

카카오맵 검색 결과를 조회하는 것만으로 내부 DB에 Place를 저장하지 않는다.

```text
카카오맵 장소 검색
→ 검색 결과 반환
→ 내부 Place 저장 안 함
```

Record 생성 시점에 내부 Place를 조회하거나 저장한다.

```text
Record 생성 요청
→ 카카오 장소 식별자로 기존 Place 조회
→ 없으면 Place 저장
→ Record와 첫 Context 생성
```

동일한 카카오 장소 식별자를 가진 Place는 동일 장소로 취급한다.

---

# 6. Record 정책

## 6.1 정의

Record는 한 User와 한 Place의 연결 단위다.

하나의 User는 동일한 Place에 대해 활성 Record를 최대 한 개만 가질 수 있다.

권장 제약:

```text
활성 상태 기준 UNIQUE(user_id, place_id)
```

## 6.2 생성 조건

활성 Record는 활성 Context를 최소 한 개 이상 가져야 한다.

Place만 저장하고 Context를 작성하지 않는 것은 허용하지 않는다.

아직 방문하지 않은 Place도 저장할 수 있지만, 가고 싶은 이유나 저장 이유를 Context로 작성해야 한다.

## 6.3 공개 범위

소유자에게 제공:

- Place
- Context 원문
- Keyword
- Record 생성일
- 연결된 Collection

다른 사용자에게 제공:

- Place
- 비식별화 Keyword
- Record 생성일

Context 원문은 Collection에 포함되어도 공개하지 않는다.

## 6.4 Record 수정과 재생성

Record 수정은 기존 행 직접 수정이 아니라 재생성 방식으로 처리한다.

트랜잭션 기준 절차:

```text
1. 기존 활성 Record 조회
2. 기존 Record가 연결된 Collection 목록 조회
3. 기존 CollectionRecord 연결 소프트 삭제
4. 기존 Record 소프트 삭제
5. 새 Record 생성
6. 활성 Context를 새 Record에 연결
7. 기존 Collection마다 새 CollectionRecord 생성
8. 활성 Context 전체와 Keyword 검색 표현을 다시 임베딩
9. 새 Record를 활성 상태로 확정
```

원칙:

- 기존 Collection 구성은 사용자 관점에서 유지된다.
- 연결 테이블은 기존 연결을 제거한 뒤 새 Record 기준으로 다시 생성한다.
- 새 Record는 새로운 생성 시각을 가진다.
- 기존 Record는 사용자 화면과 검색 대상에서 제외한다.
- 임베딩 재생성에 실패하면 새 Record 활성화와 연결 승계를 완료하지 않는 원자적 처리를 권장한다.

## 6.5 Record 삭제

Record 삭제 시 해당 Record가 포함된 모든 Collection을 확인한다.

- 해당 Record를 제거해도 다른 Record가 남는 Collection: 연결만 소프트 삭제
- 해당 Record가 마지막 Record인 Collection: 확인창 표시 후 Collection도 소프트 삭제

확인 문구 예:

```text
이 기록은 컬렉션의 마지막 기록입니다.
삭제하면 해당 컬렉션도 함께 삭제됩니다.
```

Record 삭제가 Place 삭제를 의미하지는 않는다.

## 6.6 삭제한 Place 재저장

동일 User와 Place의 삭제된 Record가 있어도 사용자 관점에서는 새 Record 생성으로 처리한다.

```text
동일 User + Place의 삭제 Record 존재
→ 새 Record 생성
→ 새 Context 생성
→ 새 임베딩 생성
→ 과거 Collection 연결 복구 안 함
```

과거 Record, Context, 임베딩, Collection 연결은 복구하지 않는다.

---

# 7. Context 정책

## 7.1 정의

Context는 Record 안에 포함되는 독립적인 서술 단위다.

포함 가능한 내용:

- 동행자
- 목적
- 분위기
- 활동
- 저장 이유
- 방문 경험
- 계절·시간대
- 발견 계기

하나의 Record에는 여러 Context가 존재할 수 있다.

## 7.2 생성·수정·삭제

사용자는 자신의 Context를 생성, 수정, 삭제할 수 있다.

Context 수정 시 해당 Context의 임베딩을 다시 생성한다.

Record 재생성 시에는 수정되지 않은 Context를 포함한 모든 활성 Context의 임베딩을 새 Record 기준으로 다시 생성한다.

## 7.3 마지막 Context 삭제

마지막 활성 Context 삭제를 시도하면 Record 삭제 확인창을 표시한다.

```text
이 맥락은 기록의 마지막 맥락입니다.
삭제하면 해당 기록이 함께 삭제됩니다.
```

사용자가 확인하면 Context와 Record를 소프트 삭제한다. Record가 Collection의 마지막 Record인 경우 관련 Collection 삭제 확인도 함께 안내한다.

---

# 8. Keyword 정책

## 8.1 생성

Keyword는 AI가 Place와 Context를 분석해 서비스가 정의한 프리셋 목록에서 매핑한다.

비식별화 원칙:

- 사람 이름 제거
- 구체적인 회사명·학교명 일반화
- 정확한 개인 일정과 날짜 제거 또는 일반화
- 사용자를 특정할 수 있는 사건 제거
- 일반적인 관계 표현은 유지 가능

## 8.2 사용자 권한

사용자는 Keyword를 조회만 할 수 있다.

사용자는 Keyword를 직접 생성, 수정, 삭제할 수 없다.

## 8.3 검색

AI 검색은 다음 데이터를 대상으로 한다.

- Place 정보
- Context 임베딩
- Keyword 임베딩 또는 검색 표현

검색 유사도 계산 후 결과는 Record 단위로 중복 제거하여 반환한다.

---

# 9. Collection 정책

## 9.1 정의

Collection은 Record를 플레이리스트처럼 묶은 사용자 정의 그룹이다.

- Context나 Place를 직접 담지 않는다.
- 하나의 Record는 여러 Collection에 포함될 수 있다.
- 동일 Collection에 같은 Record를 중복 추가할 수 없다.
- Collection 삭제 시 원본 Record는 유지한다.

## 9.2 생성

빈 Collection은 허용하지 않는다.

- Record 1개 이상 필수
- 제목 필수
- 제목 최대 20자
- 설명 필드 없음
- 생성 즉시 자동 발행
- 발행 시각 설정
- MVP에서 비공개 전환 및 발행 취소 불가

## 9.3 공개 정보

Collection 공개 화면:

- Collection 제목
- Place 개수
- Collection 생성일
- 포함된 Place
- 비식별화 Keyword
- Record 생성일

소유자 화면에서는 포함 Record의 Context 원문도 볼 수 있다.

## 9.4 정렬

Collection과 Collection 내부 Record는 `createdAt` 오름차순으로 정렬한다.

```text
오래된 항목 → 먼저 표시
```

Collection 내부 Record 정렬은 Collection에 추가된 시각이 아니라 Record의 생성 시각을 기준으로 한다.

## 9.5 수정

발행 중인 Collection도 수정할 수 있다.

- Record 추가
- Record 제거
- 제목 수정

수정 내용은 별도 발행본이나 스냅샷 없이 기존 공개 Collection에 반영된다.

## 9.6 마지막 Record 제거

마지막 Record 제거를 시도하면 Collection 삭제 확인창을 표시한다.

```text
이 기록은 컬렉션의 마지막 기록입니다.
제거하면 해당 컬렉션이 삭제됩니다.
```

확인하면 Collection을 소프트 삭제하고 Record와 Collection의 연결을 소프트 삭제한다. Record 자체는 유지한다.

## 9.7 Collection 삭제

Collection 삭제 시:

- Collection 소프트 삭제
- CollectionRecord 연결 소프트 삭제
- 원본 Record 유지

---

# 10. Shelf·Library·Follow 정책

## 10.1 Shelf

User마다 Shelf를 하나씩 가진다.

Shelf는 해당 User가 발행한 Collection 목록이다.

본인 Shelf에는 별도의 공개 이름이 없다.

## 10.2 Library

Library는 다음 Shelf를 모아 보는 개인 공간이다.

- 내 Shelf
- 내가 팔로우한 Shelf

## 10.3 Follow 대상

Follow 대상은 User나 개별 Collection이 아니라 Shelf다.

- 자신의 Shelf 팔로우 금지
- 동일 Shelf 중복 팔로우 금지

## 10.4 팔로우 Shelf 별칭

별칭은 Shelf 자체가 아니라 Follow 관계에 속한다.

| 항목 | 정책 |
|---|---|
| 입력 | 선택 |
| 미지정 | `null` |
| 최대 길이 | 20자 |
| 앞뒤 공백 | 제거 |
| 빈 문자열·공백 | `null` |
| 동일 이름 중복 | 허용 |
| 수정 | 허용 |
| 제거 | `null`로 변경 |

같은 Shelf라도 팔로워마다 서로 다른 별칭을 지정할 수 있다.

## 10.5 Shelf 삭제와 동기화

회원 탈퇴 시 해당 User의 Shelf를 소프트 삭제한다.

그 Shelf를 대상으로 한 Follow 관계도 소프트 삭제하며, 팔로워의 Library에서 해당 Shelf와 Collection이 제거된다.

---

# 11. Feed 정책

## 11.1 역할

Feed는 발행된 Collection을 개인화하여 익명으로 추천하는 영역이다.

## 11.2 추천 후보

비즈니스 정책상 후보 조건은 다음과 같다.

- Collection 소유 User가 탈퇴하지 않음

다만 모든 조회에는 공통적으로 소프트 삭제된 User, Shelf, Collection, Record, 연결 데이터를 제외하는 필터를 적용한다. 이는 별도 추천 조건이 아니라 데이터 노출의 기본 조건이다.

신고·차단 기능은 MVP에서 제공하지 않는다.

## 11.3 추천 단위

추천 단위는 Collection이다.

개별 Record 추천은 후순위다. 다른 사용자의 Context 원문은 공개하지 않으며, Collection 내부에는 Place, Keyword, Record 생성일을 제공한다.

---

# 12. AI 검색 및 RAG 정책

## 12.1 검색 대상

현재 로그인한 사용자의 다음 데이터만 검색한다.

- Record에 연결된 Place
- Record의 Context
- Record의 Keyword

다른 사용자의 Context와 Collection은 검색하지 않는다.

## 12.2 결과 단위

```text
질의 입력
→ Place·Context·Keyword 검색
→ 유사한 항목 식별
→ 관련 Record 식별
→ Record 단위 중복 제거
→ Record 목록 반환
```

하나의 Record에서 여러 Context나 Keyword가 일치하더라도 Record는 한 번만 반환한다.

검색 결과 선택 이유는 별도로 표시하지 않는다.

## 12.3 기술 방식

다음은 미확정이다.

- GraphRAG 사용 여부
- Java AI 기반 일반 RAG 사용 여부
- 그래프 관계를 검색 증강에 포함할지
- 임베딩 모델과 저장 방식
- 임베딩 재생성의 동기·비동기 처리

확정 전까지 공식 기능명은 `AI 자연어 검색`으로 사용한다.

---

# 13. 삭제·탈퇴 정책

## 13.1 삭제

사용자 소유 데이터는 소프트 삭제한다.

Place는 공용 데이터이므로 사용자 데이터 삭제에 따라 삭제하지 않는다.

사용자에게 삭제 데이터 복구 기능은 제공하지 않는다.

## 13.2 회원 탈퇴

회원 탈퇴 시 다음 데이터를 소프트 삭제한다.

- User
- SocialAccount
- Record
- Context
- Collection
- CollectionRecord
- Shelf
- User가 생성한 Follow 관계
- User의 Shelf를 대상으로 한 Follow 관계

탈퇴 즉시 Feed와 다른 사용자의 Library에서 해당 User의 Shelf와 Collection을 제외한다.

Place는 유지한다.

---

# 14. 트랜잭션 및 무결성 원칙

다음 작업은 하나의 트랜잭션으로 처리하는 것을 권장한다.

## 14.1 Record 생성

```text
Place 조회·저장
→ Record 생성
→ 첫 Context 생성
→ Keyword 생성
→ 임베딩 생성
```

## 14.2 Record 재생성

```text
기존 Collection 연결 조회
→ 기존 연결 소프트 삭제
→ 기존 Record 소프트 삭제
→ 새 Record 생성
→ Context 연결
→ Collection 연결 재생성
→ 전체 Context 및 Keyword 재임베딩
```

## 14.3 마지막 Context 삭제

```text
마지막 Context 여부 확인
→ 사용자 확인
→ 관련 Collection 영향 확인
→ Context·Record·필요 Collection 소프트 삭제
```

## 14.4 회원 탈퇴

```text
User 관련 데이터 소프트 삭제
→ Shelf 대상 Follow 관계 소프트 삭제
→ Feed·Library 노출 제외
```
