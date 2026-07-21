# PinLog 익명 SNS 및 공개 정책 — 개정본

## 1. 익명 구조

- 실명, 닉네임, 소셜 계정을 공개하지 않는다.
- 동일 User의 Collection은 Shelf 아래에서 연결된다.
- 다른 사용자는 Shelf와 그 안의 Collection을 조회할 수 있다.
- 익명성은 일회성이 아니라 지속형 익명 구조다.

## 2. 공개 데이터

### 본인 Library

- Place
- Context 원문
- Keyword
- Record 생성일
- Collection
- Shelf
- Follow 정보

### 다른 사용자에게 공개

- Collection 제목
- Collection 생성일
- Place
- Keyword
- Record 생성일

### 공개하지 않음

- Context 원문
- 이름
- 소셜 계정
- 개인 식별 표현
- 구체적 회사명·학교명
- 정확한 개인 일정
- 민감한 사건

## 3. Collection 자동 발행

Collection은 생성 즉시 발행한다.

- MVP에서 발행 취소 불가
- 비공개 전환 불가
- 수정사항은 즉시 공개본에 반영
- 별도 스냅샷 없음

자동 발행과 공개 범위는 최초 가입 약관 동의 화면에서 안내한다.

## 4. Shelf 팔로우

- 팔로우 대상은 Shelf
- 자신의 Shelf 팔로우 금지
- 중복 팔로우 금지
- 팔로워가 개인 별칭 지정 가능
- 별칭은 본인에게만 공개

## 5. Feed

Feed는 Collection을 추천한다.

후보의 비즈니스 조건:

- 소유 User가 탈퇴하지 않음

공통 데이터 접근 조건:

- 삭제된 User 제외
- 삭제된 Shelf 제외
- 삭제된 Collection 제외
- 삭제된 Record와 연결 제외

신고·차단 기능은 MVP에 없다.

## 6. 탈퇴

User 탈퇴 시 Shelf와 관련 데이터가 소프트 삭제된다.

- 다른 사용자의 Library에서 해당 Shelf 제거
- 관련 Follow 관계 제거
- Collection을 Feed에서 즉시 제외
