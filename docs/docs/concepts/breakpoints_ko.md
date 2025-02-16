# 중단점

중단점은 특정 지점에서 그래프 실행을 일시 중지하고 단계별로 실행을 진행할 수 있도록 합니다. 중단점은 각 그래프 단계 후 상태를 저장하는 LangGraph의 [**영속성 계층**](./persistence.md)에 의해 지원됩니다. 중단점은 [**인간-루프**](./human_in_the_loop.md) 작업 흐름을 활성화하는 데에도 사용할 수 있지만, 이 목적을 위해 [`interrupt` 함수](./human_in_the_loop.md#interrupt)를 사용하는 것이 좋습니다.

## 요구사항

중단점을 사용하려면 다음이 필요합니다:

1. **체크포인터**를 [**지정**](persistence.md#checkpoints)하여 각 단계 후 그래프 상태를 저장합니다.
2. 중단점을 [**설정**](#setting-breakpoints)하여 실행이 일시 중지해야 할 위치를 지정합니다.
3. [**스레드 ID**](./persistence.md#threads)를 사용하여 그래프를 **실행**하여 중단점에서 실행을 일시 중지합니다.
4. `invoke`/`ainvoke`/`stream`/`astream`을 사용하여 **실행을 재개**합니다 (자세한 내용은 [**`Command` 원시형**](./human_in_the_loop.md#the-command-primitive) 참조).

## 중단점 설정

중단점을 설정할 수 있는 두 가지 장소가 있습니다:

1. **노드 실행 전후**에 중단점을 설정하여 **컴파일 시간** 또는 **실행 시간**에 설정합니다. 이를 [**정적 중단점**](#static-breakpoints)이라고 합니다.
2. [`NodeInterrupt` 예외](#nodeinterrupt-exception)를 사용하여 **노드 내부**에서 설정합니다.

### 정적 중단점

정적 중단점은 노드를 실행하기 **전**이나 **후**에 트리거됩니다. `interrupt_before` 및 `interrupt_after`를 **"컴파일" 시간** 또는 **실행 시간**에 지정하여 정적 중단점을 설정할 수 있습니다.

=== "컴파일 시간"

    ```python
    graph = graph_builder.compile(
        interrupt_before=["node_a"], 
        interrupt_after=["node_b", "node_c"],
        checkpointer=..., # 체크포인터 지정
    )

    thread_config = {
        "configurable": {
            "thread_id": "some_thread"
        }
    }

    # 중단점까지 그래프 실행
    graph.invoke(inputs, config=thread_config)

    # 사용자 입력에 따라 그래프 상태를 선택적으로 업데이트
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

    # 사용자 입력에 따라 그래프 상태를 선택적으로 업데이트
    graph.update_state(update, config=thread_config)

    # 그래프 재개
    graph.invoke(None, config=thread_config)
    ```

    !!! 주의

        서브 그래프에 대한 **정적 중단점**은 실행 시간에 설정할 수 없습니다. 서브 그래프가 있는 경우 컴파일 시간에 중단점을 설정해야 합니다.

정적 중단점은 그래프 실행을 한 번에 하나의 노드씩 단계별로 진행하려는 경우 또는 특정 노드에서 그래프 실행을 일시 중지하려는 경우 디버깅에 특히 유용할 수 있습니다.

### `NodeInterrupt` 예외

[**대신 `interrupt` 함수**](#the-interrupt-function)를 사용하는 것이 좋습니다. `NodeInterrupt` 예외를 사용하는 것보다 [인간-루프](./human_in_the_loop.md) 작업 흐름을 구현하려는 경우 더 쉽고 유연합니다.

??? 노드 "`NodeInterrupt` 예외"

    개발자는 중단점이 트리거되기 위한 *조건*을 정의할 수 있습니다. 이러한 [동적 중단점](./low_level.md#dynamic-breakpoints)의 개념은 개발자가 *특정 조건* 하에 그래프를 중지하고 싶을 때 유용합니다. 이는 노드 내에서 특정 조건에 따라 발생할 수 있는 특수한 유형의 예외인 `NodeInterrupt`를 사용합니다. 예를 들어, `input`이 5자보다 길어질 때 트리거되는 동적 중단점을 정의할 수 있습니다.

    ```python
    def my_node(state: State) -> State:
        if len(state['input']) > 5:
            raise NodeInterrupt(f"수신된 입력이 5자보다 깁니다: {state['input']}")

        return state
    ```


    다음과 같이 동적 중단점을 트리거하는 입력으로 그래프를 실행한 후, 입력으로 `None`을 전달하여 그래프 실행을 계속 시도해 보겠습니다.

```python
# 동적 중단점에 도달한 후 상태 변경 없이 그래프 실행을 계속 시도
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

그래프는 *다시 중단*될 것입니다. 왜냐하면 이 노드는 동일한 그래프 상태로 *다시 실행*되기 때문입니다. 따라서 동적 중단점을 트리거하는 조건이 더 이상 충족되지 않도록 그래프 상태를 변경해야 합니다. 그래서 우리는 단순히 입력을 동적 중단점의 조건(< 5자)을 충족하는 값으로 업데이트하고 노드를 다시 실행할 수 있습니다.

```python 
# 동적 중단점을 통과하도록 상태 업데이트
graph.update_state(config=thread_config, values={"input": "foo"})
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

다른 방법으로, 현재 입력을 유지하고 조건 검사를 수행하는 노드(`my_node`)를 건너뛰고 싶다면 어떻게 할까요? 이를 위해, `as_node="my_node"`로 그래프 업데이트를 수행하고 값으로 `None`을 전달하면 됩니다. 이렇게 하면 그래프 상태에 대한 업데이트는 없지만 `my_node`로서 업데이트를 실행하여 노드를 효과적으로 건너뛰고 동적 중단점을 우회하게 됩니다.

```python
# 이 업데이트는 `my_node` 노드를 완전히 건너뜁니다
graph.update_state(config=thread_config, values=None, as_node="my_node")
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

## 추가 자료 📚

- [**개념 안내서: 지속성**](persistence.md): 지속성에 대한 맥락을 위해 지속성 안내서를 읽어보세요.
- [**개념 안내서: 사람과의 상호작용**](human_in_the_loop.md): 중단점을 사용하여 LangGraph 애플리케이션에 인간 피드백을 통합하는 방법에 대한 맥락을 위해 사람과의 상호작용 안내서를 읽어보세요.
- [**과거 그래프 상태 보기 및 업데이트하는 방법**](../how-tos/human_in_the_loop/time-travel.ipynb): **재생** 및 **분기** 작업을 시연하는 그래프 상태 작업에 대한 단계별 지침입니다.