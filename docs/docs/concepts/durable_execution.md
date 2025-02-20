_한국어로 기계번역됨_

# 내구성 실행

**내구성 실행**은 프로세스나 워크플로우가 주요 지점에서 진행 상황을 저장하여 일시 중지했다가 나중에 중단된 지점에서 정확히 다시 시작할 수 있도록 하는 기술입니다. 이는 사용자가 프로세스를 계속 진행하기 전에 검사, 검증 또는 수정할 수 있는 [인간 참여](./human_in_the_loop.md) 시나리오와, 중단이나 오류를 겪을 수 있는 장기 실행 작업(예: LLM 호출 타임아웃)에 특히 유용합니다. 완료된 작업을 보존함으로써 내구성 실행은 이전 단계를 다시 처리하지 않고도 프로세스를 재개할 수 있게 합니다. 심지어 상당한 지연(예: 일주일 후) 후에도 가능합니다.

LangGraph의 내장 [지속성](./persistence.md) 계층은 워크플로우에 대한 내구성 실행을 제공하여 각 실행 단계의 상태가 내구성 저장소에 저장되도록 보장합니다. 이 기능은 워크플로우가 시스템 오류 또는 [인간 참여](./human_in_the_loop.md) 상호작용으로 인해 중단된 경우에도 마지막 기록된 상태에서 재개할 수 있도록 보장합니다.

!!! 팁

    LangGraph를 체크포인터와 함께 사용하고 있다면 이미 내구성 실행이 활성화된 것입니다. 중단이나 실패가 발생하더라도 언제든지 워크플로우를 일시 중지하고 다시 시작할 수 있습니다.
    내구성 실행을 최대한 활용하려면 워크플로우가 [결정론적](#determinism-and-consistent-replay)이고 [멱등성](#determinism-and-consistent-replay)을 갖도록 설계하고, 모든 부작용이나 비결정적인 작업을 [작업](./functional_api.md#task)으로 감싸야 합니다. [작업](./functional_api.md#task)은 [StateGraph (Graph API)](./low_level.md)와 [Functional API](./functional_api.md) 모두에서 사용할 수 있습니다.

## 요구 사항

LangGraph에서 내구성 실행을 활용하려면 다음이 필요합니다:

1. 워크플로우에서 진행 상황을 저장할 [체크포인터](./persistence.md#checkpointer-libraries)를 지정하여 [지속성](./persistence.md) 기능을 활성화합니다.
2. 워크플로우를 실행할 때 [스레드 식별자](./persistence.md#threads)를 지정합니다. 이는 특정 워크플로우 인스턴스에 대한 실행 기록을 추적합니다.
3. 비결정적 작업(예: 난수 생성)이나 부작용이 있는 작업(예: 파일 쓰기, API 호출)을 [작업][langgraph.func.task]으로 감싸서 워크플로우가 재개될 때 이러한 작업이 특정 실행에 대해 반복되지 않고, 그 대신 결과가 지속성 계층에서 검색되도록 합니다. 자세한 내용은 [결정론과 일관된 재생](#determinism-and-consistent-replay)을 참조하세요.

## 결정론과 일관된 재생

워크플로우 실행을 재개할 때 코드가 실행이 중단된 **같은 코드 라인**에서 **재개되지 않습니다**. 대신, 중단된 지점에서 다시 시작할 적절한 [시작점](#starting-points-for-resuming-workflows)을 식별합니다. 이는 워크플로우가 [시작점](#starting-points-for-resuming-workflows)에서 중단된 지점까지 모든 단계를 재생한다는 의미입니다.

따라서 내구성 실행을 위한 워크플로우를 작성할 때는 비결정적인 작업(예: 난수 생성)과 부작용이 있는 작업(예: 파일 쓰기, API 호출)을 반드시 [작업](./functional_api.md#task)이나 [노드](./low_level.md#nodes)로 감싸야 합니다.

워크플로우가 결정론적이고 일관되게 재생할 수 있도록 하려면 다음 지침을 따르세요:

- **작업 반복 피하기**: [노드](./low_level.md#nodes)에서 부작용이 있는 여러 작업이 포함된 경우, 각 작업을 별도의 **작업**으로 감싸세요. 이렇게 하면 워크플로우가 재개될 때 작업이 반복되지 않고 결과가 지속성 계층에서 검색됩니다.
- **비결정적 작업 캡슐화**: 비결정적 결과를 초래할 수 있는 코드(예: 난수 생성)를 반드시 **작업** 또는 **노드**로 감싸세요. 이렇게 하면 재개 시 워크플로우가 정확히 기록된 단계 순서를 따르고 동일한 결과를 보장합니다.
- **멱등성 작업 사용**: 가능할 경우 부작용(예: API 호출, 파일 쓰기)이 멱등성을 갖도록 보장하세요. 이는 워크플로우에서 실패한 후 작업이 재시도되었을 때, 첫 번째 실행과 동일한 효과를 갖도록 합니다. 이는 데이터 쓰기를 초래하는 작업에서 특히 중요합니다. **작업**이 시작되지만 성공적으로 완료되지 않는 경우, 워크플로우의 재개는 **작업**을 다시 실행하면서 기록된 결과를 사용하여 일관성을 유지합니다. 의도치 않은 중복을 피하기 위해 멱등성 키를 사용하거나 기존 결과를 확인하여 부드럽고 예측 가능한 워크플로우 실행을 보장하세요.

피해야 할 일반적인 함정에 대한 몇 가지 예시는 기능 API의 [일반적인 함정](./functional_api.md#common-pitfalls) 섹션을 참조하세요. 이 섹션은 **작업**을 사용하여 이러한 문제를 피하기 위해 코드를 구성하는 방법을 보여줍니다. 동일한 원칙이 [StateGraph (Graph API)][langgraph.graph.state.StateGraph]에 적용됩니다.

## 노드에서 작업 사용하기

[노드](./low_level.md#nodes)에서 여러 작업이 포함된 경우 각 작업을 개별 노드로 리팩터링하는 것보다 **작업**으로 변환하는 것이 더 쉬울 수 있습니다. 

=== "원본"

```python
from typing import NotRequired
from typing_extensions import TypedDict
import uuid

from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
import requests

# 상태를 나타내는 TypedDict 정의
class State(TypedDict):
    url: str
    result: NotRequired[str]

def call_api(state: State):
    """API 요청을 수행하는 예제 노드."""
    # 다음 줄 강조
    result = requests.get(state['url']).text[:100]  # 부작용
    return {
        "result": result
    }

# StateGraph 빌더 생성 및 call_api 함수의 노드 추가
builder = StateGraph(State)
builder.add_node("call_api", call_api)

# 시작 및 종료 노드를 call_api 노드에 연결
builder.add_edge(START, "call_api")
builder.add_edge("call_api", END)

# 체크포인터 지정
checkpointer = MemorySaver()

# 체크포인터로 그래프 컴파일
graph = builder.compile(checkpointer=checkpointer)

# 스레드 ID가 포함된 구성 정의
thread_id = uuid.uuid4()
config = {"configurable": {"thread_id": thread_id}}

# 그래프 호출
graph.invoke({"url": "https://www.example.com"}, config)
```

=== "작업으로"

    ```python
    from typing import NotRequired
    from typing_extensions import TypedDict
    import uuid

    from langgraph.checkpoint.memory import MemorySaver
    from langgraph.func import task
    from langgraph.graph import StateGraph, START, END
    import requests

    # 상태를 나타내기 위해 TypedDict 정의
    class State(TypedDict):
        urls: list[str]
        result: NotRequired[list[str]]


    @task
    def _make_request(url: str):
        """요청을 수행합니다."""
        # 다음 줄 강조 표시
        return requests.get(url).text[:100]

    def call_api(state: State):
        """API 요청을 수행하는 예제 노드입니다."""
        # 다음 줄 강조 표시
        requests = [_make_request(url) for url in state['urls']]
        results = [request.result() for request in requests]
        return {
            "results": results
        }

    # StateGraph 빌더를 생성하고 call_api 함수에 대한 노드를 추가합니다.
    builder = StateGraph(State)
    builder.add_node("call_api", call_api)

    # 시작 및 끝 노드를 call_api 노드에 연결합니다.
    builder.add_edge(START, "call_api")
    builder.add_edge("call_api", END)

    # 체크포인터 지정
    checkpointer = MemorySaver()

    # 체크포인터로 그래프를 컴파일합니다.
    graph = builder.compile(checkpointer=checkpointer)

    # 스레드 ID가 포함된 구성 정의
    thread_id = uuid.uuid4()
    config = {"configurable": {"thread_id": thread_id}}

    # 그래프 호출
    graph.invoke({"urls": ["https://www.example.com"]}, config)
    ```

## 워크플로우 재개

워크플로우에서 내구성 실행을 활성화한 후 다음 시나리오에 대해 실행을 재개할 수 있습니다:

- **워크플로우 일시 중지 및 재개:** 특정 지점에서 워크플로우를 일시 중지하려면 [interrupt][langgraph.types.interrupt] 함수를 사용하고, 업데이트된 상태로 재개하려면 [Command][langgraph.types.Command] 원시를 사용하세요. 더 자세한 내용은 [**인간이 포함된 루프**](./human_in_the_loop.md)를 참조하세요.
- **실패로부터 복구:** 예외가 발생한 후 마지막 성공 체크포인트에서 워크플로우를 자동으로 재개합니다(예: LLM 제공업체 중단). 이는 동일한 스레드 식별자로 워크플로우를 실행하고 입력값으로 `None`을 제공하는 것을 포함합니다(기능 API에서 이 [예제](./functional_api.md#resuming-after-an-error)를 참조하세요).

## 워크플로우 재개를 위한 시작 지점

* [StateGraph (Graph API)][langgraph.graph.state.StateGraph]를 사용하는 경우 시작 지점은 실행이 중단된 [**노드**](./low_level.md#nodes)의 시작입니다.
* 노드 내에서 서브그래프 호출을 하는 경우 시작 지점은 서브그래프가 중단된 **부모** 노드입니다.
서브그래프 내에서는 실행이 중단된 특정 [**노드**](./low_level.md#nodes)가 시작 지점입니다.
* 기능 API를 사용하는 경우 시작 지점은 실행이 중단된 [**진입점**](./functional_api.md#entrypoint)입니다.