_한국어로 기계번역됨_

# 잘못된 그래프 노드 반환 값

LangGraph [`StateGraph`](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.state.StateGraph)가 노드에서 비딕 반환 유형을 받았습니다. 다음은 예시입니다:

```python
class State(TypedDict):
    some_key: str

def bad_node(state: State):
    # "some_key"에 대한 값을 가진 dict를 반환해야 하며, 리스트를 반환해서는 안 됩니다.
    return ["whoops"]

builder = StateGraph(State)
builder.add_node(bad_node)
...

graph = builder.compile()
```

위의 그래프를 호출하면 다음과 같은 오류가 발생합니다:

```python
graph.invoke({ "some_key": "someval" });
```

```
InvalidUpdateError: Expected dict, got ['whoops']
문제 해결을 위해 방문하세요: https://python.langchain.com/docs/troubleshooting/errors/INVALID_GRAPH_NODE_RETURN_VALUE
```

그래프의 노드는 상태에 정의된 하나 이상의 키를 포함하는 dict를 반환해야 합니다.

## 문제 해결

다음은 이 오류를 해결하는 데 도움이 될 수 있습니다:

- 노드에 복잡한 로직이 있는 경우 모든 코드 경로가 정의된 상태에 대해 적절한 dict를 반환하는지 확인하십시오.