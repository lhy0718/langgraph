_한국어로 기계번역됨_

# 중단점

중단점은 그래프 실행을 특정 지점에서 일시 중지하고 단계별로 실행을 진행할 수 있도록 해줍니다. 중단점은 LangGraph의 [**영속성 레이어**](./persistence.md)가 지원하며, 이는 각 그래프 단계 후에 상태를 저장합니다. 중단점은 [**인간 개입 루프**](./human_in_the_loop.md) 워크플로를 활성화하는 데에도 사용될 수 있지만, 이 목적을 위해 [`interrupt` 함수](./human_in_the_loop.md#interrupt) 사용을 권장합니다.

## 요구 사항

중단점을 사용하려면 다음이 필요합니다:

1. [**체크포인터 지정**](persistence.md#checkpoints)하여 각 단계 후 그래프 상태를 저장합니다.
2. [**중단점 설정**](#setting-breakpoints)하여 실행 중지 위치를 지정합니다.
3. **스레드 ID**와 함께 **그래프 실행**하여 중단점에서 실행을 일시 중지합니다.
4. `invoke`/`ainvoke`/`stream`/`astream`을 사용하여 **실행 재개**합니다 (자세한 내용은 [**커맨드 원시**](./human_in_the_loop.md#the-command-primitive) 참조).

## 중단점 설정

중단점을 설정할 수 있는 두 가지 위치가 있습니다:

1. **노드 실행 전** 또는 **후**에 중단점을 설정하여 **컴파일 시간** 또는 **실행 시간**에 설정합니다. 이를 [**정적 중단점**](#static-breakpoints)이라고 합니다.
2. [`NodeInterrupt` 예외](#nodeinterrupt-exception)를 사용하여 **노드 내부**에서 설정합니다.

### 정적 중단점

정적 중단점은 노드가 실행되기 **전** 또는 **후**에 트리거됩니다. 컴파일 시간이나 실행 시간에 `interrupt_before` 및 `interrupt_after`를 지정하여 정적 중단점을 설정할 수 있습니다.

=== "컴파일 시간"

```python
graph = graph_builder.compile(
    interrupt_before=["node_a"],
    interrupt_after=["node_b", "node_c"],
    checkpointer=...,  # 체크포인터 지정
)

thread_config = {
    "configurable": {
        "thread_id": "some_thread"
    }
}

# 중단점까지 그래프 실행
graph.invoke(inputs, config=thread_config)

# 사용자 입력에 따라 그래프 상태 업데이트 (선택 사항)
graph.update_state(update, config=thread_config)

# 그래프 재개
graph.invoke(None, config=thread_config)
```

=== "실행 시간"

```python
graph.invoke(
    inputs,
    config={"configurable": {"thread_id": "some_thread"}},
    interrupt_before=["node_a"],
    interrupt_after=["node_b", "node_c"]
)

thread_config = {
    "configurable": {
        "thread_id": "some_thread"
    }
}

# 중단점까지 그래프 실행
graph.invoke(inputs, config=thread_config)

# 사용자 입력에 따라 그래프 상태 업데이트 (선택 사항)
graph.update_state(update, config=thread_config)

# 그래프 재개
graph.invoke(None, config=thread_config)
```

!!! note

    서브 그래프에서는 실행 시간에 정적 중단점을 설정할 수 없습니다.
    서브 그래프가 있는 경우, 컴파일 시간에 중단점을 설정해야 합니다.

정적 중단점은 그래프 실행을 한 번에 하나의 노드씩 단계적으로 진행하고 싶거나 특정 노드에서 그래프 실행을 일시 중지하고 싶을 때 디버깅에 특히 유용할 수 있습니다.

### `NodeInterrupt` 예외

인간 개입 루프 작업 흐름을 구현하려는 경우 [**`interrupt` 함수 대신 사용**][langgraph.types.interrupt]할 것을 권장합니다. `interrupt` 함수는 사용하기 쉽고 더 유연합니다.

??? node "`NodeInterrupt` 예외"

    개발자는 중단점을 트리거하기 위해 충족해야 하는 일부 *조건*을 정의할 수 있습니다. 이러한 _동적 중단점_ 개념은 개발자가 *특정 조건*에서 그래프를 중단하고자 할 때 유용합니다. 이는 특정 조건에 따라 노드 내에서 발생할 수 있는 특수한 예외인 `NodeInterrupt`를 사용합니다. 예를 들어, `input` 길이가 5자 이상일 때 트리거되는 동적 중단점을 정의할 수 있습니다.

```python
def my_node(state: State) -> State:
    if len(state['input']) > 5:
        raise NodeInterrupt(f"5자보다 긴 입력을 받았습니다: {state['input']}")

    return state
```


    다이나믹 중단점을 유발하는 입력으로 그래프를 실행한 후, 입력으로 `None`을 전달하여 그래프 실행을 다시 시작하려고 가정해 보겠습니다.

```python
# 다이나믹 중단점에 도달한 후 상태를 변경하지 않고 그래프 실행을 계속 시도
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

그래프는 다시 *중단*될 것입니다. 왜냐하면 이 노드는 동일한 그래프 상태로 *재실행*되기 때문입니다. 다이나믹 중단점을 유발하는 조건이 더 이상 충족되지 않도록 그래프 상태를 변경해야 합니다. 따라서 다이나믹 중단점의 조건(< 5 문자)을 충족하는 입력으로 그래프 상태를 간단히 수정하고 노드를 다시 실행할 수 있습니다.

```python
# 다이나믹 중단점을 통과하도록 상태 업데이트
graph.update_state(config=thread_config, values={"input": "foo"})
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

또는 현재 입력을 유지하고 체크를 수행하는 노드(`my_node`)를 건너뛰고 싶다면, `as_node="my_node"`와 함께 `None`을 값으로 전달하여 그래프 업데이트를 수행하면 됩니다. 이렇게 하면 그래프 상태는 업데이트되지 않지만, `my_node`로 업데이트가 실행되어 해당 노드를 건너뛰고 다이나믹 중단점을 우회할 수 있습니다.

```python
# 이 업데이트는 `my_node`를 완전히 건너뜁니다
graph.update_state(config=thread_config, values=None, as_node="my_node")
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

## 추가 자료 📚

- [**개념 안내서: 지속성**](persistence.md): 지속성에 대한 더 많은 컨텍스트를 위한 지속성 안내서를 읽어보세요.
- [**개념 안내서: 사람 중심의 루프**](human_in_the_loop.md): 중단점을 사용하는 LangGraph 애플리케이션에 사람 피드백을 통합하는 것에 대한 더 많은 컨텍스트를 위한 사람 중심의 루프 안내서를 읽어보세요.
- [**과거 그래프 상태 보기 및 업데이트 방법**](../how-tos/human_in_the_loop/time-travel.ipynb): **재플레이** 및 **포크** 작업을 보여주는 그래프 상태 작업을 위한 단계별 지침.
