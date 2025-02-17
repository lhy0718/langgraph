_한국어로 기계번역됨_

# 환경 변수

LangGraph 클라우드 서버는 배포 구성을 위한 특정 환경 변수를 지원합니다.

## `LANGCHAIN_TRACING_SAMPLING_RATE`

LangSmith에 전송된 추적의 샘플링 비율. 유효 값: `0`과 `1` 사이의 모든 부동 소수점 수.

자세한 내용은 <a href="https://docs.smith.langchain.com/how_to_guides/tracing/sample_traces" target="_blank">LangSmith 문서</a>를 참조하세요.

## `LANGGRAPH_AUTH_TYPE`

LangGraph 클라우드 서버 배포를 위한 인증 유형. 유효 값: `langsmith`, `noop`.

LangGraph 클라우드에 배포하는 경우 이 환경 변수는 자동으로 설정됩니다. 로컬 개발이나 인증이 외부에서 처리되는 배포(예: 자체 호스팅)에서는 이 환경 변수를 `noop`으로 설정하세요.

## `LANGSMITH_RUNS_ENDPOINTS`

[Bring Your Own Cloud (BYOC)](../../concepts/bring_your_own_cloud.md) 배포에서만 [자체 호스팅된 LangSmith](https://docs.smith.langchain.com/self_hosting) 사용 시.

이 환경 변수를 설정하면 BYOC 배포가 자체 호스팅된 LangSmith 인스턴스로 추적을 전송합니다. `LANGSMITH_RUNS_ENDPOINTS`의 값은 JSON 문자열입니다: `{"<SELF_HOSTED_LANGSMITH_HOSTNAME>":"<LANGSMITH_API_KEY>"}`.

`SELF_HOSTED_LANGSMITH_HOSTNAME`은 자체 호스팅된 LangSmith 인스턴스의 호스트 이름입니다. BYOC 배포에서 접근 가능해야 합니다. `LANGSMITH_API_KEY`는 자체 호스팅된 LangSmith 인스턴스에서 생성된 LangSmith API 키입니다.

## `N_JOBS_PER_WORKER`

LangGraph 클라우드 작업 큐용 워커별 작업 수. 기본값은 `10`입니다.

## `POSTGRES_URI_CUSTOM`

[Bring Your Own Cloud (BYOC)](../../concepts/bring_your_own_cloud.md) 배포에서만.

외부에서 관리되는 Postgres 인스턴스를 사용하려면 `POSTGRES_URI_CUSTOM`을 지정하세요. `POSTGRES_URI_CUSTOM`의 값은 유효한 [Postgres 연결 URI](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING-URIS)여야 합니다.

Postgres:

- 버전 15.8 이상.
- 초기 데이터베이스가 있어야 하며 연결 URI는 해당 데이터베이스를 참조해야 합니다.

제어Plane 기능:

- `POSTGRES_URI_CUSTOM`이 지정된 경우 LangGraph 제어Plane은 서버를 위한 데이터베이스를 프로비저닝하지 않습니다.
- `POSTGRES_URI_CUSTOM`이 제거되면 LangGraph 제어Plane은 서버를 위한 데이터베이스를 프로비저닝하지 않고 외부에서 관리되는 Postgres 인스턴스를 삭제하지 않습니다.
- `POSTGRES_URI_CUSTOM`이 제거되면 리비전 배포가 성공하지 않습니다. `POSTGRES_URI_CUSTOM`이 지정되면 배포의 생애 주기 동안 항상 설정해야 합니다.
- 배포가 삭제되면 LangGraph 제어Plane은 외부에서 관리되는 Postgres 인스턴스를 삭제하지 않습니다.
- `POSTGRES_URI_CUSTOM`의 값은 업데이트할 수 있습니다. 예를 들어, URI의 비밀번호를 업데이트할 수 있습니다.

데이터베이스 연결성:

- 외부에서 관리되는 Postgres 인스턴스는 ECS 클러스터 내 LangGraph 서버 서비스에서 접근 가능해야 합니다. BYOC 사용자가 연결성을 확보할 책임이 있습니다.
- 예를 들어 AWS RDS Postgres 인스턴스가 프로비저닝된 경우, ECS 클러스터와 동일한 VPC(`langgraph-cloud-vpc`)에 `langgraph-cloud-service-sg` 보안 그룹으로 프로비저닝하여 연결성을 보장할 수 있습니다.
