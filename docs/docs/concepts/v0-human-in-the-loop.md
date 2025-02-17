_한국어로 기계번역됨_

# 인간-루프

!!! 주의 "대신 `interrupt` 함수를 사용하세요."

    LangGraph 0.2.57부터, 중단점을 설정하는 추천 방법은 [`interrupt` 함수][langgraph.types.interrupt]를 사용하는 것입니다. 이는 **인간-루프** 패턴을 간소화합니다.

    `interrupt` 함수를 사용하는 최신 버전에 대한 수정된 [인간-루프 가이드](./human_in_the_loop.md)를 참조하세요.


인간-루프(또는 "온-더-루프")는 여러 가지 일반적인 사용자 상호작용 패턴을 통해 에이전트의 능력을 향상시킵니다.

일반적인 상호작용 패턴에는 다음이 포함됩니다:

(1) `승인` - 에이전트를 중단하고, 현재 상태를 사용자에게 보여주어 사용자가 행동을 승인하도록 할 수 있습니다.

(2) `편집` - 에이전트를 중단하고, 현재 상태를 사용자에게 보여주어 사용자가 에이전트 상태를 편집할 수 있도록 할 수 있습니다.

(3) `입력` - 그래프 노드를 명시적으로 생성하여 인간 입력을 수집하고 해당 입력을 에이전트 상태에 직접 전달할 수 있습니다.

이러한 상호작용 패턴의 사용 사례에는 다음이 포함됩니다:

(1) `도구 호출 검토` - 에이전트를 중단하여 도구 호출의 결과를 검토하고 수정할 수 있습니다.

(2) `시간 여행` - 에이전트의 과거 행동을 수동으로 재생하거나 분기할 수 있습니다.

## 지속성

모든 이러한 상호작용 패턴은 LangGraph의 내장된 [지속성](./persistence.md) 레이어에 의해 가능해지며, 이는 각 단계에서 그래프 상태의 체크포인트를 작성합니다. 지속성은 그래프가 중단되고 사람이 현재 상태를 검토하거나 편집할 수 있도록 하며, 이후 사람의 입력으로 실행을 재개할 수 있게 합니다.

### 중단점

그래프 흐름의 특정 위치에 [중단점](./breakpoints.md)을 추가하는 것은 인간-루프를 활성화하는 한 가지 방법입니다. 이 경우, 개발자는 워크플로우의 *어디서* 인간 입력이 필요한지를 알고 해당 그래프 노드 이전 또는 이후에 중단점을 배치하면 됩니다.

여기에서는 체크포인터와 우리가 중단하고자 하는 노드인 `step_for_human_in_the_loop` 앞에 중단점을 두고 그래프를 컴파일합니다. 이후 위의 상호작용 패턴 중 하나를 수행하여 사람이 그래프 상태를 편집하면 새로운 체크포인트가 생성됩니다. 새로운 체크포인트는 `thread`에 저장되며, 이곳에서 `None`을 입력으로 전달하여 그래프 실행을 재개할 수 있습니다.

```python
# "step_for_human_in_the_loop" 앞에 체크포인터와 중단점을 두고 그래프 컴파일
graph = builder.compile(checkpointer=checkpointer, interrupt_before=["step_for_human_in_the_loop"])

# 중단점까지 그래프 실행
thread_config = {"configurable": {"thread_id": "1"}}
for event in graph.stream(inputs, thread_config, stream_mode="values"):
    print(event)
    
# 인간 루프가 필요한 작업 수행

# 현재 체크포인트에서 그래프 실행 계속 
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

### 동적 중단점

대안으로, 개발자는 중단점이 촉발되기 위한 몇 가지 *조건*을 정의할 수 있습니다. 이러한 [동적 중단점](./breakpoints.md) 개념은 개발자가 *특정 조건*에서 그래프를 중단하고 싶을 때 유용합니다. 이는 일부 조건에 따라 노드 내에서 발생할 수 있는 특별한 예외인 `NodeInterrupt`를 사용합니다. 예를 들어, 입력이 5자보다 길 때 트리거되는 동적 중단점을 정의할 수 있습니다.

```python
def my_node(state: State) -> State:
    if len(state['input']) > 5:
        raise NodeInterrupt(f"5자보다 긴 입력을 받았습니다: {state['input']}")
    return state
```

우리가 동적 중단점을 촉발하는 입력으로 그래프를 실행하고, 이후 입력으로 `None`을 전달하여 그래프 실행을 재개하려고 한다고 가정해 보겠습니다.

```python
# 동적 중단점에 도달한 후 상태 변경 없이 그래프 실행 계속 시도 
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

이 노드는 동일한 그래프 상태로 다시 실행되기 때문에 그래프는 다시 *중단*될 것입니다. 따라서 동적 중단점을 촉발하는 조건이 더 이상 충족되지 않도록 그래프 상태를 변경해야 합니다. 그러므로 우리는 간단히 그래프 상태를 동적 중단점의 조건(< 5자)을 충족하는 입력으로 수정하고 노드를 다시 실행할 수 있습니다.

```python 
# 동적 중단점을 통과하도록 상태 업데이트
graph.update_state(config=thread_config, values={"input": "foo"})
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

또는, 현재 입력을 유지하고 체크를 수행하는 노드(`my_node`)를 건너뛰고 싶다면 어떻게 해야 할까요? 이를 위해 우리는 `as_node="my_node"`와 함께 그래프 업데이트를 수행하고 값으로 `None`을 전달하면 됩니다. 이렇게 하면 그래프 상태 업데이트가 없지만, `my_node`로 실행되어 해당 노드를 건너뛰고 동적 중단점을 우회하게 됩니다.

```python
# 이 업데이트는 `my_node` 노드를 전혀 건너뛰게 할 것입니다
graph.update_state(config=thread_config, values=None, as_node="my_node")
for event in graph.stream(None, thread_config, stream_mode="values"):
    print(event)
```

자세한 방법은 [가이드](../how-tos/human_in_the_loop/dynamic_breakpoints.ipynb)를 참조하세요!

## 상호작용 패턴

### 승인

![](./img/human_in_the_loop/approval.png)

때때로 우리는 에이전트 실행의 특정 단계를 승인하고 싶습니다. 

우리는 승인하고자 하는 단계 이전의 [중단점](./breakpoints.md)에서 에이전트를 중단할 수 있습니다.

이는 일반적으로 민감한 작업(예: 외부 API 사용 또는 데이터베이스에 쓰기)에 권장됩니다.

지속성을 통해 현재 에이전트 상태와 다음 단계를 사용자에게 검토 및 승인을 위해 노출할 수 있습니다.

승인되면 그래프는 마지막으로 저장된 체크포인트에서 실행을 재개하며, 이는 `thread`에 저장됩니다:

```python
# 승인할 단계 이전에 체크포인터와 중단점을 포함하여 그래프를 컴파일합니다.
graph = builder.compile(checkpointer=checkpointer, interrupt_before=["node_2"])

# 중단점까지 그래프를 실행합니다.
for event in graph.stream(inputs, thread, stream_mode="values"):
    print(event)

# ... 인간의 승인을 받습니다 ...

# 승인되면 마지막으로 저장된 체크포인트에서 그래프 실행을 계속합니다.
for event in graph.stream(None, thread, stream_mode="values"):
    print(event)
```

이것을 수행하는 자세한 방법은 [우리의 가이드](../how-tos/human_in_the_loop/breakpoints.ipynb)를 참조하세요!

### 편집

![](./img/human_in_the_loop/edit_graph_state.png)

때때로 우리는 에이전트의 상태를 검토하고 편집하고 싶습니다.

승인과 마찬가지로 확인하고 싶은 단계 이전에 에이전트를 [중단점](./breakpoints.md)에서 중단할 수 있습니다.

현재 상태를 사용자에게 보여주고 사용자가 에이전트 상태를 편집할 수 있도록 합니다.

예를 들어, 에이전트가 실수를 했을 때 이를 수정하는 데 사용할 수 있습니다(예: 아래의 도구 호출 섹션 참조).

현재 체크포인트를 포크하여 그래프 상태를 편집할 수 있으며, 이는 `thread`에 저장됩니다.

그런 다음 이전과 같이 포크된 체크포인트에서 그래프를 계속 진행할 수 있습니다.

```python
# 검토할 단계 이전에 체크포인트와 중단점을 포함하여 그래프를 컴파일합니다.
graph = builder.compile(checkpointer=checkpointer, interrupt_before=["node_2"])

# 중단점까지 그래프를 실행합니다.
for event in graph.stream(inputs, thread, stream_mode="values"):
    print(event)

# 상태를 검토하고, 편집하기로 결정하고, 새로운 상태로 포크된 체크포인트를 생성합니다.
graph.update_state(thread, {"state": "new state"})

# 포크된 체크포인트에서 그래프 실행을 계속합니다.
for event in graph.stream(None, thread, stream_mode="values"):
    print(event)
```

이것을 수행하는 자세한 방법은 [이 가이드](../how-tos/human_in_the_loop/edit-graph-state.ipynb)를 참조하세요!

### 입력

![](./img/human_in_the_loop/wait_for_input.png)

때때로 우리는 그래프의 특정 단계에서 명시적으로 인간 입력을 받고 싶습니다.

이를 위해 지정된 그래프 노드를 생성할 수 있습니다(예: 예제 다이어그램의 `human_input`).

승인 및 편집과 마찬가지로 이 노드 이전에 에이전트를 [중단점](./breakpoints.md)에서 중단할 수 있습니다.

그런 다음 입력이 포함된 상태 업데이트를 수행할 수 있으며, 이는 상태 편집 시와 같습니다.

하지만 한 가지를 추가합니다:

상태 업데이트와 함께 `as_node=human_input`을 사용하여 상태 업데이트가 *노드로 처리되어야 한다*고 명시할 수 있습니다.

これは微妙ですが重要です:

편집의 경우 사용자는 그래프 상태를 편집할지 여부를 결정합니다.

입력의 경우 수집을 위한 그래프의 노드를 명시적으로 정의합니다!

그런 다음 인간 입력이 포함된 상태 업데이트는 *이 노드로서* 실행됩니다.

```python
# 인간 입력을 수집하기 위해 체크포인트와 중단점 이전에 그래프를 컴파일합니다.
graph = builder.compile(checkpointer=checkpointer, interrupt_before=["human_input"])

# 중단점까지 그래프를 실행합니다.
for event in graph.stream(inputs, thread, stream_mode="values"):
    print(event)

# 마치 human_input 노드인 것처럼 사용자 입력으로 상태를 업데이트합니다.
graph.update_state(thread, {"user_input": user_input}, as_node="human_input")

# human_input 노드로 생성된 체크포인트에서 그래프 실행을 계속합니다.
for event in graph.stream(None, thread, stream_mode="values"):
    print(event)
```

이것을 수행하는 자세한 방법은 [이 가이드](../how-tos/human_in_the_loop/wait-user-input.ipynb)를 참조하세요!

## 사용 사례

### 도구 호출 검토

일부 사용자 상호작용 패턴은 위의 아이디어를 결합합니다.

예를 들어, 많은 에이전트가 [도구 호출](https://python.langchain.com/docs/how_to/tool_calling/)을 사용하여 결정을 내립니다.

도구 호출은 두 가지를 정확하게 수행해야 하기 때문에 도전 과제가 됩니다: 

(1) 호출할 도구의 이름 

(2) 도구에 전달할 인수

도구 호출이 정확하더라도 우리는 또한 신중함을 적용할 수 있습니다: 

(3) 도구 호출이 승인해야 할 민감한 작업일 수 있습니다. 

이러한 점을 염두에 두고, 우리는 위의 아이디어를 결합하여 도구 호출에 대한 사람-검토(사람 개입)를 생성할 수 있습니다.

```python
# 검토 단계 이전에 체크포인터와 중단점을 사용하여 그래프를 컴파일합니다.
graph = builder.compile(checkpointer=checkpointer, interrupt_before=["human_review"])

# 그래프를 중단점까지 실행합니다.
for event in graph.stream(inputs, thread, stream_mode="values"):
    print(event)
    
# 도구 호출을 검토하고 필요하면 업데이트합니다. 이를 human_review 노드로 설정합니다.
graph.update_state(thread, {"tool_call": "updated tool call"}, as_node="human_review")

# 그렇지 않으면 도구 호출을 승인하고 수정 없이 그래프 실행을 계속합니다.

# 그래프 실행을 계속하는 방법:
# (1) human_review에서 생성된 forked checkpoint 또는 
# (2) 도구 호출이 원래 이루어졌을 때 저장된 체크포인트(인편에 수정 없음)
for event in graph.stream(None, thread, stream_mode="values"):
    print(event)
```

자세한 방법에 대해서는 [이 가이드](../how-tos/human_in_the_loop/review-tool-calls.ipynb)를 참조하세요!

### 시간 여행

에이전트와 작업할 때, 우리는 종종 그들의 의사 결정 과정을 면밀히 살펴보기를 원합니다: 

(1) 원하는 최종 결과에 도달하더라도 그 결과에 이르게 한 추론을 검토하는 것은 종종 중요합니다.

(2) 에이전트가 실수를 할 때, 그 이유를 이해하는 것은 종종 가치가 있습니다.

(3) 위의 두 경우 모두에서, 대안적인 의사 결정 경로를 수동으로 탐색하는 것이 유용합니다.

이 모든 개념을 통칭하여 `시간 여행(time-travel)`이라고 하며, 이는 `재생(replaying)`과 `분기(forking)`으로 구성됩니다.

#### 재생

![](./img/human_in_the_loop/replay.png)

때때로 우리는 에이전트의 과거 행동을 단순히 재생하고 싶습니다.

위에서는 그래프의 현재 상태(또는 체크포인트)에서 에이전트를 실행하는 경우를 보여주었습니다.

우리는 `thread`와 함께 입력으로 `None`을 간단히 전달함으로써 수행할 수 있습니다.

```
thread = {"configurable": {"thread_id": "1"}}
for event in graph.stream(None, thread, stream_mode="values"):
    print(event)
```

이제 우리는 체크포인트 ID를 전달하여 *특정* 체크포인트에서 과거 행동을 재생하도록 수정할 수 있습니다.

특정 체크포인트 ID를 얻으려면, 스레드에 있는 모든 체크포인트를 쉽게 가져와서 원하는 체크포인트로 필터할 수 있습니다.

```python
all_checkpoints = []
for state in app.get_state_history(thread):
    all_checkpoints.append(state)
```

각 체크포인트는 고유한 ID를 가지며, 이를 사용하여 특정 체크포인트에서 재생할 수 있습니다.

체크포인트를 검토한 결과, `xxx`라는 체크포인트에서 재생하고 싶다고 가정합니다.

그래프를 실행할 때 체크포인트 ID를 전달하면 됩니다.

```python
config = {'configurable': {'thread_id': '1', 'checkpoint_id': 'xxx'}}
for event in graph.stream(None, config, stream_mode="values"):
    print(event)
```
 
중요한 점은 그래프가 이전에 실행된 체크포인트를 알고 있다는 것입니다. 

그래서 이전에 실행된 노드를 재실행하기보다는 재생할 것입니다.

재생과 관련한 추가 컨셉 가이드는 [여기](https://langchain-ai.github.io/langgraph/concepts/persistence/#replay)를 참조하십시오.

시간 여행에 대한 자세한 방법은 [이 가이드](../how-tos/human_in_the_loop/time-travel.ipynb)를 참조하세요!

#### 포킹

![](./img/human_in_the_loop/forking.png)

가끔 우리는 에이전트의 과거 행동을 포킹하고 그래프를 통해 다양한 경로를 탐색하고 싶습니다.

위에서 논의한 `편집`은 그래프의 *현재* 상태에 대해 이를 수행하는 *정확한* 방법입니다!

하지만 그래프의 *과거* 상태를 포킹하고 싶다면 어떻게 해야 할까요?

예를 들어, 특정 체크포인트 `xxx`를 편집하고 싶다고 가정해 보겠습니다.

우리는 그래프의 상태를 업데이트할 때 이 `checkpoint_id`를 전달합니다.

```python
config = {"configurable": {"thread_id": "1", "checkpoint_id": "xxx"}}
graph.update_state(config, {"state": "updated state"}, )
```

이렇게 하면 새로운 포크된 체크포인트 `xxx-fork`가 생성되며, 이로부터 그래프를 실행할 수 있습니다.

```python
config = {'configurable': {'thread_id': '1', 'checkpoint_id': 'xxx-fork'}}
for event in graph.stream(None, config, stream_mode="values"):
    print(event)
```

포킹과 관련된 맥락에 대한 [이 추가 개념 가이드](https://langchain-ai.github.io/langgraph/concepts/persistence/#update-state)를 참조하세요.

타임 트래블을 수행하는 방법에 대한 자세한 안내는 [이 가이드](../how-tos/human_in_the_loop/time-travel.ipynb)를 참고하세요!
