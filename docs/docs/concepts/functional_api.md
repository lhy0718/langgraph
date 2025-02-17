_한국어로 기계번역됨_

# 기능 API

!!! 경고 "베타"
기능 API는 현재 **베타** 상태이며 변경될 수 있습니다. 문제나 피드백이 있는 경우 [LangGraph 팀에 보고해 주세요](https://github.com/langchain-ai/langgraph/issues).

## 개요

**기능 API**를 통해 LangGraph의 핵심 기능인 [영속성](./persistence.md), [메모리](./memory.md), [인간 개입](./human_in_the_loop.md), [스트리밍](./streaming.md)을 기존 코드에 최소한의 변경으로 추가할 수 있습니다.

이 API는 `if` 문, `for` 루프, 함수 호출과 같은 표준 언어 원시 기능을 사용하는 기존 코드에 이러한 기능을 통합하도록 설계되었습니다. 데이터 오케스트레이션 프레임워크의 많은 경우처럼 코드를 명시적인 파이프라인이나 DAG로 재구성할 필요 없이 기능 API를 통해 이러한 기능을 강제하지 않고 통합할 수 있습니다.

기능 API는 두 가지 주요 구성 요소를 사용합니다:

- **`@entrypoint`** – 워크플로의 시작점을 표시하며, 로직을 캡슐화하고 실행 흐름을 관리하며, 장기 실행 작업 및 인터럽트를 처리합니다.
- **`@task`** – API 호출이나 데이터 처리 단계와 같은 개별 작업 단위를 나타내며, 엔트리포인트 내에서 비동기적으로 실행될 수 있습니다. 작업은 대기하거나 동기적으로 해결할 수 있는 미래와 유사한 객체를 반환합니다.

이것은 상태 관리 및 스트리밍을 위한 워크플로를 구축하기 위한 최소한의 추상화를 제공합니다.

!!! 팁

    보다 선언적인 접근 방식을 선호하는 사용자에게는 LangGraph의 [그래프 API](./low_level.md)를 사용하여 그래프 패러다임으로 워크플로를 정의할 수 있습니다. 두 API는 동일한 기본 런타임을 공유하므로 같은 애플리케이션에서 함께 사용할 수 있습니다.
    두 패러다임에 대한 비교는 [기능 API 대 그래프 API](#functional-api-vs-graph-api) 섹션을 참조하세요.

## 예시

아래에서는 에세이를 작성하고 [인간 검토 요청을 위한 인터럽트](human_in_the_loop.md)를 사용하는 간단한 애플리케이션을 보여줍니다.

```python
from langgraph.func import entrypoint, task
from langgraph.types import interrupt

@task
def write_essay(topic: str) -> str:
    """주어진 주제에 대한 에세이를 작성합니다."""
    time.sleep(1) # 장기 실행 작업을 위한 자리 표시자.
    return f"주제에 대한 에세이: {topic}"

@entrypoint(checkpointer=MemorySaver())
def workflow(topic: str) -> dict:
    """에세이를 쓰고 검토 요청을 하는 간단한 워크플로."""
    essay = write_essay("고양이").result()
    is_approved = interrupt({
        # 인터럽트에 제공되는 JSON 직렬화 가능한 페이로드.
        # 워크플로에서 데이터를 스트리밍할 때 클라이언트 측에서 Interrupt로 표시됩니다.
        "essay": essay, # 검토를 원하는 에세이.
        # 필요한 추가 정보를 추가할 수 있습니다.
        # 예를 들어 "action"이라는 키를 도입하여 일부 지침을 추가합니다.
        "action": "에세이를 승인/거부해 주세요",
    })

    return {
        "essay": essay, # 생성된 에세이
        "is_approved": is_approved, # HIL로부터의 응답
    }
```

??? 예시 "상세 설명"

    이 워크플로는 "고양이"라는 주제에 대한 에세이를 작성한 다음, 인간으로부터 검토를 받기 위해 일시 중지됩니다. 검토가 제공될 때까지 워크플로는 무기한으로 인터럽트될 수 있습니다.

    워크플로가 재개되면 처음부터 실행되지만, `write_essay` 작업의 결과가 이미 저장되었기 때문에 작업 결과는 재계산되는 대신 체크포인트에서 로드됩니다.

    ```python
    import time
    import uuid

    from langgraph.func import entrypoint, task
    from langgraph.types import interrupt
    from langgraph.checkpoint.memory import MemorySaver

    @task
    def write_essay(topic: str) -> str:
        """주제에 대한 에세이를 작성합니다."""
        time.sleep(1) # 이것은 오랜 시간이 걸리는 작업을 위한 자리 표시자입니다.
        return f"주제에 대한 에세이: {topic}"

    @entrypoint(checkpointer=MemorySaver())
    def workflow(topic: str) -> dict:
        """에세이를 작성하고 리뷰를 요청하는 간단한 작업 흐름입니다."""
        essay = write_essay("cat").result()
        is_approved = interrupt({
            # interrupt의 인수로 제공된 모든 json-직렬화 가능한 페이로드.
            # 이는 작업 흐름에서 데이터를 스트리밍할 때 클라이언트 측에 Interrupt로 표시됩니다.
            "essay": essay, # 우리가 리뷰를 요청할 에세이.
            # 필요한 추가 정보를 추가할 수 있습니다.
            # 예를 들어, "action"이라는 키와 함께 어떤 지시를 추가할 수 있습니다.
            "action": "에세이를 승인/거부해 주세요",
        })

        return {
            "essay": essay, # 생성된 에세이
            "is_approved": is_approved, # HIL로부터의 응답
        }

    thread_id = str(uuid.uuid4())

    config = {
        "configurable": {
            "thread_id": thread_id
        }
    }

    for item in workflow.stream("cat", config):
        print(item)
    ```

    ```pycon
    {'write_essay': '주제에 대한 에세이: cat'}
    {'__interrupt__': (Interrupt(value={'essay': '주제에 대한 에세이: cat', 'action': '에세이를 승인/거부해 주세요'}, resumable=True, ns=['workflow:f7b8508b-21c0-8b4c-5958-4e8de74d2684'], when='during'),)}
    ```

    에세이가 작성되었고 리뷰를 받을 준비가 되었습니다. 리뷰가 제공되면 작업 흐름을 재개할 수 있습니다:

    ```python
    from langgraph.types import Command

    # 사용자로부터 리뷰 받기 (예: UI를 통해)
    # 이 경우 bool을 사용하고 있지만, 이는 json-직렬화 가능한 값일 수 있습니다.
    human_review = True

    for item in workflow.stream(Command(resume=human_review), config):
        print(item)
    ```

    ```pycon
    {'workflow': {'essay': '주제에 대한 에세이: cat', 'is_approved': False}}
    ```

    작업 흐름이 완료되었고 리뷰가 에세이에 추가되었습니다.

## 엔트리포인트

[`@entrypoint`][langgraph.func.entrypoint] 데코레이터는 함수를 작업 흐름으로 만들 때 사용됩니다. 작업 흐름 로직을 캡슐화하고 실행 흐름을 관리하며, _오랜 시간이 걸리는 작업_ 및 [interrupts](./low_level.md#interrupt)를 처리합니다.

### 정의

**엔트리포인트**는 `@entrypoint` 데코레이터로 함수를 장식하여 정의됩니다.

함수는 **하나의 위치 인수**를 수락해야 하며, 이는 작업 흐름 입력 역할을 합니다. 여러 개의 데이터를 전달해야 하는 경우, 첫 번째 인수의 입력 유형으로 사전을 사용하세요.

함수를 `entrypoint`로 장식하면 [`Pregel`][langgraph.pregel.Pregel.stream] 인스턴스가 생성되어 작업 흐름의 실행을 관리하는 데 도움을 줍니다 (예: 스트리밍, 재개, 체크포인트 처리 등).

대개 **지속성을 활성화**하고 **human-in-the-loop**와 같은 기능을 사용하기 위해 `@entrypoint` 데코레이터에 **체크포인터**를 전달하는 것이 좋습니다.

=== "동기(Sync)"

    ```python
    from langgraph.func import entrypoint

    @entrypoint(checkpointer=checkpointer)
    def my_workflow(some_input: dict) -> int:
        # API 호출과 같은 오랜 시간이 걸릴 수 있는 작업이 포함될 수 있는 일부 논리,
        # 그리고 human-in-the-loop를 위해 중단될 수 있습니다.
        ...
        return result
    ```

=== "비동기(Async)"

    ```python
    from langgraph.func import entrypoint

    @entrypoint(checkpointer=checkpointer)
    async def my_workflow(some_input: dict) -> int:
        # API 호출과 같은 긴 작업을 포함할 수 있는 일부 로직,
        # 인간의 개입이 필요한 경우 중단될 수도 있습니다.
        ...
        return result
    ```

!!! 중요 "직렬화"

    엔트리포인트의 **입력** 및 **출력**은 체크포인팅을 지원하기 위해 JSON 직렬화가 가능해야 합니다. 자세한 내용은 [직렬화](#serialization) 섹션을 참조하세요.

### 주입 가능한 매개변수

`entrypoint`를 선언할 때 런타임에 자동으로 주입될 추가 매개변수에 접근할 수 있습니다. 이러한 매개변수는 다음과 같습니다:

| 매개변수     | 설명                                                                                                                                                    |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **previous** | 주어진 스레드에 대한 이전 `checkpoint`와 관련된 상태에 접근합니다. [상태 관리](#state-management)를 참조하세요.                                         |
| **store**    | [BaseStore][langgraph.store.base.BaseStore]의 인스턴스입니다. [장기 기억](#long-term-memory)에 유용합니다.                                              |
| **writer**   | 사용자 정의 데이터를 `custom` 스트림에 쓰기 위한 스트리밍 사용자 정의 데이터입니다. [사용자 정의 데이터 스트리밍](#streaming-custom-data)에 유용합니다. |
| **config**   | 런타임 구성에 접근합니다. [RunnableConfig](https://python.langchain.com/docs/concepts/runnables/#runnableconfig) 정보를 참조하세요.                     |

!!! 중요

    매개변수를 적절한 이름과 타입 주석으로 선언하세요.

??? 예시 "주입 가능한 매개변수 요청"

    ```python
    from langchain_core.runnables import RunnableConfig
    from langgraph.func import entrypoint
    from langgraph.store.base import BaseStore
    from langgraph.store.memory import InMemoryStore

    in_memory_store = InMemoryStore(...)  # 장기 기억을 위한 InMemoryStore의 인스턴스

    @entrypoint(
        checkpointer=checkpointer,  # 체크포인터 지정
        store=in_memory_store  # 저장소 지정
    )
    def my_workflow(
        some_input: dict,  # 입력 (예: `invoke`를 통해 전달됨)
        *,
        previous: Any = None, # 단기 기억을 위한 것
        store: BaseStore,  # 장기 기억을 위한 것
        writer: StreamWriter,  # 사용자 정의 데이터 스트리밍을 위한 것
        config: RunnableConfig  # 엔트리포인트에 전달된 구성에 접근하기 위한 것
    ) -> ...:
    ```

### 실행하기

[`@entrypoint`](#entrypoint)를 사용하면 `invoke`, `ainvoke`, `stream` 및 `astream` 메서드를 사용하여 실행할 수 있는 [`Pregel`][langgraph.pregel.Pregel.stream] 객체가 생성됩니다.

=== "Invoke"

    ```python
    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }
    my_workflow.invoke(some_input, config)  # 결과를 동기적으로 기다립니다.
    ```

=== "비동기 Invoke"

    ```python
    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }
    await my_workflow.ainvoke(some_input, config)  # 결과를 비동기적으로 기다립니다.
    ```

=== "스트림"

    ```python
    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    for chunk in my_workflow.stream(some_input, config):
        print(chunk)
    ```

=== "비동기 스트림"

    ```python
    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    async for chunk in my_workflow.astream(some_input, config):
        print(chunk)
    ```

### 재개하기

[인터럽트][langgraph.types.interrupt] 후에 실행을 재개하려면 **resume** 값을 [Command][langgraph.types.Command] 원시 유형에 전달하면 됩니다.

=== "호출"

    ```python
    from langgraph.types import Command

    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    my_workflow.invoke(Command(resume=some_resume_value), config)
    ```

=== "비동기 호출"

    ```python
    from langgraph.types import Command

    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    await my_workflow.ainvoke(Command(resume=some_resume_value), config)
    ```

=== "스트림"

    ```python
    from langgraph.types import Command

    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    for chunk in my_workflow.stream(Command(resume=some_resume_value), config):
        print(chunk)
    ```

=== "비동기 스트림"

    ```python
    from langgraph.types import Command

    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    async for chunk in my_workflow.astream(Command(resume=some_resume_value), config):
        print(chunk)
    ```

**오류 후 재개하기**

오류 후 재개하려면 `entrypoint`를 `None`과 동일한 **스레드 ID**(구성)로 실행하세요.

이는 근본적인 **오류**가 해결되었고 실행이 성공적으로 진행될 수 있다고 가정합니다.

=== "호출"

    ```python

    config = {
        "configurable": {
            "thread_id": "some_thread_id"
        }
    }

    my_workflow.invoke(None, config)
    ```

=== "비동기 호출"

    ```python

config = {
"configurable": {
"thread_id": "some_thread_id"
}
}

await my_workflow.ainvoke(None, config)

````

=== "스트림"

```python
config = {
    "configurable": {
        "thread_id": "some_thread_id"
    }
}

for chunk in my_workflow.stream(None, config):
    print(chunk)
````

=== "비동기 스트림"

```python
config = {
    "configurable": {
        "thread_id": "some_thread_id"
    }
}

async for chunk in my_workflow.astream(None, config):
    print(chunk)
```

### 상태 관리

`entrypoint`가 `checkpointer`와 함께 정의되면, 같은 **스레드 ID**에 대한 연속 호출 간에 정보를 [체크포인트](persistence.md#checkpoints)에 저장합니다.

이를 통해 `previous` 매개변수를 사용하여 이전 호출의 상태에 접근할 수 있습니다.

기본적으로 `previous` 매개변수는 이전 호출의 반환 값입니다.

```python
@entrypoint(checkpointer=checkpointer)
def my_workflow(number: int, *, previous: Any = None) -> int:
    previous = previous or 0
    return number + previous

config = {
    "configurable": {
        "thread_id": "some_thread_id"
    }
}

my_workflow.invoke(1, config)  # 1 (이전 값은 None)
my_workflow.invoke(2, config)  # 3 (이전 값은 이전 호출에서의 1)
```

#### `entrypoint.final`

[entrypoint.final][langgraph.func.entrypoint.final]은 entrypoint에서 반환할 수 있는 특수 원시형으로, 체크포인트에 **저장되는** 값과 **entrypoint의 반환 값**을 **분리**할 수 있게 해줍니다.

첫 번째 값은 entrypoint의 반환 값이고, 두 번째 값은 체크포인트에 저장될 값입니다. 타입 주석은 `entrypoint.final[return_type, save_type]`입니다.

```python
@entrypoint(checkpointer=checkpointer)
def my_workflow(number: int, *, previous: Any = None) -> entrypoint.final[int, int]:
    previous = previous or 0
    # 이것은 호출자에게 이전 값을 반환하고,
    # 2 * number를 체크포인트에 저장하여 다음 호출에서
    # `previous` 매개변수로 사용됩니다.
    return entrypoint.final(value=previous, save=2 * number)

config = {
    "configurable": {
        "thread_id": "1"
    }
}

my_workflow.invoke(3, config)  # 0 (이전 값은 None)
my_workflow.invoke(1, config)  # 6 (이전 값은 이전 호출에서의 3 * 2)
```

## 작업

작업은 API 호출이나 데이터 처리 단계와 같은 독립적인 작업 단위를 나타냅니다. 작업은 두 가지 주요 특성을 갖습니다:

- **비동기 실행**: 작업은 비동기적으로 실행되도록 설계되어, 여러 작업이 동시에 실행되면서 차단되지 않습니다.
- **체크포인트**: 작업 결과는 체크포인트에 저장되어, 마지막 저장된 상태에서 워크플로를 재개할 수 있게 합니다. (자세한 내용은 [저장소](persistence.md)를 참조하세요).

### 정의

작업은 `@task` 데코레이터를 사용하여 정의되며, 이는 일반적인 Python 함수를 래핑합니다.

```python
from langgraph.func import task

@task()
def slow_computation(input_value):
    # 긴 작업을 시뮬레이션합니다.
    ...
    return result
```

!!! 중요 "직렬화"

    작업의 **출력**은 체크포인트를 지원하기 위해 JSON-직렬화 가능해야 합니다.

### 실행

**작업**은 **엔트리포인트**, 다른 **작업**, 또는 [상태 그래프 노드](./low_level.md#nodes) 내부에서만 호출할 수 있습니다.

작업은 메인 애플리케이션 코드에서 직접 호출할 수 없습니다.

작업을 호출하면 즉시 미래 객체를 반환합니다. 미래는 나중에 사용 가능한 결과에 대한 자리 표시자입니다.

작업의 결과를 얻으려면 동기적으로 기다리거나 (`result()`) 비동기적으로 기다릴 수 있습니다 (`await`).

=== "동기 호출"

    ```python
    @entrypoint(checkpointer=checkpointer)
    def my_workflow(some_input: int) -> int:
        future = slow_computation(some_input)
        return future.result()  # 결과를 동기적으로 기다립니다.
    ```

=== "비동기 호출"

    ```python
    @entrypoint(checkpointer=checkpointer)
    async def my_workflow(some_input: int) -> int:
        return await slow_computation(some_input)  # 결과를 비동기적으로 기다립니다.
    ```

## 작업을 사용하는 경우

**작업**은 다음과 같은 시나리오에서 유용합니다:

- **체크포인팅**: 긴 작업의 결과를 체크포인트에 저장해야 할 때, 워크플로우를 재개할 때 다시 계산할 필요가 없습니다.
- **인간 통제**: 인간의 개입이 필요한 워크플로우를 구축하는 경우, 랜덤성을 캡슐화하기 위해 **작업**을 사용해야 합니다 (예: API 호출) 워크플로우가 올바르게 재개될 수 있도록 보장합니다. 자세한 내용은 [결정론](#determinism) 섹션을 참조하십시오.
- **병렬 실행**: I/O가 많은 작업을 위해, **작업**은 병렬 실행을 가능하게 하여 여러 작업을 동시에 수행할 수 있습니다 (예: 여러 API 호출).
- **모니터링 가능성**: **작업** 내에 작업을 포장하면 워크플로우의 진행 상태를 추적하고 [LangSmith](https://docs.smith.langchain.com/)를 사용하여 개별 작업의 실행을 모니터링할 수 있습니다.
- **재시도 가능한 작업**: 작업이 실패나 불일치를 처리하기 위해 재시도해야 할 때, **작업**은 재시도 로직을 캡슐화하고 관리하는 방법을 제공합니다.

## 직렬화

LangGraph에서 직렬화의 주요 측면은 두 가지입니다:

1. `@entrypoint` 입력 및 출력은 JSON-직렬화 가능해야 합니다.
2. `@task` 출력은 JSON-직렬화 가능해야 합니다.

이러한 요구 사항은 체크포인팅 및 워크플로우 재개를 가능하게 합니다. 사전 정의된 파이썬 원시형인 딕셔너리, 리스트, 문자열, 숫자 및 불리언을 사용하여 입력 및 출력이 직렬화 가능하도록 보장하십시오.

직렬화는 작업 결과 및 중간 값과 같은 워크플로 상태가 신뢰할 수 있게 저장되고 복원될 수 있음을 보장합니다. 이는 인간 통제 상호 작용, 내결함성 및 병렬 실행을 가능하게 하는 데 필수적입니다.

직렬화할 수 없는 입력 또는 출력을 제공하면 체크포인트가 구성된 워크플로에서 런타임 오류가 발생합니다.

## 결정론

**인간 통제**와 같은 기능을 활용하려면 모든 무작위성을 **작업** 내에 캡슐화해야 합니다. 이렇게 하면 실행이 중단되었을 때 (예: 인간 통제) 다시 재개되더라도 동일한 *단계 순서*를 따릅니다. **작업** 결과가 비결정적이라도 마찬가지입니다.

LangGraph는 실행 중에 **작업** 및 [**서브그래프**](./low_level.md#subgraphs) 결과를 지속적으로 보존함으로써 이러한 동작을 달성합니다. 잘 설계된 워크플로우는 실행 재개가 *동일한 단계 순서*를 따르게 하여 이전에 계산된 결과를 올바르게 검색할 수 있도록 보장합니다. 이는 긴 **작업** 또는 비결정적 결과가 있는 **작업**에 특히 유용하며, 이전에 수행된 작업을 반복하지 않고 본질적으로 동일한 지점에서 재개할 수 있습니다.

워크플로의 서로 다른 실행이 서로 다른 결과를 생성할 수 있지만, **특정** 실행을 재개할 때는 항상 기록된 단계 순서를 따르게 됩니다. 이는 LangGraph가 그래프가 중단되기 전에 실행된 **작업** 및 **서브그래프** 결과를 효율적으로 조회하고 재계산을 피할 수 있도록 합니다.

## 멱등성

멱등성은 동일한 작업을 여러 번 실행하더라도 동일한 결과를 생성하도록 보장합니다. 이는 실패로 인해 단계가 다시 실행될 경우 중복 API 호출 및 불필요한 처리를 방지하는 데 도움이 됩니다. 체크포인팅을 위해 항상 API 호출을 **작업** 함수 내에 배치하고, 재실행 시 멱등성이 있도록 설계하십시오. 작업이 시작되지만 성공적으로 완료되지 않으면 재실행이 발생할 수 있습니다. 그런 다음 워크플로우가 재개될 때 **작업**이 다시 실행됩니다. 중복을 피하기 위해 멱등성 키를 사용하거나 기존 결과를 확인하십시오.

## 함수형 API와 그래프 API

**함수형 API**와 [그래프 API (상태 그래프)](./low_level.md#stategraph)는 LangGraph로 애플리케이션을 생성하기 위한 두 가지 다른 패러다임을 제공합니다. 여기 몇 가지 주요 차이점이 있습니다:

- **제어 흐름**: 함수형 API는 그래프 구조를 고려할 필요가 없습니다. 표준 파이썬 구성을 사용하여 워크플로우를 정의할 수 있으며, 이로 인해 작성해야 하는 코드량을 줄일 수 있습니다.
- **상태 관리**: **그래프 API**는 [**상태**](./low_level.md#state)를 선언해야 하며, 그래프 상태 업데이트를 관리하기 위해 [**리듀서**](./low_level.md#reducers)를 정의해야 할 수 있습니다. `@entrypoint` 및 `@task`는 명시적인 상태 관리가 필요하지 않으며, 그들의 상태는 함수에 국한되어 있으며 함수 간에 공유되지 않습니다.
- **체크포인팅**: 두 API 모두 체크포인트를 생성하고 사용합니다. **그래프 API**는 매 초단계 후에 새로운 체크포인트를 생성합니다. **함수형 API**에서는 작업이 실행될 때, 그 결과가 주어진 엔트리포인트와 연결된 기존 체크포인트에 저장됩니다.
- **시각화**: 그래프 API는 디버깅, 워크플로우 이해 및 타인과의 공유에 유용한 그래프 형태로 워크플로우를 시각화하기 쉽게 만듭니다. 함수형 API는 런타임 동안 그래프가 동적으로 생성되기 때문에 시각화를 지원하지 않습니다.

## 일반적인 실수

### 부작용 처리

부작용(예: 파일 쓰기, 이메일 발송)을 작업에 캡슐화하여 워크플로우를 재개할 때 여러 번 실행되지 않도록 합니다.

=== "잘못된 예"

    이 예제에서 부작용(파일 쓰기)이 워크플로우에 직접 포함되어 있어, 워크플로우를 재개할 때 두 번째로 실행됩니다.

    ```python
    @entrypoint(checkpointer=checkpointer)
    def my_workflow(inputs: dict) -> int:
        # 이 코드는 워크플로우를 재개할 때 두 번째로 실행됩니다.
        # 이는 아마도 여러분이 원하는 것이 아닐 것입니다.
        # 하이라이트-다음-줄
        with open("output.txt", "w") as f:
            # 하이라이트-다음-줄
            f.write("부작용이 실행되었습니다")
        value = interrupt("질문")
        return value
    ```

=== "올바른"

    이 예제에서 부작용은 작업에 캡슐화되어 있어, 재개 시 일관된 실행이 보장됩니다.

    ```python
    from langgraph.func import task

    # 하이라이트-다음-줄
    @task
    # 하이라이트-다음-줄
    def write_to_file():
        with open("output.txt", "w") as f:
            f.write("부작용이 실행되었습니다")

    @entrypoint(checkpointer=checkpointer)
    def my_workflow(inputs: dict) -> int:
        # 이제 부작용이 작업에 캡슐화되었습니다.
        write_to_file().result()
        value = interrupt("질문")
        return value
    ```

### 비결정적 제어 흐름

매번 다른 결과를 줄 수 있는 작업(예: 현재 시간 가져오기 또는 랜덤 숫자 생성)은 작업에 캡슐화되어야 합니다. 그래야 재개 시 동일한 결과가 반환됩니다.

- 작업 내: 랜덤 숫자 가져오기 (5) → 인터럽트 → 재개 → (다시 5 반환) → ...
- 작업 외: 랜덤 숫자 가져오기 (5) → 인터럽트 → 재개 → 새로운 랜덤 숫자 가져오기 (7) → ...

이는 **인간-참여형** 워크플로우를 사용할 때 특히 중요합니다. LangGraph는 각 작업/진입점에 대한 재개 값 목록을 유지합니다. 인터럽트가 발생하면 해당 재개 값과 일치시킵니다. 이 매칭은 엄밀히 **인덱스 기반**으로 이루어지므로 재개 값의 순서는 인터럽트의 순서와 일치해야 합니다.

재개 시 실행 순서가 유지되지 않으면 하나의 `interrupt` 호출이 잘못된 `resume` 값과 일치하여 잘못된 결과를 초래할 수 있습니다.

자세한 내용은 [결정론](#determinism) 섹션을 읽어보세요.

=== "잘못된"

    이 예제에서 워크플로우는 현재 시간을 사용하여 실행할 작업을 결정합니다. 이는 비결정적입니다. 왜냐하면 워크플로우의 결과가 실행되는 시간에 따라 달라지기 때문입니다.

    ```python
    from langgraph.func import entrypoint

    @entrypoint(checkpointer=checkpointer)
    def my_workflow(inputs: dict) -> int:
        t0 = inputs["t0"]
        # 하이라이트-다음-줄
        t1 = time.time()

        delta_t = t1 - t0

        if delta_t > 1:
            result = slow_task(1).result()
            value = interrupt("질문")
        else:
            result = slow_task(2).result()
            value = interrupt("질문")

        return {
            "result": result,
            "value": value
        }
    ```

=== "올바른"

    이 예제에서 워크플로우는 입력 `t0`를 사용하여 실행할 작업을 결정합니다. 이는 결정적입니다. 왜냐하면 워크플로우의 결과가 오직 입력에만 의존하기 때문입니다.

    ```python
    import time

    from langgraph.func import task

    # 하이라이트-다음-라인
    @task
    # 하이라이트-다음-라인
    def get_time() -> float:
        return time.time()

    @entrypoint(checkpointer=checkpointer)
    def my_workflow(inputs: dict) -> int:
        t0 = inputs["t0"]
        # 하이라이트-다음-라인
        t1 = get_time().result()

        delta_t = t1 - t0

        if delta_t > 1:
            result = slow_task(1).result()
            value = interrupt("question")
        else:
            result = slow_task(2).result()
            value = interrupt("question")

        return {
            "result": result,
            "value": value
        }
    ```

## 패턴

아래에는 **기능 API**를 사용하는 몇 가지 간단한 패턴이 있습니다.

`entrypoint`를 정의할 때 입력은 함수의 첫 번째 인수로 제한됩니다. 여러 개의 입력을 전달하려면 사전을 사용할 수 있습니다.

```python
@entrypoint(checkpointer=checkpointer)
def my_workflow(inputs: dict) -> int:
    value = inputs["value"]
    another_value = inputs["another_value"]
    ...

my_workflow.invoke({"value": 1, "another_value": 2})
```

### 병렬 실행

작업은 동시에 호출하고 결과를 기다림으로써 병렬로 실행할 수 있습니다. 이는 IO 중심 작업(예: LLM을 위한 API 호출)에서 성능을 개선하는 데 유용합니다.

```python
@task
def add_one(number: int) -> int:
    return number + 1

@entrypoint(checkpointer=checkpointer)
def graph(numbers: list[int]) -> list[str]:
    futures = [add_one(i) for i in numbers]
    return [f.result() for f in futures]
```

### 서브 그래프 호출

**기능 API**와 [**그래프 API**](./low_level.md)는 동일한 기본 런타임을 공유하므로 같은 애플리케이션 내에서 함께 사용할 수 있습니다.

```python
from langgraph.func import entrypoint
from langgraph.graph import StateGraph

builder = StateGraph()
...
some_graph = builder.compile()

@entrypoint()
def some_workflow(some_input: dict) -> int:
    # 그래프 API를 사용하여 정의된 그래프 호출
    result_1 = some_graph.invoke(...)
    # 다른 그래프 API를 사용하여 정의된 그래프 호출
    result_2 = another_graph.invoke(...)
    return {
        "result_1": result_1,
        "result_2": result_2
    }
```

### 다른 entrypoint 호출

**entrypoint** 내에서 다른 **entrypoints** 또는 **task**를 호출할 수 있습니다.

```python
@entrypoint() # 부모 엔트리포인트에서 체크포인터를 자동으로 사용합니다
def some_other_workflow(inputs: dict) -> int:
    return inputs["value"]

@entrypoint(checkpointer=checkpointer)
def my_workflow(inputs: dict) -> int:
    value = some_other_workflow.invoke({"value": 1})
    return value
```

### 사용자 지정 데이터 스트리밍

`StreamWriter` 유형을 사용하여 **엔트리포인트**에서 사용자 지정 데이터를 스트리밍할 수 있습니다. 이를 통해 `custom` 스트림에 사용자 지정 데이터를 쓸 수 있습니다.

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.func import entrypoint, task
from langgraph.types import StreamWriter

@task
def add_one(x):
    return x + 1

@task
def add_two(x):
    return x + 2

checkpointer = MemorySaver()

@entrypoint(checkpointer=checkpointer)
def main(inputs, writer: StreamWriter) -> int:
    """숫자에 1과 2를 더하는 간단한 워크플로우입니다."""
    writer("hello") # `custom` 스트림에 데이터를 씁니다
    add_one(inputs['number']).result() # `updates` 스트림에 데이터를 씁니다
    writer("world") # `custom` 스트림에 추가 데이터를 씁니다
    add_two(inputs['number']).result() # `updates` 스트림에 데이터를 씁니다
    return 5

config = {
    "configurable": {
        "thread_id": "1"
    }
}

for chunk in main.stream({"number": 1}, stream_mode=["custom", "updates"], config=config):
    print(chunk)
```

```pycon
('updates', {'add_one': 2})
('updates', {'add_two': 3})
('custom', 'hello')
('custom', 'world')
('updates', {'main': 5})
```

!!! 중요

    `writer` 매개변수는 런타임에 자동으로 주입됩니다. 매개변수 이름이 함수 시그니처에
    그 *정확한* 이름으로 나타나야만 주입됩니다.

### 재시도 정책

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.func import entrypoint, task
from langgraph.types import RetryPolicy

attempts = 0

# ValueError에 대해 재시도하도록 RetryPolicy를 구성합시다.
# 기본 RetryPolicy는 특정 네트워크 오류에 대한 재시도에 최적화되어 있습니다.
retry_policy = RetryPolicy(retry_on=ValueError)

@task(retry=retry_policy)
def get_info():
    global attempts
    attempts += 1

    if attempts < 2:
        raise ValueError('실패')
    return "OK"

checkpointer = MemorySaver()

@entrypoint(checkpointer=checkpointer)
def main(inputs, writer):
    return get_info().result()

config = {
    "configurable": {
        "thread_id": "1"
    }
}

main.invoke({'any_input': 'foobar'}, config=config)
```

```pycon
'OK'
```

### 오류 후 재개

```python
import time
from langgraph.checkpoint.memory import MemorySaver
from langgraph.func import entrypoint, task
from langgraph.types import StreamWriter

# 시도 횟수를 추적하는 전역 변수
attempts = 0

@task()
def get_info():
    """
    실패한 후 성공하는 작업을 시뮬레이션합니다.
    첫 번째 시도에서 예외를 발생시키고, 이후 시도에서는 "OK"를 반환합니다.
    """
    global attempts
    attempts += 1

    if attempts < 2:
        raise ValueError("실패")  # 첫 번째 시도에서 실패를 시뮬레이트
    return "OK"

# 지속성을 위한 메모리 체크포인터 초기화
checkpointer = MemorySaver()

@task
def slow_task():
    """
    1초 지연을 통해 느리게 실행되는 작업을 시뮬레이션합니다.
    """
    time.sleep(1)
    return "느린 작업이 실행되었습니다."

@entrypoint(checkpointer=checkpointer)
def main(inputs, writer: StreamWriter):
    """
    느린 작업과 정보를 가져오는 작업을 순차적으로 실행하는 주 작업 흐름 함수입니다.

    매개변수:
    - inputs: 작업 흐름 입력 값이 포함된 딕셔너리.
    - writer: 사용자 정의 데이터를 스트리밍하기 위한 StreamWriter.

    작업 흐름은 먼저 `slow_task`를 실행하고, 이후 `get_info`를 실행하는데,
    여기서 첫 번째 호출에서 실패하게 됩니다.
    """
    slow_task_result = slow_task().result()  # slow_task에 대한 블로킹 호출
    get_info().result()  # 첫 번째 시도에서 예외가 발생합니다.
    return slow_task_result

# 고유 스레드 식별자를 가진 작업 흐름 실행 구성
config = {
    "configurable": {
        "thread_id": "1"  # 작업 흐름 실행을 추적하기 위한 고유 식별자
    }
}

# 이 호출은 느린 작업 실행으로 인해 약 1초가 소요됩니다.
try:
    # 첫 번째 호출에서 `get_info` 작업이 실패하여 예외가 발생합니다.
    main.invoke({'any_input': 'foobar'}, config=config)
except ValueError:
    pass  # 실패를 우아하게 처리합니다.
```

재개 실행 시, 결과가 체크포인트에 이미 저장되어 있으므로 `slow_task`를 다시 실행할 필요가 없습니다.

```python
main.invoke(None, config=config)
```

```pycon
'느린 작업이 실행되었습니다.'
```

### 사람 개입 워크플로우

기능적 API는 `interrupt` 함수와 `Command` 기본 요소를 사용하여 [사람 개입](human_in_the_loop.md) 워크플로우를 지원합니다.

자세한 내용은 다음 예제를 참조하십시오:

- [사용자 입력 대기 방법 (기능적 API)](../how-tos/wait-user-input-functional.ipynb): 기능적 API를 사용하여 간단한 사람 개입 워크플로우를 구현하는 방법을 보여줍니다.
- [도구 호출 검토 방법 (기능적 API)](../how-tos/review-tool-calls-functional.ipynb): LangGraph 기능적 API를 사용하여 ReAct 에이전트에서 사람 개입 워크플로우를 구현하는 방법을 설명합니다.

### 단기 기억

[상태 관리](#state-management)는 **이전** 매개변수를 사용하고 선택적으로 `entrypoint.final` 기본 요소를 사용하여 [단기 기억](memory.md)을 구현하는 데 사용될 수 있습니다.

자세한 내용은 다음의 사용 가이드를 참조하십시오:

- [스레드 수준 지속성 추가 방법 (기능적 API)](../how-tos/persistence-functional.ipynb): 기능적 API 워크플로우에 스레드 수준의 지속성을 추가하는 방법과 간단한 챗봇을 구현하는 방법을 보여줍니다.

### 장기 기억

[장기 기억](memory.md#long-term-memory)은 서로 다른 **스레드 ID** 간에 정보를 저장할 수 있게 해줍니다. 이는 한 대화에서 특정 사용자에 대한 정보를 학습하고 이를 다른 대화에서 활용하는 데 유용할 수 있습니다.

자세한 내용은 다음의 가이드들을 참조하시기 바랍니다:

- [크로스 스레드 지속성 추가 방법 (기능 API)](../how-tos/cross-thread-persistence-functional.ipynb): 기능 API 워크플로우에 크로스 스레드 지속성을 추가하는 방법을 보여주고 간단한 챗봇을 구현합니다.

### 워크플로우

- [워크플로우와 에이전트](../tutorials/workflows/index.md) 가이드에서는 기능 API를 사용하여 워크플로우를 구축하는 방법에 대한 더 많은 예제를 제공합니다.

### 에이전트

- [기능 API를 사용하여 제로에서부터 리액트 에이전트 만들기](../how-tos/react-agent-from-scratch-functional.ipynb): 기능 API를 사용하여 제로에서 간단한 리액트 에이전트를 만드는 방법을 보여줍니다.
- [다중 에이전트 네트워크 구축 방법](../how-tos/multi-agent-network-functional.ipynb): 기능 API를 사용하여 다중 에이전트 네트워크를 구축하는 방법을 보여줍니다.
- [다중 에이전트 애플리케이션에서 다중 턴 대화 추가 방법 (기능 API)](../how-tos/multi-agent-multi-turn-convo-functional.ipynb): 최종 사용자가 하나 이상의 에이전트와 다중 턴 대화를 진행할 수 있도록 합니다.
