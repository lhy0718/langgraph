_한국어로 기계번역됨_

# 사람 개입 루프

!!! tip "이 가이드는 새로운 `interrupt` 함수를 사용합니다."

    LangGraph 0.2.57부터, 중단점을 설정하는 권장 방법은 **사람 개입 루프** 패턴을 단순화하는 [`interrupt` 함수][langgraph.types.interrupt]를 사용하는 것입니다.

    정적 중단점과 `NodeInterrupt` 예외에 의존했던 이전 버전의 개념적 가이드를 찾고 계신다면, [여기](v0-human-in-the-loop.md)에서 확인할 수 있습니다.

**사람 개입 루프**(또는 "사이에서") 워크플로는 자동화된 프로세스에 인간의 입력을 통합하여 주요 단계에서 결정, 검증, 또는 수정을 가능하게 합니다. 이는 모델이 때때로 부정확성을 생성할 수 있는 **LLM 기반 애플리케이션**에서 특히 유용합니다. 준수, 의사 결정 또는 콘텐츠 생성과 같은 낮은 오류 허용 시나리오에서는 인간의 개입이 모델 출력을 검토, 수정 또는 무시할 수 있게 하여 신뢰성을 보장합니다.

## 사용 사례

LLM 기반 애플리케이션에서 **사람 개입 루프** 워크플로의 주요 사용 사례는 다음과 같습니다:

1. [**🛠️ 도구 호출 검토하기**](#review-tool-calls): 인간은 LLM이 요청한 도구 호출을 도구 실행 전에 검토, 수정 또는 승인할 수 있습니다.
2. **✅ LLM 출력 검증하기**: 인간은 LLM이 생성한 콘텐츠를 검토, 수정 또는 승인할 수 있습니다.
3. **💡 맥락 제공하기**: LLM이 명확한 사안이나 추가 세부정보를 위해 인간의 입력을 요청하도록 허용하거나, 다중 턴 대화를 지원합니다.

## `interrupt`

LangGraph의 [`interrupt` 함수][langgraph.types.interrupt]는 그래프를 특정 노드에서 일시 중지하고, 정보를 인간에게 제공하며, 그들의 입력으로 그래프를 다시 시작하는 기능을 통해 사람 개입 루프 워크플로를 가능하게 합니다. 이 함수는 승인, 수정 또는 추가 입력 수집과 같은 작업에 유용합니다. [`interrupt` 함수][langgraph.types.interrupt]는 인간이 제공한 값으로 그래프를 다시 시작하기 위해 [`Command`](../reference/types.md#langgraph.types.Command) 객체와 함께 사용됩니다.

```python
from langgraph.types import interrupt

def human_node(state: State):
    value = interrupt(
        # 인간에게 제시할 수 있는 JSON 직렬화 가능 값.
        # 예: 질문 또는 텍스트 조각 또는 상태의 일부 키들
        {
           "text_to_revise": state["some_text"]
        }
    )
    # 인간의 입력으로 상태를 업데이트하거나 입력에 따라 그래프 경로를 결정합니다.
    return {
        "some_text": value
    }

graph = graph_builder.compile(
    checkpointer=checkpointer # `interrupt`가 작동하기 위해 필요합니다.
)

# 중단점까지 그래프 실행
thread_config = {"configurable": {"thread_id": "some_id"}}
graph.invoke(some_input, config=thread_config)

# 인간의 입력으로 그래프 재개
graph.invoke(Command(resume=value_from_human), config=thread_config)
```

```pycon
{'some_text': '수정된 텍스트'}
```

!!! 경고
중단은 강력하고 편리합니다. 그러나 개발자 경험 측면에서 Python의 input() 함수와 유사할 수 있지만, 중단 지점에서 자동으로 실행을 재개하지는 않기 때문에 중요합니다. 대신, 중단이 사용된 전체 노드를 다시 실행합니다.
이러한 이유로, 중단은 일반적으로 노드 시작 부분이나 전용 노드에 배치하는 것이 가장 좋습니다. 더 자세한 내용은 [중단에서 재개하기](#how-does-resuming-from-an-interrupt-work) 섹션을 읽어보시기 바랍니다.

??? "전체 코드"

      그래프에서 `interrupt`를 사용하는 방법에 대한 전체 예제입니다. 실제로 코드를 보고 싶으시면 확인해 보세요.

      ```python
      from typing import TypedDict
      import uuid

      from langgraph.checkpoint.memory import MemorySaver
      from langgraph.constants import START
      from langgraph.graph import StateGraph
      from langgraph.types import interrupt, Command

      class State(TypedDict):
         """그래프 상태."""
         some_text: str

      def human_node(state: State):
         value = interrupt(
            # 인간에게 보여줄 JSON 직렬화 가능한 값.
            # 예를 들어, 질문이나 텍스트 조각 또는 상태에 있는 키 집합
            {
               "text_to_revise": state["some_text"]
            }
         )
         return {
            # 인간의 입력으로 상태 업데이트
            "some_text": value
         }


      # 그래프 빌드
      graph_builder = StateGraph(State)
      # 그래프에 인간 노드 추가
      graph_builder.add_node("human_node", human_node)
      graph_builder.add_edge(START, "human_node")

      # `interrupt`가 작동하려면 체크포인터가 필요합니다.
      checkpointer = MemorySaver()
      graph = graph_builder.compile(
         checkpointer=checkpointer
      )

      # 그래프를 실행하기 위해 스레드 ID를 전달합니다.
      thread_config = {"configurable": {"thread_id": uuid.uuid4()}}

      # `__interrupt__` 정보를 직접 노출하기 위해 stream() 사용.
      for chunk in graph.stream({"some_text": "원본 텍스트"}, config=thread_config):
         print(chunk)

      # Command를 사용하여 재개
      for chunk in graph.stream(Command(resume="편집된 텍스트"), config=thread_config):
         print(chunk)
      ```

      ```pycon
      {'__interrupt__': (
            Interrupt(
               value={'question': '텍스트를 수정해 주세요', 'some_text': '원본 텍스트'},
               resumable=True,
               ns=['human_node:10fe492f-3688-c8c6-0d0a-ec61a43fecd6'],
               when='during'
            ),
         )
      }
      {'human_node': {'some_text': '편집된 텍스트'}}
      ```

## 요구 사항

그래프에서 `interrupt`를 사용하려면 다음이 필요합니다.

1. [**체크포인터 지정**](persistence.md#checkpoints): 각 단계 후 그래프 상태를 저장합니다.
2. **적절한 위치에서 `interrupt()` 호출**: 예시는 [디자인 패턴](#design-patterns) 섹션에서 확인할 수 있습니다.
3. **그래프 실행**: `interrupt`가 호출될 때까지 [**스레드 ID**](./persistence.md#threads)를 사용하여 실행합니다.
4. **실행 재개**: `invoke`/`ainvoke`/`stream`/`astream` 사용 (자세한 내용은 [**Command 원시**](#the-command-primitive) 참조).

## 디자인 패턴

인간-루프 작업 흐름에서 수행할 수 있는 일반적으로 세 가지 다른 **작업**이 있습니다.

1. **승인 또는 거부**: API 호출과 같은 중요한 단계 전에 그래프를 일시 중지하여 작업을 검토하고 승인합니다. 작업이 거부되면 그래프가 해당 단계를 실행하지 않도록 방지하고 대체 작업을 취할 수 있습니다. 이 패턴은 종종 인간의 입력에 따라 그래프를 **라우팅**하는 것과 관련이 있습니다.
2. **그래프 상태 편집**: 그래프를 일시 중지하고 그래프 상태를 검토 및 편집합니다. 이는 실수를 수정하거나 추가 정보를 사용하여 상태를 업데이트하는 데 유용합니다. 이 패턴은 종종 인간의 입력으로 상태를 **업데이트**하는 것과 관련이 있습니다.
3. **입력 요청**: 그래프의 특정 단계에서 인간의 입력을 명시적으로 요청합니다. 이는 추가 정보를 수집하거나 에이전트의 의사 결정 과정을 알리기 위해 또는 **다중 턴 대화**를 지원하는 데 유용합니다.

아래에서는 이러한 **작업**을 사용하여 구현할 수 있는 다양한 디자인 패턴을 보여줍니다.

### 승인 또는 거부

<figure markdown="1">
![image](img/human_in_the_loop/approve-or-reject.png){: style="max-height:400px"}
<figcaption>인간의 승인 또는 거부에 따라 그래프는 작업을 진행하거나 대체 경로를 선택할 수 있습니다.</figcaption>
</figure>

API 호출과 같은 중요한 단계 전에 그래프를 일시 중지하여 작업을 검토하고 승인합니다. 작업이 거부되면 그래프가 해당 단계를 실행하지 않도록 방지하고 대체 작업을 취할 수 있습니다.

```python
from typing import Literal
from langgraph.types import interrupt, Command

def human_approval(state: State) -> Command[Literal["some_node", "another_node"]]:
    is_approved = interrupt(
        {
            "question": "이것이 맞습니까?",
            # 검토하고 승인해야 할 출력을 보여줍니다.
            "llm_output": state["llm_output"]
        }
    )

    if is_approved:
        return Command(goto="some_node")
    else:
        return Command(goto="another_node")

# 그래프에 노드를 적절한 위치에 추가하고 관련 노드에 연결합니다.
graph_builder.add_node("human_approval", human_approval)
graph = graph_builder.compile(checkpointer=checkpointer)

# 그래프를 실행하고 인터럽트에 도달하면 그래프가 일시 중지됩니다.
# 승인 또는 거부로 재개하십시오.
thread_config = {"configurable": {"thread_id": "some_id"}}
graph.invoke(Command(resume=True), config=thread_config)
```

자세한 예제는 [도구 호출 검토 방법](../how-tos/human_in_the_loop/review-tool-calls.ipynb)을 참조하세요.

### 상태 검토 및 편집

<figure markdown="1">
![image](img/human_in_the_loop/edit-graph-state-simple.png){: style="max-height:400px"}
<figcaption>사람은 그래프의 상태를 검토하고 편집할 수 있습니다. 이는 실수를 수정하거나 추가 정보로 상태를 업데이트하는 데 유용합니다.
</figcaption>
</figure>

```python
from langgraph.types import interrupt

def human_editing(state: State):
    ...
    result = interrupt(
        # 클라이언트에게 표시할 인터럽트 정보.
        # 모든 JSON 직렬화 가능한 값이 될 수 있습니다.
        {
            "task": "LLM의 출력을 검토하고 필요한 수정을 해주세요.",
            "llm_generated_summary": state["llm_generated_summary"]
        }
    )

    # 편집된 텍스트로 상태를 업데이트합니다.
    return {
        "llm_generated_summary": result["edited_text"]
    }

# 그래프에 노드를 적절한 위치에 추가하고 관련 노드에 연결합니다.
graph_builder.add_node("human_editing", human_editing)
graph = graph_builder.compile(checkpointer=checkpointer)

...

# 그래프를 실행하고 인터럽트에 도달하면 그래프가 일시 중지됩니다.
# 편집된 텍스트로 재개하십시오.
thread_config = {"configurable": {"thread_id": "some_id"}}
graph.invoke(
    Command(resume={"edited_text": "수정된 텍스트"}),
    config=thread_config
)
```

자세한 예제는 [사용자 입력 대기 방법](../how-tos/human_in_the_loop/wait-user-input.ipynb)을 참조하세요.

### 도구 호출 검토

<figure markdown="1">
![image](img/human_in_the_loop/tool-call-review.png){: style="max-height:400px"}
<figcaption>사람은 진행하기 전에 LLM의 출력을 검토하고 편집할 수 있습니다. 이는 LLM이 요청한 도구 호출이 민감할 수 있거나 인간의 감독이 필요한 애플리케이션에서 특히 중요합니다.
</figcaption>
</figure>

```python
def human_review_node(state) -> Command[Literal["call_llm", "run_tool"]]:
    # 우리가 Command(resume=<human_review>)를 통해 제공할 값입니다.
    human_review = interrupt(
        {
            "question": "정확합니까?",
            # 리뷰를 위한 도구 호출 표면화
            "tool_call": tool_call
        }
    )

    review_action, review_data = human_review

    # 도구 호출을 승인하고 계속 진행
    if review_action == "continue":
        return Command(goto="run_tool")

    # 도구 호출을 수동으로 수정한 후 계속 진행
    elif review_action == "update":
        ...
        updated_msg = get_updated_msg(review_data)
        # 기존 메시지를 수정하려면
        # 일치하는 ID의 메시지를 전달해야 합니다.
        return Command(goto="run_tool", update={"messages": [updated_message]})

    # 자연어 피드백을 제공한 후 이를 에이전트에 전달
    elif review_action == "feedback":
        ...
        feedback_msg = get_feedback_msg(review_data)
        return Command(goto="call_llm", update={"messages": [feedback_msg]})
```

도구 호출 검토 방법에 대한 더 자세한 예는 [여기](../how-tos/human_in_the_loop/review-tool-calls.ipynb)를 참조하십시오.

### 다회 대화

<figure markdown="1">
![image](img/human_in_the_loop/multi-turn-conversation.png){: style="max-height:400px"}
<figcaption><strong>다회 대화</strong> 아키텍처로, <strong>에이전트</strong>와 <strong>인간 노드</strong>가 반복적으로 반응하며 에이전트가 대화를 다른 에이전트나 시스템의 다른 부분으로 넘기기로 결정할 때까지 계속됩니다.</figcaption>
</figure>

**다회 대화**는 에이전트와 인간 간의 여러 차례의 상호작용을 포함하며, 이를 통해 에이전트가 대화적인 방식으로 인간에게서 추가 정보를 수집할 수 있습니다.

이 디자인 패턴은 [여러 에이전트](./multi_agent.md)로 구성된 LLM 애플리케이션에서 유용합니다. 하나 이상의 에이전트가 인간과의 다회 대화를 수행할 필요가 있으며, 인간은 대화의 다양한 단계에서 입력이나 피드백을 제공합니다. 편의상 아래의 에이전트 구현은 단일 노드로 설명되지만, 실제로는 여러 노드로 구성된 더 큰 그래프의 일부일 수 있으며 조건부 엣지를 포함할 수 있습니다.

=== "에이전트당 인간 노드 사용"

이 패턴에서는 각 에이전트가 사용자 입력을 수집하기 위해 자체 인간 노드를 가집니다. 이는 인간 노드를 고유한 이름으로 지정(예: "에이전트 1의 인간", "에이전트 2의 인간")하거나, 서브그래프를 사용하여 서브그래프가 인간 노드와 에이전트 노드를 포함하도록 할 수 있습니다.

```python
from langgraph.types import interrupt

def human_input(state: State):
    human_message = interrupt("human_input")
    return {
        "messages": [
            {
                "role": "human",
                "content": human_message
            }
        ]
    }

def agent(state: State):
    # 에이전트 로직
    ...

graph_builder.add_node("human_input", human_input)
graph_builder.add_edge("human_input", "agent")
graph = graph_builder.compile(checkpointer=checkpointer)

# 그래프를 실행한 후 인터럽트에 도달하면 그래프가 일시 중지됩니다.
# 인간의 입력으로 재개합니다.
graph.invoke(
    Command(resume="안녕하세요!"),
    config=thread_config
)
```

=== "여러 에이전트 간 인간 노드 공유"

이 패턴에서는 단일 인간 노드를 사용하여 여러 에이전트의 사용자 입력을 수집합니다. 활성 에이전트는 상태에 따라 결정되므로, 인간 입력이 수집된 후 그래프는 올바른 에이전트로 라우팅할 수 있습니다.

    ```python
    from langgraph.types import interrupt

    def human_node(state: MessagesState) -> Command[Literal["agent_1", "agent_2", ...]]:
        """사용자 입력을 수집하는 노드."""
        user_input = interrupt(value="사용자 입력을 받을 준비가 되었습니다.")

        # 상태에서 **활성 에이전트**를 결정하여
        # 입력 수집 후 올바른 에이전트로 라우팅할 수 있도록 합니다.
        # 예를 들어, 상태에 필드를 추가하거나 마지막 활성 에이전트를 사용할 수 있습니다.
        # 또는 에이전트가 생성한 AI 메시지의 `name` 속성을 채울 수 있습니다.
        active_agent = ...

        return Command(
            update={
                "messages": [{
                    "role": "human",
                    "content": user_input,
                }]
            },
            goto=active_agent,
        )
    ```

멀티턴 대화 구현에 대한 자세한 예시는 [여기를 참조하세요](../how-tos/multi-agent-multi-turn-convo.ipynb).

### 인간 입력 검증하기

그래프 내에서 인간이 제공한 입력을 검증해야 하는 경우(클라이언트 측이 아니라), 단일 노드 내에서 여러 개의 interrupt 호출을 사용하여 이를 달성할 수 있습니다.

```python
from langgraph.types import interrupt

def human_node(state: State):
    """검증을 포함한 인간 노드."""
    question = "당신의 나이는 몇 살입니까?"

    while True:
        answer = interrupt(question)

        # 답변을 검증하고, 답변이 유효하지 않으면 다시 입력을 요청합니다.
        if not isinstance(answer, int) or answer < 0:
            question = f"'{answer}는 유효한 나이가 아닙니다. 당신의 나이는 몇 살입니까?"
            answer = None
            continue
        else:
            # 답변이 유효하면 진행할 수 있습니다.
            break

    print(f"루프에 있는 인간은 {answer}세 입니다.")
    return {
        "age": answer
    }
```

## `Command` 원시 타입

`interrupt` 함수를 사용할 때 그래프는 interrupt에서 일시 중지되고 사용자 입력을 기다립니다.

그래프 실행은 [Command](../reference/types.md#langgraph.types.Command) 원시 타입을 사용하여 재개할 수 있으며, 이는 `invoke`, `ainvoke`, `stream` 또는 `astream` 메서드를 통해 전달될 수 있습니다.

`Command` 원시 타입은 재개하는 동안 그래프의 상태를 제어하고 수정할 수 있는 여러 옵션을 제공합니다:

1. **`interrupt`에 값 전달하기**: `Command(resume=value)`를 사용하여 사용자 응답과 같은 데이터를 그래프에 제공합니다. 실행은 `interrupt`가 사용된 노드의 시작 지점에서 재개되지만, 이번에는 `interrupt(...)` 호출이 `Command(resume=value)`에 전달된 값을 반환하여 그래프의 일시 중지를 방지합니다.

   ```python
   # 사용자의 입력으로 그래프 실행 재개.
   graph.invoke(Command(resume={"age": "25"}), thread_config)
   ```

2. **그래프 상태 업데이트**: `Command(update=update)`를 사용하여 그래프 상태를 수정합니다. 주의할 점은 재개는 `interrupt`가 사용된 노드의 시작 지점에서 시작된다는 것입니다. 실행은 `interrupt`가 사용된 노드의 시작 지점에서 재개되지만, 업데이트된 상태로 진행됩니다.

   ```python
   # 그래프 상태를 업데이트하고 재개합니다.
   # `interrupt`를 사용할 경우 `resume` 값을 제공해야 합니다.
   graph.invoke(Command(update={"foo": "bar"}, resume="출발합시다!!!"), thread_config)
   ```

`Command`를 활용함으로써 그래프 실행을 재개하고, 사용자 입력을 처리하며, 그래프의 상태를 동적으로 조정할 수 있습니다.

## `invoke` 및 `ainvoke`와 함께 사용하기

`stream` 또는 `astream`을 사용하여 그래프를 실행하면 `interrupt`가 트리거되었음을 알려주는 `Interrupt` 이벤트를 받게 됩니다.

`invoke` 및 `ainvoke`는 interrupt 정보를 반환하지 않습니다. 이 정보를 액세스하려면 `invoke` 또는 `ainvoke` 호출 후 그래프 상태를 검색하기 위해 [get_state](../reference/graphs.md#langgraph.graph.graph.CompiledGraph.get_state) 메서드를 사용해야 합니다.

```python
# 그래프를 interrupt까지 실행합니다.
result = graph.invoke(inputs, thread_config)
# 그래프 상태를 가져와 interrupt 정보를 얻습니다.
state = graph.get_state(thread_config)
# 상태 값을 출력합니다.
print(state.values)
# 보류 중인 작업들을 출력합니다.
print(state.tasks)
# 사용자의 입력으로 그래프를 재개합니다.
graph.invoke(Command(resume={"age": "25"}), thread_config)
```

```pycon
{'foo': 'bar'} # 상태 값
(
    PregelTask(
        id='5d8ffc92-8011-0c9b-8b59-9d3545b7e553',
        name='node_foo',
        path=('__pregel_pull', 'node_foo'),
        error=None,
        interrupts=(Interrupt(value='value_in_interrupt', resumable=True, ns=['node_foo:5d8ffc92-8011-0c9b-8b59-9d3545b7e553'], when='during'),), state=None,
        result=None
    ),
) # 대기 중인 작업. 중단점
```

## 중단점에서의 재개는 어떻게 작동하나요?

!!! 경고

    중단점(`interrupt`)에서 재개하는 것은 Python의 `input()` 함수와 **다릅니다**. `input()` 함수가 호출된 정확한 지점에서 실행이 재개됩니다.

중단점을 사용할 때 중요한 점은 재개 방식에 대한 이해입니다. 중단점 이후에 실행을 재개하면 그래프 실행은 마지막 중단점이 발생한 **그래프 노드**의 **시작**부터 시작합니다.

**모든** 코드는 중단점까지의 노드 시작 부분에서 다시 실행됩니다.

```python
counter = 0
def node(state: State):
    # 그래프가 재개될 때까지의 노드 시작 부분에서 모든 코드가 다시 실행됩니다.
    global counter
    counter += 1
    print(f"> 노드에 진입했습니다: {counter} # 번")
    # 그래프를 일시 중지하고 사용자 입력을 기다립니다.
    answer = interrupt()
    print("카운터의 값은:", counter)
    ...
```

그래프를 **재개**하면 카운터가 두 번째로 증가하게 되어 다음과 같은 출력이 나옵니다:

```pycon
> 노드에 진입했습니다: 2 # 번
카운터의 값은: 2
```

## 일반적인 함정

### 부작용

API 호출과 같은 부작용이 있는 코드는 **중단점** 이후에 배치하여 중복 실행을 피해야 합니다. 노드가 재개될 때마다 이러한 경우가 다시 발생합니다.

=== "중단점 이전의 부작용 (불량)"

    노드가 중단점에서 재개될 때 API 호출이 또 한 번 실행됩니다.

    API 호출이 비가산적(idempotent)이지 않거나 비용이 많이 드는 경우 문제가 될 수 있습니다.

    ```python
    from langgraph.types import interrupt

    def human_node(state: State):
        """검증이 있는 인간 노드."""
        api_call(...) # 이 코드는 노드가 재개될 때 다시 실행됩니다.
        answer = interrupt(question)
    ```

=== "중단점 이후의 부작용 (양호)"

    ```python
    from langgraph.types import interrupt

    def human_node(state: State):
        """검증이 있는 인간 노드."""

        answer = interrupt(question)

        api_call(answer) # 중단점 이후이므로 양호
    ```

=== "별도의 노드에서의 부작용 (양호)"

    ```python
    from langgraph.types import interrupt

    def human_node(state: State):
        """검증이 있는 인간 노드."""

        answer = interrupt(question)

        return {
            "answer": answer
        }

    def api_call_node(state: State):
        api_call(...) # 별도의 노드이므로 양호
    ```

### 함수로 호출된 하위 그래프

서브그래프를 [함수처럼 호출할 때](low_level.md#as-a-function), **부모 그래프**는 서브그래프가 호출된 **노드의 시작**에서 실행을 재개합니다 (그리고 `interrupt`가 발생한 곳에서). 마찬가지로, **서브그래프**는 `interrupt()` 함수가 호출된 **노드의 시작**에서 재개됩니다.

예를 들어,

```python
def node_in_parent_graph(state: State):
    some_code()  # <-- 서브그래프가 재개될 때 이 코드가 다시 실행됩니다.
    # 함수처럼 서브그래프를 호출합니다.
    # 서브그래프는 `interrupt` 호출을 포함합니다.
    subgraph_result = subgraph.invoke(some_input)
    ...
```

??? "**예시: 부모 그래프와 서브그래프 실행 흐름**"

부모 그래프에 3개의 노드가 있다고 가정해 보겠습니다:

**부모 그래프**: `node_1` → `node_2` (서브그래프 호출) → `node_3`

서브그래프는 3개의 노드를 가지며, 두 번째 노드에 `interrupt`가 포함되어 있습니다:

**서브그래프**: `sub_node_1` → `sub_node_2` (`interrupt`) → `sub_node_3`

그래프를 재개할 때 실행 흐름은 다음과 같이 진행됩니다:

1. 부모 그래프에서 **`node_1`**을 건너뜁니다 (이미 실행되었고, 그래프 상태가 스냅샷에 저장되었습니다).
2. 부모 그래프에서 **`node_2`**를 처음부터 다시 실행합니다.
3. 서브그래프에서 **`sub_node_1`**을 건너뜁니다 (이미 실행되었고, 그래프 상태가 스냅샷에 저장되었습니다).
4. 서브그래프에서 **`sub_node_2`**를 처음부터 다시 실행합니다.
5. **`sub_node_3`** 및 이후 노드로 진행합니다.

여기 서브그래프가 `interrupt`와 함께 작동하는 방식을 이해하는 데 사용할 수 있는 약식 예시 코드가 있습니다.
각 노드에 진입한 횟수를 세고 그 카운트를 출력합니다.

      ```python
      import uuid
      from typing import TypedDict

      from langgraph.graph import StateGraph
      from langgraph.constants import START
      from langgraph.types import interrupt, Command
      from langgraph.checkpoint.memory import MemorySaver


      class State(TypedDict):
         """그래프 상태."""
         state_counter: int


      counter_node_in_subgraph = 0

      def node_in_subgraph(state: State):
         """서브 그래프의 노드."""
         global counter_node_in_subgraph
         counter_node_in_subgraph += 1  # 이 코드는 **다시 실행되지** 않습니다!
         print(f"`node_in_subgraph`에 총 {counter_node_in_subgraph}회 진입했습니다")

      counter_human_node = 0

      def human_node(state: State):
         global counter_human_node
         counter_human_node += 1 # 이 코드는 다시 실행됩니다!
         print(f"서브 그래프의 human_node에 총 {counter_human_node}회 진입했습니다")
         answer = interrupt("당신의 이름은 무엇인가요?")
         print(f"답변을 받았습니다: {answer}")


      checkpointer = MemorySaver()

      subgraph_builder = StateGraph(State)
      subgraph_builder.add_node("some_node", node_in_subgraph)
      subgraph_builder.add_node("human_node", human_node)
      subgraph_builder.add_edge(START, "some_node")
      subgraph_builder.add_edge("some_node", "human_node")
      subgraph = subgraph_builder.compile(checkpointer=checkpointer)


      counter_parent_node = 0

      def parent_node(state: State):
         """이 부모 노드는 서브 그래프를 호출할 것입니다."""
         global counter_parent_node

         counter_parent_node += 1 # 이 코드는 재개 시 다시 실행됩니다!
         print(f"`parent_node`에 총 {counter_parent_node}회 진입했습니다")

         # 그래프 상태의 상태 카운터를 의도적으로 증가시키고 있습니다
         # 서브 그래프가 동일한 키를 업데이트하더라도 부모 그래프와의 충돌이 없음을 보여드립니다.
         subgraph_state = subgraph.invoke(state)
         return subgraph_state


      builder = StateGraph(State)
      builder.add_node("parent_node", parent_node)
      builder.add_edge(START, "parent_node")

      # 인터럽트가 작동하려면 체크포인터가 활성화되어야 합니다!
      checkpointer = MemorySaver()
      graph = builder.compile(checkpointer=checkpointer)

      config = {
         "configurable": {
            "thread_id": uuid.uuid4(),
         }
      }

      for chunk in graph.stream({"state_counter": 1}, config):
         print(chunk)

      print('--- 재개 중 ---')

      for chunk in graph.stream(Command(resume="35"), config):
         print(chunk)
      ```

      이는 다음을 출력합니다.

      ```pycon
      --- 첫 번째 호출 ---
      부모 노드에서: {'foo': 'bar'}
      `parent_node`에 총 1회 진입했습니다
      `node_in_subgraph`에 총 1회 진입했습니다
      서브 그래프의 human_node에 총 1회 진입했습니다
      {'__interrupt__': (Interrupt(value='당신의 이름은 무엇인가요?', resumable=True, ns=['parent_node:0b23d72f-aaba-0329-1a59-ca4f3c8bad3b', 'human_node:25df717c-cb80-57b0-7410-44e20aac8f3c'], when='중간에'),)}

      --- 재개 중 ---
      부모 노드에서: {'foo': 'bar'}
      `parent_node`에 총 2회 진입했습니다
      서브 그래프의 human_node에 총 2회 진입했습니다
      답변을 받았습니다: 35
      {'parent_node': None}
      ```

### 여러 인터럽트 사용하기

하나의 **노드** 내에서 여러 인터럽트를 사용하는 것은 [인간 입력 검증](#validating-human-input)과 같은 패턴에 유용할 수 있습니다. 그러나 동일한 노드에서 여러 인터럽트를 사용할 경우, 주의하지 않으면 예기치 않은 동작이 발생할 수 있습니다.

노드에 여러 인터럽트 호출이 포함된 경우, LangGraph는 해당 노드를 실행하는 작업에 특정한 재개(restart) 값 목록을 유지합니다. 실행이 재개될 때마다 노드의 시작점에서 시작합니다. 각 인터럽트를 만날 때마다, LangGraph는 해당 작업의 재개 목록에 일치하는 값이 있는지 확인합니다. 매칭은 **엄격하게 인덱스 기반**이므로 노드 내 인터럽트 호출의 순서가 중요합니다.

문제를 피하려면 실행 간 노드의 구조를 동적으로 변경하는 것을 자제하십시오. 여기에는 인터럽트 호출을 추가, 제거 또는 재정렬하는 것이 포함되며, 이러한 변경은 인덱스 불일치를 초래할 수 있습니다. 이러한 문제는 종종 `Command(resume=..., update=SOME_STATE_MUTATION)`을 통해 상태를 변형하거나 전역 변수를 사용하여 노드의 구조를 동적으로 수정하는 것과 같은 비전통적인 패턴에서 발생합니다.

??? "잘못된 코드의 예"

```python
import uuid
from typing import TypedDict, Optional

from langgraph.graph import StateGraph
from langgraph.constants import START
from langgraph.types import interrupt, Command
from langgraph.checkpoint.memory import MemorySaver


class State(TypedDict):
    """그래프 상태입니다."""

    age: Optional[str]
    name: Optional[str]


def human_node(state: State):
    if not state.get('name'):
        name = interrupt("당신의 이름은 무엇인가요?")
    else:
        name = "N/A"

    if not state.get('age'):
        age = interrupt("당신의 나이는 몇 세인가요?")
    else:
        age = "N/A"

    print(f"이름: {name}. 나이: {age}")

    return {
        "age": age,
        "name": name,
    }


builder = StateGraph(State)
builder.add_node("human_node", human_node)
builder.add_edge(START, "human_node")

# 인터럽트를 작동시키기 위해 체크포인터를 활성화해야 합니다!
checkpointer = MemorySaver()
graph = builder.compile(checkpointer=checkpointer)

config = {
    "configurable": {
        "thread_id": uuid.uuid4(),
    }
}

for chunk in graph.stream({"age": None, "name": None}, config):
    print(chunk)

for chunk in graph.stream(Command(resume="John", update={"name": "foo"}), config):
    print(chunk)
```

```pycon
{'__interrupt__': (Interrupt(value='당신의 이름은 무엇인가요?', resumable=True, ns=['human_node:3a007ef9-c30d-c357-1ec1-86a1a70d8fba'], when='during'),)}
이름: N/A. 나이: John
{'human_node': {'age': 'John', 'name': 'N/A'}}
```

## 추가 자료 📚

- [**개념 가이드: 지속성**](persistence.md#replay): 재생에 대한 더 많은 맥락을 읽기 위한 지속성 가이드를 참조하세요.
- [**가이드: 인간 개입**](../how-tos/index.md#human-in-the-loop): LangGraph에서 인간 개입 워크플로를 구현하는 방법을 알아보세요.
- [**멀티 턴 대화 구현하기**](../how-tos/multi-agent-multi-turn-convo.ipynb): LangGraph에서 멀티 턴 대화를 구현하는 방법을 배워보세요.
