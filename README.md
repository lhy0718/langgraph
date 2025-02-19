_한국어로 기계번역됨_

# 🦜🕸️LangGraph

![버전](https://img.shields.io/pypi/v/langgraph)
[![다운로드](https://static.pepy.tech/badge/langgraph/month)](https://pepy.tech/project/langgraph)
[![열린 이슈](https://img.shields.io/github/issues-raw/langchain-ai/langgraph)](https://github.com/langchain-ai/langgraph/issues)
[![문서](https://img.shields.io/badge/docs-latest-blue)](https://langchain-ai.github.io/langgraph/)

⚡ 그래프로 언어 에이전트 구축하기 ⚡

> [!NOTE]
> JS 버전을 찾고 계신가요? [JS 레포](https://github.com/langchain-ai/langgraphjs) 및 [JS 문서](https://langchain-ai.github.io/langgraphjs/)를 확인하세요.

## 개요

[LangGraph](https://langchain-ai.github.io/langgraph/)는 LLM을 사용하여 상태 저장형 다중 액터 응용 프로그램을 구축하는 라이브러리로, 에이전트 및 다중 에이전트 워크플로우를 생성하는 데 사용됩니다. [여기](https://langchain-ai.github.io/langgraph/tutorials/introduction/)에서 소개 튜토리얼을 확인하세요.

LangGraph는 [Pregel](https://research.google/pubs/pub37252/)과 [Apache Beam](https://beam.apache.org/)에서 영감을 받았습니다. 공용 인터페이스는 [NetworkX](https://networkx.org/documentation/latest/)에서 영감을 얻었습니다. LangGraph는 LangChain Inc.에서 개발하였으며, LangChain 없이도 사용할 수 있습니다.

### LangGraph를 사용하는 이유는?

LangGraph는 [생산 준비 완료된 에이전트](https://www.langchain.com/built-with-langgraph)를 제공하며, 이는 Linkedin, Uber, Klarna, GitLab 등 많은 기업에서 신뢰받고 있습니다. LangGraph는 에이전트 응용 프로그램의 흐름과 상태를 미세하게 제어할 수 있는 기능을 제공합니다. 중앙 [지속성 레이어](https://langchain-ai.github.io/langgraph/concepts/persistence/)를 구현하여 대부분의 에이전트 아키텍처에서 공통적으로 사용되는 기능을 가능하게 합니다:

- **메모리**: LangGraph는 응용 프로그램 상태의 임의 측면을 지속시키며, 대화 및 기타 사용자 상호작용 내 및 사이의 업데이트를 지원합니다.
- **인간 개입**: 상태가 체크포인트되기 때문에 실행을 중단하고 재개할 수 있으며, 인간 입력을 통해 중요한 단계에서 결정, 검증 및 수정을 할 수 있습니다.

이러한 구성 요소를 표준화함으로써 개인과 팀은 에이전트의 동작에 집중할 수 있으며, 지원 인프라에 대해서는 걱정할 필요가 없습니다.

[LangGraph Platform](#langgraph-platform)을 통해 LangGraph는 응용 프로그램의 개발, 배포, 디버깅 및 모니터링을 위한 도구도 제공합니다.

LangGraph는 [LangChain](https://python.langchain.com/docs/introduction/) 및 [LangSmith](https://docs.smith.langchain.com/)와 무 seamlessly 통합되지만, 이들을 필요로 하지 않습니다.

LangGraph에 대해 더 알아보려면 첫 번째 LangChain 아카데미 과정인 *LangGraph 소개*를 확인해 보세요. 무료로 [여기](https://academy.langchain.com/courses/intro-to-langgraph)에서 제공합니다.

### LangGraph 플랫폼

[LangGraph 플랫폼](https://langchain-ai.github.io/langgraph/concepts/langgraph_platform)은 LangGraph 에이전트를 배포하기 위한 인프라입니다. 이는 오픈 소스 LangGraph 프레임워크 위에 구축된 생산용 에이전트 응용 프로그램 배포를 위한 상용 솔루션입니다. LangGraph 플랫폼은 LangGraph 응용 프로그램의 개발, 배포, 디버깅 및 모니터링을 지원하기 위해 함께 작동하는 여러 구성 요소로 구성되어 있습니다: [LangGraph 서버](https://langchain-ai.github.io/langgraph/concepts/langgraph_server) (API), [LangGraph SDK](https://langchain-ai.github.io/langgraph/concepts/sdk) (API 클라이언트), [LangGraph CLI](https://langchain-ai.github.io/langgraph/concepts/langgraph_cli) (서버 구축을 위한 명령줄 도구), 및 [LangGraph Studio](https://langchain-ai.github.io/langgraph/concepts/langgraph_studio) (UI/디버거).

배포 옵션은 [여기](https://langchain-ai.github.io/langgraph/concepts/deployment_options/)에서 확인하세요. (무료 티어 포함)

여기 LangGraph 플랫폼이 다루고 있는 복잡한 배포에서 발생하는 몇 가지 일반적인 문제입니다:

- **스트리밍 지원**: LangGraph 서버는 다양한 애플리케이션 요구 사항에 최적화된 [다중 스트리밍 모드](https://langchain-ai.github.io/langgraph/concepts/streaming)를 제공합니다.
- **백그라운드 실행**: 에이전트를 비동기적으로 백그라운드에서 실행합니다.
- **장기간 실행 에이전트 지원**: 장기간 실행 프로세스를 처리할 수 있는 인프라.
- **[이중 텍스트](https://langchain-ai.github.io/langgraph/concepts/double_texting)**: 에이전트가 응답하기 전에 사용자로부터 두 개의 메시지를 받는 경우를 처리합니다.
- **버스트 처리**: 요청이 손실 없이 고르게 처리되도록 보장하는 작업 큐.

## 설치

```shell
pip install -U langgraph
```

## 예제

도구 호출 [ReAct 스타일](https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/#react-implementation)의 에이전트를 만들어 봅시다!

```shell
pip install langchain-anthropic
```

```shell
export ANTHROPIC_API_KEY=sk-...
```

선택적으로, 최고의 관찰 가능성을 위해 [LangSmith](https://docs.smith.langchain.com/)를 설정할 수 있습니다.

```shell
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=lsv2_sk_...
```

LangGraph에서 도구 호출 에이전트를 생성하는 가장 간단한 방법은 `create_react_agent`를 사용하는 것입니다:

<details open>
  <summary>고수준 구현</summary>

```python
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool

# 에이전트가 사용할 도구 정의
@tool
def search(query: str):
    """웹을 검색하는 호출."""
    # 이것은 플레이스홀더입니다. 그러나 LLM에게는 말하지 마세요...
    if "sf" in query.lower() or "san francisco" in query.lower():
        return "현재 온도는 60도이며 흐림니다."
    return "현재 온도는 90도이며 맑습니다."


tools = [search]
model = ChatAnthropic(model="claude-3-5-sonnet-latest", temperature=0)

# 그래프 실행 간 상태를 유지하기 위해 메모리 초기화
checkpointer = MemorySaver()

app = create_react_agent(model, tools, checkpointer=checkpointer)

# 에이전트 사용
final_state = app.invoke(
    {"messages": [{"role": "user", "content": "sf의 날씨는 어떤가요?"}]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```
```
"검색 결과에 기반하여, 샌프란시스코의 현재 날씨는:\n\n온도: 60도 화씨\n조건: 흐림\n\n샌프란시스코는 미세 기후와 잦은 안개로 유명하며, 여름철에 특히 나타납니다. 60°F(약 15.5°C)의 온도는 이 도시에서 매우 일반적입니다. 이 도시는 연중 온도가 온화한 경향이 있습니다. 지역 주민들이 '카를 더 포그'라고 부르는 안개는 샌프란시스코 날씨의 특징이며, 특히 아침과 저녁에 자주 나타납니다.\n\n샌프란시스코 또는 다른 장소의 날씨에 대해 더 알고 싶은 것이 있나요?"
```

이제 같은 <code>"thread_id"</code>를 전달하면 저장된 상태를 통해 대화 문맥이 유지됩니다 (즉, 저장된 메시지 목록).

```python
final_state = app.invoke(
    {"messages": [{"role": "user", "content": "뉴욕은 어떤가요?"}]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```

```
"검색 결과에 기반하여, 뉴욕시의 현재 날씨는:\n\n온도: 90도 화씨 (약 32.2도 섭씨)\n조건: 맑음\n\n이 날씨는 우리가 방금 본 샌프란시스코의 날씨와는 매우 다릅니다. 뉴욕은 현재 훨씬 더 따뜻한 온도를 경험하고 있습니다. 몇 가지 포인트를 유의하세요:\n\n1. 90°F의 온도는 매우 덥고, 뉴욕시의 여름 날씨에서 일반적입니다.\n2. 맑은 날씨는 맑은 하늘을 시사하며, 이는 야외 활동에 좋지만 직사광선으로 인해 더 뜨겁게 느껴질 수 있습니다.\n3. 뉴욕의 이런 날씨는 종종 높은 습도와 함께해 실제 온도보다 더 뜨겁게 느껴질 수 있습니다.\n\n샌프란시스코의 온화하고 흐린 날씨와 뉴욕의 더운 맑은 날씨 간의 뚜렷한 대조를 보는 것은 흥미롭습니다. 이 차이는 같은 날 미국의 다양한 지역에서 날씨가 얼마나 다양할 수 있는지를 보여줍니다.\n\n뉴욕 또는 다른 장소의 날씨에 대해 더 알고 싶은 것이 있나요?"
```
</details>

> [!TIP]
> LangGraph는 사용자 정의 에이전트 아키텍처를 구현할 수 있는 **저수준** 프레임워크입니다. 아래의 저수준 구현을 클릭하여 도구 호출 에이전트를 처음부터 구현하는 방법을 확인하세요.

<details>
<summary>저수준 구현</summary>

```python
from typing import Literal

from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import END, START, StateGraph, MessagesState
from langgraph.prebuilt import ToolNode


# 에이전트가 사용할 도구 정의
@tool
def search(query: str):
    """웹 서핑을 위한 호출."""
    # 이것은 자리 표시자이며, LLM에게 말하지 마세요...
    if "sf" in query.lower() or "san francisco" in query.lower():
        return "60도이고 안개가 끼어 있습니다."
    return "90도이고 맑습니다."


tools = [search]

tool_node = ToolNode(tools)

model = ChatAnthropic(model="claude-3-5-sonnet-latest", temperature=0).bind_tools(tools)

# 계속할지 여부를 결정하는 함수 정의
def should_continue(state: MessagesState) -> Literal["tools", END]:
    messages = state['messages']
    last_message = messages[-1]
    # LLM이 도구 호출을 하면 "tools" 노드로 라우팅합니다.
    if last_message.tool_calls:
        return "tools"
    # 그렇지 않으면 중지합니다 (사용자에게 응답).
    return END


# 모델을 호출하는 함수 정의
def call_model(state: MessagesState):
    messages = state['messages']
    response = model.invoke(messages)
    # 기존 목록에 추가되기 때문에 목록을 반환합니다.
    return {"messages": [response]}


# 새로운 그래프 정의
workflow = StateGraph(MessagesState)

# 우리가 순환할 두 노드 정의
workflow.add_node("agent", call_model)
workflow.add_node("tools", tool_node)

# 진입점을 `agent`로 설정
# 이는 이 노드가 처음 호출된다는 것을 의미합니다.
workflow.add_edge(START, "agent")

# 조건부 엣지 추가
workflow.add_conditional_edges(
    # 먼저 시작 노드를 정의합니다. `agent`를 사용합니다.
    # 이는 `agent` 노드가 호출된 후의 엣지를 의미합니다.
    "agent",
    # 다음으로, 어떤 노드가 다음에 호출될지 결정할 함수를 전달합니다.
    should_continue,
)

# `tools`에서 `agent`로의 일반 엣지 추가.
# 이는 `tools`가 호출된 후 `agent` 노드가 다음에 호출된다는 것을 의미합니다.
workflow.add_edge("tools", 'agent')

# 그래프 실행 간 상태를 유지하기 위해 메모리 초기화
checkpointer = MemorySaver()

# 마지막으로 컴파일합니다!
# 이는 LangChain Runnable로 컴파일되어,
# 다른 runnable처럼 사용할 수 있습니다.
# 그래프를 컴파일할 때 메모리를 선택적으로 전달하고 있다는 점에 유의하세요.
app = workflow.compile(checkpointer=checkpointer)

# 에이전트 사용
final_state = app.invoke(
    {"messages": [{"role": "user", "content": "sf의 날씨는 어떠한가요?"}]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```

<b>단계별 분석</b>:

<details>
<summary>모델 및 도구 초기화.</summary>
<ul>
  <li>
    <code>ChatAnthropic</code>를 우리의 LLM으로 사용합니다. <strong>참고:</strong> 모델이 이 도구를 호출할 수 있도록 알려야 합니다. <code>.bind_tools()</code> 메서드를 사용하여 LangChain 도구를 OpenAI 도구 호출 형식으로 변환함으로써 이를 수행할 수 있습니다.
  </li>
  <li>
    우리가 사용할 도구를 정의합니다 - 이 경우 검색 도구입니다. 자신의 도구를 만드는 것은 매우 쉽습니다 - 여기에 대한 문서는 <a href="https://python.langchain.com/docs/how_to/custom_tools/">여기</a>에서 확인하세요.
  </li>
</ul>
</details>

<details>
<summary>상태로 그래프 초기화하기.</summary>

<ul>
    <li>우리는 상태 스키마(<code>MessagesState</code>)를 전달하여 그래프(<code>StateGraph</code>)를 초기화합니다.</li>
    <li><code>MessagesState</code>는 LangChain <code>Message</code> 객체의 목록이라는 하나의 속성을 가진 미리 구축된 상태 스키마이며, 각 노드에서 업데이트를 상태로 병합하는 로직도 포함되어 있습니다.</li>
</ul>
</details>

<details>
<summary>그래프 노드 정의하기.</summary>

우리가 필요한 두 개의 주요 노드가 있습니다:

<ul>
    <li><code>agent</code> 노드: 어떤 조치를 취할지 결정하는 책임이 있습니다.</li>
    <li>조치를 취하기로 결정하면 이 노드가 실행할 <code>tools</code> 노드입니다.</li>
</ul>
</details>

<details>
<summary>진입점과 그래프 엣지 정의하기.</summary>

먼저 그래프 실행을 위한 진입점을 설정해야 합니다 - <code>agent</code> 노드.

그런 다음 하나의 일반 엣지와 하나의 조건 엣지를 정의합니다. 조건 엣지는 목적지가 그래프 상태(<code>MessagesState</code>)의 내용에 따라 달라진다는 것을 의미합니다. 우리 경우, 목적지는 에이전트(LLM)가 결정할 때까지 알려지지 않습니다.

<ul>
  <li>조건 엣지: 에이전트가 호출된 후, 우리는 다음 두 가지 중 하나를 수행해야 합니다:
    <ul>
      <li>a. 에이전트가 조치를 취하라고 말했다면 도구를 실행하고, 또는</li>
      <li>b. 에이전트가 도구 실행을 요청하지 않았다면 종료(사용자에게 응답)합니다.</li>
    </ul>
  </li>
  <li>일반 엣지: 도구가 호출된 후, 그래프는 항상 에이전트에게 돌아가 다음에 무엇을 할지 결정을 내려야 합니다.</li>
</ul>
</details>

<details>
<summary>그래프 컴파일하기.</summary>

<ul>
  <li>
    그래프를 컴파일할 때, 우리는 이를 LangChain <a href="https://python.langchain.com/docs/concepts/runnables/">Runnable</a>로 변환하여 <code>.invoke()</code>, <code>.stream()</code> 및 <code>.batch()</code>를 입력과 함께 자동으로 호출할 수 있도록 합니다.
  </li>
  <li>
    또한 선택적으로 그래프 실행 간 상태를 지속하도록 하는 체크포인터 객체를 전달하여 메모리, 인간-in-the-loop 워크플로우, 시간 여행 등을 활성화할 수 있습니다. 우리 경우 <code>MemorySaver</code>를 사용합니다 - 간단한 인메모리 체크포인터입니다.
  </li>
</ul>
</details>

<details>
<summary>그래프 실행하기.</summary>

<ol>
  <li>LangGraph는 입력 메시지를 내부 상태에 추가한 후 상태를 진입점 노드인 <code>"agent"</code>로 전달합니다.</li>
  <li><code>"agent"</code> 노드가 실행되며, 챗 모델을 호출합니다.</li>
  <li>챗 모델이 <code>AIMessage</code>를 반환합니다. LangGraph는 이를 상태에 추가합니다.</li>
  <li>그래프는 더 이상 <code>AIMessage</code>에 <code>tool_calls</code>가 없을 때까지 다음 단계를 순환합니다:
    <ul>
      <li>만약 <code>AIMessage</code>가 <code>tool_calls</code>를 가지고 있다면, <code>"tools"</code> 노드가 실행됩니다.</li>
      <li>그 후 <code>"agent"</code> 노드가 다시 실행되어 <code>AIMessage</code>를 반환합니다.</li>
    </ul>
  </li>
  <li>실행은 특별한 <code>END</code> 값으로 진행되며 최종 상태를 출력합니다. 그 결과, 우리는 모든 챗 메시지의 목록을 출력으로 받습니다.</li>
</ol>
</details>

</details>

## 문서

* [튜토리얼](https://langchain-ai.github.io/langgraph/tutorials/): 가이드 예제를 통해 LangGraph로 구축하는 방법을 배웁니다.
* [방법 가이드](https://langchain-ai.github.io/langgraph/how-tos/): 스트리밍, 메모리 및 지속성 추가, 일반 디자인 패턴(분기, 서브그래프 등)과 같은 LangGraph 내에서 특정 작업을 수행하는 방법입니다. 특정 코드 스니펫을 복사하고 실행하려면 이곳이 적합합니다.
* [개념 가이드](https://langchain-ai.github.io/langgraph/concepts/high_level/): 노드, 엣지, 상태 등 LangGraph의 주요 개념 및 원리에 대한 심층 설명입니다.
* [API 참조](https://langchain-ai.github.io/langgraph/reference/graphs/): 중요한 클래스 및 메서드, 그래프 및 체크포인팅 API를 사용하는 간단한 예제, 고급 미리 구축된 구성 요소 등을 검토할 수 있습니다.
* [LangGraph 플랫폼](https://langchain-ai.github.io/langgraph/concepts/#langgraph-platform): LangGraph 플랫폼은 오픈 소스 LangGraph 프레임워크를 기반으로 한 상업적 솔루션으로, 프로덕션에서 에이전틱 애플리케이션을 배포하기 위한 것입니다.

## 리소스

* [LangGraph로 구축된 사례](https://www.langchain.com/built-with-langgraph): 산업 리더들이 LangGraph를 사용하여 강력한 프로덕션 준비 AI 애플리케이션을 출시하는 방법을 들어보세요.

## 기여

기여에 대한 자세한 정보는 [여기](https://github.com/langchain-ai/langgraph/blob/main/CONTRIBUTING.md)를 참조하십시오.
