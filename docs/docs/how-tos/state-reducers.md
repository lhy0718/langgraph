_한국어로 기계번역됨_

# 노드에서 그래프 상태 업데이트 방법

이 가이드는 LangGraph에서 [상태](../concepts/low_level.md/#state)를 정의하고 업데이트하는 방법을 보여줍니다. 우리는 다음을 시연할 것입니다:

1. 상태를 사용하여 그래프의 [스키마](../concepts/low_level.md/#schema)를 정의하는 방법
2. [리듀서](../concepts/low_level.md/#reducers)를 사용하여 상태 업데이트가 처리되는 방식을 제어하는 방법

우리의 예제에서는 [메시지](../concepts/low_level.md/#messagesstate)를 사용할 것입니다. 이는 여러 LLM 애플리케이션에 대한 상태의 다재다능한 표현을 나타냅니다. 자세한 내용은 우리의 [개념 페이지](../concepts/low_level.md/#working-with-messages-in-graph-state)를 참조하십시오.

## 설정

먼저, langgraph를 설치합시다:

```python
%%capture --no-stderr
%pip install -U langgraph
```

<div class="admonition tip">
     <p class="admonition-title">디버깅을 개선하기 위해 <a href="https://smith.langchain.com">LangSmith</a>를 설정하세요</p>
     <p style="padding-top: 5px;">
         LangSmith에 가입하여 문제를 신속하게 발견하고 LangGraph 프로젝트의 성능을 향상시키세요. LangSmith는 LangGraph로 구축된 LLM 앱을 디버깅, 테스트 및 모니터링하기 위해 추적 데이터를 사용할 수 있게 해줍니다 — <a href="https://docs.smith.langchain.com">문서</a>에서 시작하는 방법을 읽어보세요.
     </p>
 </div>

## 예제 그래프

### 상태 정의
LangGraph의 [상태](../concepts/low_level.md/#state)는 `TypedDict`, `Pydantic` 모델 또는 데이터 클래스로 정의될 수 있습니다. 아래에서는 `TypedDict`를 사용할 것입니다. Pydantic 사용에 대한 자세한 내용은 [이 가이드](../how-tos/state-model.ipynb)를 참조하세요.

기본적으로 그래프는 동일한 입력 및 출력 스키마를 가지며, 상태가 해당 스키마를 결정합니다. 구별된 입력 및 출력 스키마를 정의하는 방법에 대한 자세한 내용은 [이 가이드](../how-tos/input_output_schema.ipynb)를 참조하세요.

간단한 예를 살펴보겠습니다:

```python exec="on" source="above" session="1"
from langchain_core.messages import AnyMessage
from typing_extensions import TypedDict


class State(TypedDict):
    messages: list[AnyMessage]
    extra_field: int
```

이 상태는 [메시지](https://python.langchain.com/docs/concepts/messages/) 객체 목록과 추가 정수 필드를 추적합니다.

### 그래프 구조 정의

하나의 노드가 있는 예제 그래프를 만들어 보겠습니다. 우리의 [노드](../concepts/low_level.md#nodes)는 그래프의 상태를 읽고 이를 업데이트하는 Python 함수입니다. 이 함수의 첫 번째 인수는 항상 상태가 됩니다:

```python exec="on" source="above" session="1"
from langchain_core.messages import AIMessage


def node(state: State):
    messages = state["messages"]
    new_message = AIMessage("Hello!")

    return {"messages": messages + [new_message], "extra_field": 10}
```

이 노드는 단순히 메시지를 메시지 목록에 추가하고 추가 필드를 채웁니다.

!!! 중요

    노드는 상태를 변형하는 대신 상태에 대한 업데이트를 직접 반환해야 합니다.

이제 이 노드를 포함하는 간단한 그래프를 정의해 보겠습니다. 우리는 [StateGraph](../concepts/low_level.md#stategraph)를 사용하여 이 상태에서 작동하는 그래프를 정의합니다. 그런 다음 [add_node](../concepts/low_level.md#messagesstate)를 사용하여 그래프를 채웁니다.

```python exec="on" source="above" session="1"
from langgraph.graph import StateGraph

graph_builder = StateGraph(State)
graph_builder.add_node(node)
graph_builder.set_entry_point("node")
graph = graph_builder.compile()
```

LangGraph는 그래프를 시각화하기 위한 내장 유틸리티를 제공합니다. 우리 그래프를 살펴봅시다. 시각화에 대한 더 자세한 내용은 [이 가이드](../how-tos/visualization.ipynb)를 참조하세요.

```python
from IPython.display import Image, display

display(Image(graph.get_graph().draw_mermaid_png()))
```

![](data:image/png;base64,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)

이 경우, 우리의 그래프는 단일 노드만 실행합니다.

### 그래프 사용

간단한 호출을 진행해 보겠습니다:


```python exec="on" source="above" session="1" result="ansi"
from langchain_core.messages import HumanMessage

result = graph.invoke({"messages": [HumanMessage("안녕하세요")]})
result
```

유의할 점:

- 우리는 상태의 단일 키를 업데이트하여 호출을 시작했습니다.
- 호출 결과로 전체 상태를 받습니다.

편의상, 우리는 [메시지 객체](https://python.langchain.com/docs/concepts/messages/)의 내용을 종종 예쁘게 출력하여 확인합니다:


```python exec="on" source="above" session="1" result="ansi"
for message in result["messages"]:
    message.pretty_print()
```

## 리듀서를 통한 상태 업데이트 처리

상태의 각 키는 업데이트가 어떻게 적용되는지를 제어하는 독립적인 [리듀서](../concepts/low_level.md#reducers) 함수를 가질 수 있습니다. 리듀서 함수가 명시적으로 지정되지 않는 경우, 해당 키에 대한 모든 업데이트가 이를 덮어써야 한다고 가정합니다.

`TypedDict` 상태 스키마의 경우, 상태의 해당 필드를 리듀서 함수로 주석을 달아 리듀서를 정의할 수 있습니다.

이전 예에서, 우리의 노드는 메시지를 추가함으로써 상태의 `"messages"` 키를 업데이트했습니다. 아래에서는 이 키에 자동으로 업데이트가 추가되도록 리듀서를 추가합니다:


```python exec="on" source="above" session="1"
from typing_extensions import Annotated


def add(left, right):
    """`operator` 내장 모듈에서 `add`를 가져올 수도 있습니다."""
    return left + right


class State(TypedDict):
    # 하이라이트 다음 줄
    messages: Annotated[list[AnyMessage], add]
    extra_field: int
```

이제 우리의 노드는 단순화될 수 있습니다:


```python exec="on" source="above" session="1"
def node(state: State):
    new_message = AIMessage("안녕하세요!")
    # 하이라이트 다음 줄
    return {"messages": [new_message], "extra_field": 10}
```


```python exec="on" source="above" session="1" result="ansi"
from langgraph.graph import START


graph = StateGraph(State).add_node(node).add_edge(START, "node").compile()

result = graph.invoke({"messages": [HumanMessage("안녕하세요")]})

for message in result["messages"]:
    message.pretty_print()
```

### MessagesState

실제로 메시지 목록을 업데이트할 때 추가적인 고려 사항이 있습니다:

- 우리는 상태에서 기존 메시지를 업데이트하고 싶을 수 있습니다.
- 우리는 [메시지 형식](../concepts/low_level.md#using-messages-in-your-graph)에 대한 단축어를 수용하고 싶을 수 있습니다, 예를 들어 [OpenAI 형식](https://python.langchain.com/docs/concepts/messages/#openai-format).

LangGraph에는 이러한 고려 사항을 처리하는 내장 리듀서인 `add_messages`가 포함되어 있습니다:


```python exec="on" source="above" session="1"
from langgraph.graph.message import add_messages


class State(TypedDict):
    # 다음 줄 강조
    messages: Annotated[list[AnyMessage], add_messages]
    extra_field: int


def node(state: State):
    new_message = AIMessage("안녕하세요!")
    return {"messages": [new_message], "extra_field": 10}


graph = StateGraph(State).add_node(node).set_entry_point("node").compile()
```


```python exec="on" source="above" session="1" result="ansi"
# 다음 줄 강조
input_message = {"role": "user", "content": "안녕"}

result = graph.invoke({"messages": [input_message]})

for message in result["messages"]:
    message.pretty_print()
```

이는 [챗 모델](https://python.langchain.com/docs/concepts/chat_models/)과 관련된 애플리케이션에 대한 상태의 다재다능한 표현입니다. LangGraph는 편리함을 위해 미리 구축된 `MessagesState`를 포함하고 있어 다음과 같이 사용할 수 있습니다:


```python exec="on" source="above" session="1"
from langgraph.graph import MessagesState


class State(MessagesState):
    extra_field: int
```

## 다음 단계

- [Graph API 기본](index.md#graph-api-basics) 가이드를 계속 진행하세요.
- [상태 관리](index.md#state-management)에 대한 자세한 내용을 참조하세요.
