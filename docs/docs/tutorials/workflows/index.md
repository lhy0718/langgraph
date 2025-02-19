_한국어로 기계번역됨_

# 워크플로우와 에이전트

이 가이드는 에이전틱 시스템에 대한 일반적인 패턴을 검토합니다. 이러한 시스템을 설명할 때 "워크플로우"와 "에이전트"를 구분하는 것이 유용할 수 있습니다. 이 차이를 생각하는 한 가지 방법은 Anthropic의 [여기](https://www.anthropic.com/research/building-effective-agents)에서 잘 설명되어 있습니다:

> 워크플로우는 LLM과 도구가 미리 정의된 코드 경로를 통해 조정된 시스템입니다.
> 반면 에이전트는 LLM이 자신의 프로세스와 도구 사용을 동적으로 조정하여 작업을 수행하는 방식을 제어하는 시스템입니다.

다음은 이러한 차이를 시각화하는 간단한 방법입니다:

![Agent Workflow](../../concepts/img/agent_workflow.png)

에이전트와 워크플로우를 구축할 때, LangGraph는 [지속성, 스트리밍, 디버깅 및 배포 지원](https://langchain-ai.github.io/langgraph/concepts/high_level/) 등 여러 가지 이점을 제공합니다.

## 설정

구조화된 출력 및 도구 호출을 지원하는 [모든 채팅 모델](https://python.langchain.com/docs/integrations/chat/)을 사용할 수 있습니다. 아래에서는 패키지 설치, API 키 설정, Anthropic의 구조화된 출력 / 도구 호출 테스트 과정을 보여줍니다.

??? "종속성 설치"

    ```bash
    pip install langchain_core langchain-anthropic langgraph
    ```

LLM 초기화

```python
import os
import getpass

from langchain_anthropic import ChatAnthropic

def _set_env(var: str):
    if not os.environ.get(var):
        os.environ[var] = getpass.getpass(f"{var}: ")

_set_env("ANTHROPIC_API_KEY")

llm = ChatAnthropic(model="claude-3-5-sonnet-latest")
```

## 빌딩 블록: 증강된 LLM

LLM은 [워크플로우와 에이전트 구축을 지원하는 증강](https://www.anthropic.com/research/building-effective-agents) 기능이 있습니다. 여기에는 [구조화된 출력](https://python.langchain.com/docs/concepts/structured_outputs/)과 [도구 호출](https://python.langchain.com/docs/concepts/tool_calling/)이 포함됩니다. 이는 Anthropic의 [블로그](https://www.anthropic.com/research/building-effective-agents)에서 보여지는 이미지입니다:

![augmented_llm.png](./img/augmented_llm.png)

```python
# 구조화 출력 스키마
from pydantic import BaseModel, Field

class SearchQuery(BaseModel):
    search_query: str = Field(None, description="최적화된 웹 검색 쿼리.")
    justification: str = Field(
        None, description="이 쿼리가 사용자의 요청과 관련된 이유."
    )

# 구조화 출력을 위한 스키마로 LLM 증강
structured_llm = llm.with_structured_output(SearchQuery)

# 증강된 LLM 호출
output = structured_llm.invoke("칼슘 CT 점수는 고콜레스테롤과 어떻게 관련이 있습니까?")

# 도구 정의
def multiply(a: int, b: int) -> int:
    return a * b

# 도구로 LLM 증강
llm_with_tools = llm.bind_tools([multiply])

# 도구 호출을 트리거하는 입력으로 LLM 호출
msg = llm_with_tools.invoke("2 곱하기 3은 얼마입니까?")

# 도구 호출 확인
msg.tool_calls
```

## 프롬프트 체인

프롬프트 체인에서는 각 LLM 호출이 이전 호출의 출력을 처리합니다.

[Anthropic 블로그](https://www.anthropic.com/research/building-effective-agents)에서 언급한 바와 같이:

> 프롬프트 체인은 작업을 일련의 단계로 분해하며, 각 LLM 호출이 이전 호출의 출력을 처리합니다. 프로그래밍 검사를 추가하여(아래 다이어그램에서 "게이트" 참조) 프로세스가 여전히 궤도에 있는지 확인할 수 있습니다.

> 이 워크플로우를 사용하는 적절한 시점: 이 워크플로우는 작업이 고정된 하위 작업으로 쉽게 분해될 수 있는 상황에 이상적입니다. 주요 목표는 각 LLM 호출을 더 쉬운 작업으로 만들어 지연 시간을 높이는 대신 정확성을 향상시키는 것입니다.

![prompt_chain.png](./img/prompt_chain.png)

=== "그래프 API"

    ```python
    from typing_extensions import TypedDict
    from langgraph.graph import StateGraph, START, END
    from IPython.display import Image, display


    # 그래프 상태
    class State(TypedDict):
        topic: str
        joke: str
        improved_joke: str
        final_joke: str


    # 노드
    def generate_joke(state: State):
        """초기 농담을 생성하기 위한 첫 번째 LLM 호출"""

        msg = llm.invoke(f"{state['topic']}에 대한 짧은 농담을 작성하세요.")
        return {"joke": msg.content}


    def check_punchline(state: State):
        """농담에 펀치라인이 있는지 확인하는 게이트 함수"""

        # 간단한 검사 - 농담에 "?" 또는 "!"가 포함되어 있는지
        if "?" in state["joke"] or "!" in state["joke"]:
            return "Fail"
        return "Pass"


    def improve_joke(state: State):
        """농담을 개선하기 위한 두 번째 LLM 호출"""

        msg = llm.invoke(f"이 농담을 유머있게 만들기 위해 언어유희를 추가하세요: {state['joke']}")
        return {"improved_joke": msg.content}


    def polish_joke(state: State):
        """마무리 다듬기를 위한 세 번째 LLM 호출"""

        msg = llm.invoke(f"이 농담에 놀라운 전환을 추가하세요: {state['improved_joke']}")
        return {"final_joke": msg.content}


    # 워크플로우 구축
    workflow = StateGraph(State)

    # 노드 추가
    workflow.add_node("generate_joke", generate_joke)
    workflow.add_node("improve_joke", improve_joke)
    workflow.add_node("polish_joke", polish_joke)

    # 노드를 연결하는 엣지 추가
    workflow.add_edge(START, "generate_joke")
    workflow.add_conditional_edges(
        "generate_joke", check_punchline, {"Fail": "improve_joke", "Pass": END}
    )
    workflow.add_edge("improve_joke", "polish_joke")
    workflow.add_edge("polish_joke", END)

    # 컴파일
    chain = workflow.compile()

    # 워크플로우 표시
    display(Image(chain.get_graph().draw_mermaid_png()))

    # 호출
    state = chain.invoke({"topic": "고양이"})
    print("초기 농담:")
    print(state["joke"])
    print("\n--- --- ---\n")
    if "improved_joke" in state:
        print("개선된 농담:")
        print(state["improved_joke"])
        print("\n--- --- ---\n")

        print("최종 농담:")
        print(state["final_joke"])
    else:
        print("농담이 품질 게이트를 통과하지 못했습니다 - 펀치라인이 감지되지 않았습니다!")
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/a0281fca-3a71-46de-beee-791468607b75/r

    **리소스:**

    **LangChain 아카데미**

    프롬프트 체인에 대한 강의는 [여기](https://github.com/langchain-ai/langchain-academy/blob/main/module-1/chain.ipynb)에서 확인하세요.

=== "Functional API (beta)"

    ```python
    from langgraph.func import entrypoint, task


    # 작업
    @task
    def generate_joke(topic: str):
        """초기 농담을 생성하기 위한 첫 번째 LLM 호출"""
        msg = llm.invoke(f"{topic}에 대한 짧은 농담을 작성하세요.")
        return msg.content


    def check_punchline(joke: str):
        """농담에 펀치라인이 있는지 확인하는 게이트 함수"""
        # 간단한 확인 - 농담에 "?" 또는 "!"가 포함되어 있는지
        if "?" in joke or "!" in joke:
            return "실패"

        return "합격"


    @task
    def improve_joke(joke: str):
        """농담을 개선하기 위한 두 번째 LLM 호출"""
        msg = llm.invoke(f"이 농담을 말장난을 추가하여 더 재미있게 만드세요: {joke}")
        return msg.content


    @task
    def polish_joke(joke: str):
        """최종 수정을 위한 세 번째 LLM 호출"""
        msg = llm.invoke(f"이 농담에 놀라운 반전을 추가하세요: {joke}")
        return msg.content


    @entrypoint()
    def parallel_workflow(topic: str):
        original_joke = generate_joke(topic).result()
        if check_punchline(original_joke) == "합격":
            return original_joke

        improved_joke = improve_joke(original_joke).result()
        return polish_joke(improved_joke).result()

    # 호출
    for step in parallel_workflow.stream("고양이", stream_mode="업데이트"):
        print(step)
        print("\n")
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/332fa4fc-b6ca-416e-baa3-161625e69163/r

## 병렬화

병렬화를 통해 LLM이 작업을 동시에 수행합니다:

> LLM은 때때로 작업을 동시에 수행할 수 있으며, 그 결과물을 프로그래밍 방식으로 집계할 수 있습니다. 이 작업 흐름, 즉 병렬화는 두 가지 주요 변형으로 나타납니다: 섹션화: 작업을 독립적인 하위 작업으로 나누어 병렬로 실행합니다. 투표: 동일한 작업을 여러 번 실행하여 다양한 출력을 얻습니다.

> 이 작업 흐름을 사용할 때: 병렬화는 나눠진 하위 작업이 속도를 위해 병렬화될 수 있거나, 더 높은 신뢰성의 결과를 위해 여러 관점이나 시도가 필요할 때 효과적입니다. 여러 고려 사항이 있는 복잡한 작업의 경우, LLM은 일반적으로 각 고려 사항이 별도의 LLM 호출에 의해 처리되면 각 특정 측면에 집중할 수 있어 더 나은 성능을 보입니다.

![parallelization.png](./img/parallelization.png)

=== "그래프 API"

    ```python
    # 그래프 상태
    class State(TypedDict):
        topic: str
        joke: str
        story: str
        poem: str
        combined_output: str


    # 노드
    def call_llm_1(state: State):
        """첫 번째 LLM 호출로 초기 농담 생성"""

        msg = llm.invoke(f"{state['topic']}에 대한 농담을 작성해 주세요.")
        return {"joke": msg.content}


    def call_llm_2(state: State):
        """두 번째 LLM 호출로 이야기 생성"""

        msg = llm.invoke(f"{state['topic']}에 대한 이야기를 작성해 주세요.")
        return {"story": msg.content}


    def call_llm_3(state: State):
        """세 번째 LLM 호출로 시 생성"""

        msg = llm.invoke(f"{state['topic']}에 대한 시를 작성해 주세요.")
        return {"poem": msg.content}


    def aggregator(state: State):
        """농담과 이야기를 하나의 출력으로 결합"""

        combined = f"{state['topic']}에 대한 이야기, 농담, 시입니다!\n\n"
        combined += f"이야기:\n{state['story']}\n\n"
        combined += f"농담:\n{state['joke']}\n\n"
        combined += f"시:\n{state['poem']}"
        return {"combined_output": combined}


    # 워크플로우 구축
    parallel_builder = StateGraph(State)

    # 노드 추가
    parallel_builder.add_node("call_llm_1", call_llm_1)
    parallel_builder.add_node("call_llm_2", call_llm_2)
    parallel_builder.add_node("call_llm_3", call_llm_3)
    parallel_builder.add_node("aggregator", aggregator)

    # 노드를 연결하는 엣지 추가
    parallel_builder.add_edge(START, "call_llm_1")
    parallel_builder.add_edge(START, "call_llm_2")
    parallel_builder.add_edge(START, "call_llm_3")
    parallel_builder.add_edge("call_llm_1", "aggregator")
    parallel_builder.add_edge("call_llm_2", "aggregator")
    parallel_builder.add_edge("call_llm_3", "aggregator")
    parallel_builder.add_edge("aggregator", END)
    parallel_workflow = parallel_builder.compile()

    # 워크플로우 보여주기
    display(Image(parallel_workflow.get_graph().draw_mermaid_png()))

    # 호출
    state = parallel_workflow.invoke({"topic": "고양이"})
    print(state["combined_output"])
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/3be2e53c-ca94-40dd-934f-82ff87fac277/r

    **리소스:**

    **문서**

    병렬화에 대한 문서는 [여기](https://langchain-ai.github.io/langgraph/how-tos/branching/)에서 확인하세요.

    **LangChain 아카데미**

    병렬화에 대한 수업은 [여기](https://github.com/langchain-ai/langchain-academy/blob/main/module-1/simple-graph.ipynb)에서 확인하세요.

=== "기능적 API (베타)"

    ```python
    @task
    def call_llm_1(topic: str):
        """첫 번째 LLM 호출로 초기 농담 생성"""
        msg = llm.invoke(f"{topic}에 관한 농담을 써 주세요.")
        return msg.content


    @task
    def call_llm_2(topic: str):
        """두 번째 LLM 호출로 이야기 생성"""
        msg = llm.invoke(f"{topic}에 관한 이야기를 써 주세요.")
        return msg.content


    @task
    def call_llm_3(topic):
        """세 번째 LLM 호출로 시 생성"""
        msg = llm.invoke(f"{topic}에 관한 시를 써 주세요.")
        return msg.content


    @task
    def aggregator(topic, joke, story, poem):
        """농담과 이야기를 단일 출력으로 결합"""

        combined = f"{topic}에 관한 이야기, 농담 및 시입니다!\n\n"
        combined += f"이야기:\n{story}\n\n"
        combined += f"농담:\n{joke}\n\n"
        combined += f"시:\n{poem}"
        return combined


    # 워크플로우 구축
    @entrypoint()
    def parallel_workflow(topic: str):
        joke_fut = call_llm_1(topic)
        story_fut = call_llm_2(topic)
        poem_fut = call_llm_3(topic)
        return aggregator(
            topic, joke_fut.result(), story_fut.result(), poem_fut.result()
        ).result()

    # 호출
    for step in parallel_workflow.stream("고양이", stream_mode="updates"):
        print(step)
        print("\n")
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/623d033f-e814-41e9-80b1-75e6abb67801/r

## 라우팅

라우팅은 입력을 분류하고 후속 작업으로 안내합니다. [Anthropic 블로그](https://www.anthropic.com/research/building-effective-agents)에서 언급했듯이:

> 라우팅은 입력을 분류하고 이를 전문화된 후속 작업으로 안내합니다. 이 워크플로우는 관심사의 분리를 허용하고 더 전문화된 프롬프트를 구축할 수 있습니다. 이 워크플로우가 없을 경우, 하나의 입력 유형에 최적화하면 다른 입력에서 성능이 저하될 수 있습니다.

> 이 워크플로우를 사용해야 할 때: 라우팅은 복잡한 작업에 매우 잘 작동하며, 서로 다르게 처리하는 것이 더 나은 구별된 범주가 존재하고, LLM 또는 보다 전통적인 분류 모델/알고리즘에 의해 분류가 정확하게 처리될 수 있는 경우에 적합합니다.

![routing.png](./img/routing.png)

=== "그래프 API"

```python

from typing_extensions import Literal
from langchain_core.messages import HumanMessage, SystemMessage

# 구조화된 출력을 위한 스키마로 라우팅 로직 사용

class Route(BaseModel):
step: Literal["poem", "story", "joke"] = Field(
None, description="라우팅 과정의 다음 단계"
)

# 구조화된 출력을 위한 스키마로 LLM 보강

router = llm.with_structured_output(Route)

# 상태

class State(TypedDict):
input: str
decision: str
output: str

# 노드

def llm_call_1(state: State):
"""이야기 쓰기"""

    result = llm.invoke(state["input"])
    return {"output": result.content}

def llm_call_2(state: State):
"""농담 쓰기"""

    result = llm.invoke(state["input"])
    return {"output": result.content}

def llm_call_3(state: State):
"""시 쓰기"""

    result = llm.invoke(state["input"])
    return {"output": result.content}

def llm_call_router(state: State):
"""입력을 적절한 노드로 라우팅"""

    # 라우팅 로직으로서 구조화된 출력을 사용하여 보강된 LLM 실행
    decision = router.invoke(
        [
            SystemMessage(
                content="사용자의 요청에 따라 입력을 이야기, 농담, 또는 시로 라우팅하세요."
            ),
            HumanMessage(content=state["input"]),
        ]
    )

    return {"decision": decision.step}

# 적절한 노드로 라우팅하기 위한 조건부 엣지 함수

def route_decision(state: State): # 방문할 다음 노드 이름 반환
if state["decision"] == "story":
return "llm_call_1"
elif state["decision"] == "joke":
return "llm_call_2"
elif state["decision"] == "poem":
return "llm_call_3"

# 워크플로우 구축

router_builder = StateGraph(State)

# 노드 추가

router_builder.add_node("llm_call_1", llm_call_1)
router_builder.add_node("llm_call_2", llm_call_2)
router_builder.add_node("llm_call_3", llm_call_3)
router_builder.add_node("llm_call_router", llm_call_router)

# 노드를 연결하기 위한 엣지 추가

router_builder.add_edge(START, "llm_call_router")
router_builder.add_conditional_edges(
"llm_call_router",
route_decision,
{ # route_decision에서 반환된 이름 : 다음 방문할 노드 이름
"llm_call_1": "llm_call_1",
"llm_call_2": "llm_call_2",
"llm_call_3": "llm_call_3",
},
)
router_builder.add_edge("llm_call_1", END)
router_builder.add_edge("llm_call_2", END)
router_builder.add_edge("llm_call_3", END)

# 워크플로우 컴파일

router_workflow = router_builder.compile()

# 워크플로우 표시

display(Image(router_workflow.get_graph().draw_mermaid_png()))

# 호출

state = router_workflow.invoke({"input": "고양이에 대한 농담을 만들어 줘"})
print(state["output"])

```

**LangSmith 추적**

https://smith.langchain.com/public/c4580b74-fe91-47e4-96fe-7fac598d509c/r

**자원:**

**LangChain 아카데미**

라우팅에 대한 우리의 수업을 [여기](https://github.com/langchain-ai/langchain-academy/blob/main/module-1/router.ipynb)에서 확인하세요.

**예시**

질문을 라우팅하는 RAG 작업 흐름은 [여기](https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_adaptive_rag_local/)에서 확인할 수 있습니다. 우리의 비디오는 [여기](https://www.youtube.com/watch?v=bq1Plo2RhYI)에서 확인하세요.

=== "기능적 API (베타)"

    ```python
    from typing_extensions import Literal
    from pydantic import BaseModel
    from langchain_core.messages import HumanMessage, SystemMessage


    # 라우팅 논리로 사용할 구조화된 출력의 스키마
    class Route(BaseModel):
        step: Literal["poem", "story", "joke"] = Field(
            None, description="라우팅 프로세스의 다음 단계"
        )


    # 구조화된 출력을 위한 스키마로 LLM를 증강
    router = llm.with_structured_output(Route)


    @task
    def llm_call_1(input_: str):
        """이야기를 작성합니다"""
        result = llm.invoke(input_)
        return result.content


    @task
    def llm_call_2(input_: str):
        """농담을 작성합니다"""
        result = llm.invoke(input_)
        return result.content


    @task
    def llm_call_3(input_: str):
        """시를 작성합니다"""
        result = llm.invoke(input_)
        return result.content


    def llm_call_router(input_: str):
        """입력을 적절한 노드로 라우팅합니다"""
        # 라우팅 논리로 사용할 구조화된 출력을 가진 증강된 LLM을 실행
        decision = router.invoke(
            [
                SystemMessage(
                    content="사용자의 요청에 따라 입력을 이야기, 농담 또는 시로 라우팅합니다."
                ),
                HumanMessage(content=input_),
            ]
        )
        return decision.step


    # 워크플로우 생성
    @entrypoint()
    def router_workflow(input_: str):
        next_step = llm_call_router(input_)
        if next_step == "story":
            llm_call = llm_call_1
        elif next_step == "joke":
            llm_call = llm_call_2
        elif next_step == "poem":
            llm_call = llm_call_3

        return llm_call(input_).result()

    # 호출
    for step in router_workflow.stream("고양이에 관한 농담을 만들어주세요", stream_mode="updates"):
        print(step)
        print("\n")

````

    ```

**LangSmith 추적**

https://smith.langchain.com/public/5e2eb979-82dd-402c-b1a0-a8cceaf2a28a/r

## 오케스트레이터-작업자

오케스트레이터-작업자는 오케스트레이터가 작업을 세분화하고 각 하위 작업을 작업자에게 위임합니다. [Anthropic 블로그](https://www.anthropic.com/research/building-effective-agents)에서 언급한 바와 같이:

> 오케스트레이터-작업자 워크플로우에서, 중앙 LLM은 작업을 동적으로 세분화하고, 이를 작업자 LLM에 위임하며 결과를 종합합니다.

> 이 워크플로우를 사용할 때: 이 워크플로우는 하위 작업에 대한 예측이 불가능한 복잡한 작업에 적합합니다(예를 들어, 코딩에서는 변경해야 할 파일의 수와 각 파일에서의 변경 내용이 작업에 따라 다를 수 있습니다). 지형적으로 유사하지만, 병렬화와의 주요 차이점은 유연성입니다—하위 작업은 미리 정의된 것이 아니라 특정 입력에 따라 오케스트레이터에 의해 결정됩니다.

![worker.png](./img/worker.png)

=== "그래프 API"

```python
from typing import Annotated, List
import operator


# 계획 수립에 사용할 구조화된 출력의 스키마
class Section(BaseModel):
    name: str = Field(
        description="보고서의 이 섹션에 대한 이름.",
    )
    description: str = Field(
        description="이 섹션에서 다룰 주요 주제와 개념에 대한 간략한 개요.",
    )


class Sections(BaseModel):
    sections: List[Section] = Field(
        description="보고서의 섹션.",
    )


# 구조화된 출력을 위한 스키마로 LLM 증강
planner = llm.with_structured_output(Sections)
```

**LangGraph에서 작업자 생성하기**

오케스트레이터-작업자 워크플로우는 일반적이기 때문에, LangGraph **는 이를 지원하기 위한 `Send` API를 보유하고 있습니다**. 이를 통해 동적으로 작업자 노드를 생성하고 각 작업자에게 특정 입력을 보낼 수 있습니다. 각 작업자는 고유한 상태를 가지며, 모든 작업자의 출력은 오케스트레이터 그래프에서 접근 가능한 *공유 상태 키*에 기록됩니다. 이를 통해 오케스트레이터는 모든 작업자의 출력에 접근할 수 있으며, 이를 최종 출력으로 종합할 수 있습니다. 아래에서 볼 수 있듯이, 우리는 섹션 목록을 반복하고 `Send`를 통해 각 작업자 노드에 전송합니다. 더 자세한 문서는 [여기](https://langchain-ai.github.io/langgraph/how-tos/map-reduce/)와 [여기](https://langchain-ai.github.io/langgraph/concepts/low_level/#send)에서 확인할 수 있습니다.

```python
from langgraph.constants import Send


# 그래프 상태
class State(TypedDict):
    topic: str  # 보고서 주제
    sections: list[Section]  # 보고서 섹션 목록
    completed_sections: Annotated[
        list, operator.add
    ]  # 모든 작업자가 이 키에 병렬로 기록
    final_report: str  # 최종 보고서


# 작업자 상태
class WorkerState(TypedDict):
    section: Section
    completed_sections: Annotated[list, operator.add]


# 노드
def orchestrator(state: State):
    """보고서 계획을 생성하는 오케스트레이터"""

    # 쿼리 생성
    report_sections = planner.invoke(
        [
            SystemMessage(content="보고서 계획을 생성하세요."),
            HumanMessage(content=f"여기 보고서 주제가 있습니다: {state['topic']}"),
        ]
    )

    return {"sections": report_sections.sections}


def llm_call(state: WorkerState):
    """작업자가 보고서 섹션을 작성함"""

    # 섹션 생성
    section = llm.invoke(
        [
            SystemMessage(
                content="제공된 이름과 설명을 따라 보고서 섹션을 작성하세요. 각 섹션에 대한 서문은 포함하지 마세요. 마크다운 형식을 사용하세요."
            ),
            HumanMessage(
                content=f"여기 섹션 이름: {state['section'].name} 및 설명: {state['section'].description}가 있습니다."
            ),
        ]
    )

    # 완료된 섹션에 업데이트된 섹션 기록
    return {"completed_sections": [section.content]}
```

    ```python

def synthesizer(state: State):
"""섹션에서 전체 보고서를 합성합니다."""

        # 완료된 섹션의 목록
        completed_sections = state["completed_sections"]

        # 완료된 섹션을 문자로 포맷하여 최종 섹션의 맥락으로 사용
        completed_report_sections = "\n\n---\n\n".join(completed_sections)

        return {"final_report": completed_report_sections}


    # 각 보고서 섹션에 작성자를 할당하는 조건부 엣지 함수
    def assign_workers(state: State):
        """계획에서 각 섹션에 작업자를 할당합니다."""

        # Send() API를 통해 섹션 작성을 병렬로 시작
        return [Send("llm_call", {"section": s}) for s in state["sections"]]


    # 워크플로우 빌드
    orchestrator_worker_builder = StateGraph(State)

    # 노드 추가
    orchestrator_worker_builder.add_node("orchestrator", orchestrator)
    orchestrator_worker_builder.add_node("llm_call", llm_call)
    orchestrator_worker_builder.add_node("synthesizer", synthesizer)

    # 노드를 연결하는 엣지 추가
    orchestrator_worker_builder.add_edge(START, "orchestrator")
    orchestrator_worker_builder.add_conditional_edges(
        "orchestrator", assign_workers, ["llm_call"]
    )
    orchestrator_worker_builder.add_edge("llm_call", "synthesizer")
    orchestrator_worker_builder.add_edge("synthesizer", END)

    # 워크플로우 컴파일
    orchestrator_worker = orchestrator_worker_builder.compile()

    # 워크플로우 보여주기
    display(Image(orchestrator_worker.get_graph().draw_mermaid_png()))

    # 호출
    state = orchestrator_worker.invoke({"topic": "LLM 스케일링 법칙에 대한 보고서 작성"})

    from IPython.display import Markdown
    Markdown(state["final_report"])
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/78cbcfc3-38bf-471d-b62a-b299b144237d/r

    **자료:**

    **LangChain 아카데미**

    [여기](https://github.com/langchain-ai/langchain-academy/blob/main/module-4/map-reduce.ipynb)에서 오케스트레이터-작업자에 대한 강의를 확인하세요.

    **예시**

    [여기](https://github.com/langchain-ai/report-mAIstro)에는 보고서 계획 및 작성을 위한 오케스트레이터-작업자를 사용하는 프로젝트가 있습니다. 우리의 동영상은 [여기](https://www.youtube.com/watch?v=wSxZ7yFbbas)에서 확인하세요.

=== "기능적 API (베타)"

    ```python
    from typing import List


    # 계획에 사용할 구조화된 출력을 위한 스키마
    class Section(BaseModel):
        name: str = Field(
            description="보고서의 이 섹션에 대한 이름입니다.",
        )
        description: str = Field(
            description="이 섹션에서 다룰 주요 주제 및 개념에 대한 간략한 개요입니다.",
        )


    class Sections(BaseModel):
        sections: List[Section] = Field(
            description="보고서의 섹션입니다.",
        )


    # 구조화된 출력을 위한 스키마로 LLM을 확장
    planner = llm.with_structured_output(Sections)


    @task
    def orchestrator(topic: str):
        """보고서 계획을 생성하는 오케스트레이터"""
        # 쿼리 생성
        report_sections = planner.invoke(
            [
                SystemMessage(content="보고서 계획을 생성합니다."),
                인간 메시지(content=f"여기 보고서 주제가 있습니다: {topic}"),
            ]
        )

        보고서 섹션을 반환합니다.


    @task
    def llm_call(section: Section):
        """작업자가 보고서 섹션을 작성합니다."""

        # 섹션 생성
        result = llm.invoke(
            [
                SystemMessage(content="보고서 섹션을 작성하세요."),
                HumanMessage(
                    content=f"여기 섹션 이름이 있습니다: {section.name} 및 설명: {section.description}"
                ),
            ]
        )

        # 완료된 섹션에 업데이트된 섹션을 작성합니다.
        return result.content


    @task
    def synthesizer(completed_sections: list[str]):
        """섹션에서 전체 보고서를 합성합니다."""
        final_report = "\n\n---\n\n".join(completed_sections)
        return final_report


    @entrypoint()
    def orchestrator_worker(topic: str):
        sections = orchestrator(topic).result()
        section_futures = [llm_call(section) for section in sections]
        final_report = synthesizer(
            [section_fut.result() for section_fut in section_futures]
        ).result()
        return final_report

    # 호출
    report = orchestrator_worker.invoke("LLM 스케일링 법칙에 대한 보고서 작성")
    from IPython.display import Markdown
    Markdown(report)
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/75a636d0-6179-4a12-9836-e0aa571e87c5/r

## 평가자-최적화기

평가자-최적화기 워크플로우에서는 한 LLM 호출이 응답을 생성하고 다른 LLM 호출이 평가 및 피드백을 제공하는 루프가 형성됩니다:

> 평가자-최적화기 워크플로우에서는 한 LLM 호출이 응답을 생성하고 다른 호출이 평가 및 피드백을 제공하는 루프가 형성됩니다.

> 이 워크플로우를 사용하는 시점: 이 워크플로우는 명확한 평가 기준이 있을 때 특히 효과적이며, 반복적인 개선이 측정 가능한 가치를 제공할 때 유용합니다. 잘 맞는 두 가지 신호는 첫째, LLM의 응답이 인간이 그들의 피드백을 명확히 할 때 입증 가능하게 개선될 수 있다는 점이며, 둘째, LLM이 그러한 피드백을 제공할 수 있어야 합니다. 이는 인간 작가가 세련된 문서를 작성할 때 겪을 수 있는 반복적인 글쓰기 과정과 유사합니다.

![evaluator_optimizer.png](./img/evaluator_optimizer.png)

=== "그래프 API"

    ```python
    # 그래프 상태
    class State(TypedDict):
        joke: str
        topic: str
        feedback: str
        funny_or_not: str


    # 평가에 사용할 구조화된 출력의 스키마
    class Feedback(BaseModel):
        grade: Literal["funny", "not funny"] = Field(
            description="농담이 웃긴지 아닌지 결정합니다.",
        )
        feedback: str = Field(
            description="농담이 웃기지 않으면, 개선 방법에 대한 피드백을 제공합니다.",
        )


    # 구조화된 출력을 위한 스키마로 LLM을 보강합니다.
    evaluator = llm.with_structured_output(Feedback)


    # 노드
    def llm_call_generator(state: State):
        """LLM이 농담을 생성합니다."""

        if state.get("feedback"):
            msg = llm.invoke(
                f"{state['topic']}에 관한 농담을 작성하되 피드백을 고려하세요: {state['feedback']}"
            )
        else:
            msg = llm.invoke(f"{state['topic']}에 관한 농담을 작성하세요.")
        return {"joke": msg.content}

    def llm_call_evaluator(state: State):
    """LLM이 농담을 평가합니다."""

        grade = evaluator.invoke(f"농담을 평가해 주세요: {state['joke']}")
        return {"재미있음_or_not": grade.grade, "피드백": grade.feedback}


    # 평가자의 피드백에 따라 농담 생성기로 되돌리거나 종료하는 조건부 경로 함수
    def route_joke(state: State):
        """평가자의 피드백에 따라 농담 생성기로 되돌리거나 종료합니다."""

        if state["재미있음_or_not"] == "재미있음":
            return "승인됨"
        elif state["재미있음_or_not"] == "재미없음":
            return "거부됨 + 피드백"


    # 워크플로우 구축
    optimizer_builder = StateGraph(State)

    # 노드를 추가합니다.
    optimizer_builder.add_node("llm_call_generator", llm_call_generator)
    optimizer_builder.add_node("llm_call_evaluator", llm_call_evaluator)

    # 노드를 연결하는 엣지를 추가합니다.
    optimizer_builder.add_edge(START, "llm_call_generator")
    optimizer_builder.add_edge("llm_call_generator", "llm_call_evaluator")
    optimizer_builder.add_conditional_edges(
        "llm_call_evaluator",
        route_joke,
        {  # route_joke에 의해 반환된 이름 : 다음 방문할 노드의 이름
            "승인됨": END,
            "거부됨 + 피드백": "llm_call_generator",
        },
    )

    # 워크플로우를 컴파일합니다.
    optimizer_workflow = optimizer_builder.compile()

    # 워크플로우를 표시합니다.
    display(Image(optimizer_workflow.get_graph().draw_mermaid_png()))

    # 호출합니다.
    state = optimizer_workflow.invoke({"주제": "고양이"})
    print(state["농담"])
    ```

    **LangSmith 추적**

    https://smith.langchain.com/public/86ab3e60-2000-4bff-b988-9b89a3269789/r

    **자료:**

    **예시**

    [여기](https://github.com/langchain-ai/research-rabbit)에는 evaluator-optimizer를 사용하여 보고서를 개선하는 도우미가 있습니다. 비디오는 [여기](https://www.youtube.com/watch?v=XGuTzHoqlj8)에서 확인하세요.

    [여기](https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_adaptive_rag_local/)는 환각이나 오류에 대해 답변을 평가하는 RAG 워크플로우입니다. 비디오는 [여기](https://www.youtube.com/watch?v=bq1Plo2RhYI)에서 확인하세요.

=== "기능적 API (베타)"

    ```python
    # 평가에 사용할 구조화된 출력의 스키마
    class Feedback(BaseModel):
        grade: Literal["재미있음", "재미없음"] = Field(
            description="농담이 재미있는지 여부를 결정합니다.",
        )
        feedback: str = Field(
            description="농담이 재미없다면 개선 방법에 대한 피드백을 제공합니다.",
        )


    # 구조화된 출력 스키마로 LLM을 보강합니다.
    evaluator = llm.with_structured_output(Feedback)


    # 노드
    @task
    def llm_call_generator(topic: str, feedback: Feedback):
        """LLM이 농담을 생성합니다."""
        if feedback:
            msg = llm.invoke(
                f"{topic}에 대한 농담을 작성하되 피드백: {feedback}을 고려하세요."
            )
        else:
            msg = llm.invoke(f"{topic}에 대한 농담을 작성하세요.")
        return msg.content


    @task
    def llm_call_evaluator(joke: str):
        """LLM이 농담을 평가합니다."""
        feedback = evaluator.invoke(f"농담을 평가해 주세요: {joke}")
        return feedback


    @entrypoint()
    def optimizer_workflow(topic: str):
        feedback = None


        #무한 루프:

    while True:
        joke = llm_call_generator(topic, feedback).result()
        feedback = llm_call_evaluator(joke).result()
        if feedback.grade == "funny":
            break

    return joke

    # 호출
    for step in optimizer_workflow.stream("Cats", stream_mode="updates"):
        print(step)
        print("\n")

````

**LangSmith Trace**

https://smith.langchain.com/public/f66830be-4339-4a6b-8a93-389ce5ae27b4/r

## 에이전트

에이전트는 일반적으로 환경 피드백에 따라 행동(도구 호출을 통해)을 수행하는 LLM으로 구현됩니다. [Anthropic 블로그](https://www.anthropic.com/research/building-effective-agents)에서 언급했듯이:

> 에이전트는 정교한 작업을 처리할 수 있지만, 그 구현은 종종 간단합니다. 일반적으로 환경 피드백을 기반으로 도구를 사용하는 LLM입니다. 따라서 도구 세트와 그 문서를 명확하고 신중하게 설계하는 것이 중요합니다.

> 에이전트를 사용할 때: 에이전트는 필요한 단계 수를 예측하기 어렵거나 불가능한 개방형 문제에 사용할 수 있으며, 고정된 경로를 하드코딩할 수 없는 경우에 적합합니다. LLM은 여러 턴 동안 작동할 수 있으며, 의사 결정에 일정 수준의 신뢰가 필요합니다. 에이전트의 자율성은 신뢰할 수 있는 환경에서 작업을 확장하는 데 이상적입니다.

![agent.png](./img/agent.png)

```python
from langchain_core.tools import tool

# 도구 정의
@tool
def multiply(a: int, b: int) -> int:
    """a와 b를 곱합니다.

    인수:
        a: 첫 번째 정수
        b: 두 번째 정수
    """
    return a * b

@tool
def add(a: int, b: int) -> int:
    """a와 b를 더합니다.

    인수:
        a: 첫 번째 정수
        b: 두 번째 정수
    """
    return a + b

@tool
def divide(a: int, b: int) -> float:
    """a를 b로 나눕니다.

    인수:
        a: 첫 번째 정수
        b: 두 번째 정수
    """
    return a / b

# 도구로 LLM 확장
tools = [add, multiply, divide]
tools_by_name = {tool.name: tool for tool in tools}
llm_with_tools = llm.bind_tools(tools)
```

=== "그래프 API"

```python
from langgraph.graph import MessagesState
from langchain_core.messages import SystemMessage, HumanMessage, ToolMessage

# 노드
def llm_call(state: MessagesState):
    """LLM이 도구를 호출할지 여부를 결정합니다."""

    return {
        "messages": [
            llm_with_tools.invoke(
                [
                    SystemMessage(
                        content="당신은 입력 세트에 대한 산술을 수행하는 유용한 어시스턴트입니다."
                    )
                ]
                + state["messages"]
            )
        ]
    }

def tool_node(state: dict):
```

        """도구 호출을 수행합니다."""

        결과 = []
        for tool_call in state["messages"][-1].tool_calls:
            도구 = tools_by_name[tool_call["name"]]
            관찰 = 도구.invoke(tool_call["args"])
            결과.append(ToolMessage(content=관찰, tool_call_id=tool_call["id"]))
        return {"messages": 결과}


    # LLM이 도구 호출을 했는지 여부에 따라 도구 노드 또는 종료로 라우팅하는 조건부 엣지 함수
    def should_continue(state: MessagesState) -> Literal["environment", END]:
        """LLM이 도구 호출을 했는지 여부에 따라 루프를 계속할지 중단할지 결정합니다."""

        메시지 = state["messages"]
        마지막_메시지 = 메시지[-1]
        # LLM이 도구 호출을 하면 작업을 수행합니다.
        if 마지막_메시지.tool_calls:
            return "Action"
        # 그렇지 않으면 중단합니다 (사용자에게 응답)
        return END


    # 워크플로우 구성
    agent_builder = StateGraph(MessagesState)

    # 노드 추가
    agent_builder.add_node("llm_call", llm_call)
    agent_builder.add_node("environment", tool_node)

    # 노드를 연결하는 엣지 추가
    agent_builder.add_edge(START, "llm_call")
    agent_builder.add_conditional_edges(
        "llm_call",
        should_continue,
        {
            # should_continue가 반환하는 이름 : 방문할 다음 노드 이름
            "Action": "environment",
            END: END,
        },
    )
    agent_builder.add_edge("environment", "llm_call")

    # 에이전트 컴파일
    agent = agent_builder.compile()

    # 에이전트 표시
    display(Image(agent.get_graph(xray=True).draw_mermaid_png()))

    # 호출
    메시지 = [HumanMessage(content="3과 4를 더하세요.")]
    메시지 = agent.invoke({"messages": 메시지})
    for m in 메시지["messages"]:
        m.pretty_print()

**LangSmith 추적**

https://smith.langchain.com/public/051f0391-6761-4f8c-a53b-22231b016690/r

**자원:**

**LangChain 아카데미**

에이전트에 대한 레슨은 [여기](https://github.com/langchain-ai/langchain-academy/blob/main/module-1/agent.ipynb)에서 확인하실 수 있습니다.

**예제**

[여기](https://github.com/langchain-ai/memory-agent)에는 도구 호출 에이전트를 사용하여 장기 기억을 생성/저장하는 프로젝트가 있습니다.

=== "기능적 API(베타)"

````python
from langgraph.graph import add_messages
from langchain_core.messages import (
    SystemMessage,
    HumanMessage,
    BaseMessage,
    ToolCall,
)


@task
def call_llm(messages: list[BaseMessage]):
    """LLM이 도구를 호출할지 여부를 결정합니다."""
    return llm_with_tools.invoke(
        [
            SystemMessage(
                content="당신은 입력 세트에서 산술 작업을 수행하는 유용한 비서입니다."
            )
        ]
        + messages
    )


@task
def call_tool(tool_call: ToolCall):
    """도구 호출을 수행합니다."""
    도구 = tools_by_name[tool_call["name"]]
        ```
tool.invoke(tool_call를 반환합니다)


    @entrypoint()
    def agent(messages: list[BaseMessage]):
        llm_response = call_llm(messages).result()

        while True:
            if not llm_response.tool_calls:
                break

            # 도구 실행
            tool_result_futures = [
                call_tool(tool_call) for tool_call in llm_response.tool_calls
            ]
            tool_results = [fut.result() for fut in tool_result_futures]
            messages = add_messages(messages, [llm_response, *tool_results])
            llm_response = call_llm(messages).result()

        messages = add_messages(messages, llm_response)
        return messages

    # 호출
    messages = [HumanMessage(content="3과 4를 더하세요.")]
    for chunk in agent.stream(messages, stream_mode="updates"):
        print(chunk)
        print("\n")
    ```

    **LangSmith Trace**

    https://smith.langchain.com/public/42ae8bf9-3935-4504-a081-8ddbcbfc8b2e/r

#### 미리 구축된

LangGraph는 위에서 정의된 에이전트를 생성하기 위한 **미리 구축된 방법**도 제공합니다 ( [`create_react_agent`][langgraph.prebuilt.chat_agent_executor.create_react_agent] 함수를 사용):

https://langchain-ai.github.io/langgraph/how-tos/create-react-agent/

```python
from langgraph.prebuilt import create_react_agent

# 다음을 전달합니다:
# (1) 도구가 포함된 증강된 LLM
# (2) 도구 목록 (도구 노드 생성을 위해 사용됨)
pre_built_agent = create_react_agent(llm, tools=tools)

# 에이전트 표시
display(Image(pre_built_agent.get_graph().draw_mermaid_png()))

# 호출
messages = [HumanMessage(content="3과 4를 더하세요.")]
messages = pre_built_agent.invoke({"messages": messages})
for m in messages["messages"]:
    m.pretty_print()
````

```

**LangSmith 추적**

https://smith.langchain.com/public/abab6a44-29f6-4b97-8164-af77413e494d/r

## LangGraph가 제공하는 것

LangGraph에서 위의 각 요소를 구성함으로써 몇 가지를 얻을 수 있습니다:

### 지속성: 인간 개입

LangGraph의 지속성 계층은 작업의 중단 및 승인을 지원합니다(예: 인간 개입). [LangChain 아카데미의 3모듈](https://github.com/langchain-ai/langchain-academy/tree/main/module-3)을 참조하십시오.

### 지속성: 기억

LangGraph의 지속성 계층은 대화형(단기) 기억과 장기 기억을 지원합니다. [LangChain 아카데미의 2모듈](https://github.com/langchain-ai/langchain-academy/tree/main/module-2)과 [5모듈](https://github.com/langchain-ai/langchain-academy/tree/main/module-5)을 참조하십시오:

### 스트리밍

LangGraph는 워크플로우/에이전트 출력 또는 중간 상태를 스트리밍할 수 있는 여러 가지 방법을 제공합니다. [LangChain 아카데미의 3모듈](https://github.com/langchain-ai/langchain-academy/blob/main/module-3/streaming-interruption.ipynb)을 참조하십시오.

### 배포

LangGraph는 배포, 관측 가능성 및 평가를 위한 쉬운 진입점을 제공합니다. [LangChain 아카데미의 6모듈](https://github.com/langchain-ai/langchain-academy/tree/main/module-6)을 참조하십시오.
```
