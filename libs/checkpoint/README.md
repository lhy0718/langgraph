_한국어로 기계번역됨_

# LangGraph 체크포인트

이 라이브러리는 LangGraph 체크포인터의 기본 인터페이스를 정의합니다. 체크포인터는 LangGraph를 위한 지속성 레이어를 제공합니다. 이들은 그래프의 상태와 상호작용하고 이를 관리할 수 있게 해줍니다. 체크포인터가 있는 그래프를 사용할 때, 체크포인터는 매 슈퍼스텝에서 그래프 상태의 _체크포인트_를 저장하여, 인간 개입, 상호작용 간의 "메모리" 등 여러 강력한 기능을 가능하게 합니다.

## 주요 개념

### 체크포인트

체크포인트는 특정 시점의 그래프 상태를 스냅샷한 것입니다. 체크포인트 튜플은 체크포인트와 관련된 구성, 메타데이터, 대기 중인 쓰기를 포함하는 객체를 의미합니다.

### 스레드

스레드는 여러 가지 다른 실행을 체크포인트 할 수 있게 해주며, 별도의 상태를 유지해야 하는 멀티 테넌트 채팅 애플리케이션 및 기타 시나리오에서 필수적입니다. 스레드는 체크포인터가 저장한 일련의 체크포인트에 할당된 고유한 ID입니다. 체크포인터를 사용할 때 `thread_id`와 선택적으로 `checkpoint_id`를 지정해야 합니다.

- `thread_id`는 스레드의 ID입니다. 이는 항상 필요합니다.
- `checkpoint_id`는 선택적으로 전달될 수 있습니다. 이 식별자는 스레드 내의 특정 체크포인트를 나타냅니다. 이는 스레드 중간의 어떤 지점에서 그래프 실행을 시작하는 데 사용할 수 있습니다.

이들은 구성의 구성 가능 부분의 일환으로 그래프를 호출할 때 전달해야 합니다. 예를 들어:

```python
{"configurable": {"thread_id": "1"}}  # 유효한 구성
{"configurable": {"thread_id": "1", "checkpoint_id": "0c62ca34-ac19-445d-bbb0-5b4984975b2a"}}  # 또한 유효한 구성
```

### 직렬화/역직렬화(Serde)

`langgraph_checkpoint`는 또한 직렬화/역직렬화(serde) 프로토콜을 정의하고 LangChain 및 LangGraph 원시값, 날짜 시간, 열거형 등을 포함한 다양한 유형을 처리하는 기본 구현(`langgraph.checkpoint.serde.jsonplus.JsonPlusSerializer`)을 제공합니다.

### 대기 중인 쓰기

그래프 노드가 주어진 슈퍼스텝에서 실행 중에 실패하면, LangGraph는 해당 슈퍼스텝에서 성공적으로 완료된 다른 노드의 대기 중인 체크포인트 쓰기를 저장하여, 해당 슈퍼스텝부터 그래프 실행을 재개할 때 성공적인 노드를 다시 실행하지 않도록 합니다.

## 인터페이스

각 체크포인터는 `langgraph.checkpoint.base.BaseCheckpointSaver` 인터페이스를 준수해야 하며, 다음과 같은 메서드를 구현해야 합니다:

- `.put` - 체크포인트와 그 구성 및 메타데이터를 저장합니다.
- `.put_writes` - 체크포인트와 연결된 중간 쓰기(즉, 대기 중인 쓰기)를 저장합니다.
- `.get_tuple` - 주어진 구성(`thread_id` 및 `thread_ts`)에 대해 체크포인트 튜플을 가져옵니다.
- `.list` - 주어진 구성과 필터 기준에 맞는 체크포인트를 나열합니다.

비동기 그래프 실행(즉, `.ainvoke`, `.astream`, `.abatch`를 통한 그래프 실행)과 함께 체크포인터를 사용할 경우, 체크포인터는 위의 메서드의 비동기 버전(`.aput`, `.aput_writes`, `.aget_tuple`, `.alist`)을 구현해야 합니다.

## 사용 예시

```python
from langgraph.checkpoint.memory import MemorySaver

write_config = {"configurable": {"thread_id": "1", "checkpoint_ns": ""}}
read_config = {"configurable": {"thread_id": "1"}}

checkpointer = MemorySaver()
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

# 체크포인트 목록 나열
list(checkpointer.list(read_config))
```
