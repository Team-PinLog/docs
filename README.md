# PinLog Release Documents

PinLog의 최종 Release 문서 모음입니다.

PinLog은 장소를 `Context`와 함께 `Record`로 저장하고, 자연어로 다시 찾으며, 익명 `Collection`을 통해 새로운 장소를 발견하는 서비스입니다.

## 문서 구조

| 문서 | 역할 |
|---|---|
| [Quick Start](docs/00_Quick_Start.md) | 프로젝트 5분 요약 |
| [서비스 기획서](docs/01_서비스_기획서.md) | 문제, 사용자, 핵심 가치 |
| [정책 정의서](docs/02_정책_정의서.md) | 서비스 동작과 데이터 공개 정책 |
| [공식 용어사전](docs/03_공식_용어사전.md) | 제품·UI·DB 공통 용어 |
| [MVP 기능범위](docs/04_MVP_기능범위.md) | 포함·제외 기능 |
| [AI 설계](docs/05_AI_설계.md) | AI 자연어 검색과 임베딩 처리 |
| [익명 SNS 공개정책](docs/06_익명SNS_공개정책.md) | 익명성, 공개 범위, Follow |
| [데이터 모델 및 무결성](docs/07_데이터모델_및_무결성.md) | 테이블과 제약, 트랜잭션 |
| [ERD](docs/08_ERD.md) | Mermaid 관계도 |
| [API 명세](docs/09_API_명세.md) | MVP Endpoint 목록 |
| [유저플로우](docs/10_유저플로우.md) | 핵심 사용자 흐름 |
| [개발 컨벤션](docs/11_개발_컨벤션.md) | 현재 프로젝트에서 확인된 구현 규칙 |

## 기준

- 공식 기록 단위는 `Record`, 독립 서술 단위는 `Context`입니다.
- `Memory` 등 폐기 용어는 사용하지 않습니다.
- 변경이력과 미결정사항은 Release 문서에 포함하지 않습니다.
