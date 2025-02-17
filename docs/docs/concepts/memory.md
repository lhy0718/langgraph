_한국어로 기계번역됨_

# 메모리

## 메모리란 무엇인가?

[메모리](https://pmc.ncbi.nlm.nih.gov/articles/PMC10410470/)는 사람들이 정보를 저장하고, 검색하고, 사용하여 현재와 미래를 이해할 수 있게 해주는 인지 기능입니다. 당신이 말한 모든 것을 잊어버리는 동료와 함께 일할 때의 좌절감을 생각해 보세요. 끊임없는 반복이 필요하겠죠! AI 에이전트가 수많은 사용자 상호작용을 포함하는 더 복잡한 작업을 수행하게 되면서, 메모리를 갖추는 것이 효율성과 사용자 만족을 위해 똑같이 중요해졌습니다. 메모리를 통해 에이전트는 피드백에서 학습하고 사용자 선호에 적응할 수 있습니다. 이 가이드는 회상 범위에 따라 두 가지 유형의 메모리를 다룹니다:

**단기 기억** 또는 [스레드](persistence.md#threads) 범위 메모리는 사용자와의 단일 대화 스레드 내에서 **언제든지** 호출할 수 있습니다. LangGraph는 단기 기억을 에이전트의 [상태](low_level.md#state) 일부분으로 관리합니다. 상태는 [체크포인터](persistence.md#checkpoints)를 사용하여 데이터베이스에 저장되어 스레드를 언제든지 재개할 수 있습니다. 단기 기억은 그래프가 호출되거나 단계가 완료될 때 업데이트되며 각 단계 시작 시 상태가 읽힙니다.

**장기 기억**은 대화 스레드 **전체에 걸쳐** 공유됩니다. 이는 _언제든지_ **모든 스레드에서** 호출할 수 있습니다. 기억은 단일 스레드 ID 내에서뿐만 아니라 사용자 정의 네임스페이스에 따라 범위가 설정됩니다. LangGraph는 장기 기억을 저장하고 회상할 수 있도록 [저장소](persistence.md#memory-store) ([참조 문서](https://langchain-ai.github.io/langgraph/reference/store/#langgraph.store.base.BaseStore))를 제공합니다.

모두 애플리케이션을 이해하고 구현하는 데 중요합니다.

![](img/memory/short-vs-long.png)

## 단기 기억

단기 기억은 애플리케이션이 단일 [스레드](persistence.md#threads) 또는 대화 내에서 이전 상호작용을 기억하게 해 줍니다. [스레드](persistence.md#threads)는 여러 상호작용을 세션 내에서 조직하며, 이는 이메일이 단일 대화에서 메시지를 그룹화하는 것과 유사합니다.

LangGraph는 단기 기억을 에이전트의 상태의 일부분으로 관리하며, 스레드 범위의 체크포인트를 통해 지속됩니다. 이 상태는 대화 기록은 물론 업로드된 파일, 검색된 문서, 생성된 아티팩트와 같은 다른 상태 데이터도 포함될 수 있습니다. 정보를 그래프의 상태에 저장함으로써 봇은 원하는 대화에 대한 전체 맥락에 접근할 수 있고 서로 다른 스레드 간의 분리를 유지할 수 있습니다.

대화 기록은 단기 기억을 표현하는 가장 일반적인 형태이므로, 다음 섹션에서는 메시지 목록이 **길어질** 때 대화 기록을 관리하는 기술을 다루겠습니다. 고급 개념에 집중하고 싶다면 [장기 기억](#long-term-memory) 섹션으로 진행해 주세요.

### 긴 대화 기록 관리하기

긴 대화는 오늘날의 LLM에 도전이 됩니다. 전체 기록이 LLM의 컨텍스트 윈도우에 들어맞지 않아서 복구할 수 없는 오류가 발생할 수 있습니다. 당신의 LLM이 기술적으로 전체 컨텍스트 길이를 지원하더라도, 대부분의 LLM은 여전히 긴 컨텍스트에서 성능이 저조합니다. 그들은 오래된 콘텐츠나 주제와 관련 없는 콘텐츠에 "산만해지며", 동시에 응답 시간 지연과 비용 증가로 고통받습니다.

단기 기억 관리는 [정밀도와 재현율](https://en.wikipedia.org/wiki/Precision_and_recall#:~:text=Precision%20can%20be%20seen%20as,irrelevant%20ones%20are%20also%20returned)을 애플리케이션의 다른 성능 요구 사항(대기 시간 및 비용)과 균형 있게 맞추는 작업입니다. 항상 정보를 어떻게 표현할지에 대해서 비판적으로 생각하고 데이터를 살펴보는 것이 중요합니다. 아래에서는 메시지 목록 관리를 위한 몇 가지 일반적인 기술을 다루고, 여러분이 애플리케이션에 대한 최상의 절충안을 선택할 수 있도록 충분한 맥락을 제공하려고 합니다:

- [메시지 목록 편집하기](#editing-message-lists): 언어 모델에 전달하기 전에 메시지 목록을 잘라내고 필터링하는 방법.
- [과거 대화 요약하기](#summarizing-past-conversations): 메시지 목록을 단순히 필터링하고 싶지 않을 때 사용하는 일반적인 기술입니다.

### 메시지 목록 편집하기

채팅 모델은 [메시지](https://python.langchain.com/docs/concepts/#messages)를 사용하여 컨텍스트를 수용하며, 여기에는 개발자가 제공한 지침(시스템 메시지)과 사용자 입력(인간 메시지)이 포함됩니다. 채팅 애플리케이션에서는 인간 입력과 모델 응답이 번갈아가며 메시지 목록이 시간이 지남에 따라 길어집니다. 컨텍스트 윈도우가 제한되어 있고, 토큰이 많은 메시지 목록은 비용이 많이 들 수 있기 때문에, 많은 애플리케이션이 오래된 정보를 수동으로 제거하거나 잊어버리는 기술로 이득을 볼 수 있습니다.

![](img/memory/filter.png)

가장 직접적인 접근법은 목록에서 오래된 메시지를 제거하는 것입니다(최소 최근 사용 캐시와 유사하게). 

LangGraph에서 목록에서 콘텐츠를 삭제하는 일반적인 기술은 노드에서 시스템에 목록의 일부를 삭제하라는 업데이트를 반환하는 것입니다. 이 업데이트가 어떻게 생길지는 당신이 정하지만, 일반적인 접근은 어떤 값을 유지할 것인지 지정하는 객체나 사전을 반환하도록 하는 것입니다.

```python
def manage_list(existing: list, updates: Union[list, dict]):
    if isinstance(updates, list):
        # 정상적인 경우, 역사에 추가
        return existing + updates
    elif isinstance(updates, dict) and updates["type"] == "keep":
        # 이것이 어떻게 생길지는 당신이 정합니다.
        # 예를 들어, 문자열 "DELETE"를 간단히 받아 전체 목록을 지울 수 있습니다.
        return existing[updates["from"]:updates["to"]]
    # 기타 등등. 업데이트를 해석하는 방법을 정의합니다.

class State(TypedDict):
    my_list: Annotated[list, manage_list]

def my_node(state: State):
    return {
        # "my_list" 필드에 대한 업데이트를 반환하여
        # 인덱스 -5부터 끝까지의 값만 유지하도록 요청합니다(나머지는 삭제)
        "my_list": {"type": "keep", "from": -5, "to": None}
    }
```

LangGraph는 "my_list" 키 아래서 업데이트가 반환될 때마다 `manage_list` "[감소기](low_level.md#reducers)" 함수를 호출합니다. 그 함수 내에서는 어떤 종류의 업데이트를 수용할지 정의합니다. 대개 메시지는 현재 목록에 추가될 것이지만(대화가 성장할 것이지만), 특정 상태의 일부를 "유지"하도록 허용하는 사전을 수용하는 것도 지원을 추가했습니다. 이를 통해 오래된 메시지 맥락을 프로그래밍적으로 삭제할 수 있습니다.

또 다른 일반적인 방법은 삭제할 모든 메시지의 ID를 지정하는 "제거" 객체 목록을 반환하도록 하는 것입니다. 만약 LangChain 메시지와 LangGraph에서 [`add_messages`](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.message.add_messages) 감소기(또는 동일한 기본 기능을 사용하는 `MessagesState`)를 사용하고 있다면, `RemoveMessage`를 사용하여 이를 수행할 수 있습니다.

```python
from langchain_core.messages import RemoveMessage, AIMessage
from langgraph.graph import add_messages
# ... 기타 임포트

class State(TypedDict):
    # add_messages는 ID로 기존 목록에 메시지를 업서트하는 기본값을 가집니다.
    # RemoveMessage가 반환되면, 메시지 목록에서 ID로 메시지를 삭제합니다.
    messages: Annotated[list, add_messages]

def my_node_1(state: State):
    # 상태의 `messages` 목록에 AI 메시지를 추가합니다.
    return {"messages": [AIMessage(content="안녕하세요")]}

def my_node_2(state: State):
    # 상태의 `messages` 목록에서 마지막 2개의 메시지를 제외한 모든 메시지를 삭제합니다.
    delete_messages = [RemoveMessage(id=m.id) for m in state['messages'][:-2]]
    return {"messages": delete_messages}

```

위의 예에서 `add_messages` 감소기는 새로운 메시지를 `messages` 상태 키에 [추가](https://langchain-ai.github.io/langgraph/concepts/low_level/#serialization)할 수 있게 해줍니다. `RemoveMessage`가 보이면 메시지가 목록에서 해당 ID로 삭제됩니다(그리고 RemoveMessage는 버려집니다). LangChain 특정 메시지 처리에 대한 더 많은 정보는 [이 RemoveMessage 사용 방법](https://langchain-ai.github.io/langgraph/how-tos/memory/delete-messages/)을 참조하세요.

이 [가이드](https://langchain-ai.github.io/langgraph/how-tos/memory/manage-conversation-history/)와 [LangChain 아카데미](https://github.com/langchain-ai/langchain-academy/tree/main/module-2) 과정의 모듈 2를 확인하면 예제 사용을 볼 수 있습니다.

### 과거 대화 요약하기

메시지를 잘라내거나 제거하는 과정에서 메시지 큐의 정보를 잃어버릴 위험이 있습니다. 이 때문에 일부 애플리케이션은 메시지 기록을 요약하기 위한 보다 정교한 접근법으로 혜택을 볼 수 있습니다.

![](img/memory/summary.png)

단순한 프롬프트 및 조정 로직을 사용하여 이를 달성할 수 있습니다. 예를 들어, LangGraph에서 우리는 [MessagesState](https://langchain-ai.github.io/langgraph/concepts/low_level/#working-with-messages-in-graph-state)를 확장하여 `summary` 키를 포함할 수 있습니다.

```python
from langgraph.graph import MessagesState
class State(MessagesState):
    summary: str
```

그런 다음, 기존 요약을 다음 요약의 맥락으로 사용하여 채팅 기록의 요약을 생성할 수 있습니다. 이 `summarize_conversation` 노드는 `messages` 상태 키에 메시지가 일정 수 쌓인 후에 호출될 수 있습니다.

```python
def summarize_conversation(state: State):

    # 먼저, 기존 요약을 가져옵니다.
    summary = state.get("summary", "")

    # 요약 프롬프트를 생성합니다.
    if summary:

        # 이미 요약이 존재합니다.
        summary_message = (
            f"현재까지의 대화 요약입니다: {summary}\n\n"
            "위의 새로운 메시지를 고려하여 요약을 확장하세요:"
        )

    else:
        summary_message = "위의 대화의 요약을 작성하세요:"

    # 프롬프트를 우리의 기록에 추가합니다.
    messages = state["messages"] + [HumanMessage(content=summary_message)]
    response = model.invoke(messages)

    # 가장 최근 메시지 2개를 제외한 모든 메시지를 삭제합니다.
    delete_messages = [RemoveMessage(id=m.id) for m in state["messages"][:-2]]
    return {"summary": response.content, "messages": delete_messages}
```

여기서 이 방법을 확인하세요 [here](https://langchain-ai.github.io/langgraph/how-tos/memory/add-summary-conversation-history/) 및 우리의 [LangChain Academy](https://github.com/langchain-ai/langchain-academy/tree/main/module-2) 수업의 모듈 2에서 사용 예를 확인하세요.

### **언제** 메시지를 제거할지 아는 것

대부분의 LLM은 최대 지원 문맥 창(토큰 기준)이 있습니다. 메시지 기록에서 토큰 수를 세아서 이 한계에 접근할 때마다 메시지를 잘라내는 것이 간단한 방법입니다. 단순한 잘라내기는 스스로 구현하기 쉽지만 몇 가지 "머리 아픈" 사항이 있습니다. 일부 모델 API는 메시지 유형의 연속성을 추가로 제한합니다(인간 메시지로 시작해야 하고, 동일한 유형의 메시지가 연속으로 있을 수 없음 등). LangChain을 사용하는 경우, [`trim_messages`](https://python.langchain.com/docs/how_to/trim_messages/#trimming-based-on-token-count) 유틸리티를 사용하여 목록에서 유지할 토큰 수와 경계 처리를 위한 `strategy`(예: 마지막 `max_tokens` 유지)를 지정할 수 있습니다.

아래는 예시입니다.

```python
from langchain_core.messages import trim_messages
trim_messages(
    messages,
    # 마지막 <= n_count 토큰을 메시지에서 유지합니다.
    strategy="last",
    # 모델에 따라 조정하는 것을 잊지 마세요
    # 그렇지 않으면 사용자 정의 토큰 인코더를 전달하세요.
    token_counter=ChatOpenAI(model="gpt-4"),
    # 원하는 대화 길이에 따라 조정하는 것을 잊지 마세요.
    max_tokens=45,
    # 대부분의 채팅 모델은 채팅 기록이 다음 중 하나로 시작되기를 기대합니다:
    # (1) HumanMessage 또는
    # (2) SystemMessage 다음에 HumanMessage
    start_on="human",
    # 대부분의 채팅 모델은 채팅 기록이 다음 중 하나로 끝나기를 기대합니다:
    # (1) HumanMessage 또는
    # (2) ToolMessage
    end_on=("human", "tool"),
    # 일반적으로 우리는 SystemMessage가 원래 기록에 존재한다면 유지하고 싶습니다.
    # SystemMessage는 모델에 대한 특별한 지침이 있습니다.
    include_system=True,
)
```

## 장기 메모리

LangGraph의 장기 메모리는 시스템이 다양한 대화나 세션 간에 정보를 유지할 수 있게 해줍니다. 짧은 메모리와는 달리 **스레드 범위**가 아닌 장기 메모리는 사용자 정의 "네임스페이스" 내에 저장됩니다.

### 메모리 저장

LangGraph는 장기 메모리를 [스토어](persistence.md#memory-store) 내의 JSON 문서로 저장합니다([참조 문서](https://langchain-ai.github.io/langgraph/reference/store/#langgraph.store.base.BaseStore)). 각 메모리는 사용자 정의 `namespace`(폴더와 유사) 및 고유한 `key`(파일명과 유사) 아래 구성됩니다. 네임스페이스는 종종 사용자 또는 조직 ID 또는 정보를 정리하는 데 도움이 되는 다른 레이블을 포함합니다. 이 구조는 메모리의 계층적 구성을 지원합니다. 그런 다음 내용을 필터링하여 네임스페이스 간 검색을 지원합니다. 아래 예제를 참조하세요.

```python
from langgraph.store.memory import InMemoryStore


def embed(texts: list[str]) -> list[list[float]]:
    # 실제 임베딩 함수 또는 LangChain 임베딩 객체로 교체하십시오.
    return [[1.0, 2.0] * len(texts)]


# InMemoryStore는 데이터를 메모리 내 사전으로 저장합니다. 운영 환경에서는 DB 기반 저장소를 사용하세요.
store = InMemoryStore(index={"embed": embed, "dims": 2})
user_id = "my-user"
application_context = "chitchat"
namespace = (user_id, application_context)
store.put(
    namespace,
    "a-memory",
    {
        "rules": [
            "사용자는 짧고 직접적인 언어를 선호합니다.",
            "사용자는 영어와 파이썬만 사용합니다.",
        ],
        "my-key": "my-value",
    },
)
# ID로 "기억"을 가져옵니다.
item = store.get(namespace, "a-memory")
# 이 네임스페이스에서 "기억"을 검색하고, 내용을 필터링하며 벡터 유사도에 따라 정렬합니다.
items = store.search(
    namespace, filter={"my-key": "my-value"}, query="언어 선호도"
)
```

### 장기 기억에 대한 사고 프레임워크

장기 기억은 복잡한 문제로 보편적인 해결책이 없습니다. 그러나 다음 질문들은 다양한 기술을 탐색하는 데 도움이 되는 구조적 프레임워크를 제공합니다:

**기억의 종류는 무엇인가요?**

인간은 기억을 통해 [사실](https://en.wikipedia.org/wiki/Semantic_memory), [경험](https://en.wikipedia.org/wiki/Episodic_memory), 그리고 [규칙](https://en.wikipedia.org/wiki/Procedural_memory)을 기억합니다. AI 에이전트도 동일한 방식으로 기억을 사용할 수 있습니다. 예를 들어, AI 에이전트는 특정 사용자의 사실을 기억하여 작업을 수행할 수 있습니다. [아래 섹션](#memory-types)에서는 여러 유형의 기억을 확장합니다.

**언제 기억을 업데이트하고 싶나요?**

기억은 에이전트의 애플리케이션 논리의 일환으로 업데이트될 수 있습니다(예: "핫 경로에서"). 이 경우 에이전트는 일반적으로 사용자에게 응답하기 전에 사실을 기억하기로 결정합니다. 또는 기억은 백그라운드 작업으로 업데이트될 수 있습니다(백그라운드에서 실행되는 논리 / 비동기적으로 메모리를 생성합니다). 이러한 접근 방식 간의 절충점을 [아래 섹션](#writing-memories)에서 설명합니다.

## 기억의 종류

각기 다른 애플리케이션은 다양한 종류의 기억을 요구합니다. 비유가 완벽하지는 않지만, [인간 기억의 종류](https://www.psychologytoday.com/us/basics/memory/types-of-memory?ref=blog.langchain.dev)를 살펴보는 것은 통찰력을 제공할 수 있습니다. 일부 연구(예: [CoALA 논문](https://arxiv.org/pdf/2309.02427))에서는 이러한 인간 기억 유형을 AI 에이전트가 사용하는 기억 유형에 매핑하기도 했습니다.

| 기억의 종류 | 저장되는 내용 | 인간의 예 | 에이전트의 예 |
|-------------|----------------|---------------|---------------|
| 의미적 | 사실 | 학교에서 배운 것들 | 사용자에 대한 사실 |
| 서사적 | 경험 | 내가 한 일들 | 과거의 에이전트 행동 |
| 절차적 | 지침 | 본능 또는 운동 기술 | 에이전트 시스템 프롬프트 |

### 의미적 기억

[의미적 기억](https://en.wikipedia.org/wiki/Semantic_memory)은 인간과 AI 에이전트 모두에서 특정 사실과 개념의 유지를 포함합니다. 인간의 경우, 학교에서 배운 정보와 개념 및 그 관계에 대한 이해가 포함될 수 있습니다. AI 에이전트의 경우, 의미적 기억은 과거 상호작용으로부터 사실이나 개념을 기억함으로써 애플리케이션을 개인화하는 데 자주 사용됩니다. 

> 주의: "의미 검색"과 혼동하지 마세요. 의미 검색은 "의미"(일반적으로 임베딩)를 사용하여 유사한 콘텐츠를 찾는 기술입니다. 의미적 기억은 사실과 지식을 저장하는 심리학의 용어인 반면, 의미 검색은 정확한 일치를 넘어서 의미에 따라 정보를 검색하는 방법입니다.


#### 프로필

의미적 기억은 다양한 방식으로 관리될 수 있습니다. 예를 들어, 기억은 사용자의 특정 정보에 대한 단일의 지속적으로 업데이트되는 "프로필"이 될 수 있습니다. 프로필은 일반적으로 도메인을 나타내기 위해 선택한 다양한 키-값 쌍으로 구성된 JSON 문서일 뿐입니다. 

프로필을 기억할 때마다 프로필을 **업데이트**하고 있는지 확인해야 합니다. 따라서 이전 프로필을 전달하고 [모델에 새 프로필을 생성하도록 요청](https://github.com/langchain-ai/memory-template)해야 합니다(또는 이전 프로필에 적용할 [JSON 패치](https://github.com/hinthornw/trustcall)). 프로필이 커질수록 오류가 발생하기 쉬워지고, 여러 문서로 프로필을 분할하거나 문서를 생성할 때 **엄격한** 디코딩을 고려하여 메모리 구조가 유효성을 유지하도록 하는 것이 유리할 수 있습니다.

![](img/memory/update-profile.png)

#### 수집

또한, 기억은 시간이 지남에 따라 지속적으로 업데이트되고 확장되는 문서 모음이 될 수 있습니다. 각 개별 기억은 더 좁은 범위로 정의되며 생성이 더 쉽기 때문에 시간이 지남에 따라 정보가 **소실**될 가능성이 적습니다. LLM은 새로운 정보를 위해 _새로운_ 객체를 생성하는 것이 기존 프로필의 새로운 정보와 조화시키는 것보다 훨씬 쉽습니다. 결과적으로, 문서 수집은 [더 높은 재현율을 초래](https://en.wikipedia.org/wiki/Precision_and_recall)하는 경향이 있습니다.

하지만 이는 기억 업데이트의 일부 복잡성을 전가합니다. 이제 모델은 목록에서 기존 항목을 _삭제_하거나 _업데이트_해야 하며, 이는 까다로울 수 있습니다. 게다가 일부 모델은 과도하게 삽입되는 기본값을 가지고 있고, 다른 모델은 과도하게 업데이트하는 기본값을 가질 수 있습니다. 이를 관리하는 하나의 방법은 [Trustcall](https://github.com/hinthornw/trustcall) 패키지를 참조하고, 행동 조정을 도와줄 수 있는 평가(e.g., [LangSmith](https://docs.smith.langchain.com/tutorials/Developers/evaluation)와 같은 도구)를 고려하세요.

문서 수집 작업은 목록에 대한 기억 **검색**의 복잡 제출도 전가합니다. `Store`는 현재 [의미 검색](https://langchain-ai.github.io/langgraph/reference/store/#langgraph.store.base.SearchOp.query)과 [내용 필터링](https://langchain-ai.github.io/langgraph/reference/store/#langgraph.store.base.SearchOp.filter)을 모두 지원합니다.

마지막으로, 기억의 모음을 사용하는 것은 모델에 포괄적인 맥락을 제공하기 어렵게 만들 수 있습니다. 개별 기억은 특정 구조를 따르지만, 이 구조는 모든 기억 간의 완전한 맥락이나 관계를 포착하지 못할 수도 있습니다. 결과적으로, 이러한 기억을 사용하여 응답을 생성할 때 모델은 통합 프로필 접근 방식을 통해 더 쉽게 이용할 수 있는 중요한 맥락 정보를 결여할 수 있습니다.

![](img/memory/update-list.png)

기억 관리 접근 방식에 관계없이, 중심 포인트는 에이전트가 의미적 기억을 사용하여 [응답을 기반화](https://python.langchain.com/docs/concepts/rag/)하고, 이는 종종 더 개인화되고 관련성이 높은 상호작용으로 이어진다는 것입니다.

### 서사적 기억

[서사적 기억](https://en.wikipedia.org/wiki/Episodic_memory)은 인간과 AI 에이전트 모두에서 과거 사건이나 행동을 회상하는 것을 포함합니다. [CoALA 논문](https://arxiv.org/pdf/2309.02427)은 이를 잘 설명합니다: 사실은 의미적 기억에 기록될 수 있는 반면, *경험*은 서사적 기억에 기록됩니다. AI 에이전트의 경우, 서사적 기억은 에이전트가 작업을 수행하는 방법을 기억하도록 돕는 데 자주 사용됩니다. 

실제로, 서사적 기억은 종종 [few-shot 예시 프롬프트](https://python.langchain.com/docs/concepts/few_shot_prompting/)를 통해 구현되며, 에이전트는 과거의 연속으로부터 학습하여 작업을 올바르게 수행합니다. 때때로 "말하는" 것보다 "보여주는" 것이 더 쉽고, LLM은 예제로부터 잘 학습합니다. Few-shot 학습은 입력-출력 예시로 프롬프트를 업데이트하여 원하는 동작을 설명함으로써 LLM을 ["프로그래밍"](https://x.com/karpathy/status/1627366413840322562)할 수 있게 합니다. 다양한 [모범 사례](https://python.langchain.com/docs/concepts/#1-generating-examples)를 사용하여 few-shot 예시를 생성할 수 있지만, 종종 도전 과제는 사용자 입력에 기반하여 가장 관련성이 높은 예제를 선택하는 데 있습니다.

기억 [저장소](persistence.md#memory-store)는 few-shot 예제를 저장하는 한 방법일 뿐입니다. 더 많은 개발자의 참여를 원하거나 few-shot을 평가에 더 밀접하게 연결하고 싶다면, [LangSmith 데이터셋](https://docs.smith.langchain.com/evaluation/how_to_guides/datasets/index_datasets_for_dynamic_few_shot_example_selection)을 사용하여 데이터를 저장할 수 있습니다. 그런 다음 동적 few-shot 예시 선택기가 기본적으로 사용할 수 있어 동일한 목표를 달성할 수 있습니다. LangSmith는 데이터셋을 색인화하고, 키워드 유사성을 기반으로 사용자 입력에 가장 관련성이 높은 few-shot 예시를 검색할 수 있게 합니다 ([BM25 유사성 알고리즘](https://docs.smith.langchain.com/how_to_guides/datasets/index_datasets_for_dynamic_few_shot_example_selection)을 사용하여 키워드 기반 유사성).

동적 few-shot 예시 선택 사용에 대한 [영상](https://www.youtube.com/watch?v=37VaU7e7t5o)을 참조하세요. 또한, 도구 호출 성능을 개선하기 위해 few-shot 프롬프트를 사용하는 [블로그 게시물](https://blog.langchain.dev/few-shot-prompting-to-improve-tool-calling-performance/)과 LLM을 인간의 선호도에 맞추기 위해 few-shot 예시를 사용하는 [블로그 게시물](https://blog.langchain.dev/aligning-llm-as-a-judge-with-human-preferences/)도 확인하세요.

### 절차적 기억

[절차적 기억](https://en.wikipedia.org/wiki/Procedural_memory)은 인간과 AI 에이전트 모두에서 작업 수행에 사용되는 규칙을 기억하는 것과 관련이 있습니다. 인간의 경우, 절차적 기억은 자전거를 타는 것과 같은 작업 수행 방법에 대한 내재화된 지식으로, 기본적인 운동 기술과 균형이 포함됩니다. 반면에 서사적 기억은 훈련 바퀴 없이 자전거를 처음으로 성공적으로 타거나 경치 좋은 길을 따라 자전거를 타고 간 기억과 같은 특정 경험을 회상하는 것입니다. AI 에이전트의 경우, 절차적 기억은 에이전트의 기능을 결정하는 모델 가중치, 에이전트 코드 및 에이전트의 프롬프트의 조합입니다.

실제로 에이전트가 모델 가중치를 수정하거나 코드를 재작성하는 것은 드뭅니다. 그러나 에이전트가 [자신의 프롬프트를 수정하는](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-generator) 것은 더 일반적입니다. 

에이전트의 지침을 개선하는 효과적인 접근 방식 중 하나는 ["Reflection"](https://blog.langchain.dev/reflection-agents/) 또는 메타 프롬프트 방법입니다. 이는 에이전트를 현재 지침(예: 시스템 프롬프트)과 최근 대화 또는 명시적인 사용자 피드백과 함께 프롬프트하는 것을 포함합니다. 그러면 에이전트는 이 입력을 기반으로 자신의 지침을 정제합니다. 이 방법은 지침을 사전에 구체화하기 어려운 작업에 특히 유용합니다. 왜냐하면 에이전트가 상호작용에서 배우고 적응할 수 있기 때문입니다.

예를 들어, 우리는 외부 피드백과 프롬프트 재작성 방법을 사용하여 Twitter에 고품질의 논문 요약을 생성하는 [트윗 생성기](https://www.youtube.com/watch?v=Vn8A3BxfplE)를 만들었습니다. 이 경우, 특정 요약 프롬프트는 *사전에* 구체화하기 어려웠지만, 사용자가 생성된 트윗에 대한 비판을 하고 요약 프로세스를 개선하기 위한 피드백을 제공하는 것은 비교적 쉬웠습니다.

아래의 의사 코드는 LangGraph 메모리 [저장소](persistence.md#memory-store)를 사용하여 이를 구현하는 방법을 보여줍니다. 저장소를 사용하여 프롬프트를 저장하고, `update_instructions` 노드를 통해 현재 프롬프트(및 `state["messages"]`에서 캡처된 사용자와의 대화 피드백)를 가져온 뒤, 프롬프트를 업데이트하고 새로운 프롬프트를 저장소에 다시 저장합니다. 그런 다음 `call_model`은 저장소에서 업데이트된 프롬프트를 가져와 응답을 생성하는 데 사용됩니다.

```python
# 지침을 *사용하는* 노드
def call_model(state: State, store: BaseStore):
    namespace = ("agent_instructions", )
    instructions = store.get(namespace, key="agent_a")[0]
    # 애플리케이션 로직
    prompt = prompt_template.format(instructions=instructions.value["instructions"])
    ...

# 지침을 업데이트하는 노드
def update_instructions(state: State, store: BaseStore):
    namespace = ("instructions",)
    current_instructions = store.search(namespace)[0]
    # 메모리 로직
    prompt = prompt_template.format(instructions=instructions.value["instructions"], conversation=state["messages"])
    output = llm.invoke(prompt)
    new_instructions = output['new_instructions']
    store.put(("agent_instructions",), "agent_a", {"instructions": new_instructions})
    ...
```

![](img/memory/update-instructions.png)

## 기억 쓰기

[인간은 종종 수면 중에 장기 기억을 형성합니다](https://medicine.yale.edu/news-article/sleeps-crucial-role-in-preserving-memory/), 하지만 AI 에이전트는 다른 접근 방식이 필요합니다. 에이전트가 새로운 기억을 언제, 어떻게 생성해야 할까요? 에이전트가 기억을 쓰기 위한 두 가지 주요 방법이 있습니다: "핫 패스에서"와 "백그라운드에서".

![](img/memory/hot_path_vs_background.png)

### 핫 패스에서의 기억 쓰기

런타임 중에 기억을 생성하는 것은 장점과 도전 과제를 모두 제공합니다. 긍정적인 측면에서 이 접근 방식은 실시간 업데이트를 가능하게 하여 새로운 기억을 즉시 후속 상호작용에서 사용할 수 있게 합니다. 또한 사용자가 기억이 생성되고 저장될 때 알림을 받을 수 있기 때문에 투명성을 제공합니다.

그러나 이 방법은 또 다른 도전 과제를 제시합니다. 에이전트가 어떤 것을 기억에 남길지 결정하기 위해 새로운 도구가 필요할 경우 복잡성이 증가할 수 있습니다. 또한, 무엇을 기억으로 저장할지에 대한 추론 과정이 에이전트의 지연에 영향을 미칠 수 있습니다. 마지막으로 에이전트는 기억 생성과 다른 책임 사이에서 멀티태스킹을 해야 하므로 생성되는 기억의 양과 질에 영향을 줄 수 있습니다.

예를 들어, ChatGPT는 [save_memories](https://openai.com/index/memory-and-new-controls-for-chatgpt/) 도구를 사용하여 콘텐츠 문자열로 기억을 추가하거나 수정하며, 각 사용자 메시지와 함께 이 도구를 사용할지 여부와 방법을 결정합니다. 우리의 [memory-agent](https://github.com/langchain-ai/memory-agent) 템플릿을 참조 구현으로 확인해 보세요.

### 백그라운드에서의 기억 쓰기

별도의 백그라운드 작업으로 기억을 생성하는 것은 몇 가지 장점을 제공합니다. 이는 주요 애플리케이션의 지연을 없애고, 애플리케이션 로직과 기억 관리를 분리하며, 에이전트가 보다 집중된 작업을 수행할 수 있게 합니다. 이 접근 방식은 또한 중복 작업을 피하기 위해 기억 생성을 위한 타이밍을 조절할 수 있는 유연성을 제공합니다.

그러나 이 방법에는 자체적인 도전 과제가 있습니다. 기억 작성을 얼마나 자주 할지 결정하는 것이 중요해지며, 빈번하지 않은 업데이트는 다른 스레드에 새로운 컨텍스트가 없게 만들 수 있습니다. 기억 형성을 언제 트리거할지를 결정하는 것도 중요합니다. 일반적인 전략으로는 정해진 시간 후에 일정한 스케줄을 통한 예약(새로운 이벤트가 발생할 경우 재예약), 크론 스케줄 사용, 사용자나 애플리케이션 로직에 의해 수동으로 트리거를 허용하는 방법 등이 있습니다.

우리의 [memory-service](https://github.com/langchain-ai/memory-template) 템플릿을 참조 구현으로 확인해 보세요.
