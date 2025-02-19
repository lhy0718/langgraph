_한국어로 기계번역됨_

# Kafka를 위한 LangGraph 스케줄러

이 라이브러리는 Kafka를 메시지 브로커로 사용하여 LangGraph를 위한 분산 스케줄러를 구현합니다.

## 아키텍처

![](./langgraph-distributed.png)

- Kafka(최소 한번)와 LangGraph 체크포인터의 조합은 오케스트레이터와 실행자 메시지 모두에 대해 정확히 한 번의 의미를 제공합니다.
- 체크포인터는 주어진 작업에 대해 작성이 한 번만 저장되도록 보장하며, 작업이 재실행되더라도 동일합니다.
- 체크포인터는 각 단계에서 각 작업이 Kafka에 성공적으로 게시되었는지를 기록하여 작업이 잃어버리거나 두 번 이상 게시되지 않도록 합니다.
- 오케스트레이터와 실행자는 작업이 처리 완료된 후에만 완료로 표시할 수 있도록 오프셋 커밋을 수동으로 관리합니다.
- 오케스트레이터와 실행자는 다시 시작할 때 아직 소비되지 않은 가장 이른 메시지부터 시작하여 메시지가 손실되지 않도록 하고, 메시지를 두 번 이상 처리하지 않도록 합니다.
- 오케스트레이터 메시지는 스레드 ID와 체크포인트 NS를 기준으로 키가 부여되어 동일한 스레드의 동일 단계에 대한 업데이트를 두 개의 소비자가 동시에 처리할 수 없도록 보장합니다.
- 실행자 메시지는 키가 부여되지 않으며, 병렬로 처리될 수 있습니다.
- 오케스트레이터와 실행자는 구성 가능한 배치에서 메시지를 실행하며(최대 N 메시지를 X 초 내에서), 적절할 경우 배치 내 메시지를 중복 제거합니다(이는 순수한 성능 최적화에 해당하며 적용 여부가 올바름에 영향을 미치지 않습니다).

## 기본 사용법

오케스트레이터 및 실행자 프로세스를 실행합니다:

`orchestrator.py`

```python
import asyncio
import logging
import os

from langgraph.scheduler.kafka.orchestrator import AsyncKafkaOrchestrator
from langgraph.scheduler.kafka.types import Topics

from your_lib import graph # graph는 컴파일된 LangGraph 그래프여야 합니다.

logger = logging.getLogger(__name__)

topics = Topics(
    orchestrator=os.environ['KAFKA_TOPIC_ORCHESTRATOR'],
    executor=os.environ['KAFKA_TOPIC_EXECUTOR'],
    error=os.environ['KAFKA_TOPIC_ERROR'],
)

async def main():
    async with AsyncKafkaOrchestrator(graph, topics) as orch:
        async for msgs in orch:
            logger.info('처리된 메시지 수: %d', len(msgs))

if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    asyncio.run(main())
```

`executor.py`

```python
import asyncio
import logging
import os

from langgraph.scheduler.kafka.executor import AsyncKafkaExecutor
from langgraph.scheduler.kafka.types import Topics

from your_lib import graph # graph는 컴파일된 LangGraph 그래프여야 합니다.

logger = logging.getLogger(__name__)

topics = Topics(
    orchestrator=os.environ['KAFKA_TOPIC_ORCHESTRATOR'],
    executor=os.environ['KAFKA_TOPIC_EXECUTOR'],
    error=os.environ['KAFKA_TOPIC_ERROR'],
)

async def main():
    async with AsyncKafkaExecutor(graph, topics) as orch:
        async for msgs in orch:
            logger.info('처리된 메시지 수: %d', len(msgs))

if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    asyncio.run(main())
```

```bash
export KAFKA_TOPIC_ORCHESTRATOR='orchestrator'
export KAFKA_TOPIC_EXECUTOR='executor'
export KAFKA_TOPIC_ERROR='error'
python orchestrator.py &
python executor.py &
```

## 구성

우리는 오케스트레이터 및 실행자의 동기 및 비동기 버전인 `KafkaOrchestrator` 및 `AsyncKafkaOrchestrator`, `KafkaExecutor` 및 `AsyncKafkaExecutor`를 제공합니다. 비동기 버전이 추천되며, 특히 배치로 작업을 처리하려는 경우 좋습니다. 비동기 클래스와 함께 `uvloop`를 사용하여 성능을 향상시키는 것도 권장됩니다.

다음 값 중 하나를 `kwargs`로 `KafkaOrchestrator` 또는 `KafkaExecutor`에 전달하여 소비자를 구성할 수 있습니다:

- batch_max_n (int): 단일 배치에 포함할 최대 메시지 수. 기본값: 10.
- batch_max_ms (int): 배치에 포함하기 위해 메시지를 기다릴 최대 시간(밀리초). 기본값: 1000.
- retry_policy (langgraph.types.RetryPolicy): 메시지를 처리할 때 재시도할 그래프 수준 오류를 제어합니다. 이 경우 체크포인터에서 발생하는 데이터베이스 오류를 재시도하는 것이 좋습니다. 기본값은 None입니다.

### 연결 설정

기본적으로 오케스트레이터와 실행자는 `localhost:9092`에서 실행 중인 Kafka 브로커에 연결을 시도합니다. 다음 값 중 하나를 `KafkaOrchestrator` 또는 `KafkaExecutor`에 `kwargs`로 전달하여 연결 설정을 변경할 수 있습니다:

- bootstrap_servers: 소비자가 초기 클러스터 메타데이터를 부트스트랩하기 위해 연락해야 하는 'host[:port]' 문자열(또는 'host[:port]' 문자열의 목록). 전체 노드 목록일 필요는 없습니다. 메타데이터 API 요청에 응답할 수 있는 브로커가 하나 이상 있어야 합니다. 기본 포트는 9092입니다. 서버가 지정되지 않으면 기본값으로 localhost:9092로 설정됩니다.
- client_id (str): 이 클라이언트의 이름. 이 문자열은 각 요청에 서버로 전달되며, 이 클라이언트에 해당하는 특정 서버 측 로그 항목을 식별하는 데 사용할 수 있습니다. 소비자 그룹 관리와 관련하여 GroupCoordinator에 로깅을 위해 제출됩니다. 기본값: 'aiokafka-{ver}'
- request_timeout_ms (int): 밀리초 단위의 클라이언트 요청 타임아웃. 기본값: 40000.
- metadata_max_age_ms (int): 파티션 리더십 변경 사항을 보지 않은 경우에도 메타데이터를 강제로 새로 고치는 시간을 밀리초 단위로 지정합니다. 새로운 브로커나 파티션을 능동적으로 발견하기 위함입니다. 기본값: 300000.
- retry_backoff_ms (int): 오류 발생 시 재시도를 위한 대기 시간을 밀리초로 지정합니다. 기본값: 100.
- api_version (str): 사용할 Kafka API 버전을 지정합니다. AIOKafka는 Kafka API 버전 >=0.9만 지원합니다. 'auto'로 설정하면 다양한 API를 탐색하여 브로커 버전을 유추하려고 시도합니다. 기본값: auto.
- security_protocol (str): 브로커와 통신하는 데 사용되는 프로토콜입니다. 유효한 값은: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL입니다. 기본값: PLAINTEXT.
- ssl_context (ssl.SSLContext): 소켓 연결을 래핑하기 위해 사전 구성된 SSLContext입니다. 자세한 내용은 :ref:`ssl_auth`를 참조하십시오. 기본값: None.
- connections_max_idle_ms (int): 이 구성에서 지정된 밀리초 후에 유휴 연결을 닫습니다. `None`을 지정하면 유휴 검사 기능이 비활성화됩니다. 기본값: 540000 (9분).

### 사용자 정의 소비자/프로듀서

오케스트레이터와 실행자는 각각 `Consumer` 또는 `Producer` 프로토콜을 구현해야 하는 `consumer` 및 `producer` 인수를 허용합니다. 소비자는 자동 커밋이 비활성화되어야 하며, 프로듀서와 소비자는 직렬 변환기/역직렬 변환기가 설정되지 않아야 합니다.
