# PinLog Release Documents

PinLog의 최종 Release 문서 모음입니다.

PinLog은 장소를 `Context`와 함께 `Record`로 저장하고, 자연어로 다시 찾으며, 익명 `Collection`을 통해 새로운 장소를 발견하는 서비스입니다.

## 문서 구조

| 문서 | 역할 |
|---|---|
| [Quick Start](static/00_Quick_Start.md) | 프로젝트 5분 요약 |
| [서비스 기획서](static/01_서비스_기획서.md) | 문제, 사용자, 핵심 가치 |
| [정책 정의서](static/02_정책_정의서.md) | 서비스 동작과 데이터 공개 정책 |
| [공식 용어사전](static/03_공식_용어사전.md) | 제품·UI·DB 공통 용어 |
| [익명 SNS 공개정책](static/04_익명SNS_공개정책.md) | 익명성, 공개 범위, Follow |
| [AI 설계](static/05_AI_설계.md) | AI 아키텍처, 비동기 처리, 임베딩, 키워드, 자연어 검색 및 파트 간 계약 |
| [파트 간 요구사항](static/06_파트간_요구사항.md) | Front·Infra가 수행할 요구사항 |
| [데이터 모델 및 무결성](draft/06_데이터모델_및_무결성.md) | 테이블과 제약, 트랜잭션 |
| [ERD](draft/07_ERD.md) | Mermaid 관계도 |
| [API 명세](draft/08_API_명세.md) | MVP Endpoint 목록 |
| [API 상세 명세](draft/11_API_상세명세.md) | 디자인 화면 → 엔드포인트 매핑, 요청·응답 초안 |
| [유저플로우](draft/09_유저플로우.md) | 핵심 사용자 흐름 |
| [MVP 기능범위](draft/10_MVP_기능범위.md) | 포함·제외 기능 |

## 기준

- 공식 기록 단위는 `Record`, 독립 서술 단위는 `Context`입니다.
- `Memory` 등 폐기 용어는 사용하지 않습니다.
- 변경이력과 미결정사항은 Release 문서에 포함하지 않습니다.
