_한국어로 기계번역됨_

# LangGraph의 셀프 호스팅 배포 방법

!!! 정보 "사전 요구 사항"

    - [애플리케이션 구조](../concepts/application_structure.md)
    - [배포 옵션](../concepts/deployment_options.md)

이 가이드는 기존 LangGraph 애플리케이션에서 도커 이미지를 생성하여 자신만의 인프라에 배포하는 방법을 안내합니다.

## 작동 방식

셀프 호스팅 배포 옵션을 사용하면 필요한 데이터베이스, Redis 인스턴스 및 기타 서비스를 설정하고 유지管理하는 책임이 있습니다.

다음 작업을 수행해야 합니다:

1. 자신의 인프라에 Redis 및 Postgres 인스턴스를 배포합니다.
2. [LangGraph CLI](../concepts/langgraph_cli.md)를 사용하여 [LangGraph 서버](../concepts/langgraph_server.md)로 도커 이미지를 빌드합니다.
3. 도커 이미지를 실행하고 필요한 환경 변수를 전달할 웹 서버를 배포합니다.

## Helm 차트

Kubernetes에 LangGraph Cloud를 배포하려면 [Helm 차트](https://github.com/langchain-ai/helm/blob/main/charts/langgraph-cloud/README.md)를 사용할 수 있습니다.

## 환경 변수

최종적으로 LangGraph Deploy 서버에 다음 환경 변수를 전달해야 합니다:

- `REDIS_URI`: Redis 인스턴스의 연결 세부 정보입니다. Redis는 백그라운드 실행에서 실시간 출력을 스트리밍할 수 있도록 게시-구독 브로커로 사용됩니다. `REDIS_URI`의 값은 유효한 [Redis 연결 URI](https://redis-py.readthedocs.io/en/stable/connections.html#redis.Redis.from_url)여야 합니다.

    !!! 주의 "공유 Redis 인스턴스"
        여러 개의 셀프 호스팅 배포가 동일한 Redis 인스턴스를 공유할 수 있습니다. 예를 들어, `배포 A`의 경우 `REDIS_URI`를 `redis://<hostname_1>:<port>/1`로 설정하고, `배포 B`의 경우 `REDIS_URI`를 `redis://<hostname_1>:<port>/2`로 설정할 수 있습니다.

        `1`과 `2`는 동일한 인스턴스 내에서 서로 다른 데이터베이스 번호지만 `<hostname_1>`은 공유됩니다. **별도의 배포에 대해 동일한 데이터베이스 번호를 사용할 수 없습니다**.

- `DATABASE_URI`: Postgres 연결 세부 정보입니다. Postgres는 비서, 스레드, 실행을 저장하고 스레드 상태 및 장기 기억을 유지하며, '정확히 한 번' 의미론을 가진 백그라운드 작업 큐의 상태를 관리하는 데 사용됩니다. `DATABASE_URI`의 값은 유효한 [Postgres 연결 URI](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING-URIS)여야 합니다.

    !!! 주의 "공유 Postgres 인스턴스"
        여러 개의 셀프 호스팅 배포가 동일한 Postgres 인스턴스를 공유할 수 있습니다. 예를 들어, `배포 A`의 경우 `DATABASE_URI`를 `postgres://<user>:<password>@/<database_name_1>?host=<hostname_1>`로 설정하고, `배포 B`의 경우 `DATABASE_URI`를 `postgres://<user>:<password>@/<database_name_2>?host=<hostname_1>`로 설정할 수 있습니다.

        `<database_name_1>`과 `database_name_2`는 동일한 인스턴스 내에서 서로 다른 데이터베이스지만 `<hostname_1>`은 공유됩니다. **별도의 배포에 대해 동일한 데이터베이스를 사용할 수 없습니다**.

- `LANGSMITH_API_KEY`: ([Self-Hosted Lite](../concepts/deployment_options.md#self-hosted-lite) 사용 시) LangSmith API 키입니다. 서버 시작 시 한 번만 인증하는 데 사용됩니다.
- `LANGGRAPH_CLOUD_LICENSE_KEY`: ([Self-Hosted Enterprise](../concepts/deployment_options.md#self-hosted-enterprise) 사용 시) LangGraph 플랫폼 라이선스 키입니다. 서버 시작 시 한 번만 인증하는 데 사용됩니다.
- `LANGCHAIN_ENDPOINT`: [셀프 호스팅 LangSmith](https://docs.smith.langchain.com/self_hosting) 인스턴스에 추적을 보내기 위해 `LANGCHAIN_ENDPOINT`를 셀프 호스팅 LangSmith 인스턴스의 호스트 이름으로 설정합니다.

## 도커 이미지 빌드

LangGraph 애플리케이션을 어떻게 구조화하는지 이해하기 위해 [애플리케이션 구조](../concepts/application_structure.md) 가이드를 읽어보십시오.

애플리케이션이 올바르게 구조화되어 있다면, LangGraph Deploy 서버로 도커 이미지를 빌드할 수 있습니다.

도커 이미지를 빌드하려면 먼저 CLI를 설치해야 합니다:

```shell
pip install -U langgraph-cli
```

그런 다음 다음을 사용할 수 있습니다:

```
langgraph build -t my-image
```

이 명령은 LangGraph Deploy 서버로 도커 이미지를 빌드합니다. `-t my-image`는 이미지에 이름을 태그하는 데 사용됩니다.

서버를 실행할 때는 세 가지 환경 변수를 전달해야 합니다:

## 애플리케이션을 로컬에서 실행하기

### 도커 사용

```shell
docker run \
    --env-file .env \
    -p 8123:8000 \
    -e REDIS_URI="foo" \
    -e DATABASE_URI="bar" \
    -e LANGSMITH_API_KEY="baz" \
    my-image
```

별도의 Redis 및 Postgres 인스턴스를 설정하지 않고 빠르게 실행하고 싶다면 이 도커 컴포즈 파일을 사용할 수 있습니다.

!!! 주의

    * `my-image`를 이전 단계에서 빌드한 이미지의 이름으로 교체해야 합니다 (`langgraph build`에서).
    * `REDIS_URI`, `DATABASE_URI` 및 `LANGSMITH_API_KEY`에 대해 적절한 값을 제공해야 합니다.
    * 애플리케이션에서 추가적인 환경 변수가 필요한 경우 비슷한 방식으로 전달할 수 있습니다.
    * [Self-Hosted Enterprise](../concepts/deployment_options.md#self-hosted-enterprise)를 사용하는 경우 `LANGGRAPH_CLOUD_LICENSE_KEY`를 추가 환경 변수로 제공해야 합니다.

### 도커 컴포즈 사용

```yml
볼륨:
    langgraph-data:
        드라이버: local
서비스:
    langgraph-redis:
        이미지: redis:6
        헬스체크:
            테스트: redis-cli ping
            간격: 5s
            시간 초과: 1s
            재시도: 5
    langgraph-postgres:
        이미지: postgres:16
        포트:
            - "5433:5432"
        환경:
            POSTGRES_DB: postgres
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: postgres
        볼륨:
            - langgraph-data:/var/lib/postgresql/data
        헬스체크:
            테스트: pg_isready -U postgres
            시작 기간: 10s
            시간 초과: 1s
            재시도: 5
            간격: 5s
    langgraph-api:
        이미지: ${IMAGE_NAME}
        포트:
            - "8123:8000"
        의존성:
            langgraph-redis:
                조건: service_healthy
            langgraph-postgres:
                조건: service_healthy
        env_file:
            - .env
        환경:
            REDIS_URI: redis://langgraph-redis:6379
            LANGSMITH_API_KEY: ${LANGSMITH_API_KEY}
            POSTGRES_URI: postgres://postgres:postgres@langgraph-postgres:5432/postgres?sslmode=disable
```

그런 다음 이 Docker Compose 파일이 있는 동일한 폴더에서 `docker compose up`을 실행할 수 있습니다.

이것은 LangGraph Deploy을 포트 `8123`에서 시작합니다 (변경하고 싶다면 `langgraph-api` 볼륨의 포트를 변경하여 변경할 수 있습니다).

응용 프로그램이 정상적으로 실행되고 있는지 확인하려면 다음을 확인하십시오:

```shell
curl --request GET --url 0.0.0.0:8123/ok
```
모든 것이 올바르게 실행되고 있다면, 다음과 같은 응답을 볼 수 있습니다:

```shell
{"ok":true}
```

