_한국어로 기계번역됨_

# 멀티 에이전트 시스템

[에이전트](./agentic_concepts.md#agent-architectures)는 *애플리케이션의 제어 흐름을 결정하기 위해 LLM을 사용하는 시스템*입니다. 이러한 시스템을 개발하다 보면 시간이 지남에 따라 복잡해져 관리와 확장이 어려워질 수 있습니다. 예를 들어, 다음과 같은 문제에 직면할 수 있습니다:

- 에이전트가 사용할 수 있는 도구가 너무 많아 어떤 도구를 호출할지 결정하는 데 문제가 발생
- 하나의 에이전트가 모든 컨텍스트를 추적하기에 복잡도가 너무 높음
- 시스템에 여러 가지 전문 분야(예: 계획자, 연구자, 수학 전문가 등)가 필요함

이를 해결하기 위해 애플리케이션을 여러 개의 독립적인 에이전트로 나누고 이를 **멀티 에이전트 시스템**으로 구성하는 것을 고려할 수 있습니다. 이러한 독립적인 에이전트는 간단한 프롬프트와 LLM 호출일 수도 있고, 복잡한 [ReAct](./agentic_concepts.md#react-implementation) 에이전트일 수도 있습니다(그리고 더 많은 형태가 있습니다!).

멀티 에이전트 시스템을 사용하는 주요 이점은 다음과 같습니다:

- **모듈화**: 독립적인 에이전트는 에이전트 시스템을 개발, 테스트 및 유지 관리하는 데 더 용이합니다.
- **전문화**: 특정 도메인에 집중하는 전문가 에이전트를 만들어 시스템의 전체 성능을 개선할 수 있습니다.
- **제어**: 에이전트 간의 통신 방법을 명확히 제어할 수 있습니다(함수 호출에 의존하는 대신).

## 멀티 에이전트 아키텍처

![](./img/multi_agent/architectures.png)

멀티 에이전트 시스템에서 에이전트를 연결하는 몇 가지 방법이 있습니다:

- **네트워크**: 각 에이전트는 [다른 모든 에이전트와](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/multi-agent-collaboration/) 통신할 수 있습니다. 어떤 에이전트도 다른 에이전트를 호출할지 결정할 수 있습니다.
- **감독자**: 각 에이전트는 하나의 [감독자](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/agent_supervisor/) 에이전트와 통신합니다. 감독자 에이전트는 어떤 에이전트를 호출할지 결정합니다.
- **감독자 (도구 호출)**: 이는 감독자 아키텍처의 특별한 경우입니다. 개별 에이전트는 도구로 나타낼 수 있습니다. 이 경우, 감독자 에이전트는 도구 호출 LLM을 사용하여 호출할 에이전트 도구를 결정하고 해당 에이전트에 전달할 인수를 결정합니다.
- **계층형**: [감독자의 감독자](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/hierarchical_agent_teams/)를 정의하여 멀티 에이전트 시스템을 구성할 수 있습니다. 이는 감독자 아키텍처의 일반화이며 더 복잡한 제어 흐름을 가능하게 합니다.
- **사용자 정의 멀티 에이전트 워크플로**: 각 에이전트는 일부 에이전트 집합과만 통신합니다. 흐름의 일부는 결정적이며, 일부 에이전트만 다음에 호출할 다른 에이전트를 결정할 수 있습니다.

### 핸드오프

멀티 에이전트 아키텍처에서 에이전트는 그래프 노드로 나타낼 수 있습니다. 각 에이전트 노드는 자신의 단계를 실행하고 실행을 마칠지 아니면 다른 에이전트로 제어를 넘길지 결정합니다. 에이전트가 제어를 다른 에이전트에게 넘기는 패턴을 **핸드오프**라고 합니다. 핸드오프를 사용하면 다음을 지정할 수 있습니다:

- **목적지**: 이동할 대상 에이전트(예: 이동할 노드 이름)
- **페이로드**: 해당 에이전트로 전달할 [정보](#communication-between-agents) (예: 상태 업데이트)

LangGraph에서 핸드오프를 구현하려면 에이전트 노드가 [`Command`](./low_level.md#command) 객체를 반환할 수 있습니다. 이 객체는 제어 흐름과 상태 업데이트를 결합할 수 있습니다:

```python
def agent(state) -> Command[Literal["agent", "another_agent"]]:
    # 라우팅/중지 조건은 무엇이든 될 수 있습니다. 예: LLM 도구 호출 / 구조화된 출력 등.
    goto = get_next_agent(...)  # 'agent' / 'another_agent'
    return Command(
        # 어떤 에이전트를 다음으로 호출할지 지정
        goto=goto,
        # 그래프 상태 업데이트
        update={"my_state_key": "my_state_value"}
    )
```

각 에이전트 노드가 자체 그래프(즉, [서브그래프](./low_level.md#subgraphs))일 때 더 복잡한 시나리오가 발생할 수 있습니다. 예를 들어, `alice`와 `bob`이라는 두 에이전트가 있고(`alice`와 `bob`이 부모 그래프의 서브그래프 노드일 경우), `alice`가 `bob`으로 이동해야 할 때 `Command` 객체에서 `graph=Command.PARENT`를 설정하여 이를 처리할 수 있습니다:

```python
def some_node_inside_alice(state)
    return Command(
        goto="bob",
        update={"my_state_key": "my_state_value"},
        # 이동할 그래프를 지정 (기본값은 현재 그래프)
        graph=Command.PARENT,
    )
```

!!! 주의

    서브그래프가 `Command(graph=Command.PARENT)`를 사용하여 통신하는 시각화를 지원하려면 이를 `Command` 주석이 있는 노드 함수로 감싸야 합니다. 예를 들어, 다음과 같이 해야 합니다:

```python
builder.add_node(alice)
```

    대신 다음과 같이 해야 합니다:

```python
def call_alice(state) -> Command[Literal["bob"]]:
    return alice.invoke(state)

builder.add_node("alice", call_alice)
```

#### 핸드오프를 도구로 사용

가장 일반적인 에이전트 유형 중 하나는 ReAct 스타일의 도구 호출 에이전트입니다. 이러한 유형의 에이전트에서 일반적인 패턴은 핸드오프를 도구 호출로 래핑하는 것입니다. 예를 들어:

```python
def transfer_to_bob(state):
    """bob으로 전송."""
    return Command(
        goto="bob",
        update={"my_state_key": "my_state_value"},
        graph=Command.PARENT,
    )
```

이는 도구에서 그래프 상태 업데이트와 제어 흐름을 포함하여 두 가지를 함께 처리하는 특별한 경우입니다.

!!! 중요

    `Command`를 반환하는 도구를 사용하려면 미리 구축된 [`create_react_agent`][langgraph.prebuilt.chat_agent_executor.create_react_agent] / [`ToolNode`][langgraph.prebuilt.tool_node.ToolNode] 컴포넌트를 사용하거나, 도구에서 반환된 `Command` 객체를 수집하여 이를 반환하는 자체 도구 실행 노드를 구현할 수 있습니다. 예를 들어:

```python
def call_tools(state):
    ...
    commands = [tools_by_name[tool_call["name"]].invoke(tool_call) for tool_call in tool_calls]
    return commands
```

이제 다양한 멀티 에이전트 아키텍처를 살펴보겠습니다.

### 네트워크

이 아키텍처에서는 에이전트를 그래프 노드로 정의합니다. 각 에이전트는 다른 모든 에이전트와 통신할 수 있으며(다대다 연결), 어떤 에이전트를 호출할지 결정할 수 있습니다. 이 아키텍처는 에이전트 간에 명확한 계층 구조나 특정 호출 순서가 없는 문제에 적합합니다.

```python
from typing import Literal
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, MessagesState, START, END

model = ChatOpenAI()

def agent_1(state: MessagesState) -> Command[Literal["agent_2", "agent_3", END]]:
    # LLM에 전달할 상태의 관련 부분을 전달하여 어떤 에이전트를 호출할지 결정
    response = model.invoke(...)
    # LLM의 결정에 따라 에이전트를 호출하거나 종료
    # LLM이 "__end__"를 반환하면 그래프 실행이 종료됨
    return Command(
        goto=response["next_agent"],
        update={"messages": [response["content"]]},
    )

def agent_2(state: MessagesState) -> Command[Literal["agent_1", "agent_3", END]]:
    response = model.invoke(...)
    return Command(
        goto=response["next_agent"],
        update={"messages": [response["content"]]},
    )

def agent_3(state: MessagesState) -> Command[Literal["agent_1", "agent_2", END]]:
    ...
    return Command(
        goto=response["next_agent"],
        update={"messages": [response["content"]]},
    )

builder = StateGraph(MessagesState)
builder.add_node(agent_1)
builder.add_node(agent_2)
builder.add_node(agent_3)

builder.add_edge(START, "agent_1")
network = builder.compile()
```

### 감독자

이 아키텍처에서는 에이전트를 노드로 정의하고, 어떤 에이전트 노드를 호출할지 결정하는 감독자 노드(LLM)를 추가합니다. 우리는 [`Command`](./low_level.md#command)를 사용하여 감독자의 결정에 따라 실행을 적절한 에이전트 노드로 라우팅합니다. 이 아키텍처는 여러 에이전트를 병렬로 실행하거나 [map-reduce](../how-tos/map-reduce.ipynb) 패턴을 사용할 때 적합합니다.

```python
from typing import Literal
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, MessagesState, START, END

model = ChatOpenAI()

def supervisor(state: MessagesState) -> Command[Literal["agent_1", "agent_2", END]]:
    # LLM에 전달할 상태의 관련 부분을 전달하여 어떤 에이전트를 호출할지 결정
    response = model.invoke(...)
    # 감독자의 결정에 따라 에이전트를 호출하거나 종료
    # 감독자가 "__end__"를 반환하면 그래프 실행이 종료됨
    return Command(goto=response["next_agent"])

def agent_1(state: MessagesState) -> Command[Literal["supervisor"]]:
    response = model.invoke(...)
    return Command(
        goto="supervisor",
        update={"messages": [response]},
    )

def agent_2(state: MessagesState) -> Command[Literal["supervisor"]]:
    response = model.invoke(...)
    return Command(
        goto="supervisor",
        update={"messages": [response]},
    )

builder = StateGraph(MessagesState)
builder.add_node(supervisor)
builder.add_node(agent_1)
builder.add_node(agent_2)

builder.add_edge(START, "supervisor")

supervisor = builder.compile()
```

[이 튜토리얼](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/agent_supervisor/)에서 감독자 멀티 에이전트 아키텍처의 예제를 확인하세요.

### 감독자 (도구 호출)

[감독자](#supervisor) 아키텍처의 변형에서는 각 에이전트를 **도구**로 정의하고, 감독자 노드에서 도구 호출 LLM을 사용합니다. 이는 두 개의 노드로 구성된 [ReAct](./agentic_concepts.md#react-implementation) 스타일의 에이전트로 구현될 수 있습니다. 하나는 LLM 노드(감독자)이고 다른 하나는 도구를 실행하는 도구 호출 노드입니다(이 경우 에이전트).

```python
from typing import Annotated
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import InjectedState, create_react_agent

model = ChatOpenAI()

# 이 노드는 도구로 호출될 에이전트 함수입니다.
# 주목할 점은 InjectedState 주석을 통해 상태를 도구에 전달할 수 있다는 것입니다.
def agent_1(state: Annotated[dict, InjectedState]):
    # 상태의 관련 부분을 LLM에 전달하여 다음 에이전트를 결정
    response = model.invoke(...)
    # LLM 응답을 문자열로 반환 (도구 응답 형식)
    # 이는 자동으로 ToolMessage로 변환됩니다.
    return response.content

def agent_2(state: Annotated[dict, InjectedState]):
    response = model.invoke(...)
    return response.content

tools = [agent_1, agent_2]
# 도구 호출을 사용하는 감독자를 만드는 가장 간단한 방법은 미리 구축된 ReAct 에이전트 그래프를 사용하는 것입니다.
# 이 그래프는 도구 호출 LLM 노드(즉, 감독자)와 도구 실행 노드를 포함합니다.
supervisor = create_react_agent(model, tools)
```

### 계층적

시스템에 에이전트를 더 추가하면 감독자가 모든 에이전트를 관리하는 것이 어려워질 수 있습니다. 감독자는 다음에 호출할 에이전트를 결정하는 데 잘못된 결정을 내리기 시작할 수 있으며, 맥락이 너무 복잡해져서 하나의 감독자가 이를 추적하는 것이 불가능해질 수 있습니다. 즉, 멀티 에이전트 아키텍처에서 처음 문제를 해결하려 했던 것과 같은 문제가 발생할 수 있습니다.

이를 해결하려면 시스템을 _계층적으로_ 설계할 수 있습니다. 예를 들어, 개별 감독자가 관리하는 별도의 전문화된 에이전트 팀을 만들고, 상위 수준의 감독자가 이 팀들을 관리할 수 있습니다.

```python
from typing import Literal
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.types import Command
model = ChatOpenAI()

# 팀 1 정의 (위의 단일 감독자 예시와 동일)

def team_1_supervisor(state: MessagesState) -> Command[Literal["team_1_agent_1", "team_1_agent_2", END]]:
    response = model.invoke(...)
    return Command(goto=response["next_agent"])

def team_1_agent_1(state: MessagesState) -> Command[Literal["team_1_supervisor"]]:
    response = model.invoke(...)
    return Command(goto="team_1_supervisor", update={"messages": [response]})

def team_1_agent_2(state: MessagesState) -> Command[Literal["team_1_supervisor"]]:
    response = model.invoke(...)
    return Command(goto="team_1_supervisor", update={"messages": [response]})

team_1_builder = StateGraph(Team1State)
team_1_builder.add_node(team_1_supervisor)
team_1_builder.add_node(team_1_agent_1)
team_1_builder.add_node(team_1_agent_2)
team_1_builder.add_edge(START, "team_1_supervisor")
team_1_graph = team_1_builder.compile()

# 팀 2 정의 (위의 단일 감독자 예시와 동일)
class Team2State(MessagesState):
    next: Literal["team_2_agent_1", "team_2_agent_2", "__end__"]

def team_2_supervisor(state: Team2State):
    ...

def team_2_agent_1(state: Team2State):
    ...

def team_2_agent_2(state: Team2State):
    ...

team_2_builder = StateGraph(Team2State)
...
team_2_graph = team_2_builder.compile()


# 상위 수준의 감독자 정의

builder = StateGraph(MessagesState)
def top_level_supervisor(state: MessagesState) -> Command[Literal["team_1_graph", "team_2_graph", END]]:
    response = model.invoke(...)
    return Command(goto=response["next_team"])

builder = StateGraph(MessagesState)
builder.add_node(top_level_supervisor)
builder.add_node("team_1_graph", team_1_graph)
builder.add_node("team_2_graph", team_2_graph)
builder.add_edge(START, "top_level_supervisor")
builder.add_edge("team_1_graph", "top_level_supervisor")
builder.add_edge("team_2_graph", "top_level_supervisor")
graph = builder.compile()
```

### 커스텀 멀티 에이전트 워크플로

이 아키텍처에서는 개별 에이전트를 그래프 노드로 추가하고, 에이전트가 호출되는 순서를 미리 정의하여 커스텀 워크플로를 만듭니다. LangGraph에서는 워크플로를 두 가지 방법으로 정의할 수 있습니다:

- **명시적 제어 흐름 (정상 엣지)**: LangGraph는 응용 프로그램의 제어 흐름을 명시적으로 정의할 수 있게 해줍니다(즉, 에이전트 간의 통신 순서). 이는 위의 아키텍처 중 가장 결정적인 방식으로, 미리 어떤 에이전트가 다음에 호출될지 알 수 있습니다.

- **동적 제어 흐름 (커맨드)**: LangGraph에서는 LLM이 응용 프로그램 제어 흐름의 일부를 결정하도록 허용할 수 있습니다. 이는 [`커맨드`](./low_level.md#command)를 사용하여 달성할 수 있습니다. 이 경우, 감독자 도구 호출 아키텍처의 특수한 경우입니다. 이 경우, 감독자 에이전트를 구동하는 도구 호출 LLM이 에이전트가 호출되는 순서에 대한 결정을 내립니다.

### 에이전트 간 통신

멀티 에이전트 시스템을 구축할 때 가장 중요한 점은 에이전트들이 어떻게 통신할지를 결정하는 것입니다. 이를 위해 몇 가지 중요한 고려 사항이 있습니다:

- 에이전트들이 [**그래프 상태 또는 도구 호출을 통해**](#graph-state-vs-tool-calls) 통신하는가?
- 두 에이전트가 [**다른 상태 스키마**](#different-state-schemas)를 가질 경우 어떻게 처리하는가?
- [**공유된 메시지 목록**](#shared-message-list)을 통해 어떻게 통신할 것인가?

### 그래프 상태 vs 도구 호출

에이전트 간에 전달되는 "페이로드"는 무엇일까요? 위에서 논의한 대부분의 아키텍처에서는 에이전트들이 [그래프 상태](./low_level.md#state)를 통해 통신합니다. [도구 호출을 사용하는 감독자 아키텍처](#supervisor-tool-calling)의 경우, 페이로드는 도구 호출 인자입니다.

#### 그래프 상태

그래프 상태를 통해 통신하려면 개별 에이전트를 [그래프 노드](./low_level.md#nodes)로 정의해야 합니다. 이 노드는 함수로 정의하거나 전체 [서브그래프](./low_level.md#subgraphs)로 추가할 수 있습니다. 그래프 실행의 각 단계에서 에이전트 노드는 그래프의 현재 상태를 받아서 에이전트 코드를 실행하고, 그 후 업데이트된 상태를 다음 노드로 전달합니다.

일반적으로 에이전트 노드들은 단일 [상태 스키마](./low_level.md#schema)를 공유합니다. 그러나 에이전트 노드에 [다른 상태 스키마](#different-state-schemas)를 사용해야 할 수도 있습니다.

### 다른 상태 스키마

에이전트는 다른 에이전트들과 다른 상태 스키마를 가질 필요가 있을 수 있습니다. 예를 들어, 검색 에이전트는 쿼리와 검색된 문서만 추적하면 될 수 있습니다. LangGraph에서 이를 구현하는 두 가지 방법은 다음과 같습니다:

- [서브그래프](./low_level.md#subgraphs) 에이전트를 별도의 상태 스키마로 정의합니다. 서브그래프와 부모 그래프 간에 상태 키(채널)가 공유되지 않는 경우, 부모 그래프가 서브그래프와 통신할 수 있도록 [입력/출력 변환](https://langchain-ai.github.io/langgraph/how-tos/subgraph-transform-state/)을 추가하는 것이 중요합니다.
- 에이전트 노드 함수에 [개인 입력 상태 스키마](https://langchain-ai.github.io/langgraph/how-tos/pass_private_state/)를 정의하여 전체 그래프 상태 스키마와 다른 상태를 사용할 수 있습니다. 이를 통해 해당 에이전트를 실행하는 데 필요한 정보만 전달할 수 있습니다.

### 공유된 메시지 목록

에이전트들이 통신하는 가장 일반적인 방법은 공유된 상태 채널을 통해서이며, 일반적으로 메시지 목록을 공유합니다. 이는 에이전트들이 공유하는 최소한 하나의 상태 채널(키)이 있다는 것을 전제로 합니다. 공유된 메시지 목록을 통해 통신할 때 추가적인 고려 사항이 있습니다: 에이전트들이 [**전체 생각 과정**](#share-full-history)을 공유할지, 아니면 [**최종 결과**](#share-final-result)만 공유할지 결정해야 합니다.

#### 전체 생각 과정 공유

에이전트들이 **전체 생각 과정**(즉, "스래치패드")을 다른 모든 에이전트들과 공유할 수 있습니다. 이 "스래치패드"는 일반적으로 [메시지 목록](./low_level.md#why-use-messages)처럼 보입니다. 전체 생각 과정을 공유하는 장점은 다른 에이전트들이 더 나은 결정을 내리는 데 도움이 될 수 있고, 시스템 전체의 추론 능력을 향상시킬 수 있다는 점입니다. 단점은 에이전트 수와 복잡도가 커질수록 "스래치패드"도 빠르게 커지므로, [메모리 관리](./memory.md/#managing-long-conversation-history)에 대한 추가 전략이 필요할 수 있다는 점입니다.

#### 최종 결과만 공유

에이전트들이 각자의 개인적인 "스래치패드"를 가지고 **최종 결과만** 나머지 에이전트들과 공유할 수 있습니다. 이 접근 방식은 많은 에이전트나 복잡한 에이전트가 있는 시스템에서 더 효과적일 수 있습니다. 이 경우, 에이전트들은 [다른 상태 스키마](#different-state-schemas)를 사용해야 합니다.

도구로 호출된 에이전트의 경우, 감독자는 도구 스키마를 기반으로 입력을 결정합니다. 또한 LangGraph에서는 [런타임에 상태 전달](https://langchain-ai.github.io/langgraph/how-tos/pass-run-time-values-to-tools/#pass-graph-state-to-tools)을 통해 하위 에이전트가 상위 상태에 접근할 수 있도록 지원합니다.
