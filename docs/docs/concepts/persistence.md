_한국어로 기계번역됨_

## 지속성

LangGraph는 내장된 지속성 계층을 제공하며, 이는 체크포인터를 통해 구현됩니다. 그래프를 체크포인터와 함께 컴파일하면, 체크포인트는 각 슈퍼 단계에서 그래프 상태의 `checkpoint`를 저장합니다. 이 체크포인트는 `thread`에 저장되며, 그래프 실행 후에 접근할 수 있습니다. `threads`를 통해 그래프의 실행 후 상태에 접근할 수 있기 때문에, 인간 개입, 메모리, 시간 여행, 내결함성 등 여러 강력한 기능을 지원할 수 있습니다. 체크포인트를 그래프에 추가하고 사용하는 방법에 대한 전체 예시는 [이 가이드](../how-tos/persistence.ipynb)에서 확인할 수 있습니다. 아래에서는 각 개념을 더 자세히 설명합니다.

![체크포인트](img/persistence/checkpoints.jpg)

## 스레드

스레드는 체크포인트가 저장한 각 체크포인트에 할당된 고유한 ID 또는 [스레드 식별자](#threads)입니다. 체크포인트를 사용해 그래프를 호출할 때는 **반드시** `thread_id`를 구성 요소의 일부로 지정해야 합니다:

```python
{"configurable": {"thread_id": "1"}}
```

## 체크포인트

체크포인트는 각 슈퍼 단계에서 저장된 그래프 상태의 스냅샷으로, 다음과 같은 주요 속성들을 포함하는 `StateSnapshot` 객체로 표현됩니다:

- `config`: 이 체크포인트와 관련된 구성
- `metadata`: 이 체크포인트와 관련된 메타데이터
- `values`: 이 시점에서 상태 채널의 값
- `next`: 그래프에서 다음에 실행할 노드들의 튜플
- `tasks`: 실행할 다음 작업들에 대한 정보를 담고 있는 `PregelTask` 객체들의 튜플입니다. 이전에 시도된 경우 오류 정보도 포함됩니다. 그래프가 [동적으로](../how-tos/human_in_the_loop/dynamic_breakpoints.ipynb) 노드 내에서 중단된 경우, `tasks`에는 중단과 관련된 추가 데이터가 포함됩니다.

간단한 그래프가 호출될 때 저장되는 체크포인트를 살펴보겠습니다:

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from typing import Annotated
from typing_extensions import TypedDict
from operator import add

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]

def node_a(state: State):
    return {"foo": "a", "bar": ["a"]}

def node_b(state: State):
    return {"foo": "b", "bar": ["b"]}


workflow = StateGraph(State)
workflow.add_node(node_a)
workflow.add_node(node_b)
workflow.add_edge(START, "node_a")
workflow.add_edge("node_a", "node_b")
workflow.add_edge("node_b", END)

checkpointer = MemorySaver()
graph = workflow.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "1"}}
graph.invoke({"foo": ""}, config)
```

그래프를 실행한 후, 우리는 총 4개의 체크포인트를 보게 됩니다:

- `START`가 다음 실행 노드로 지정된 빈 체크포인트
- 사용자 입력 `{'foo': '', 'bar': []}`와 `node_a`가 다음 실행 노드로 지정된 체크포인트
- `node_a`의 출력 `{'foo': 'a', 'bar': ['a']}`와 `node_b`가 다음 실행 노드로 지정된 체크포인트
- `node_b`의 출력 `{'foo': 'b', 'bar': ['a', 'b']}`와 더 이상 실행할 노드가 없는 체크포인트

`bar` 채널 값에는 `bar`에 대한 리듀서가 있기 때문에 두 노드에서 반환된 값이 포함됩니다.

### 상태 가져오기

저장된 그래프 상태와 상호작용할 때는 **반드시** [스레드 식별자](#threads)를 지정해야 합니다. `graph.get_state(config)`를 호출하여 그래프의 _최신_ 상태를 볼 수 있습니다. 이는 제공된 스레드 ID와 연결된 최신 체크포인트 또는 제공된 체크포인트 ID에 연결된 체크포인트와 일치하는 `StateSnapshot` 객체를 반환합니다.

```python
# 최신 상태 스냅샷 가져오기
config = {"configurable": {"thread_id": "1"}}
graph.get_state(config)

# 특정 checkpoint_id에 대한 상태 스냅샷 가져오기
config = {"configurable": {"thread_id": "1", "checkpoint_id": "1ef663ba-28fe-6528-8002-5a559208592c"}}
graph.get_state(config)
```

위 예제에서 `get_state`의 출력은 다음과 같습니다:

```
StateSnapshot(
    values={'foo': 'b', 'bar': ['a', 'b']},
    next=(),
    config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28fe-6528-8002-5a559208592c'}},
    metadata={'source': 'loop', 'writes': {'node_b': {'foo': 'b', 'bar': ['b']}}, 'step': 2},
    created_at='2024-08-29T19:19:38.821749+00:00',
    parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f9-6ec4-8001-31981c2c39f8'}}, tasks=()
)
```

### 상태 기록 가져오기

`graph.get_state_history(config)`를 호출하면 주어진 스레드에 대한 전체 그래프 실행 기록을 가져올 수 있습니다. 이 메서드는 제공된 스레드 ID와 관련된 `StateSnapshot` 객체들의 목록을 반환합니다. 중요한 점은 체크포인트가 시간 순으로 정렬되어 있으며, 가장 최근의 체크포인트/`StateSnapshot`이 목록의 첫 번째에 있다는 것입니다.

```python
config = {"configurable": {"thread_id": "1"}}
list(graph.get_state_history(config))
```

예제에서 `get_state_history`의 출력은 다음과 같습니다:

```
[
    StateSnapshot(
        values={'foo': 'b', 'bar': ['a', 'b']},
        next=(),
        config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28fe-6528-8002-5a559208592c'}},
        metadata={'source': 'loop', 'writes': {'node_b': {'foo': 'b', 'bar': ['b']}}, 'step': 2},
        created_at='2024-08-29T19:19:38.821749+00:00',
        parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f9-6ec4-8001-31981c2c39f8'}},
        tasks=(),
    ),
    StateSnapshot(
        values={'foo': 'a', 'bar': ['a']}, next=('node_b',),
        config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f9-6ec4-8001-31981c2c39f8'}},
        metadata={'source': 'loop', 'writes': {'node_a': {'foo': 'a', 'bar': ['a']}}, 'step': 1},
        created_at='2024-08-29T19:19:38.819946+00:00',
        parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f4-6b4a-8000-ca575a13d36a'}},
        tasks=(PregelTask(id='6fb7314f-f114-5413-a1f3-d37dfe98ff44', name='node_b', error=None, interrupts=()),),
    ),
    StateSnapshot(
        values={'foo': '', 'bar': []},
        next=('node_a',),
        config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f4-6b4a-8000-ca575a13d36a'}},
        metadata={'source': 'loop', 'writes': None, 'step': 0},
        created_at='2024-08-29T19:19:38.817813+00:00',
        parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f0-6c66-bfff-6723431e8481'}},
        tasks=(PregelTask(id='f1b14528-5ee5-579c-949b-23ef9bfbed58', name='node_a', error=None, interrupts=()),),
    ),
    StateSnapshot(
        values={'bar': []},
        next=('__start__',),
        config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1ef663ba-28f0-6c66-bfff-6723431e8481'}},
        metadata={'source': 'input', 'writes': {'foo': ''}, 'step': -1},
        created_at='2024-08-29T19:19:38.816205+00:00',
        parent_config=None,
        tasks=(PregelTask(id='6d27aa2e-d72b-5504-a36f-8620e54a76dd', name='__start__', error=None, interrupts=()),),
    )
]
```

![상태](img/persistence/get_state.jpg)

### 재생

이전 그래프 실행을 재생할 수도 있습니다. 만약 `thread_id`와 `checkpoint_id`를 사용하여 그래프를 `invoke`하면, `checkpoint_id`에 해당하는 체크포인트 이전에 실행된 단계를 *재생*하고, 체크포인트 이후의 단계만 실행합니다.

- `thread_id`: 스레드의 ID입니다.
- `checkpoint_id`: 스레드 내 특정 체크포인트를 나타내는 식별자입니다.

이들을 그래프 호출 시 `configurable` 부분에 포함하여 전달해야 합니다:

```python
config = {"configurable": {"thread_id": "1", "checkpoint_id": "0c62ca34-ac19-445d-bbb0-5b4984975b2a"}}
graph.invoke(None, config=config)
```

중요한 점은 LangGraph가 특정 단계가 이전에 실행되었는지 여부를 인식한다는 것입니다. 만약 실행되었다면, LangGraph는 해당 단계를 단순히 *재생*하며, 제공된 `checkpoint_id` 이전의 단계만 다시 실행하지 않습니다. `checkpoint_id` 이후의 모든 단계는 다시 실행됩니다 (즉, 새로운 분기).

[시간 여행을 배우는 방법](../how-tos/human_in_the_loop/time-travel.ipynb)에 대한 가이드를 참조하여 재생에 대해 더 알아보세요.

![재생](img/persistence/re_play.png)

### 상태 업데이트

특정 `checkpoints`에서 그래프를 재생하는 것 외에도, 그래프 상태를 *편집*할 수도 있습니다. 이를 위해 `graph.update_state()`를 사용합니다. 이 메서드는 세 가지 인자를 받습니다:

#### `config`

구성은 업데이트할 스레드의 `thread_id`를 포함해야 합니다. `thread_id`만 전달하면 현재 상태를 업데이트(또는 분기)합니다. 선택적으로 `checkpoint_id` 필드를 포함하면 선택한 체크포인트에서 분기할 수 있습니다.

#### `values`

이 값들은 상태를 업데이트하는 데 사용됩니다. 이 업데이트는 노드에서 발생한 업데이트와 동일하게 처리됩니다. 즉, 이 값들은 그래프 상태의 일부 채널에 대해 정의된 [리듀서](./low_level.md#reducers) 함수에 전달됩니다. 이 말은 `update_state`가 모든 채널 값을 자동으로 덮어쓰지 않고, 리듀서가 없는 채널만 덮어쓴다는 뜻입니다. 예제를 살펴봅시다.

그래프 상태를 다음과 같은 스키마로 정의했다고 가정해봅시다 (위의 전체 예제 참조):

```python
from typing import Annotated
from typing_extensions import TypedDict
from operator import add

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]
```

현재 그래프 상태가 다음과 같다고 가정합시다:

```
{"foo": 1, "bar": ["a"]}
```

다음과 같이 상태를 업데이트한다고 가정합시다:

```
graph.update_state(config, {"foo": 2, "bar": ["b"]})
```

그렇다면 그래프의 새 상태는 다음과 같습니다:

```
{"foo": 2, "bar": ["a", "b"]}
```

`foo` 키(채널)는 완전히 변경됩니다 (이 채널에 대해 리듀서가 정의되지 않았으므로 `update_state`가 이를 덮어씁니다). 그러나 `bar` 키에는 리듀서가 정의되어 있으므로 `"b"`가 `bar` 상태에 추가됩니다.

#### `as_node`

`update_state`를 호출할 때 선택적으로 지정할 수 있는 마지막 항목은 `as_node`입니다. 이를 제공하면, 업데이트는 마치 `as_node`에서 온 것처럼 적용됩니다. `as_node`가 제공되지 않으면, 상태를 마지막으로 업데이트한 노드가 설정됩니다(모호하지 않은 경우). 이는 그래프에서 다음에 실행할 단계가 마지막으로 상태를 업데이트한 노드에 의존하므로, 어떤 노드가 다음에 실행될지 제어하는 데 사용할 수 있습니다. 이 기능에 대해 더 알고 싶다면 [시간 여행 가이드](../how-tos/human_in_the_loop/time-travel.ipynb)를 참고하세요.

![업데이트](img/persistence/checkpoints_full_story.jpg)

## 메모리 저장소

![공유 상태 모델](img/persistence/shared_state.png)

[상태 스키마](low_level.md#schema)는 그래프가 실행될 때 채워지는 키 세트를 지정합니다. 앞서 언급했듯이, 상태는 체크포인터가 그래프 단계마다 스레드에 기록하여 상태 지속성을 가능하게 합니다.

하지만 _스레드 간_ 정보를 유지하려면 어떻게 해야 할까요? 예를 들어, 사용자와의 모든 채팅 대화(즉, 스레드)에서 특정 정보를 유지하고 싶은 챗봇을 고려해보세요!

체크포인터만으로는 스레드 간에 정보를 공유할 수 없습니다. 이로 인해 [`Store`](../reference/store.md#langgraph.store.base.BaseStore) 인터페이스의 필요성이 생깁니다. 예를 들어, 사용자의 정보를 여러 스레드에 걸쳐 저장할 수 있는 `InMemoryStore`를 정의할 수 있습니다. 우리는 그래프를 체크포인터와 함께 컴파일하고 새로운 `in_memory_store` 변수를 사용하여 이를 할 수 있습니다.

### 기본 사용법

먼저, LangGraph를 사용하지 않고 이를 독립적으로 보여줍니다.

```python
from langgraph.store.memory import InMemoryStore
in_memory_store = InMemoryStore()
```

메모리는 `tuple`로 네임스페이스가 지정됩니다. 이 예시에서는 `(<user_id>, "memories")`가 사용됩니다. 네임스페이스는 길이가 다를 수 있으며, 사용자 특정일 필요는 없습니다.

```python
user_id = "1"
namespace_for_memory = (user_id, "memories")
```

`store.put` 메서드를 사용하여 메모리를 네임스페이스에 저장합니다. 이때 네임스페이스와 메모리의 키-값 쌍을 지정합니다. 키는 메모리의 고유 식별자(`memory_id`)이고 값은 메모리 자체를 나타내는 딕셔너리입니다.

```python
memory_id = str(uuid.uuid4())
memory = {"food_preference" : "I like pizza"}
in_memory_store.put(namespace_for_memory, memory_id, memory)
```

`store.search` 메서드를 사용하여 네임스페이스에서 메모리를 읽을 수 있으며, 주어진 사용자의 모든 메모리를 목록으로 반환합니다. 가장 최근 메모리가 목록의 마지막에 있습니다.

```python
memories = in_memory_store.search(namespace_for_memory)
memories[-1].dict()
{'value': {'food_preference': 'I like pizza'},
 'key': '07e0caf4-1631-47b7-b15f-65515d4c1843',
 'namespace': ['1', 'memories'],
 'created_at': '2024-10-02T17:22:31.590602+00:00',
 'updated_at': '2024-10-02T17:22:31.590605+00:00'}
```

각 메모리 유형은 특정 속성을 가진 파이썬 클래스([`Item`](https://langchain-ai.github.io/langgraph/reference/store/#langgraph.store.base.Item))입니다. `.dict`로 변환하여 딕셔너리 형태로 접근할 수 있습니다. 이 클래스가 가진 속성은 다음과 같습니다:

- `value`: 이 메모리의 값(딕셔너리)
- `key`: 이 네임스페이스에서 메모리의 고유 키
- `namespace`: 이 메모리 유형의 네임스페이스 목록
- `created_at`: 이 메모리가 생성된 타임스탬프
- `updated_at`: 이 메모리가 업데이트된 타임스탬프

### 의미 기반 검색

단순 검색을 넘어서, 저장소는 의미 기반 검색도 지원하여, 정확한 일치가 아닌 의미를 기준으로 메모리를 찾을 수 있습니다. 이를 활성화하려면, 저장소를 임베딩 모델로 구성해야 합니다:

```python
from langchain.embeddings import init_embeddings

store = InMemoryStore(
    index={
        "embed": init_embeddings("openai:text-embedding-3-small"),  # 임베딩 제공자
        "dims": 1536,                              # 임베딩 차원
        "fields": ["food_preference", "$"]              # 임베딩할 필드
    }
)
```

이제 검색할 때 자연어 쿼리를 사용하여 관련 메모리를 찾을 수 있습니다:

```python
# 음식 선호에 대한 메모리 찾기
# (메모리가 저장된 후 검색 가능합니다)
memories = store.search(
    namespace_for_memory,
    query="사용자는 무엇을 먹는 것을 좋아합니까?",
    limit=3  # 상위 3개 일치 항목 반환
)
```

메모리에서 어떤 부분을 임베딩할지 제어하려면 `fields` 매개변수를 설정하거나 메모리를 저장할 때 `index` 매개변수를 지정할 수 있습니다:

```python
# 특정 필드를 임베딩하여 저장
store.put(
    namespace_for_memory,
    str(uuid.uuid4()),
    {
        "food_preference": "나는 이탈리안 요리를 좋아합니다",
        "context": "저녁 계획에 대해 논의 중"
    },
    index=["food_preference"]  # "food_preference" 필드만 임베딩
)

# 임베딩 없이 저장 (여전히 검색 가능하지만 검색은 불가능)
store.put(
    namespace_for_memory,
    str(uuid.uuid4()),
    {"system_info": "마지막 업데이트: 2024-01-01"},
    index=False
)
```

### LangGraph에서 사용하기

모든 준비가 끝났으면, 이제 `in_memory_store`를 LangGraph에서 사용합니다. `in_memory_store`는 체크포인터와 함께 작동합니다: 체크포인터는 위에서 설명한 대로 상태를 스레드에 저장하고, `in_memory_store`는 스레드 간에 임의의 정보를 저장할 수 있게 해줍니다. 우리는 그래프를 체크포인터와 `in_memory_store`로 컴파일할 수 있습니다.

```python
from langgraph.checkpoint.memory import MemorySaver

# 스레드(대화)를 활성화하려면 체크포인터가 필요합니다
checkpointer = MemorySaver()

# ... 그래프 정의 ...

# 체크포인터와 저장소를 사용하여 그래프 컴파일
graph = graph.compile(checkpointer=checkpointer, store=in_memory_store)
```

그래프를 `thread_id`와 함께 호출하며, `user_id`도 전달하여 사용자별 메모리를 네임스페이싱할 수 있습니다.

```python
# 그래프 호출
user_id = "1"
config = {"configurable": {"thread_id": "1", "user_id": user_id}}

# 먼저 AI에게 인사하기
for update in graph.stream(
    {"messages": [{"role": "user", "content": "안녕"}]}, config, stream_mode="updates"
):
    print(update)
```

`in_memory_store`와 `user_id`는 *모든 노드*에서 `store: BaseStore`와 `config: RunnableConfig`를 노드 인수로 전달하여 접근할 수 있습니다. 다음은 노드에서 의미 검색을 사용하여 관련 메모리를 찾는 방법입니다:

```python
def update_memory(state: MessagesState, config: RunnableConfig, *, store: BaseStore):

    # config에서 사용자 ID 가져오기
    user_id = config["configurable"]["user_id"]

    # 메모리 네임스페이스 설정
    namespace = (user_id, "memories")

    # ... 대화 분석 후 새 메모리 생성

    # 새 메모리 ID 생성
    memory_id = str(uuid.uuid4())

    # 새 메모리 생성
    store.put(namespace, memory_id, {"memory": memory})
```

위에서 보았듯이, `store`를 사용하여 메모리에 접근하고 `store.search` 메서드를 사용하여 메모리를 검색할 수 있습니다. 메모리는 객체 목록으로 반환되며, 딕셔너리로 변환할 수 있습니다.

```python
memories[-1].dict()
{'value': {'food_preference': '나는 피자를 좋아합니다'},
 'key': '07e0caf4-1631-47b7-b15f-65515d4c1843',
 'namespace': ['1', 'memories'],
 'created_at': '2024-10-02T17:22:31.590602+00:00',
 'updated_at': '2024-10-02T17:22:31.590605+00:00'}
```

메모리를 사용하여 모델 호출에 활용할 수 있습니다.

```python
def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
    # config에서 사용자 ID 가져오기
    user_id = config["configurable"]["user_id"]

    # 가장 최근 메시지를 기반으로 검색
    memories = store.search(
        namespace,
        query=state["messages"][-1].content,
        limit=3
    )
    info = "\n".join([d.value["memory"] for d in memories])

    # ... 메모리를 모델 호출에 사용
```

새로운 스레드를 생성해도 `user_id`가 동일하면 동일한 메모리를 계속 사용할 수 있습니다.

```python
# 그래프 호출
config = {"configurable": {"thread_id": "2", "user_id": "1"}}

# 다시 인사하기
for update in graph.stream(
    {"messages": [{"role": "user", "content": "hi, tell me about my memories"}]}, config, stream_mode="updates"
):
    print(update)
```

LangGraph 플랫폼을 사용할 때, 로컬(예: LangGraph Studio) 또는 LangGraph Cloud에서 기본 저장소를 사용하여 그래프를 컴파일할 때 별도로 지정할 필요가 없습니다. 그러나 의미 검색을 활성화하려면 `langgraph.json` 파일에서 인덱싱 설정을 구성해야 합니다. 예를 들면:

```json
{
    ...
    "store": {
        "index": {
            "embed": "openai:text-embeddings-3-small",
            "dims": 1536,
            "fields": ["$"]
        }
    }
}
```

자세한 내용과 구성 옵션은 [배포 가이드](../cloud/deployment/semantic_search.md)를 참조하세요.

### 체크포인터 라이브러리

LangGraph는 체크포인터 객체를 통해 체크포인팅을 제공합니다. 이 객체는 [BaseCheckpointSaver](langgraph.checkpoint.base.BaseCheckpointSaver) 인터페이스를 따르며, LangGraph는 여러 가지 체크포인터 구현을 제공합니다. 모든 구현은 독립적인 설치 가능한 라이브러리로 제공됩니다:

- `langgraph-checkpoint`: 체크포인터 저장소 인터페이스 ([BaseCheckpointSaver](langgraph.checkpoint.base.BaseCheckpointSaver))와 직렬화/역직렬화 인터페이스 ([SerializerProtocol](langgraph.checkpoint.serde.base.SerializerProtocol))를 포함합니다. 실험용으로 인메모리 체크포인터 구현 ([MemorySaver](langgraph.checkpoint.memory.MemorySaver))도 포함되어 있습니다. LangGraph는 기본적으로 `langgraph-checkpoint`을 포함합니다.
- `langgraph-checkpoint-sqlite`: SQLite 데이터베이스를 사용하는 LangGraph 체크포인터 구현 ([SqliteSaver](langgraph.checkpoint.sqlite.SqliteSaver) / [AsyncSqliteSaver](langgraph.checkpoint.sqlite.aio.AsyncSqliteSaver)). 실험용 및 로컬 워크플로에 적합합니다. 별도로 설치해야 합니다.
- `langgraph-checkpoint-postgres`: Postgres 데이터베이스를 사용하는 고급 체크포인터 구현 ([PostgresSaver](langgraph.checkpoint.postgres.PostgresSaver) / [AsyncPostgresSaver](langgraph.checkpoint.postgres.aio.AsyncPostgresSaver)), LangGraph Cloud에서 사용됩니다. 생산 환경에 적합합니다. 별도로 설치해야 합니다.

### 체크포인터 인터페이스

각 체크포인터는 [BaseCheckpointSaver](langgraph.checkpoint.base.BaseCheckpointSaver) 인터페이스를 구현하며, 다음과 같은 메서드를 제공합니다:

- `.put` - 체크포인트와 해당 구성 및 메타데이터를 저장합니다.
- `.put_writes` - 체크포인트와 연결된 중간 쓰기(예: [보류 중인 쓰기](#pending-writes))를 저장합니다.
- `.get_tuple` - 주어진 구성 (`thread_id` 및 `checkpoint_id`)에 대한 체크포인트 튜플을 가져옵니다. 이는 `graph.get_state()`에서 `StateSnapshot`을 채우는 데 사용됩니다.
- `.list` - 주어진 구성과 필터 기준에 맞는 체크포인트 목록을 반환합니다. 이는 `graph.get_state_history()`에서 상태 히스토리를 채우는 데 사용됩니다.

비동기적으로 그래프를 실행하는 경우 (즉, `.ainvoke`, `.astream`, `.abatch`를 통해 그래프를 실행할 때) 위의 메서드들은 비동기 버전(`.aput`, `.aput_writes`, `.aget_tuple`, `.alist`)을 사용합니다.

!!! 참고
그래프를 비동기적으로 실행하려면 `MemorySaver`나 비동기 버전의 Sqlite/Postgres 체크포인터인 `AsyncSqliteSaver` / `AsyncPostgresSaver` 체크포인터를 사용할 수 있습니다.

### 직렬화기

체크포인터가 그래프 상태를 저장할 때, 상태의 채널 값을 직렬화해야 합니다. 이는 직렬화 객체를 사용하여 수행됩니다. `langgraph_checkpoint`는 [직렬화 프로토콜](langgraph.checkpoint.serde.base.SerializerProtocol)을 정의하여 직렬화기를 구현하며, 기본 구현인 [JsonPlusSerializer](langgraph.checkpoint.serde.jsonplus.JsonPlusSerializer)가 제공됩니다. 이 구현은 LangChain 및 LangGraph 기본 데이터형, 날짜/시간, 열거형 등을 포함한 다양한 유형을 처리할 수 있습니다.

## 기능

### 인간-in-the-loop

첫째, 체크포인터는 [인간-in-the-loop](agentic_concepts.md#human-in-the-loop) 워크플로를 지원합니다. 이를 통해 인간은 그래프 단계를 검사, 중단 및 승인할 수 있습니다. 체크포인터는 이러한 워크플로에 필수적입니다. 인간은 그래프 상태를 언제든지 확인할 수 있어야 하고, 그래프는 인간이 상태를 업데이트한 후 실행을 재개할 수 있어야 합니다. [이들 하우투 가이드](../how-tos/human_in_the_loop/breakpoints.ipynb)를 통해 구체적인 예제를 확인할 수 있습니다.

### 메모리

둘째, 체크포인터는 [메모리](agentic_concepts.md#memory) 기능을 지원합니다. 반복적인 인간 상호작용(예: 대화)에서 후속 메시지는 해당 스레드에 전송되며 이전 메시지에 대한 기억을 유지합니다. 체크포인터를 사용하여 대화형 메모리를 추가하고 관리하는 방법에 대한 예제를 보려면 [이 하우투 가이드](../how-tos/memory/manage-conversation-history.ipynb)를 참조하세요.

### 시간 여행

셋째, 체크포인트는 [시간 여행](time-travel.md)을 지원하여 사용자가 이전 그래프 실행을 재생하여 특정 그래프 단계를 검토하거나 디버깅할 수 있도록 합니다. 또한 체크포인터를 사용하면 그래프 상태를 임의의 체크포인트에서 분기시켜 대체 경로를 탐색할 수 있습니다.

### 내결함성

마지막으로, 체크포인트는 내결함성 및 오류 복구 기능을 제공합니다. 하나 이상의 노드가 특정 슈퍼 단계에서 실패하면, 그래프는 마지막 성공적인 단계에서 재시작할 수 있습니다. 또한, 그래프 노드가 특정 슈퍼 단계에서 실행 도중 실패하면, LangGraph는 다른 노드에서 완료된 모든 성공적인 쓰기를 보류 중인 체크포인트로 저장하여, 그래프 실행이 재개될 때 성공적인 노드를 다시 실행하지 않도록 합니다.

#### 보류 중인 쓰기

추가로, 그래프 노드가 실행 도중 실패하면 LangGraph는 해당 슈퍼 단계에서 완료된 다른 노드들의 보류 중인 체크포인트 쓰기를 저장합니다. 그래프 실행이 재개될 때 성공적인 노드는 다시 실행되지 않도록 합니다.
