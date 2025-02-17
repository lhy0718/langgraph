_한국어로 기계번역됨_

# 단계 순서 만드는 방법

!!! 정보 "사전 요구 사항"
    이 가이드는 다음에 대한 이해도가 있다고 가정합니다:

    - [그래프 상태 정의 및 업데이트 방법](../how-tos/state-reducers.md)

이 가이드는 간단한 단계 순서를 구성하는 방법을 보여줍니다. 우리는 다음을 시연할 것입니다:

1. 순차적 그래프 만드는 방법
2. 유사한 그래프를 구축하기 위한 내장 단축키.


# 요약

노드의 순서를 추가하기 위해 우리는 [그래프](../concepts/low_level.md#stategraph)의 `.add_node` 및 `.add_edge` 메서드를 사용합니다:

```python
from langgraph.graph import START, StateGraph

graph_builder = StateGraph(State)

# 노드 추가
graph_builder.add_node(step_1)
graph_builder.add_node(step_2)
graph_builder.add_node(step_3)

# 엣지 추가
graph_builder.add_edge(START, "step_1")
graph_builder.add_edge("step_1", "step_2")
graph_builder.add_edge("step_2", "step_3")
```

우리는 또한 내장 단축키인 `.add_sequence`를 사용할 수 있습니다:

```python
graph_builder = StateGraph(State).add_sequence([step_1, step_2, step_3])
graph_builder.add_edge(START, "step_1")
```


<details>
<summary>왜 LangGraph로 애플리케이션 단계를 순서로 나누나요?</summary>

LangGraph는 애플리케이션에 기본적인 지속성 계층을 쉽게 추가할 수 있게 해줍니다.
이로 인해 노드 실행 간 상태를 체크포인트로 만들 수 있으며, 따라서 LangGraph 노드는 다음을 관리합니다:

<ul>
<li>상태 업데이트가 어떻게 [체크포인트화](../concepts/persistence/.md)되는지</li>
<li>인터럽션이 [인간-루프](../concepts/human_in_the_loop/.md) 워크플로우에서 어떻게 재개되는지</li>
<li>LangGraph의 [타임 트래블](../concepts/time-travel/.md) 기능을 사용하여 어떻게 "되감기"와 분기 실행을 할 수 있는지</li>
</ul>

이들은 또한 실행 단계가 어떻게 [스트리밍](../concepts/streaming.md)되는지, 그리고 LangGraph Studio를 사용하여 애플리케이션이 어떻게 시각화되고 디버그되는지를 결정합니다.

</details>

## 설정

먼저, langgraph를 설치합시다:


```python
%%capture --no-stderr
%pip install -U langgraph
```

<div class="admonition tip">
     <p class="admonition-title"><a href="https://smith.langchain.com">LangSmith</a>로 더 나은 디버깅을 설정하세요</p>
     <p style="padding-top: 5px;">
         LangSmith에 가입하여 문제를 빠르게 발견하고 LangGraph 프로젝트의 성능을 향상시킵니다. LangSmith는 LangGraph로 구축된 LLM 앱을 디버그, 테스트 및 모니터링하는 데 사용할 수 있는 추적 데이터를 제공합니다 — <a href="https://docs.smith.langchain.com">문서</a>에서 시작하는 방법에 대한 자세한 내용을 읽어보세요. 
     </p>
 </div>

## 그래프 구축

간단한 사용 예제를 보여줍시다. 우리는 세 단계의 순서를 만들 것입니다:

1. 상태의 키에 값을 채웁니다.
2. 같은 값을 업데이트합니다.
3. 다른 값을 채웁니다.

### 상태 정의

먼저 우리의 [상태](../concepts/low_level.md#state)를 정의합시다. 이것은 [그래프의 스키마](../concepts/low_level.md#schema)를 관리하며, 업데이트를 적용하는 방법을 지정할 수도 있습니다. 더 자세한 내용은 [이 가이드](../how-tos/state-reducers.md)를 참조하세요.

우리의 경우, 두 개의 값을 간단히 추적할 것입니다:


```python
from typing_extensions import TypedDict

class State(TypedDict):
    value_1: str
    value_2: int
```

### 노드 정의

우리 [노드](../concepts/low_level.md#nodes)는 그래프의 상태를 읽고 이를 업데이트하는 파이썬 함수입니다. 이 함수의 첫 번째 인수는 항상 상태입니다:


```python exec="on" source="above" session="1"
def step_1(state: State):
    return {"value_1": "a"}


def step_2(state: State):
    current_value_1 = state["value_1"]
    return {"value_1": f"{current_value_1} b"}


def step_3(state: State):
    return {"value_2": 10}
```

!!! 노트

    상태를 업데이트할 때 각 노드는 업데이트하려는 키의 값만 지정하면 됩니다.

기본적으로 이는 해당 키의 값을 **덮어쓰기** 합니다. 또한 [리듀서](../concepts/low_level.md#reducers)를 사용하여 업데이트가 처리되는 방식을 제어할 수 있습니다—예를 들어, 키에 대한 연속적인 업데이트를 추가할 수 있습니다. 자세한 내용은 [이 가이드](../how-tos/state-reducers.md)를 참조하세요.

### 그래프 정의

우리는 [StateGraph](../concepts/low_level.md#stategraph)를 사용하여 이 상태에서 작동하는 그래프를 정의합니다.

그런 다음 [add_node](../concepts/low_level.md#messagesstate)와 [add_edge](../concepts/low_level.md#edges)를 사용하여 그래프를 채우고 제어 흐름을 정의합니다.


```python exec="on" source="above" session="1"
from langgraph.graph import START, StateGraph

graph_builder = StateGraph(State)

# 노드 추가
graph_builder.add_node(step_1)
graph_builder.add_node(step_2)
graph_builder.add_node(step_3)

# 엣지 추가
graph_builder.add_edge(START, "step_1")
graph_builder.add_edge("step_1", "step_2")
graph_builder.add_edge("step_2", "step_3")
```

!!! 팁 "사용자 정의 이름 지정"

    `.add_node`를 사용하여 노드에 대한 사용자 정의 이름을 지정할 수 있습니다:

    ```python
    graph_builder.add_node("my_node", step_1)
    ```

다음 사항에 유의하세요:

- `.add_edge`는 노드의 이름을 받으며, 함수의 경우 기본적으로 `node.__name__`입니다.
- 그래프의 진입점을 지정해야 합니다. 이를 위해 [START 노드](../concepts/low_level.md#start-node)와 엣지를 추가합니다.
- 더 이상 실행할 노드가 없을 때 그래프는 중단됩니다.

다음으로 우리는 그래프를 [컴파일](../concepts/low_level.md#compiling-your-graph)합니다. 이는 그래프 구조에 대한 몇 가지 기본 검사를 제공합니다(예: 고아 노드 식별). 애플리케이션에 [체크포인터](../concepts/persistence.md)를 통해 지속성을 추가하고 있다면, 여기에서 전달됩니다.


```python exec="on" source="above" session="1"
graph = graph_builder.compile()
```

LangGraph는 그래프를 시각화하기 위한 내장 유틸리티를 제공합니다. 우리의 시퀀스를 검사해 보겠습니다. 시각화에 대한 자세한 내용은 [이 가이드](../how-tos/visualization.ipynb)를 참조하세요.


```python exec="on" source="above" session="1"
from IPython.display import Image, display

display(Image(graph.get_graph().draw_mermaid_png()))
```

![](data:image/png;base64,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)

### 사용법

간단한 호출을 진행해 보겠습니다:


```python exec="on" source="above" session="1" result="ansi"
graph.invoke({"value_1": "c"})
```


다음 사항에 유의하세요:

- 하나의 상태 키에 대한 값을 제공하여 호출을 시작했습니다. 항상 적어도 하나의 키에 대한 값을 제공해야 합니다.
- 우리가 전달한 값은 첫 번째 노드에 의해 덮어졌습니다.
- 두 번째 노드는 값을 업데이트했습니다.
- 세 번째 노드는 다른 값을 할당했습니다.

## 내장 약어

!!! 정보 "전제 조건"
    `.add_sequence`는 `langgraph>=0.2.46`가 필요합니다.


LangGraph는 편리하게 사용할 수 있는 내장 약어 `.add_sequence`를 포함하고 있습니다:


```python exec="on" source="above" session="1" result="ansi"
# highlight-next-line
graph_builder = StateGraph(State).add_sequence([step_1, step_2, step_3])
graph_builder.add_edge(START, "step_1")

graph = graph_builder.compile()

graph.invoke({"value_1": "c"})
```





