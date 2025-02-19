_한국어로 기계번역됨_

# LangGraph 체크포인트 Postgres

Postgres를 사용하는 LangGraph CheckpointSaver의 구현.

## 의존성

기본적으로 `langgraph-checkpoint-postgres`는 추가 기능 없이 `psycopg`(Psycopg 3)를 설치합니다. 그러나 필요에 맞는 특정 설치를 선택할 수 있습니다 [여기](https://www.psycopg.org/psycopg3/docs/basic/install.html)에서 (예: `psycopg[binary]`).

## 사용법

> [!중요]
> Postgres 체크포인터를 처음 사용할 때는 필요한 테이블을 생성하기 위해 `.setup()` 메서드를 호출해야 합니다. 아래 예제를 참조하세요.

> [!중요]
> Postgres 연결을 수동으로 생성하고 이를 `PostgresSaver` 또는 `AsyncPostgresSaver`에 전달할 때는 `autocommit=True`와 `row_factory=dict_row` (`from psycopg.rows import dict_row`)를 포함해야 합니다. 전체 예제는 이 [사용 가이드](https://langchain-ai.github.io/langgraph/how-tos/persistence_postgres/)에서 확인하세요.

```python
from langgraph.checkpoint.postgres import PostgresSaver

write_config = {"configurable": {"thread_id": "1", "checkpoint_ns": ""}}
read_config = {"configurable": {"thread_id": "1"}}

DB_URI = "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable"
with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    # 체크포인터를 처음 사용할 때 .setup()을 호출합니다.
    checkpointer.setup()
    checkpoint = {
        "v": 1,
        "ts": "2024-07-31T20:14:19.804150+00:00",
        "id": "1ef4f797-8335-6428-8001-8a1503f9b875",
        "channel_values": {
            "my_key": "meow",
            "node": "node"
        },
        "channel_versions": {
            "__start__": 2,
            "my_key": 3,
            "start:node": 3,
            "node": 3
        },
        "versions_seen": {
            "__input__": {},
            "__start__": {
            "__start__": 1
            },
            "node": {
            "start:node": 2
            }
        },
        "pending_sends": [],
    }

    # 체크포인트 저장
    checkpointer.put(write_config, checkpoint, {}, {})

    # 체크포인트 로드
    checkpointer.get(read_config)

    # 체크포인트 목록
    list(checkpointer.list(read_config))
```

### 비동기

```python
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

async with AsyncPostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpoint = {
        "v": 1,
        "ts": "2024-07-31T20:14:19.804150+00:00",
        "id": "1ef4f797-8335-6428-8001-8a1503f9b875",
        "channel_values": {
            "my_key": "미야옹",
            "node": "노드"
        },
        "channel_versions": {
            "__start__": 2,
            "my_key": 3,
            "start:node": 3,
            "node": 3
        },
        "versions_seen": {
            "__input__": {},
            "__start__": {
            "__start__": 1
            },
            "node": {
            "start:node": 2
            }
        },
        "pending_sends": [],
    }

    # 체크포인트 저장
    await checkpointer.aput(write_config, checkpoint, {}, {})

    # 체크포인트 로드
    await checkpointer.aget(read_config)

    # 체크포인트 목록
    [c async for c in checkpointer.alist(read_config)]
```
