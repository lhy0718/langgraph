_한국어로 기계번역됨_

# 스트리밍

LangGraph는 스트리밍에 대한 1급 지원을 제공하며, 그래프 실행 결과를 스트리밍하는 여러 가지 방법이 있습니다.

## 스트리밍 그래프 출력(`.stream` 및 `.astream`)

`.stream` 및 `.astream`은 그래프 실행 결과를 동기 또는 비동기적으로 스트리밍하는 방법입니다. 이러한 메서드를 호출할 때 여러 가지 모드를 지정할 수 있습니다(예: `graph.stream(..., mode="...")`):

- [`"values"`](../how-tos/streaming.ipynb#values): 그래프의 각 단계 후 상태의 전체 값을 스트리밍합니다.
- [`"updates"`](../how-tos/streaming.ipynb#updates): 그래프의 각 단계 후 상태의 업데이트를 스트리밍합니다. 같은 단계에서 여러 업데이트가 이루어지면(e.g. 여러 노드를 실행하는 경우) 그 업데이트는 개별적으로 스트리밍됩니다.
- [`"custom"`](../how-tos/streaming.ipynb#custom): 그래프 노드 내부에서 사용자 정의 데이터를 스트리밍합니다.
- [`"messages"`](../how-tos/streaming-tokens.ipynb): LLM이 호출되는 그래프 노드의 LLM 토큰 및 메타데이터를 스트리밍합니다.
- [`"debug"`](../how-tos/streaming.ipynb#debug): 그래프 실행 중 가능한 많은 정보를 스트리밍합니다.

여러 스트리밍 모드를 동시에 지정할 수도 있으며, 이 경우 스트리밍 출력은 튜플 `(stream_mode, data)` 형식으로 제공됩니다. 예를 들어:

```python
graph.stream(..., stream_mode=["updates", "messages"])
```

```
...
('messages', (AIMessageChunk(content='Hi'), {'langgraph_step': 3, 'langgraph_node': 'agent', ...}))
...
('updates', {'agent': {'messages': [AIMessage(content="Hi, how can I help you?")]}})
```

아래 시각화는 `values` 모드와 `updates` 모드의 차이를 보여줍니다:

![values vs updates](../static/values_vs_updates.png)


## LLM 토큰 및 이벤트 스트리밍(`.astream_events`)

추가적으로, `astream_events` 메서드를 사용하여 노드 내부에서 발생하는 이벤트를 스트리밍할 수 있습니다. 이는 [LLM 호출의 토큰 스트리밍](../how-tos/streaming-tokens.ipynb)에 유용합니다.

이것은 모든 [LangChain 객체](https://python.langchain.com/docs/concepts/#runnable-interface)에서 표준 메서드입니다. 즉, 그래프가 실행되는 동안 특정 이벤트가 발생하여 `.astream_events`를 사용하여 그래프를 실행하면 이 이벤트를 볼 수 있습니다.

모든 이벤트는 (다른 요소들과 함께) `event`, `name`, `data` 필드를 포함하고 있습니다. 이들은 다음을 의미합니다:

- `event`: 발생하고 있는 이벤트의 유형입니다. 모든 콜백 이벤트 및 트리거에 대한 자세한 표는 [여기](https://python.langchain.com/docs/concepts/#callback-events)에서 확인할 수 있습니다.
- `name`: 이벤트의 이름입니다.
- `data`: 이벤트와 관련된 데이터입니다.

이벤트가 발생하게 하는 것들은 무엇일까요?

* 각 노드(실행 가능)는 실행을 시작할 때 `on_chain_start`를 방출하고, 노드 실행 중에 `on_chain_stream`을 방출하며, 노드가 종료되면 `on_chain_end`를 방출합니다. 노드 이벤트는 이벤트의 `name` 필드에 노드 이름을 가집니다.
* 그래프는 그래프 실행 시작 시 `on_chain_start`를 방출하고, 각 노드 실행 후 `on_chain_stream`을 방출하며, 그래프가 종료되면 `on_chain_end`를 방출합니다. 그래프 이벤트는 이벤트의 `name` 필드에 `LangGraph`를 가집니다.
* 상태 채널에 대한 모든 쓰기 작업(즉, 상태 키의 값을 업데이트할 때마다)은 `on_chain_start` 및 `on_chain_end` 이벤트를 방출합니다.

또한, 노드 내부에서 생성된 모든 이벤트(LLM 이벤트, 도구 이벤트, 수동 방출된 이벤트 등)도 `.astream_events`의 출력에서 확인할 수 있습니다.

이것을 더 구체적으로 이해하고 간단한 그래프를 실행할 때 어떤 이벤트가 반환되는지 살펴보겠습니다:

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, MessagesState, START, END

model = ChatOpenAI(model="gpt-4o-mini")


def call_model(state: MessagesState):
    response = model.invoke(state['messages'])
    return {"messages": response}

workflow = StateGraph(MessagesState)
workflow.add_node(call_model)
workflow.add_edge(START, "call_model")
workflow.add_edge("call_model", END)
app = workflow.compile()

inputs = [{"role": "user", "content": "hi!"}]
async for event in app.astream_events({"messages": inputs}, version="v1"):
    kind = event["event"]
    print(f"{kind}: {event['name']}")
```
```shell
on_chain_start: LangGraph
on_chain_start: __start__
on_chain_end: __start__
on_chain_start: call_model
on_chat_model_start: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_stream: ChatOpenAI
on_chat_model_end: ChatOpenAI
on_chain_start: ChannelWrite<call_model,messages>
on_chain_end: ChannelWrite<call_model,messages>
on_chain_stream: call_model
on_chain_end: call_model
on_chain_stream: LangGraph
on_chain_end: LangGraph
```

우리는 전체 그래프 시작(`on_chain_start: LangGraph`)으로 시작합니다. 그런 다음 `__start__` 노드에 작성합니다 (이 노드는 입력을 처리하기 위한 특별한 노드입니다).
그다음 `call_model` 노드를 시작합니다 (`on_chain_start: call_model`). 그 후 대화 모델 호출(`on_chat_model_start: ChatOpenAI`)을 시작하고,
토큰을 순차적으로 스트리밍합니다 (`on_chat_model_stream: ChatOpenAI`) 그리고 대화 모델을 마칩니다 (`on_chat_model_end: ChatOpenAI`). 그 후,
결과를 채널에 다시 작성합니다 (`ChannelWrite<call_model,messages>`) 그리고 `call_model` 노드를 마치고 전체 그래프를 종료합니다.

이것은 간단한 그래프에서 어떤 이벤트가 발생하는지에 대한 좋은 감각을 제공할 것입니다. 그러나 이러한 이벤트가 포함하는 데이터는 무엇일까요?
각 유형의 이벤트는 다른 형식으로 데이터를 포함합니다. `on_chat_model_stream` 이벤트가 어떻게 생겼는지 살펴보겠습니다. 이는 LLM 응답에서 토큰을 스트리밍하기 위해 필요한 중요한 유형의 이벤트입니다.

이 이벤트 형태는 다음과 같습니다:

```shell
{'event': 'on_chat_model_stream',
 'name': 'ChatOpenAI',
 'run_id': '3fdbf494-acce-402e-9b50-4eab46403859',
 'tags': ['seq:step:1'],
 'metadata': {'langgraph_step': 1,
  'langgraph_node': 'call_model',
  'langgraph_triggers': ['start:call_model'],
  'langgraph_task_idx': 0,
  'checkpoint_id': '1ef657a0-0f9d-61b8-bffe-0c39e4f9ad6c',
  'checkpoint_ns': 'call_model',
  'ls_provider': 'openai',
  'ls_model_name': 'gpt-4o-mini',
  'ls_model_type': 'chat',
  'ls_temperature': 0.7},
 'data': {'chunk': AIMessageChunk(content='Hello', id='run-3fdbf494-acce-402e-9b50-4eab46403859')},
 'parent_ids': []}
```
우리는 이벤트 유형과 이름(우리가 이전에 알았던 것)을 확인할 수 있습니다.

또한 메타데이터에 많은 내용이 포함되어 있습니다. 특히, `'langgraph_node': 'call_model',`는 이 모델이 호출된 노드를 알려주는 매우 유용한 정보입니다.

마지막으로 `data`는 매우 중요한 필드입니다. 이 필드는 이 이벤트에 대한 실제 데이터를 포함합니다! 이 경우에는 AIMessageChunk입니다. 이는 메시지의 `content`와 `id`를 포함합니다.
이것은 전체 AIMessage의 ID(단지 이 청크가 아님)이며, 매우 유용합니다 - 같은 메시지의 어느 청크가 함께 있는지를 추적하는 데 도움이 됩니다(그래서 UI에서 함께 보여줄 수 있습니다).

이 정보는 스트리밍 LLM 토큰에 대한 UI를 생성하는 데 필요한 모든 것을 포함합니다. 이에 대한 가이드를 [여기](../how-tos/streaming-tokens.ipynb)에서 확인할 수 있습니다.


!!! 경고 "PYTHON<=3.10의 비동기"
    Python <= 3.10에서 `.astream_events`를 사용할 때 노드 내부에서 이벤트가 발생하는 것을 보지 못할 수 있습니다. 노드 안에서 Langchain RunnableLambda, RunnableGenerator 또는 Tool을 비동기로 사용하는 경우, 이러한 객체에 콜백을 수동으로 전파해야 합니다. 이는 이 경우 LangChain이 자식 객체에 콜백을 자동으로 전파할 수 없기 때문입니다.


## LangGraph 플랫폼

스트리밍은 LLM 애플리케이션이 최종 사용자에게 반응적으로 느껴지도록 만드는 데 중요합니다. 스트리밍 실행을 생성할 때 스트리밍 모드는 API 클라이언트에 어떤 데이터가 스트리밍되는지를 결정합니다. LangGraph 플랫폼은 다섯 가지 스트리밍 모드를 지원합니다:

- `values`: 각 [슈퍼 스텝](https://langchain-ai.github.io/langgraph/concepts/low_level/#graphs) 실행 후 그래프의 전체 상태를 스트리밍합니다. 값 스트리밍에 대한 [가이드](../cloud/how-tos/stream_values.md)를 참조하십시오.
- `messages-tuple`: 노드 내에서 생성된 모든 메시지에 대해 LLM 토큰을 스트리밍합니다. 이 모드는 주로 채팅 애플리케이션에 사용됩니다. 메시지 스트리밍에 대한 [가이드](../cloud/how-tos/stream_messages.md)를 참조하십시오.
- `updates`: 각 노드 실행 후 그래프의 상태에 대한 업데이트를 스트리밍합니다. 업데이트 스트리밍에 대한 [가이드](../cloud/how-tos/stream_updates.md)를 참조하십시오.
- `events`: 그래프 실행 중 발생하는 모든 이벤트(그래프 상태 포함)를 스트리밍합니다. 이벤트 스트리밍에 대한 [가이드](../cloud/how-tos/stream_events.md)를 참조하십시오. 이는 LLM에 대한 토큰별 스트리밍으로 사용할 수 있습니다.
- `debug`: 그래프 실행 중 디버그 이벤트를 스트리밍합니다. 디버그 이벤트 스트리밍에 대한 [가이드](../cloud/how-tos/stream_debug.md)를 참조하십시오.

동시에 여러 스트리밍 모드를 지정할 수도 있습니다. 여러 스트리밍 모드를 동시에 구성하기 위한 [가이드](../cloud/how-tos/stream_multiple.md)를 참조하십시오.

스트리밍 실행을 생성하는 방법에 대한 [API 참조](../cloud/reference/api/api_ref.html#tag/threads-runs/POST/threads/{thread_id}/runs/stream)를 참조하세요.

스트리밍 모드 `values`, `updates`, `messages-tuple` 및 `debug`는 LangGraph 라이브러리에서 사용할 수 있는 모드와 매우 유사합니다 - 이에 대한 더 깊은 개념 설명은 [이전 섹션](#streaming-graph-outputs-stream-and-astream)을 참조할 수 있습니다.

스트리밍 모드 `events`는 LangGraph 라이브러리에서 `.astream_events`를 사용하는 것과 동일합니다 - 이에 대한 더 깊은 개념 설명은 [이전 섹션](#streaming-graph-outputs-stream-and-astream)을 참조할 수 있습니다.

모든 발생한 이벤트는 두 개의 속성을 가지고 있습니다:

- `event`: 이는 이벤트의 이름입니다.
- `data`: 이는 이벤트와 관련된 데이터입니다.