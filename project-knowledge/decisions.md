# 주요 결정 기록

아래 항목은 기존 결정·제안·설계 문서에서 프로젝트 전체 관계를 이해하는 데 중요한 내용을 추린 것이다. 각 결정은 문제, 선택, 이유, 대안, 트레이드오프, 근거를 같은 형식으로 정리한다. 숫자는 항목 식별자이며 우선순위가 아니다.

## D-01. 장소가 아니라 Context가 있는 Record를 핵심 단위로 둔다

- 문제: 장소만 저장하면 나중에 저장 이유·동행·상황으로 다시 찾기 어렵다.
- 선택: Place와 User의 연결을 Record로 두고 하나 이상의 불변 Context를 필수로 한다.
- 이유: 미래 계획, 과거 경험, 발견 후 개인화를 하나의 모델로 표현할 수 있다.
- 대안: 장소 북마크, 자유형 리뷰, 모드별 별도 모델.
- 트레이드오프: 빈 Record를 허용하지 않으므로 마지막 Context 삭제가 상위 삭제와 연결된다.
- 근거: [서비스 기획서](../static/01_서비스_기획서.md), [Quick Start](../static/00_Quick_Start.md), [데이터 모델](../static/06_데이터모델_및_무결성.md).

## D-02. Context는 불변이며 수정은 새 Context로 교체한다

- 문제: 비동기 AI 결과가 수정 전 본문에 대한 것인지 두 서비스가 버전으로 판별해야 했다.
- 선택: 동일한 Context 식별자는 항상 동일한 본문을 뜻하게 하고, 수정은 새 행 생성과 기존 행 삭제로 처리한다.
- 이유: 수정 경합을 삭제 경합에 흡수하고 별도 본문 버전을 없앤다.
- 대안: 도메인 `body_version`, JPA 낙관적 락.
- 트레이드오프: 새 식별자 반영이 FE 계약에 필요하고, AI 파생 데이터 재생성·삭제 행 누적을 감수한다.
- 근거: [BD-07](https://github.com/Team-PinLog/back/blob/dev/docs/backend/decisions/BD-07-context-immutability.md), [AI 공용 설계](../static/05_AI_설계.md).

## D-03. 공개 발견은 사람 신원이 아니라 Collection·Shelf 중심이다

- 문제: 사람 중심 SNS는 개인 Context 노출과 신원 추론 위험을 키운다.
- 선택: Feed는 익명 Collection을 추천하고 Follow 대상은 Shelf로 둔다. 타인의 Context 원문은 공개하지 않는다.
- 이유: 장소 취향 발견 가치와 개인 기록 보호를 함께 지킨다.
- 대안: 공개 프로필·User 팔로우, Context 원문 공유.
- 트레이드오프: 사회적 관계 표현이 제한되고, 권한 실패를 존재 은닉 응답으로 다뤄야 한다.
- 근거: [익명 SNS 공개정책](../static/04_익명SNS_공개정책.md), [유저플로우](../static/09_유저플로우.md).

## D-04. AI를 Core 기능의 비동기 부가 계층으로 둔다

- 문제: 모델 호출 지연·실패가 Record 저장이나 Collection 발행을 막을 수 있다.
- 선택: Core commit 뒤 AI 처리를 요청하고, Keyword·검색 반영의 일시적 공백을 정상 상태로 취급한다.
- 이유: 저장·조회·발행의 가용성을 외부 모델 상태와 분리한다.
- 대안: AI 완료까지 동기 대기, AI 실패 시 Core rollback.
- 트레이드오프: UI가 빈 Keyword와 eventual consistency를 표현해야 하고 복구 경로가 필요하다.
- 근거: [AI 공용 설계](../static/05_AI_설계.md), [Frontend 아키텍처](https://github.com/Team-PinLog/front/blob/dev/docs/architecture.md).

## D-05. 메시지 브로커 대신 영속 AI State와 Scheduler를 사용한다

- 문제: commit 이후 AI 호출이 유실될 수 있다.
- 선택: DB의 단계별 상태를 진실의 원본으로 두고 Backend Scheduler가 미완료·만료 작업을 재스캔한다.
- 이유: 이미 필요한 상태 행으로 재처리 대상을 찾을 수 있어 MVP에서 별도 운영 구성요소를 피한다.
- 대안: 메시지 브로커, Outbox와 relay.
- 트레이드오프: polling 비용·재스캔 지연·배치 처리량 한계를 애플리케이션이 부담한다.
- 근거: [BD-17](https://github.com/Team-PinLog/back/blob/dev/docs/backend/decisions/BD-17-async-without-message-queue.md), [AI 재스캔 명세](https://github.com/Team-PinLog/back/blob/dev/docs/ai/spec/ai-rescan-scheduler.md).

## D-06. AI는 `ai` 스키마에만 접근하고 migration은 Backend가 소유한다

- 문제: Backend와 AI가 같은 DB를 쓰면 Core 소유권과 DDL 책임이 섞일 수 있다.
- 선택: AI runtime role과 search path를 `ai` 중심으로 제한하고, `core`·`ai` migration은 Backend Flyway가 적용한다.
- 이유: 계산 서비스가 Core 상태를 우회 변경하지 못하게 하고 schema history를 하나로 유지한다.
- 대안: AI가 자체 migration 실행, FastAPI의 `core` 직접 조회.
- 트레이드오프: AI 배포가 Backend migration 준비에 의존하고 파트 간 선행조건 조율이 필요하다.
- 근거: [AI 아키텍처](https://github.com/Team-PinLog/ai/blob/dev/docs/spec/architecture.md), [DB migration README](https://github.com/Team-PinLog/back/blob/dev/src/main/resources/db/migration/README.md), [AI 배포 선행조건](https://github.com/Team-PinLog/infra/blob/main/docs/ai-dev-prerequisites.md).

## D-07. 개인 검색은 exact cosine을 사용한다

- 문제: pgvector 검색에 정확 스캔과 근사 인덱스 중 무엇을 쓸지 정해야 했다.
- 선택: 사용자 범위와 삭제·상태 필터를 먼저 적용한 뒤 정확 cosine으로 정렬한다.
- 이유: 개인 Context 범위에서 recall 손실 없는 단순한 검색을 우선했다.
- 대안: HNSW, IVFFlat.
- 트레이드오프: 개인 데이터가 커져 스캔이 병목이 되면 근사 검색을 재평가해야 한다.
- 근거: [P5 exact cosine](https://github.com/Team-PinLog/ai/blob/dev/docs/proposals/P5-exact-cosine.md), [개인 검색 명세](https://github.com/Team-PinLog/ai/blob/dev/docs/spec/personal-search.md).

## D-08. Keyword는 자유 생성이 아니라 preset 후보와 구조화 판정으로 만든다

- 문제: 자유 생성 Keyword는 표현이 흔들리고 공개·추천 계약을 통제하기 어렵다.
- 선택: Context embedding으로 preset 후보를 찾고 모델은 전달된 후보 식별자 중에서만 선택한다.
- 이유: 표시·추천에 쓰는 분류 vocabulary를 통제하고 목록 밖 결과를 폐기할 수 있다.
- 대안: LLM 자유 생성, embedding top 결과를 그대로 확정.
- 트레이드오프: preset 설계와 재분류 운영이 필요하고, 후보 recall과 모델 정밀도 사이를 조정해야 한다.
- 근거: [P26 Keyword 판정](https://github.com/Team-PinLog/ai/blob/dev/docs/proposals/P26-keyword-preset-judgment.md), [Keyword preset 명세](https://github.com/Team-PinLog/ai/blob/dev/docs/spec/keyword-preset.md).

## D-09. Frontend는 서버 상태와 UI 상태를 분리한다

- 문제: 서버 응답을 화면 Context에 복사하면 캐시·무효화·로컬 상태가 충돌한다.
- 선택: 서버 상태는 Query 계층, 모달·선택·입력 같은 화면 상태는 Context API에 둔다.
- 이유: 재요청·캐시 일관성을 데이터 도구에 맡기고 UI 흐름을 독립시킨다.
- 대안: 단일 전역 store, Component의 직접 API 호출.
- 트레이드오프: feature별 hook·API·schema 계층을 유지해야 한다.
- 근거: [Frontend 아키텍처](https://github.com/Team-PinLog/front/blob/dev/docs/architecture.md), [Frontend conventions](https://github.com/Team-PinLog/front/blob/dev/docs/conventions.md).

## D-10. 인증 토큰은 서버 관리 쿠키에 두고 비대칭 서명을 택한다

- 문제: 브라우저 토큰 노출을 줄이고 향후 검증 주체 분리 가능성도 고려해야 했다.
- 선택: Frontend가 토큰을 저장하지 않는 HttpOnly cookie 모델과 RS256 서명을 사용하며 운영 키 누락은 fail-fast로 처리한다.
- 이유: 브라우저 스크립트의 토큰 취급을 없애고, 나중에 공개키 검증자를 분리할 때 비밀 공유를 피한다.
- 대안: 브라우저 저장 Bearer token, HS256, ES256.
- 트레이드오프: cookie refresh·CSRF·cross-tab 동시성, PEM 주입과 키 회전 과제가 생긴다.
- 근거: [인증 설계](../static/11_인증_설계.md), [BD-31](https://github.com/Team-PinLog/back/blob/dev/docs/backend/decisions/BD-31-jwt-rs256-key-management.md), [Frontend 아키텍처](https://github.com/Team-PinLog/front/blob/dev/docs/architecture.md).

## D-11. Collection과 표지 생성을 분리하고 Frontend가 비동기 단계를 조정한다

- 문제: GPU 표지 생성은 느리거나 실패할 수 있고 사용자의 스타일 선택이 필요하다.
- 선택: Collection을 표지 없이 먼저 생성하고, Frontend가 Image 후보·선택·최종본 단계를 수행한 뒤 표지 경로를 Backend에 등록한다.
- 이유: Core 생성 버튼을 GPU 큐에 묶지 않고 사용자 선택 단계를 보존한다.
- 대안: Backend 동기 생성, Collection 생성 transaction에 표지 포함.
- 트레이드오프: 표지 없는 정상 상태, polling 정리, 서로 다른 API 형식용 전용 client가 필요하다.
- 근거: [Frontend API 계약](https://github.com/Team-PinLog/front/blob/dev/docs/api-contract.md), [Image README](https://github.com/Team-PinLog/image/blob/feat/gpu-cover-worker-mvp/README.md).

## D-12. 배포는 범용 Helm chart와 디렉터리 기반 ApplicationSet을 사용한다

- 문제: 초기에는 서비스 수와 이름이 확정되지 않았고 서비스마다 배포 YAML을 복제하면 drift가 커진다.
- 선택: 공용 microservice chart 하나와 `apps/<env>/<service>/values.yaml`을 사용하고 ApplicationSet이 서비스를 발견한다.
- 이유: 안전 기본값을 중앙화하고 디렉터리 추가로 확장한다.
- 대안: 서비스별 chart, Helm과 Kustomize 병용, 수동 Application 작성.
- 트레이드오프: 공용 chart 변경의 영향 범위가 넓어 render guardrail이 필수다.
- 근거: [Infra 아키텍처](https://github.com/Team-PinLog/infra/blob/main/docs/architecture.md), [공용 Helm 값](https://github.com/Team-PinLog/infra/blob/main/charts/microservice/values.yaml), [prod ApplicationSet](https://github.com/Team-PinLog/infra/blob/main/argocd/applicationsets/services-prod.yaml).

## D-13. 이미지 승격은 commit 태그와 digest를 함께 검증하는 GitOps 변경이다

- 문제: mutable tag drift, 출처가 다른 이미지, 기준 브랜치 직접 변경은 재현성과 감사를 깨뜨린다.
- 선택: 서비스 CI 결과와 registry digest를 검증하고 Infra values의 tag·digest만 기능 브랜치 변경 요청으로 갱신한다.
- 이유: 소스 commit, publish run, registry artifact, 배포 선언을 연결하고 Git 이력을 rollback 기준으로 삼는다.
- 대안: mutable latest, 런타임 image 직접 변경, 자동 image updater의 직접 write-back.
- 트레이드오프: 교차 저장소 최소권한 credential과 exact-head 검증 자동화가 복잡해진다.
- 근거: [Git/CI 거버넌스](https://github.com/Team-PinLog/infra/blob/main/docs/git-governance.md), [Backend 이미지 결정](https://github.com/Team-PinLog/back/blob/dev/docs/backend/decisions/BD-44-image-takes-prebuilt-jar.md).

## D-14. probe의 의미를 나누고 readiness에 필요한 의존성만 포함한다

- 문제: 외부 의존성 장애가 있어도 Pod가 Ready로 남거나, 반대로 liveness가 의존성 순단 때문에 재시작을 유발할 수 있다.
- 선택: startup은 느린 기동 보호, liveness는 외부 의존성을 제외한 생존, readiness는 트래픽 수용 조건으로 사용한다. Backend readiness에는 DB를 포함하고 Redis는 현재 제외한다.
- 이유: DB 장애 트래픽은 차단하되 Redis 장애의 전체 서비스 의미가 확정되기 전 과잉 격리를 피한다.
- 대안: 전체 health 하나, 모든 의존성을 liveness/readiness에 포함, 내부 상태만 확인.
- 트레이드오프: 단일 replica에서는 DB 장애가 곧 endpoint 부재이며, Redis 의존성이 커지면 재검토가 필요하다.
- 근거: [BD-28](https://github.com/Team-PinLog/back/blob/dev/docs/backend/decisions/BD-28-readiness-includes-db.md), [공용 Deployment template](https://github.com/Team-PinLog/infra/blob/main/charts/microservice/templates/deployment.yaml), [AI 배포 gate](https://github.com/Team-PinLog/ai/blob/dev/docs/implements/2026-07-29-dev-deployment-gates.md).

## D-15. 단일 노드 제약에서 관측·백업의 실패 도메인을 분리한다

- 문제: 노드 전체 장애 시 노드 안의 모니터링과 백업도 함께 사라진다.
- 선택: 내부 상태는 Prometheus·Alertmanager·로그 파이프라인으로 보고, 외부 가용성은 노드 밖 probe로 보완한다. DB backup은 노드 밖 사본과 복원 검증을 원칙으로 한다.
- 이유: 내부 관측만으로 자기 자신의 완전한 실패를 탐지하거나 같은 디스크 손실을 복구할 수 없다.
- 대안: 클러스터 내부 알림만 사용, 로컬 backup만 유지.
- 트레이드오프: 외부 probe·전송 경로와 정기 반출·복원 훈련의 운영 부담이 생긴다.
- 근거: [Infra 아키텍처](https://github.com/Team-PinLog/infra/blob/main/docs/architecture.md), [운영 알림](https://github.com/Team-PinLog/infra/blob/main/docs/alerting.md), [운영 런북](https://github.com/Team-PinLog/infra/blob/main/docs/runbook.md).

## 재검토 신호

- 개인 검색 스캔이 측정 가능한 병목이 되면 D-07의 ANN 대안을 재평가한다.
- AI 처리량·지연이 Scheduler polling 한계를 넘거나 다른 소비자가 이벤트를 구독해야 하면 D-05의 broker·outbox 대안을 재평가한다.
- Redis가 세션·Feed 가용성에 필수 의존성이 되면 D-14의 readiness와 영속성 계약을 함께 재평가한다.
- Image 저장소가 다중 writer나 수평 확장을 요구하면 D-11의 SQLite singleton·`Recreate` 계약을 재평가한다.
- 노드·DB가 다중화되면 D-15의 단일 실패 도메인 전제를 다시 작성한다.
