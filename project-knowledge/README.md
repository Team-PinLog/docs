# PinLog 프로젝트 지식 지도

이 디렉터리는 프로젝트 진행 중 저장소에 축적된 비밀이 아닌 지식을 제품 가치부터 운영까지 한 흐름으로 연결한다. 현재 실행 상태를 주장하는 운영 대시보드가 아니라, Team-PinLog의 GitHub 저장소 문서와 선언형 설정을 연결한 탐색용 인덱스다.

## 읽는 순서

1. [지식 그래프](knowledge-graph.md) — 제품 가치, 영역, 저장소 근거의 관계
2. [아키텍처](architecture.md) — 전체 구조와 요청·배포·데이터 흐름
3. [주요 결정](decisions.md) — 문제, 선택, 이유, 대안, 트레이드오프
4. [운영 원칙](operations.md) — 배포·관측·백업·장애 대응 및 기록의 한계
5. [용어사전](glossary.md) — 제품·기술·운영 공통 언어

## 범위와 근거 원칙

조사 범위는 Team-PinLog의 `ai`, `back`, `front`, `image`, `infra`, `docs`, `cowork` GitHub 저장소 문서와 설정이다. `docs` 저장소 자체 근거는 실제 상대경로로, 다른 저장소 근거는 검증한 GitHub 파일 URL로 연결했다.

- 제품 정책과 파트 간 계약의 정본: [Release 문서 인덱스](../README.md)
- 선언과 당시 기록이 다르면 이 문서는 차이를 숨기지 않고 `설계`, `선언`, `당시 기록`, `미확인`으로 구분한다.
- 운영 주소, 운영 IP, 자격증명 값, 실제 데이터는 싣지 않는다.
- 실행 중인 클러스터·DB·외부 서비스는 조사하거나 변경하지 않았다.

## 영역별 책임 지도

| 영역 | 주 책임 | 책임 밖 | 대표 근거 |
|---|---|---|---|
| Product docs | 문제·가치·정책·공통 계약 | 런타임 구현 상태 | [Quick Start](../static/00_Quick_Start.md), [서비스 기획서](../static/01_서비스_기획서.md) |
| Frontend | PC 웹 UX, 서버/UI 상태 분리, Core·Image 경계 오케스트레이션 | Core 인가, AI 내부 처리, 작업자 인증정보 | [Frontend 아키텍처](https://github.com/Team-PinLog/front/blob/dev/docs/architecture.md), [API 계약](https://github.com/Team-PinLog/front/blob/dev/docs/api-contract.md) |
| Backend | 인증·인가, Core 도메인·트랜잭션, 최종 응답, Flyway, AI 작업 상태·재스캔 | 모델 호출과 벡터 계산 | [Backend README](https://github.com/Team-PinLog/back/blob/dev/README.md), [AI 공용 설계](../static/05_AI_설계.md) |
| AI | 임베딩·키워드 판정·개인 의미 검색, `ai` 스키마 파생 데이터 | `core` 접근, 사용자 인증, migration 실행 | [AI README](https://github.com/Team-PinLog/ai/blob/dev/README.md), [AI 아키텍처](https://github.com/Team-PinLog/ai/blob/dev/docs/spec/architecture.md) |
| Image | 표지 후보·최종본 작업 API와 파일 저장, GPU 작업자 큐 계약 | Collection 영속 계약과 브라우저용 작업자 토큰 | [Image README](https://github.com/Team-PinLog/image/blob/feat/gpu-cover-worker-mvp/README.md), [Frontend Image 계약](https://github.com/Team-PinLog/front/blob/dev/docs/frontend-image-runtime-contract.md) |
| Infra | k3s·Argo CD GitOps, 배포 선언, 데이터·캐시 플랫폼, Secret 봉인, 관측·백업 연결 | 앱 비즈니스 로직과 DB migration 내용 | [Infra README](https://github.com/Team-PinLog/infra/blob/main/README.md), [공용 Helm 값](https://github.com/Team-PinLog/infra/blob/main/charts/microservice/values.yaml) |
| Cowork | 자연어 작업 입력을 팀 작업 티켓 초안·생성으로 연결하는 내부 보조 도구 | 제품 런타임과 Jira 대체 | [Cowork 명세](https://github.com/Team-PinLog/cowork/blob/main/SPEC.md) |

## 핵심 경계

- 외부 클라이언트의 제품 API는 Backend가 소유하고, AI 내부 API는 Backend만 호출한다.
- Backend가 `core` 도메인과 migration을 소유하며, AI는 `ai` 스키마에 한정된 계산·파생 데이터 책임을 가진다.
- Context 원문은 개인 데이터이고 타인 공개면에는 나오지 않는다. 공개 발견은 Collection·Shelf와 비식별화 표현을 중심으로 한다.
- Collection 저장과 표지 생성은 분리된다. 표지가 없어도 Collection은 유효하며 Frontend가 Image 비동기 상태를 조정한다.
- 배포의 원하는 상태는 Infra 선언이며, 서비스 소스에서 만든 불변 이미지가 검증된 변경 요청을 통해 승격된다.
- 실제 비밀 값은 배포 Secret 경계에 머물며 문서·로그·브라우저 번들에 넣지 않는다.
