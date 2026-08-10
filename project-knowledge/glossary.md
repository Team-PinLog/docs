# 프로젝트 용어사전

제품 용어는 [공식 용어사전](../static/03_공식_용어사전.md)을 우선한다. 아래 표는 제품·구현·운영 문서를 함께 읽기 위해 관계와 책임 경계를 덧붙인 요약이다.

## 제품·도메인

| 용어 | 뜻 | 경계·주의 |
|---|---|---|
| User | 서비스 계정 주체 | 공개 프로필과 같지 않으며 내부 식별자를 타인 응답에 노출하지 않는다. |
| Place | 외부 지도에서 유래한 공용 장소 정보 | 사용자 소유 Record가 아니며 장소 검색만으로 내부 저장되지 않는다. |
| Record | 한 User와 한 Place의 연결 단위 | 활성 Context를 하나 이상 가져야 한다. 같은 사용자의 같은 Place에는 활성 Record 중복을 막는다. |
| Context | Record 안의 독립적인 불변 서술 | 수정은 기존 Context 삭제와 새 Context 생성이며 식별자가 바뀐다. 원문은 개인 데이터다. |
| Keyword | Context에서 AI가 preset으로 매핑한 표현 | 사용자 자유 태그가 아니며 원문 대신 공개·추천 특징으로 쓸 수 있다. |
| Keyword Preset | AI가 선택할 수 있는 사전 정의 vocabulary | 모델은 후보 밖 Keyword를 새로 만들지 않는다. |
| Collection | 하나 이상의 Record를 묶은 공개 그룹 | 생성과 동시에 공개되며 Context 원문은 포함하지 않는다. 표지는 없어도 유효하다. |
| CollectionRecord | Collection과 Record의 연결 | Record 원본 복제가 아니며 하나의 Record가 여러 Collection에 포함될 수 있다. |
| Shelf | 한 User가 발행한 Collection 목록이라는 화면 개념 | 별도 Shelf 생성 절차나 공개 신원 프로필이 아니다. |
| Library | 내 Shelf와 팔로우한 Shelf를 보는 개인 공간 | 공개 프로필이 아니다. |
| Follow | 특정 Shelf를 Library에서 계속 보는 관계 | User 신원 팔로우와 구분한다. |
| Feed | 공개 Collection을 익명으로 추천하는 영역 | 개인 자연어 검색 결과와 다르며 Context 원문을 노출하지 않는다. |
| AI 자연어 검색 | 본인 Context를 의미 기준으로 찾아 Record로 반환 | 외부 지도 장소 검색이나 내 지도 keyword 검색과 다르다. |
| 장소 검색 | 외부 지도 공급자에서 세상의 장소를 찾는 기능 | 개인 Record·Context 검색과 구분한다. |
| 표지 후보 | Image 서비스가 생성한 선택 전 미리보기 | Collection 저장 완료 조건이 아니다. |
| 최종 표지 | 사용자가 후보 스타일을 선택한 뒤 생성된 파일 | Frontend가 완료 경로를 Backend Collection에 등록한다. |

## AI·데이터

| 용어 | 뜻 | 경계·주의 |
|---|---|---|
| Embedding | 텍스트의 벡터 표현 | Context 원문 대신 개인 의미 검색 계산에 사용하며 profile이 맞아야 비교할 수 있다. |
| Embedding Profile | 모델·차원·거리 기준을 묶은 식별자 | Backend 요청과 AI 설정이 다르면 빈 결과로 숨기지 않고 거부한다. |
| exact cosine | 후보 전체를 cosine 거리로 정확 정렬하는 방식 | 개인 사용자 범위에 적용하며 ANN과 구분한다. |
| ANN | HNSW·IVFFlat 같은 근사 최근접 검색 | 현재 개인 검색 선택이 아니며 실제 병목이 생길 때 재평가 대상이다. |
| AI State | Context별 embedding·keyword 처리 상태 | 두 단계가 독립 상태를 가지며 부분 재개 근거가 된다. |
| PENDING | 처리 대기 | Backend가 최초 생성·운영 재처리를 시작한다. |
| PROCESSING | AI가 작업을 선점해 처리 중 | 만료된 작업만 조건부 재선점한다. |
| COMPLETED | 해당 단계 완료 | Keyword가 0개여도 완료일 수 있다. |
| FAILED | 명시적 또는 재시도 소진 종결 | 같은 Context의 일반 직접 재시작 상태가 아니다. |
| CANCELLED | 삭제·교체로 더 이상 유효하지 않은 작업 | 모델 호출·결과 저장·검색·재스캔 대상에서 제외한다. |
| 부분 재개 | 완료된 embedding을 재사용하고 남은 keyword 단계부터 진행 | 수정으로 새 Context가 된 경우 구 embedding을 승계하는 뜻은 아니다. |
| pgvector | PostgreSQL vector type과 거리 연산 확장 | DB image, extension catalog, Flyway schema가 호환돼야 한다. |
| `core` schema | Backend 소유 도메인 데이터 영역 | AI runtime의 접근 금지 경계다. |
| `ai` schema | AI 상태·embedding·keyword·preset 영역 | runtime 계산은 AI가 하지만 migration 실행은 Backend가 소유한다. |
| Flyway | 순서가 있는 DB migration 도구·history | 앱 startup 임의 DDL과 구분한다. |
| Redis | Backend의 세션·cache 저장소 | DB와 같은 readiness·영속성 의미를 자동으로 갖지 않는다. |

## Frontend·API

| 용어 | 뜻 | 경계·주의 |
|---|---|---|
| BFF | 브라우저 요청을 위한 Backend 경계 | 이 프로젝트에서는 별도 계층이 아니라 Spring 앱이 제품 API와 함께 담당한다. |
| 서버 상태 | API에서 가져온 Record·Collection·Feed 등 | Query 계층에서 관리하고 UI Context에 복사하지 않는다. |
| UI 상태 | 모달·선택·입력·확인 단계 | 서버 캐시와 분리한다. |
| response envelope | 성공 data 또는 실패 error를 감싸는 Core API 형식 | Image API의 raw JSON과 다르며 no-content 응답에는 본문이 없다. |
| single-flight refresh | 동시 인증 갱신 요청을 하나의 실행으로 묶는 방식 | 회전 refresh token 경쟁과 무한 재시도를 막는다. |
| CSRF | cookie 인증의 상태 변경 요청 위조 위험 | 서버가 준 검증 cookie와 header의 일치를 요구한다. |
| same-origin routing | Frontend와 Core 경로를 한 origin에서 경로로 분기 | 운영 주소 자체가 아니라 배포 설계 원칙으로 이해한다. |
| polling | 비동기 상태를 일정 간격으로 다시 조회 | 종료 조건과 component unmount 정리가 필요하며 무한 polling을 금지한다. |

## 배포·운영

| 용어 | 뜻 | 경계·주의 |
|---|---|---|
| immutable image | commit tag와 digest로 실행 artifact를 고정한 image | mutable latest를 사용하지 않는다. |
| private registry | 인증된 주체만 image를 읽는 저장소 | CI publish, updater 검증, cluster pull credential을 분리한다. |
| GitOps | Git의 선언을 원하는 runtime 상태의 기준으로 삼는 운영 방식 | live 직접 수정은 drift를 만들며 정상 rollback은 선언을 되돌린다. |
| Argo CD | GitOps 선언을 Kubernetes에 동기화하는 controller | 현재 sync·health 상태는 live readback 없이는 알 수 없다. |
| ApplicationSet | 디렉터리와 template으로 서비스 Application을 생성 | 서비스별 수동 Application 복제를 줄인다. |
| Helm chart | 공용 Kubernetes template과 안전 기본값 | 서비스 차이는 values로 덮어쓰며 render 회귀 검증이 필요하다. |
| startup probe | 느린 시작 동안 liveness/readiness를 유예하는 검사 | startup 성공 전 재시작 루프를 줄인다. |
| liveness probe | 프로세스를 재시작해야 하는 생존 실패 검사 | DB·외부 모델 순단을 섞지 않는다. |
| readiness probe | 새 요청을 받을 수 있는지 판단하는 검사 | 실패는 endpoint 격리이며 반드시 container restart를 뜻하지 않는다. |
| RollingUpdate | 새 Pod를 순차 투입하는 Deployment 전략 | 단일 노드·단일 DB 고가용성을 보장하지 않는다. |
| Recreate | 기존 Pod를 내린 뒤 새 Pod를 만드는 전략 | singleton writable state에서 동시 writer를 피하지만 교체 중 중단을 감수한다. |
| resource request | scheduler가 예약 판단에 쓰는 자원량 | 실제 사용량·limit와 다르다. |
| resource limit | container 사용 상한 | 너무 낮으면 OOM·throttle, 너무 높으면 노드 생존 여유를 줄일 수 있다. |
| ServiceMonitor | Prometheus가 서비스 metric endpoint를 발견하게 하는 선언 | 앱이 실제 metric을 제공할 때만 켠다. |
| Alertmanager | metric alert grouping·routing·repeat 관리 | 원인 분석 자체보다 전달 정책을 담당한다. |
| Sentinel Receiver | alert payload를 안전한 운영 메시지로 가공하는 host 서비스 | 자동 remediation을 하지 않고 redaction·fallback·형식을 강제한다. |
| SealedSecret | 저장소에 둘 수 있도록 공개키로 봉인된 Kubernetes Secret 선언 | 평문 Secret이 아니며 controller 복구 키의 별도 backup이 필요하다. |
| backup | 복구를 위한 데이터 사본 생성 | job 성공만으로 충분하지 않고 archive·복원 검증이 필요하다. |
| restore drill | 격리 환경에 backup을 실제 복원해 유효성을 확인 | live DB 덮어쓰기와 구분한다. |
| drift | Git 선언과 runtime 상태의 차이 | 임시 live 변경을 영구 해결책으로 쓰지 않는 이유다. |
| rollback | 이전의 검증된 원하는 상태로 되돌리는 절차 | runtime undo만이 아니라 Git·Argo·probe·외부 smoke까지 확인한다. |
| provenance | source commit, CI run, image digest, 배포 선언의 연결 증거 | tag 이름만으로 artifact 출처를 추정하지 않는다. |

## 협업·문서화

| 용어 | 뜻 | 경계·주의 |
|---|---|---|
| Cowork | 자연어 입력에서 팀 Task를 생성하는 내부 도구 | 제품 도메인 서비스나 Jira 대체물이 아니다. |
| idempotency key | 중복 네트워크 요청을 같은 작업으로 식별하는 값 | 결과가 불명확한 외부 side effect를 자동 반복하지 않게 한다. |

## 폐기·혼동 금지 표현

- `Memory`를 공식 기록 단위로 쓰지 않는다. 공식 단위는 Record와 Context다.
- 장소 검색, 개인 지도 검색, AI 자연어 검색을 하나의 “검색 API”로 뭉뚱그리지 않는다.
- `Running`, `Ready`, `Healthy`, `Synced`를 같은 상태로 취급하지 않는다.
- Keyword 빈 배열을 AI 실패와 동일시하지 않는다.
- 표지 없음과 Collection 생성 실패를 동일시하지 않는다.
- 당시 측정값이나 과거 작업 완료 기록을 현재 live 상태로 표현하지 않는다.
