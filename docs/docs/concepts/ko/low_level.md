# LangGraph 용어집

## 그래프

LangGraph의 핵심은 에이전트 작업 흐름을 그래프로 모델링하는 것입니다. 에이전트의 동작을 정의하기 위해 다음의 세 가지 주요 구성 요소를 사용합니다:

1. [`State`](#state): 애플리케이션의 현재 스냅샷을 나타내는 공유 데이터 구조입니다. Python 타입이면 무엇이든 될 수 있지만, 일반적으로 `TypedDict`나 Pydantic `BaseModel`이 사용됩니다.

2. [`Nodes`](#nodes): 에이전트의 논리를 인코딩한 Python 함수입니다. 현재 `State`를 입력으로 받아서 계산을 수행하거나 부수 효과를 발생시키고, 업데이트된 `State`를 반환합니다.

3. [`Edges`](#edges): 현재 `State`를 기반으로 어떤 `Node`를 다음에 실행할지 결정하는 Python 함수입니다. 조건부 분기나 고정된 전환이 있을 수 있습니다.

`Nodes`와 `Edges`를 조합하여 시간에 따라 `State`를 발전시키는 복잡하고 반복적인 작업 흐름을 만들 수 있습니다. 하지만 진정한 강점은 LangGraph가 `State`를 어떻게 관리하는지에 있습니다. 다시 말해, `Nodes`와 `Edges`는 단지 Python 함수일 뿐입니다 — 그 안에는 LLM이나 일반적인 Python 코드가 있을 수 있습니다.

간단히 말해서: _노드는 작업을 수행하고, 엣지는 무엇을 할지 결정합니다_.

LangGraph의 기본 그래프 알고리즘은 [메시지 전달](https://en.wikipedia.org/wiki/Message_passing)을 사용하여 일반 프로그램을 정의합니다. Node가 작업을 완료하면 메시지를 하나 이상의 엣지를 통해 다른 노드로 전달합니다. 이러한 수신 노드는 그 후 자신의 함수를 실행하고 결과 메시지를 다음 노드로 전달하며 이 과정이 계속됩니다. Google의 [Pregel](https://research.google/pubs/pregel-a-system-for-large-scale-graph-processing/) 시스템에서 영감을 받아, 프로그램은 별도의 "슈퍼 스텝"으로 진행됩니다.

슈퍼 스텝은 그래프 노드를 한 번 반복하는 것과 같습니다. 병렬로 실행되는 노드는 동일한 슈퍼 스텝에 속하며, 순차적으로 실행되는 노드는 별도의 슈퍼 스텝에 속합니다. 그래프 실행이 시작되면 모든 노드는 `inactive` 상태로 시작합니다. 노드는 새로운 메시지를 받으면 `active` 상태가 되어 해당 함수를 실행하고 업데이트를 반환합니다. 각 슈퍼 스텝이 끝나면, 메시지를 받지 않은 노드는 `inactive` 상태로 표시되어 `halt` 투표를 하게 됩니다. 그래프 실행은 모든 노드가 `inactive` 상태이고, 메시지가 더 이상 전송되지 않을 때 종료됩니다.

### StateGraph

`StateGraph` 클래스는 주요 그래프 클래스입니다. 이 클래스는 사용자 정의 `State` 객체로 매개변수화됩니다.

### 그래프 컴파일

그래프를 구축하려면 먼저 [state](#state)를 정의하고, 그 다음에 [nodes](#nodes)와 [edges](#edges)를 추가하고, 마지막으로 이를 컴파일해야 합니다. 컴파일이 정확히 무엇을 의미하고 왜 필요한지에 대해 살펴보겠습니다.

컴파일은 꽤 간단한 단계입니다. 그래프 구조에 대한 몇 가지 기본 검사를 제공합니다 (고아 노드 없음 등). 또한 [checkpointers](./persistence.md)나 [breakpoints](#breakpoints)와 같은 실행 시간 인자를 지정할 수 있는 곳입니다. 그래프를 컴파일하려면 `.compile` 메서드를 호출하기만 하면 됩니다:

```python
graph = graph_builder.compile(...)
```

그래프를 사용하기 전에 **반드시** 컴파일해야 합니다.

## State

그래프를 정의할 때 가장 먼저 해야 할 일은 `State`를 정의하는 것입니다. `State`는 그래프의 [스키마](#schema)와 `reducer` 함수들로 구성됩니다. `State`의 스키마는 그래프의 모든 `Nodes`와 `Edges`에 대한 입력 스키마가 되며, `TypedDict`나 `Pydantic` 모델이 될 수 있습니다. 모든 `Nodes`는 `State`에 업데이트를 내보내며, 그 업데이트는 지정된 `reducer` 함수에 의해 적용됩니다.

### 스키마

그래프의 스키마를 지정하는 주요 방법은 `TypedDict`를 사용하는 것입니다. 하지만 우리는 또한 그래프 상태로 Pydantic `BaseModel`을 사용하여 **기본값**과 추가 데이터 유효성 검사를 할 수 있습니다.

기본적으로, 그래프는 입력 및 출력 스키마가 동일합니다. 이를 변경하려면 명시적으로 입력 및 출력 스키마를 지정할 수도 있습니다. 이는 많은 키가 있고 일부는 입력용이고 다른 일부는 출력용일 때 유용합니다. 사용하는 방법에 대한 [노트북](../how-tos/input_output_schema.ipynb)을 참조하세요.

#### 여러 개의 스키마

일반적으로 모든 그래프 노드는 단일 스키마와 통신합니다. 이는 동일한 상태 채널을 읽고 쓰게 됨을 의미합니다. 하지만 이를 더 세밀하게 제어하고자 하는 경우도 있습니다:

- 내부 노드는 그래프의 입력/출력에서 필요하지 않은 정보를 전달할 수 있습니다.
- 또한 그래프의 입력/출력에 대해 다른 스키마를 사용할 수도 있습니다. 예를 들어, 출력에는 관련 있는 하나의 출력 키만 포함될 수 있습니다.

그래프 내에서 노드가 내부 노드 간 통신을 위해 개인 상태 채널에 쓸 수 있도록 하려면, `PrivateState`와 같은 개인 스키마를 정의할 수 있습니다. 이에 대한 자세한 내용은 [이 노트북](../how-tos/pass_private_state.ipynb)을 참조하세요.

또한 그래프에 대해 명시적인 입력 및 출력 스키마를 정의할 수도 있습니다. 이 경우, 그래프 작업에 관련된 _모든_ 키를 포함하는 "내부" 스키마를 정의합니다. 하지만 그래프의 입력과 출력을 제한하기 위해 `input`과 `output` 스키마를 "내부" 스키마의 하위 집합으로 정의합니다. 자세한 내용은 [이 노트북](../how-tos/input_output_schema.ipynb)을 참조하세요.

예시를 살펴보겠습니다:

```python
class InputState(TypedDict):
    user_input: str

class OutputState(TypedDict):
    graph_output: str

class OverallState(TypedDict):
    foo: str
    user_input: str
    graph_output: str

class PrivateState(TypedDict):
    bar: str

def node_1(state: InputState) -> OverallState:
    # OverallState에 작성
    return {"foo": state["user_input"] + " name"}

def node_2(state: OverallState) -> PrivateState:
    # OverallState에서 읽고, PrivateState에 작성
    return {"bar": state["foo"] + " is"}

def node_3(state: PrivateState) -> OutputState:
    # PrivateState에서 읽고, OutputState에 작성
    return {"graph_output": state["bar"] + " Lance"}

builder = StateGraph(OverallState,input=InputState,output=OutputState)
builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
builder.add_node("node_3", node_3)
builder.add_edge(START, "node_1")
builder.add_edge("node_1", "node_2")
builder.add_edge("node_2", "node_3")
builder.add_edge("node_3", END)

graph = builder.compile()
graph.invoke({"user_input":"My"})
{'graph_output': 'My name is Lance'}
```

여기서 주목해야 할 두 가지 중요한 점이 있습니다:

1. 우리는 `state: InputState`를 `node_1`에 입력 스키마로 전달합니다. 그러나 `foo` 채널에 대해 `OverallState`에 작성합니다. 왜 입력 스키마에 포함되지 않은 상태 채널에 쓸 수 있을까요? 이는 노드가 _그래프 상태의 모든 상태 채널에 쓸 수 있기 때문입니다._ 그래프 상태는 초기화 시 정의된 상태 채널의 합집합이며, 여기에는 `OverallState`와 `InputState`, `OutputState`가 포함됩니다.

2. 우리는 `StateGraph(OverallState,input=InputState,output=OutputState)`로 그래프를 초기화합니다. 그렇다면 `node_2`에서 `PrivateState`에 어떻게 쓸 수 있을까요? `StateGraph` 초기화 시 `PrivateState`가 전달되지 않았는데 어떻게 그래프가 이 스키마에 접근할 수 있을까요? 노드는 _추가적인 상태 채널을 선언할 수 있기 때문입니다._ 이 경우, `PrivateState` 스키마가 정의되어 있어 `bar`를 그래프의 새로운 상태 채널로 추가하고 쓸 수 있습니다.

### 리듀서

리듀서는 노드에서 업데이트된 값이 `State`에 어떻게 적용되는지 이해하는 데 중요합니다. `State`의 각 키에는 고유한 리듀서 함수가 있습니다. 만약 리듀서 함수가 명시적으로 지정되지 않으면, 해당 키에 대한 모든 업데이트는 그 값을 덮어쓰는 것으로 간주됩니다. 리듀서에는 몇 가지 유형이 있으며, 기본 리듀서 유형부터 살펴보겠습니다:

#### 기본 리듀서

다음 두 예시에서는 기본 리듀서를 사용하는 방법을 보여줍니다:

**예시 A:**

```python
from typing_extensions import TypedDict

class State(TypedDict):
    foo: int
    bar: list[str]
```

이 예시에서는 어떤 키에도 리듀서 함수가 지정되지 않았습니다. 그래프의 입력이 `{"foo": 1, "bar": ["hi"]}`라고 가정해 보겠습니다. 첫 번째 `Node`가 `{"foo": 2}`를 반환한다고 가정해 보겠습니다. 이것은 상태 업데이트로 처리됩니다. `Node`가 전체 `State` 스키마를 반환할 필요는 없고, 업데이트만 반환하면 됩니다. 이 업데이트를 적용한 후, `State`는 `{"foo": 2, "bar": ["hi"]}`가 됩니다. 두 번째 노드가 `{"bar": ["bye"]}`를 반환하면 `State`는 `{"foo": 2, "bar": ["bye"]}`가 됩니다.

**예시 B:**

```python
from typing import Annotated
from typing_extensions import TypedDict
from operator import add

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]
```

이 예시에서는 두 번째 키인 `bar`에 대해 리듀서 함수(`operator.add`)를 지정했습니다. 첫 번째 키는 변경되지 않습니다. 입력이 `{"foo": 1, "bar": ["hi"]}`일 때, 첫 번째 `Node`가 `{"foo": 2}`를 반환한다고 가정해 보겠습니다. 업데이트가 적용된 후 `State`는 `{"foo": 2, "bar": ["hi"]}`가 됩니다. 두 번째 노드가 `{"bar": ["bye"]}`를 반환하면 `State`는 `{"foo": 2, "bar": ["hi", "bye"]}`로 변경됩니다. 여기서 `bar` 키는 두 개의 리스트를 더하여 업데이트됩니다.

### 그래프 상태에서 메시지 사용

#### 왜 메시지를 사용할까요?

대부분의 최신 LLM 제공자는 메시지 목록을 입력으로 받는 채팅 모델 인터페이스를 제공합니다. 특히 LangChain의 [`ChatModel`](https://python.langchain.com/docs/concepts/#chat-models)은 `Message` 객체 목록을 입력으로 받습니다. 메시지 객체는 `HumanMessage`(사용자 입력)나 `AIMessage`(LLM 응답) 등 다양한 형태로 존재합니다. 메시지 객체에 대해 더 읽으려면 [이](https://python.langchain.com/docs/concepts/#messages) 개념 가이드를 참조하세요.

#### 그래프에서 메시지 사용하기

많은 경우, 그래프 상태에서 이전 대화 내역을 메시지 목록으로 저장하는 것이 유용합니다. 이를 위해, 그래프 상태에 `Message` 객체 목록을 저장하는 키(채널)를 추가하고, 이를 리듀서 함수로 주석 처리할 수 있습니다(아래 예시의 `messages` 키 참고). 리듀서 함수는 노드가 업데이트를 보낼 때 그래프 상태의 메시지 목록을 어떻게 업데이트할지 알려주는 역할을 합니다. 리듀서를 지정하지 않으면, 모든 상태 업데이트가 가장 최근에 제공된 값으로 메시지 목록을 덮어쓰게 됩니다. 기존 목록에 메시지를 추가하려면 `operator.add`를 리듀서로 사용할 수 있습니다.

하지만, 그래프 상태에서 메시지를 수동으로 업데이트하려는 경우(예: 인간이 개입하는 경우)도 있을 수 있습니다. 이 경우, `operator.add`를 사용하면 그래프에 보내는 수동 상태 업데이트가 기존 메시지 목록에 추가되는 방식이므로 기존 메시지를 업데이트하는 것이 아니라 새로 추가하게 됩니다. 이를 피하려면 메시지 ID를 추적하고 기존 메시지를 덮어쓸 수 있는 리듀서를 사용해야 합니다. 이를 위해, 미리 만들어진 `add_messages` 함수를 사용할 수 있습니다. 새로운 메시지에 대해서는 기존 목록에 단순히 추가되지만, 기존 메시지에 대해서는 올바르게 업데이트를 처리합니다.

#### 직렬화

메시지 ID를 추적하는 것 외에도, `add_messages` 함수는 상태 업데이트가 `messages` 채널에 도달할 때 LangChain `Message` 객체로 메시지를 역직렬화하려고 시도합니다. LangChain 직렬화/역직렬화에 대한 자세한 내용은 [여기](https://python.langchain.com/docs/how_to/serialization/)에서 확인할 수 있습니다. 이렇게 하면 다음과 같은 형식으로 그래프 입력/상태 업데이트를 보낼 수 있습니다:

```python
# 이 형식은 지원됩니다
{"messages": [HumanMessage(content="message")]}

# 이 형식도 지원됩니다
{"messages": [{"type": "human", "content": "message"}]}
```

`add_messages`를 사용할 때 상태 업데이트가 항상 LangChain `Message`로 역직렬화되므로, 메시지 속성에 액세스할 때 점 표기법을 사용해야 합니다. 예를 들어, `state["messages"][-1].content`와 같이 접근할 수 있습니다. 아래는 `add_messages`를 리듀서 함수로 사용하는 그래프 예시입니다.

```python
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages
from typing import Annotated
from typing_extensions import TypedDict

class GraphState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
```

#### MessagesState

그래프 상태에서 메시지 목록을 유지하는 것이 매우 일반적이기 때문에, `MessagesState`라는 미리 만들어진 상태가 있습니다. 이는 메시지를 쉽게 사용할 수 있게 해줍니다. `MessagesState`는 단일 `messages` 키를 가지고 있으며, 이 키는 `AnyMessage` 객체의 목록이고, `add_messages` 리듀서를 사용합니다. 일반적으로 그래프에서 추적해야 할 상태는 메시지뿐만 아니라 더 많기 때문에, 사람들이 이 상태를 서브클래스하여 추가 필드를 더 많이 정의하기도 합니다. 예를 들어:

```python
from langgraph.graph import MessagesState

class State(MessagesState):
    documents: list[str]
```

## 노드

LangGraph에서 노드는 일반적으로 Python 함수(동기 또는 비동기)로, **첫 번째** 위치 인자는 [state](#state)이며, **두 번째** 위치 인자는 선택적으로 "config"로, 여기에는 선택적인 [configurable parameters](#configuration)가 포함됩니다(예: `thread_id`).

`NetworkX`와 유사하게, 이 노드를 그래프에 추가하려면 [add_node](https://langgraph.docs/langgraph.graph.StateGraph.add_node) 메서드를 사용합니다:

```python
from langchain_core.runnables import RunnableConfig
from langgraph.graph import StateGraph

builder = StateGraph(dict)

def my_node(state: dict, config: RunnableConfig):
    print("노드 안: ", config["configurable"]["user_id"])
    return {"results": f"안녕하세요, {state['input']}!"}

# 두 번째 인자는 선택 사항
def my_other_node(state: dict):
    return state

builder.add_node("my_node", my_node)
builder.add_node("other_node", my_other_node)
...
```

배경에서, 함수들은 [RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html#langchain_core.runnables.base.RunnableLambda)로 변환되며, 이는 함수에 배치 및 비동기 지원을 추가하고, 기본적인 추적 및 디버깅 기능도 제공합니다.

만약 이름을 지정하지 않고 노드를 그래프에 추가하면, 해당 함수 이름과 동일한 기본 이름이 할당됩니다.

```python
builder.add_node(my_node)
# 그런 다음 이 노드를 참조하여 엣지를 만들 수 있습니다: `"my_node"`
```

### `START` 노드

`START` 노드는 그래프에 사용자 입력을 보낸 노드를 나타내는 특별한 노드입니다. 이 노드를 참조하는 주요 목적은 어떤 노드를 먼저 호출할지 결정하는 것입니다.

```python
from langgraph.graph import START

graph.add_edge(START, "node_a")
```

### `END` 노드

`END` 노드는 터미널 노드를 나타내는 특별한 노드입니다. 이 노드는 작업이 완료된 후에 더 이상 실행할 작업이 없는 엣지를 나타낼 때 참조됩니다.

```
from langgraph.graph import END

graph.add_edge("node_a", END)
```

## 엣지

엣지는 논리가 어떻게 라우팅되고 그래프가 멈추는지 결정하는 부분입니다. 이는 에이전트가 작동하는 방식과 서로 다른 노드들이 어떻게 통신하는지에 큰 역할을 합니다. 엣지에는 몇 가지 주요 유형이 있습니다:

- **일반 엣지**: 한 노드에서 다른 노드로 직접 이동합니다.
- **조건부 엣지**: 다음에 이동할 노드를 결정하기 위한 함수를 호출합니다.
- **입력 포인트**: 사용자 입력이 도달했을 때 호출할 첫 번째 노드를 정의합니다.
- **조건부 입력 포인트**: 사용자 입력이 도달했을 때 호출할 첫 번째 노드를 결정하기 위한 함수를 호출합니다.

노드는 **여러 개의 나가는 엣지**를 가질 수 있습니다. 만약 노드가 여러 개의 나가는 엣지를 가지고 있으면, **모든** 목적지 노드는 다음 슈퍼 스텝의 일환으로 병렬로 실행됩니다.

### 일반 엣지

노드 A에서 노드 B로 **항상** 이동하려면, [add_edge][langgraph.graph.StateGraph.add_edge] 메서드를 직접 사용할 수 있습니다.

```python
graph.add_edge("node_a", "node_b")
```

### 조건부 엣지

1개 이상의 엣지로 **옵션으로** 라우팅하거나 종료하고자 할 경우, [add_conditional_edges][langgraph.graph.StateGraph.add_conditional_edges] 메서드를 사용할 수 있습니다. 이 메서드는 노드 이름과 해당 노드가 실행된 후 호출할 "라우팅 함수"를 받습니다:

```python
graph.add_conditional_edges("node_a", routing_function)
```

노드처럼, `routing_function`은 현재 `state`를 받아 값을 반환합니다.

기본적으로, `routing_function`의 반환값은 다음에 상태를 보낼 노드의 이름(또는 노드 목록)으로 사용됩니다. 이 모든 노드는 다음 슈퍼 스텝의 일환으로 병렬로 실행됩니다.

옵션으로, `routing_function`의 출력을 다음 노드의 이름으로 매핑하는 딕셔너리를 제공할 수 있습니다.

```python
graph.add_conditional_edges("node_a", routing_function, {True: "node_b", False: "node_c"})
```

!!! tip
상태 업데이트와 라우팅을 하나의 함수로 결합하려면 조건부 엣지 대신 [`Command`](#command)를 사용하는 것이 좋습니다.

### 입력 포인트

입력 포인트는 그래프가 시작될 때 실행될 첫 번째 노드입니다. 가상 [`START`][langgraph.constants.START] 노드에서 첫 번째 실행할 노드로 [`add_edge`][langgraph.graph.StateGraph.add_edge] 메서드를 사용하여 그래프에 진입하는 위치를 지정할 수 있습니다.

```python
from langgraph.graph import START

graph.add_edge(START, "node_a")
```

### 조건부 입력 포인트

조건부 입력 포인트는 사용자 지정 로직에 따라 다른 노드에서 시작할 수 있게 해줍니다. 이를 위해 가상 [`START`][langgraph.constants.START] 노드에서 [`add_conditional_edges`][langgraph.graph.StateGraph.add_conditional_edges] 메서드를 사용할 수 있습니다.

```python
from langgraph.graph import START

graph.add_conditional_edges(START, routing_function)
```

옵션으로, `routing_function`의 출력을 다음 노드의 이름으로 매핑하는 딕셔너리를 제공할 수 있습니다.

```python
graph.add_conditional_edges(START, routing_function, {True: "node_b", False: "node_c"})
```

### `Send`

기본적으로, `Nodes`와 `Edges`는 미리 정의되어 있으며 동일한 공유 상태에서 작업합니다. 그러나 특정 엣지가 미리 알려지지 않거나 서로 다른 버전의 `State`를 동시에 존재하게 해야 할 경우가 있을 수 있습니다. 이 디자인 패턴에서 흔한 예는 `map-reduce` 디자인 패턴입니다. 이 패턴에서는 첫 번째 노드가 객체 목록을 생성하고, 그 목록에 다른 노드를 적용하려고 할 때 유용합니다. 객체의 개수는 미리 알 수 없으며(즉, 엣지의 개수가 미리 정해지지 않음), 입력 `State`는 각 생성된 객체마다 달라야 합니다.

이 디자인 패턴을 지원하기 위해, LangGraph는 조건부 엣지에서 [`Send`](langgraph.types.Send) 객체를 반환하는 것을 지원합니다. `Send`는 두 개의 인수를 받습니다: 첫 번째는 노드의 이름이고, 두 번째는 해당 노드로 전달할 상태입니다.

```python
def continue_to_jokes(state: OverallState):
    return [Send("generate_joke", {"subject": s}) for s in state['subjects']]

graph.add_conditional_edges("node_a", continue_to_jokes)
```

## `Command`

제어 흐름(엣지)과 상태 업데이트(노드)를 결합하는 것이 유용할 수 있습니다. 예를 들어, 상태 업데이트를 수행하고 **다음에 이동할 노드를 결정하는** 작업을 **동일한 노드에서** 수행하고자 할 때 유용합니다. LangGraph는 노드 함수에서 [`Command`](langgraph.types.Command) 객체를 반환하여 이를 처리할 수 있습니다:

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    return Command(
        # 상태 업데이트
        update={"foo": "bar"},
        # 제어 흐름
        goto="my_other_node"
    )
```

`Command`를 사용하면 동적 제어 흐름을 구현할 수도 있습니다(이는 [조건부 엣지](#conditional-edges)와 동일):

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    if state["foo"] == "bar":
        return Command(update={"foo": "baz"}, goto="my_other_node")
```

!!! 중요

    `Command`를 노드 함수에서 반환할 때는 노드가 라우팅할 노드 목록을 반환 타입 주석으로 명시해야 합니다. 예: `Command[Literal["my_other_node"]]`. 이는 그래프 렌더링에 필요하며, LangGraph가 `my_node`가 `my_other_node`로 이동할 수 있음을 알려줍니다.

[`Command` 사용법에 대한 전체 예시](../how-tos/command.ipynb)를 참조하세요.

### 언제 조건부 엣지 대신 `Command`를 사용해야 하나요?

상태 업데이트와 **제어 흐름**을 **둘 다** 처리해야 할 때는 `Command`를 사용하세요. 예를 들어, [다중 에이전트 핸드오프](./multi_agent.md#handoffs)를 구현할 때, 다른 에이전트로 라우팅하고 그 에이전트에게 정보를 전달하는 것이 중요할 때 유용합니다.

[조건부 엣지](#conditional-edges)는 상태를 업데이트하지 않고 노드 간에 조건적으로 라우팅할 때 사용하세요.

### 부모 그래프의 노드로 네비게이션

[하위 그래프](#subgraphs)를 사용하고 있다면, 하위 그래프의 노드에서 부모 그래프의 다른 노드로 이동하고 싶을 수 있습니다. 이를 위해, `Command`에서 `graph=Command.PARENT`를 지정할 수 있습니다:

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    return Command(
        update={"foo": "bar"},
        goto="other_subgraph",  # `other_subgraph`는 부모 그래프의 노드
        graph=Command.PARENT
    )
```

!!! 참고

    `graph`를 `Command.PARENT`로 설정하면 가장 가까운 부모 그래프로 이동합니다.

!!! 중요 "상태 업데이트와 `Command.PARENT`"

    하위 그래프 노드에서 부모 그래프 노드로 상태 업데이트를 보낼 때, 부모 그래프와 하위 그래프 모두에서 공유되는 키에 대해 업데이트를 보내면, **부모 그래프 상태에서 해당 키에 대한 리듀서를 정의해야** 합니다. [예시](../how-tos/command.ipynb#navigating-to-a-node-in-a-parent-graph)를 참조하세요.

이 방식은 특히 [다중 에이전트 핸드오프](./multi_agent.md#handoffs)를 구현할 때 유용합니다.

### 도구 내에서 사용하기

일반적인 사용 사례는 도구 내에서 그래프 상태를 업데이트하는 것입니다. 예를 들어, 고객 지원 애플리케이션에서는 대화 초기에 계좌 번호나 ID를 기반으로 고객 정보를 조회할 수 있습니다. 도구에서 그래프 상태를 업데이트하려면, 도구에서 `Command(update={"my_custom_key": "foo", "messages": [...]})`를 반환할 수 있습니다:

```python
@tool
def lookup_user_info(tool_call_id: Annotated[str, InjectedToolCallId], config: RunnableConfig):
    """사용자 정보를 조회하여 질문을 더 잘 도와줍니다."""
    user_info = get_user_info(config.get("configurable", {}).get("user_id"))
    return Command(
        update={
            # 상태 키 업데이트
            "user_info": user_info,
            # 메시지 기록 업데이트
            "messages": [ToolMessage("사용자 정보를 성공적으로 조회했습니다.", tool_call_id=tool_call_id)]
        }
    )
```

!!! 중요

    도구에서 `Command`를 반환할 때 `messages`(또는 메시지 기록에 사용되는 상태 키)를 반드시 포함해야 합니다. `messages` 목록은 반드시 `ToolMessage`를 포함해야 합니다. 이는 도구 호출 후 LLM 제공자가 AI 메시지 뒤에 도구 결과 메시지가 필요하기 때문에 유효한 메시지 기록을 생성하는 데 필요합니다.

도구가 `Command`를 통해 상태를 업데이트하는 경우, 자동으로 `Command` 객체를 반환하고 이를 그래프 상태로 전파하는 미리 만들어진 [`ToolNode`](langgraph.prebuilt.tool_node.ToolNode)를 사용하는 것이 좋습니다. 도구를 호출하는 커스텀 노드를 작성하는 경우, 도구에서 반환된 `Command` 객체를 노드에서 상태 업데이트로 수동으로 전파해야 합니다.

### Human-in-the-loop

`Command`는 인간이 개입하는 워크플로우에서 중요한 부분입니다: `interrupt()`를 사용하여 사용자 입력을 수집하고, 이후 `Command(resume="User input")`을 사용하여 입력을 제공하고 실행을 재개할 수 있습니다. 더 자세한 정보는 [이 개념 가이드](./human_in_the_loop.md)를 참조하세요.

## 지속성

LangGraph는 에이전트의 상태에 대한 내장된 지속성을 제공하며, 이를 위해 [checkpointers][langgraph.checkpoint.base.BaseCheckpointSaver]를 사용합니다. Checkpointers는 그래프 상태의 스냅샷을 각 슈퍼스텝에서 저장하여 언제든지 재개할 수 있게 해줍니다. 이를 통해 인간 개입, 메모리 관리 및 장애 내성을 지원하는 기능을 제공합니다. 실행 후 적절한 `get` 및 `update` 메서드를 사용하여 그래프의 상태를 직접 조작할 수도 있습니다. 더 자세한 사항은 [지속성 개념 가이드](./persistence.md)를 참조하세요.

## 스레드

LangGraph에서 스레드는 그래프와 사용자 간의 개별 세션 또는 대화를 나타냅니다. 체크포인팅을 사용할 때, 단일 대화의 턴(심지어 단일 그래프 실행 내의 단계)들은 고유한 스레드 ID로 구성됩니다.

## 저장소

LangGraph는 [BaseStore][langgraph.store.base.BaseStore] 인터페이스를 통해 내장된 문서 저장소를 제공합니다. 체크포인터와 달리, 저장소는 데이터를 구성하는 데 커스텀 네임스페이스를 사용합니다. 이를 통해 스레드 간 지속성을 지원하며, 에이전트가 장기적인 기억을 유지하고, 과거 상호작용에서 배우고, 시간이 지남에 따라 지식을 축적할 수 있습니다. 일반적인 사용 사례에는 사용자 프로필 저장, 지식 베이스 구축, 모든 스레드에서 전역 설정 관리 등이 포함됩니다.

## 그래프 마이그레이션

LangGraph는 그래프 정의(노드, 엣지 및 상태)의 마이그레이션을 쉽게 처리할 수 있습니다. 체크포인터를 사용하여 상태를 추적하는 경우에도 마찬가지입니다.

- 그래프의 끝에 있는 스레드(즉, 중단되지 않은 경우)에서는 그래프의 전체 토폴로지를 변경할 수 있습니다(즉, 모든 노드와 엣지를 제거, 추가, 이름 변경 등).
- 현재 중단된 스레드에 대해서는 노드를 이름 변경/제거하는 것 외의 모든 토폴로지 변경이 지원됩니다(이 스레드가 더 이상 존재하지 않는 노드를 실행하려고 할 수 있기 때문입니다) — 이 문제로 막히면 연락을 주시면 해결을 우선적으로 고려하겠습니다.
- 상태 수정의 경우, 키 추가 및 제거에 대해 완전한 이전 및 향후 호환성이 제공됩니다.
- 이름이 변경된 상태 키는 기존 스레드에서 저장된 상태를 잃게 됩니다.
- 타입이 호환되지 않는 방식으로 변경된 상태 키는 변경 전의 상태가 있는 스레드에서 문제가 발생할 수 있습니다 — 이 문제가 막히면 연락을 주시면 해결을 우선적으로 고려하겠습니다.

## 구성

그래프를 생성할 때, 그래프의 일부가 구성 가능하도록 마크할 수도 있습니다. 이는 모델이나 시스템 프롬프트를 쉽게 전환할 수 있도록 할 때 자주 사용됩니다. 이를 통해 단일 "인지 아키텍처"(그래프)를 생성하되, 여러 다른 인스턴스를 만들 수 있습니다.

그래프를 생성할 때 `config_schema`를 선택적으로 지정할 수 있습니다.

```python
class ConfigSchema(TypedDict):
    llm: str

graph = StateGraph(State, config_schema=ConfigSchema)
```

그런 다음 이 구성을 `configurable` 구성 필드를 사용하여 그래프에 전달할 수 있습니다.

```python
config = {"configurable": {"llm": "anthropic"}}

graph.invoke(inputs, config=config)
```

이 구성을 노드 내부에서 액세스하고 사용할 수 있습니다:

```python
def node_a(state, config):
    llm_type = config.get("configurable", {}).get("llm", "openai")
    llm = get_llm(llm_type)
    ...
```

[구성에 대한 전체 설명을 보려면](../how-tos/configuration.ipynb)을 참조하세요.

### 재귀 한도

재귀 한도는 그래프가 단일 실행 동안 실행할 수 있는 최대 [슈퍼 스텝](#graphs)의 수를 설정합니다. 한도가 초과되면 LangGraph는 `GraphRecursionError`를 발생시킵니다. 기본적으로 이 값은 25단계로 설정되어 있습니다. 재귀 한도는 런타임에 설정할 수 있으며, `.invoke`/`.stream`에 있는 구성 딕셔너리를 통해 전달됩니다. 중요하게도, `recursion_limit`은 독립적인 `config` 키이며 다른 사용자 정의 구성처럼 `configurable` 키 안에 전달되어서는 안 됩니다. 아래 예시를 확인하세요:

```python
graph.invoke(inputs, config={"recursion_limit": 5, "configurable":{"llm": "anthropic"}})
```

재귀 한도가 어떻게 작동하는지에 대해 더 배우려면 [이 가이드](https://langchain-ai.github.io/langgraph/how-tos/recursion-limit/)를 참조하세요.

## `interrupt`

[interrupt](../reference/types.md/#langgraph.types.interrupt) 함수는 **특정 지점에서** 그래프를 일시 중지하여 사용자 입력을 수집하는 데 사용됩니다. `interrupt` 함수는 인터럽트 정보를 클라이언트로 전달하여 개발자가 사용자 입력을 수집하고, 그래프 상태를 검증하거나, 실행을 재개하기 전에 결정을 내릴 수 있게 합니다.

```python
from langgraph.types import interrupt

def human_approval_node(state: State):
    ...
    answer = interrupt(
        # 이 값은 클라이언트로 전송됩니다.
        # JSON으로 직렬화 가능한 값일 수 있습니다.
        {"question": "계속해도 괜찮나요?"},
    )
    ...
```

그래프를 재개하려면 `interrupt` 함수에서 반환된 값으로 `resume` 키가 설정된 [`Command`](#command) 객체를 그래프에 전달합니다.

**인간 개입** 워크플로우에서 `interrupt` 사용에 대해 더 배우려면 [이 개념 가이드](./human_in_the_loop.md)를 참조하세요.

## 중단점

중단점은 특정 지점에서 그래프 실행을 일시 중지하고 실행을 한 단계씩 진행할 수 있게 합니다. 중단점은 LangGraph의 [**지속성 계층**](./persistence.md)을 기반으로 하며, 각 그래프 단계 후 상태를 저장합니다. 중단점은 [**인간 개입**](./human_in_the_loop.md) 워크플로우에도 사용할 수 있지만, 이 목적을 위해서는 [`interrupt` 함수](#interrupt-function)를 사용하는 것이 더 좋습니다.

중단점에 대해 더 배우려면 [중단점 개념 가이드](./breakpoints.md)를 참조하세요.

## 하위 그래프

하위 그래프는 다른 그래프의 [노드](#nodes)로 사용되는 [그래프](#graphs)입니다. 이는 LangGraph에 적용된 고전적인 캡슐화 개념에 불과합니다. 하위 그래프를 사용하는 이유는 다음과 같습니다:

- [다중 에이전트 시스템](./multi_agent.md) 구축

- 여러 그래프에서 노드 집합을 재사용하려는 경우, 일부 상태를 공유할 수 있는 노드를 하위 그래프에서 한 번 정의하고 여러 부모 그래프에서 사용 가능

- 서로 다른 팀이 그래프의 다른 부분을 독립적으로 작업해야 하는 경우, 각 부분을 하위 그래프로 정의하고 하위 그래프의 인터페이스(입력 및 출력 스키마)를 존중하는 한, 부모 그래프는 하위 그래프의 세부 사항을 알지 않고도 구축 가능

하위 그래프를 부모 그래프에 추가하는 방법은 두 가지가 있습니다:

- 컴파일된 하위 그래프를 노드로 추가: 이는 부모 그래프와 하위 그래프가 상태 키를 공유하고 상태를 변환할 필요가 없을 때 유용합니다.

```python
builder.add_node("subgraph", subgraph_builder.compile())
```

- 하위 그래프를 호출하는 함수로 노드를 추가: 이는 부모 그래프와 하위 그래프가 다른 상태 스키마를 가지고 있고, 하위 그래프를 호출하기 전후에 상태를 변환해야 할 때 유용합니다.

```python
subgraph = subgraph_builder.compile()

def call_subgraph(state: State):
    return subgraph.invoke({"subgraph_key": state["parent_key"]})

builder.add_node("subgraph", call_subgraph)
```

각각의 예를 살펴보겠습니다.

### 컴파일된 그래프로서

하위 그래프 노드를 생성하는 가장 간단한 방법은 [컴파일된 하위 그래프](#compiling-your-graph)를 직접 사용하는 것입니다. 이렇게 할 때, 부모 그래프와 하위 그래프의 [상태 스키마](#state)가 최소한 하나의 키를 공유하는 것이 **중요**합니다. 만약 부모 그래프와 하위 그래프가 공유하는 키가 없다면, 대신 하위 그래프를 [호출하는 함수](#as-a-function)를 작성해야 합니다.

!!! 참고
하위 그래프 노드에 추가적인 키를 전달하면(공유된 키 외에), 이 키들은 하위 그래프 노드에서 무시됩니다. 마찬가지로, 하위 그래프에서 반환된 추가적인 키들도 부모 그래프에서 무시됩니다.

```python
from langgraph.graph import StateGraph
from typing import TypedDict

class State(TypedDict):
    foo: str

class SubgraphState(TypedDict):
    foo: str  # 부모 그래프 상태와 공유되는 키
    bar: str

# 하위 그래프 정의
def subgraph_node(state: SubgraphState):
    # 하위 그래프 노드는 공유된 "foo" 키를 통해 부모 그래프와 통신할 수 있습니다.
    return {"foo": state["foo"] + "bar"}

subgraph_builder = StateGraph(SubgraphState)
subgraph_builder.add_node(subgraph_node)
...
subgraph = subgraph_builder.compile()

# 부모 그래프 정의
builder = StateGraph(State)
builder.add_node("subgraph", subgraph)
...
graph = builder.compile()
```

### 함수로서

하위 그래프를 완전히 다른 스키마로 정의하고 싶을 때가 있을 수 있습니다. 이 경우, 하위 그래프를 호출하는 노드 함수를 생성할 수 있습니다. 이 함수는 하위 그래프를 호출하기 전에 부모 상태를 하위 그래프 상태로 [변환](../how-tos/subgraph-transform-state.ipynb)해야 하며, 하위 그래프에서 반환된 결과를 부모 상태로 변환하여 상태 업데이트를 반환해야 합니다.

```python
class State(TypedDict):
    foo: str

class SubgraphState(TypedDict):
    # 부모 그래프 상태와 공유되지 않는 키들
    bar: str
    baz: str

# 하위 그래프 정의
def subgraph_node(state: SubgraphState):
    return {"bar": state["bar"] + "baz"}

subgraph_builder = StateGraph(SubgraphState)
subgraph_builder.add_node(subgraph_node)
...
subgraph = subgraph_builder.compile()

# 부모 그래프 정의
def node(state: State):
    # 상태를 하위 그래프 상태로 변환
    response = subgraph.invoke({"bar": state["foo"]})
    # 응답을 부모 상태로 변환
    return {"foo": response["bar"]}

builder = StateGraph(State)
# 컴파일된 하위 그래프 대신 `node` 함수 사용
builder.add_node(node)
...
graph = builder.compile()
```

## 시각화

그래프가 더 복잡해질수록 그래프를 시각화할 수 있는 기능이 필요할 때가 많습니다. LangGraph는 그래프를 시각화할 수 있는 몇 가지 내장 기능을 제공합니다. 자세한 내용은 [이 가이드](../how-tos/visualization.ipynb)를 참조하세요.

## 스트리밍

LangGraph는 실행 중에 그래프 노드에서의 스트리밍 업데이트, LLM 호출에서의 토큰 스트리밍 등을 포함한 스트리밍을 기본적으로 지원합니다. 스트리밍에 대한 더 많은 정보는 [이 개념 가이드](./streaming.md)를 참조하세요.
