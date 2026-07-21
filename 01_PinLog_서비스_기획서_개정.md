# PinLog 서비스 기획서 — 개정본

## 1. 서비스 개요

| 항목 | 내용 |
|---|---|
| 서비스명 | PinLog / 핀로그 |
| 서비스 형태 | 맥락 기반 장소 기록·검색 및 익명 장소 큐레이션 SNS |
| 핵심 기록 단위 | Record |
| 서술 단위 | Context |
| 개인 공간 | Library |
| 공개 큐레이션 | Collection |
| 익명 사용자 단위 | Shelf |
| 추천 영역 | Feed |

PinLog은 사용자가 Place를 Context와 함께 Record로 저장하고, 이후 자연어로 다시 찾으며, 다른 사용자의 익명 Collection을 통해 새로운 Place를 발견하는 서비스다.

## 2. 핵심 구조

```text
User + Place = 활성 Record 최대 1개
Record = Place + Context 1개 이상 + Keyword
Collection = Record 1개 이상
Shelf = 한 User의 발행 Collection 목록
Library = 내 Shelf + 팔로우한 Shelf
Feed = 익명 Collection 추천 영역
```

## 3. 핵심 가치

### 3.1 맥락이 남는 장소 기록

사용자는 Place만 저장하지 않는다. 왜 저장했는지, 누구와 무엇을 하고 싶은지, 방문 후 무엇이 기억에 남았는지를 Context로 기록한다.

### 3.2 장소명이 없어도 찾는 검색

AI 자연어 검색은 Record 내부의 Place, Context, Keyword를 검색해 관련 Record를 반환한다.

예:

- “비 오는 날 친구와 가려고 저장한 카페”
- “부모님과 갔던 조용한 한식당”
- “피드에서 발견한 성수 전시”

### 3.3 익명 취향 발견

Feed는 User 신원이 아니라 Collection을 추천한다. 사용자는 같은 익명 Shelf의 다른 Collection을 보고 Shelf를 팔로우할 수 있다.

### 3.4 발견을 내 기록으로 전환

다른 사용자의 Collection에서 Place를 발견하면 자신의 Context를 새로 작성해 개인 Record로 저장한다. 다른 사용자의 Context 원문은 가져오지 않는다.

## 4. 주요 사용자 흐름

### 4.1 Place 저장

```text
카카오맵 Place 검색
→ Place 선택
→ 첫 Context 작성
→ Record 생성
→ Keyword 매핑
→ 임베딩 생성
```

### 4.2 기존 Place에 Context 추가

```text
Place 선택
→ 기존 활성 Record 확인
→ 기존 Context 확인
→ Context 추가
→ 임베딩 생성
```

### 4.3 Record 수정

Record 수정은 기존 행을 직접 수정하지 않는다.

```text
기존 Record와 Collection 연결 조회
→ 기존 연결 제거
→ 기존 Record 소프트 삭제
→ 새 Record 생성
→ 기존 Collection 연결 재생성
→ 전체 Context 재임베딩
```

사용자 화면에서는 Collection 구성이 유지되는 것처럼 보인다.

### 4.4 Collection 생성

```text
Record 1개 이상 선택
→ 제목 입력
→ Collection 생성
→ 자동 발행
→ Shelf와 Feed에 반영
```

Collection 설명은 제공하지 않는다.

### 4.5 Feed에서 Place 가져오기

```text
Collection 탐색
→ Place 선택
→ 내 기존 Record 확인
→ 없으면 첫 Context 작성 후 Record 생성
→ 있으면 Context 추가 또는 수정
```

### 4.6 Shelf 팔로우

```text
Collection에서 Shelf 이동
→ Shelf 팔로우
→ 선택적으로 개인 별칭 지정
→ Library에 Shelf 표시
```

## 5. 공개 범위

### 본인

- Place
- Context 원문
- Keyword
- Record 생성일
- Collection 연결

### 다른 사용자

- Collection 제목
- Collection 생성일
- Place
- Keyword
- Record 생성일

비공개:

- Context 원문
- 실명·닉네임·소셜 계정
- 개인을 식별할 수 있는 구체적 표현

## 6. MVP 범위

### 포함

- Google·Kakao·Naver 로그인
- 카카오맵 Place 검색
- Record·Context 생성·조회·수정·삭제
- Keyword 자동 매핑 및 조회
- Place·Context·Keyword 기반 AI 자연어 검색
- Collection 생성·자동 발행·수정·삭제
- Shelf 조회·팔로우·별칭
- Feed에서 Collection 추천
- 회원 탈퇴와 소프트 삭제

### 제외

- 자체 아이디·비밀번호 로그인
- Collection 비공개 전환
- Collection 설명
- 사용자의 Keyword 생성·수정·삭제
- 검색 결과 선택 이유
- User·Collection 직접 검색
- 신고·차단
- 개별 Record 추천 Feed
- AI Collection 자동 생성
- 자연어 기반 생성·수정·삭제

## 7. 미확정 기술

- GraphRAG 또는 일반 RAG
- Java AI 기반 구현 범위
- 임베딩 모델
- 그래프 관계 활용 여부
- 임베딩 생성 동기·비동기 방식
