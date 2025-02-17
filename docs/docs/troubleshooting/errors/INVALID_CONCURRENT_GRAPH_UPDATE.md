_한국어로 기계번역됨_

# 유효하지 않은 동시 그래프 업데이트

LangGraph의 [`StateGraph`](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.state.StateGraph)가 여러 노드로부터 상태 속성에 대한 동시 상태 업데이트를 수신했지만 이를 지원하지 않습니다.

이런 상황은 [fanout](https://langchain-ai.github.io/langgraph/how-tos/map-reduce/) 또는 그래프에서의 다른 병렬 실행을 사용할 때 발생할 수 있으며, 다음과 같은 그래프를 정의할 경우 발생할 수 있습니다:

```python hl_lines="2"
class State(TypedDict):
    some_key: str

def node(state: State):
    return {"some_key": "some_string_value"}

def other_node(state: State):
    return {"some_key": "some_string_value"}


builder = StateGraph(State)
builder.add_node(node)
builder.add_node(other_node)
builder.add_edge(START, "node")
builder.add_edge(START, "other_node")
graph = builder.compile()
```

위의 그래프에서 노드가 `{ "some_key": "some_string_value" }`를 반환하면, 이는 `"some_key"`에 대한 상태 값을 `"some_string_value"`로 덮어쓰게 됩니다. 그러나 만약 여러 노드가 e.g. 하나의 단계 내에서 `"some_key"`에 대한 값을 반환하면, 내부 상태를 어떻게 업데이트할지에 대한 불확실성 때문에 그래프는 이 오류를 발생시킵니다.

이 문제를 해결하기 위해, 여러 값을 결합하는 리듀서를 정의할 수 있습니다:

```python hl_lines="5-6"
import operator
from typing import Annotated

class State(TypedDict):
    # operator.add 리듀서 함수는 이것을 추가 전용으로 만듭니다
    some_key: Annotated[list, operator.add]
```

이렇게 하면 병렬 실행된 여러 노드에서 반환된 동일한 키를 처리하는 로직을 정의할 수 있습니다.

## 문제 해결

다음 사항이 이 오류 해결에 도움이 될 수 있습니다:

- 그래프가 노드를 병렬로 실행하는 경우, 관련 상태 키를 리듀서와 함께 정의했는지 확인하세요.