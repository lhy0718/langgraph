_한국어로 기계번역됨_

# 미리 구축된 ReAct 에이전트 사용 방법

<div class="admonition tip">
    <p class="admonition-title">전제 조건</p>
    <p>
        이 가이드는 다음에 대한 친숙함을 가정합니다:
        <ul>
            <li>
                <a href="https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/">
                    에이전트 아키텍처
                </a>                   
            </li>
            <li>
                <a href="https://python.langchain.com/docs/concepts/chat_models/">
                    채팅 모델
                </a>
            </li>
            <li>
                <a href="https://python.langchain.com/docs/concepts/tools/">
                    도구
                </a>
            </li>
        </ul>
    </p>
</div>

이 방법에서는 간단한 [ReAct](https://arxiv.org/abs/2210.03629) 에이전트 앱을 만들어 날씨를 확인할 수 있습니다. 이 앱은 에이전트 (LLM)와 도구로 구성되어 있습니다. 앱과 상호작용하면서 먼저 에이전트 (LLM)를 호출하여 도구를 사용할지 결정합니다. 그런 다음 반복 작업을 수행합니다:

1. 에이전트가 행동을 취하라고 말하면 (즉, 도구를 호출하라고 하면), 도구를 실행하고 결과를 에이전트에게 전달합니다.
2. 에이전트가 도구를 실행하라고 요청하지 않으면, 사용자에게 응답하여 종료합니다.

<div class="admonition warning">
    <p class="admonition-title">사전 구축된 에이전트</p>
    <p>
        여기서는 <a href="https://langchain-ai.github.io/langgraph/reference/prebuilt/#langgraph.prebuilt.chat_agent_executor.create_react_agent">사전 구축된 에이전트</a>를 사용할 것입니다. LangGraph의 큰 장점 중 하나는 쉽게 자신의 에이전트 아키텍처를 만들 수 있다는 것입니다. 따라서 여기서 빠르게 에이전트를 구축하는 것은 괜찮지만, LangGraph의 모든 이점을 활용할 수 있도록 자신의 에이전트를 구축하는 방법을 배우는 것을 강력히 추천합니다.
    </p>
</div>

## 설정

먼저 필요한 패키지를 설치하고 API 키를 설정합시다.

```python
%%capture --no-stderr
%pip install -U langgraph langchain-openai
```

```python
import getpass
import os


def _set_env(var: str):
    if not os.environ.get(var):
        os.environ[var] = getpass.getpass(f"{var}: ")


_set_env("OPENAI_API_KEY")
```

<div class="admonition tip">
    <p class="admonition-title"><a href="https://smith.langchain.com">LangSmith</a>를 LangGraph 개발을 위해 설정하기</p>
    <p style="padding-top: 5px;">
        LangSmith에 가입하여 LangGraph 프로젝트의 문제를 신속하게 발견하고 성능을 개선하세요. LangSmith는 추적 데이터를 사용하여 LangGraph로 구축된 LLM 앱을 디버깅하고 테스트하며 모니터링할 수 있게 해줍니다 — 시작하는 방법에 대한 자세한 내용은 <a href="https://docs.smith.langchain.com">여기</a>에서 읽어보세요.
    </p>
</div>

## 코드


```python exec="on" source="above" session="1"
# 먼저 사용하고자 하는 모델을 초기화합니다.
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o", temperature=0)


# 이 튜토리얼에서는 두 도시(NYC 및 SF)의 날씨에 대한 미리 정의된 값을 반환하는 사용자 지정 도구를 사용합니다.

from typing import Literal

from langchain_core.tools import tool


@tool
def get_weather(city: Literal["nyc", "sf"]):
    """이것을 사용하여 날씨 정보를 가져옵니다."""
    if city == "nyc":
        return "뉴욕은 흐릴 수 있습니다"
    elif city == "sf":
        return "샌프란시스코는 항상 화창합니다"
    else:
        raise AssertionError("알 수 없는 도시")


tools = [get_weather]


# 그래프 정의

from langgraph.prebuilt import create_react_agent

graph = create_react_agent(model, tools=tools)
```

## 사용법

먼저 방금 생성한 그래프를 시각화해 보겠습니다.


```python exec="on" source="above" session="1"
from IPython.display import Image, display

display(Image(graph.get_graph().draw_mermaid_png()))
```

![](data:image/jpg;base64,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)


```python exec="on" source="above" session="1"
def print_stream(stream):
    for s in stream:
        message = s["messages"][-1]
        if isinstance(message, tuple):
            print(message)
        else:
            message.pretty_print()
```

도구 호출이 필요한 입력으로 앱을 실행해 보겠습니다.


```python exec="on" source="above" session="1" result="ansi"
inputs = {"messages": [("user", "샌프란시스코의 날씨는 어때?")]}
print_stream(graph.stream(inputs, stream_mode="values"))
```

이제 도구가 필요 없는 질문을 시도해 보겠습니다.


```python exec="on" source="above" session="1" result="ansi"
inputs = {"messages": [("user", "너를 만든 사람은 누구야?")]}
print_stream(graph.stream(inputs, stream_mode="values"))
```
