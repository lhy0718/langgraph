_한국어로 기계번역됨_

# 그래프 재귀 제한

귀하의 LangGraph [`StateGraph`](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.state.StateGraph)가 정지 조건에 도달하기 전에 최대 단계 수에 도달했습니다. 이는 종종 아래와 같은 예제 코드로 인한 무한 루프 때문입니다:

```python
class State(TypedDict):
    some_key: str

builder = StateGraph(State)
builder.add_node("a", ...)
builder.add_node("b", ...)
builder.add_edge("a", "b")
builder.add_edge("b", "a")
...

graph = builder.compile()
```

그러나 복잡한 그래프는 자연스럽게 기본 제한에 도달할 수 있습니다.

## 문제 해결

- 그래프가 많은 반복을 거칠 것으로 예상하지 않는 경우, 사이클이 존재할 가능성이 높습니다. 무한 루프에 대한 논리를 확인하세요.
- 복잡한 그래프가 있는 경우, 그래프를 호출할 때 `config` 객체에 더 높은 `recursion_limit` 값을 전달할 수 있습니다. 예:

```python
graph.invoke({...}, {"recursion_limit": 100})
```