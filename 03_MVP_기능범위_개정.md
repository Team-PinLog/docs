# PinLog MVP 기능 범위 — 개정본

## 1. 계정

- Google 소셜 로그인
- Kakao 소셜 로그인
- Naver 소셜 로그인
- 최초 로그인 시 User 생성
- 필수 약관 동의
- 로그아웃
- 회원 탈퇴
- SocialAccount 분리 저장

## 2. Place

- 카카오맵 Place 검색
- Place 선택
- Place 기본 정보 조회
- 카카오 Place 식별자 기준 중복 처리
- Record 생성 시점에만 내부 Place 저장
- 지도 마커 표시

## 3. Record

- User당 Place별 활성 Record 최대 1개
- 첫 Context 필수
- Record 조회
- Record 재생성 방식 수정
- Record 소프트 삭제
- 삭제 Place 재저장 시 새 Record 생성
- Record 생성일 표시
- Collection 연결 승계

## 4. Context

- Context 복수 생성
- Context 조회
- Context 수정
- Context 삭제
- 마지막 Context 삭제 시 Record 삭제 확인
- Context 임베딩 생성
- Record 재생성 시 전체 활성 Context 재임베딩

## 5. Keyword

- AI 프리셋 매핑
- 비식별화
- 사용자 조회
- 사용자 생성·수정·삭제 불가
- 검색 데이터로 활용

## 6. AI 자연어 검색

- Place 검색 표현
- Context 임베딩 검색
- Keyword 검색
- Record 단위 중복 제거
- Record 결과 반환
- 검색 결과 없음 처리

검색 결과 선택 이유는 제공하지 않는다.

GraphRAG 사용 여부는 MVP 기능 확정과 분리하여 기술 검토한다.

## 7. Collection

- Record 1개 이상으로 생성
- 제목 필수, 최대 20자
- 설명 없음
- 자동 발행
- Record 추가·제거
- 제목 수정
- 소프트 삭제
- 마지막 Record 제거 시 Collection 삭제 확인
- Record 재생성 시 연결 승계
- `createdAt ASC` 정렬

## 8. Shelf·Library·Follow

- User당 Shelf 1개
- 내 Shelf 조회
- 팔로우한 Shelf 조회
- Shelf 팔로우·해제
- 개인 별칭 지정
- 별칭 최대 20자
- 팔로워·팔로잉 수 본인 조회
- 목록은 제공하지 않음
- 탈퇴 Shelf를 팔로워 Library에서 제거

## 9. Feed

- 발행 Collection 추천
- Collection 상세 조회
- 같은 Shelf의 다른 Collection 조회
- Shelf 팔로우
- Place 선택 후 내 Record 생성 또는 Context 추가
- 탈퇴 User 데이터 제외
- 소프트 삭제 데이터 제외

## 10. 개인정보

- Context 원문 비공개
- Place, Keyword, Record 생성일 공개
- 신원 정보 비공개
- Collection 자동 발행을 가입 약관에서 고지
- 개인 식별 표현 비식별화

## 11. MVP 제외

- 자체 로그인
- 비밀번호 찾기·변경
- Collection 비공개 전환
- Collection 설명
- Keyword 사용자 편집
- 검색 결과 선택 이유
- User·Collection 검색
- 신고·차단
- 개별 Record 추천 Feed
- AI Collection 자동 생성
- 자연어 CUD
