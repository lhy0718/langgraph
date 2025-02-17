_한국어로 기계번역됨_

# 사용자 입력 대기 방법

인간 상호작용 패턴 중 하나는 인간 입력을 기다리는 것입니다. 주요 사용 사례 중 하나는 사용자에게 명확한 질문을 하는 것입니다. 이를 수행하는 한 가지 방법은 `END` 노드로 이동하여 그래프를 종료하는 것입니다. 그러면 사용자 응답이 그래프의 새 호출로 돌아옵니다. 이는 기본적으로 챗봇 아키텍처를 만드는 것입니다.

문제는 그래프의 특정 지점으로 다시 돌아가기가 어렵다는 것입니다. 종종 에이전트는 어떤 프로세스의 중간에 있으며, 약간의 사용자 입력만 필요합니다. 사용자 메시지를 올바른 위치로 라우팅할 수 있는 `conditional_entry_point`를 가지고 그래프를 설계하는 것이 가능하지만, 이는 초점이 거의 어디에나 도달할 수 있는 라우팅 함수를 포함하므로 크게 확장 가능하지 않습니다.

별도의 방법은 사용자 입력을 받기 위해 명시적인 노드를 두는 것입니다. 이는 노트북 설정에서 구현하기 쉽습니다. 노드에 `input()` 호출을 넣으면 됩니다. 그러나 이는 생산-ready 상태가 아닙니다.

다행히도, LangGraph는 생산적인 방식으로 유사한 작업을 수행하는 것을 가능하게 합니다. 기본 아이디어는 다음과 같습니다:

- 인간 입력을 나타내는 노드를 설정합니다. 이 노드는 특정한 들어오는/나가는 엣지를 가질 수 있습니다(원하는 대로). 이 노드 내부에 실제로 로직은 없어야 합니다.
- 이 노드 전에 중단점을 추가합니다. 이는 이 노드가 실행되기 전에 그래프를 중단합니다(이는 좋은데, 어차피 진짜 로직이 없으므로)
- `.update_state`를 사용하여 그래프의 상태를 업데이트합니다. 사용자 응답을 전달합니다. 여기서 핵심은 이 업데이트를 **그 노드인 것처럼** 적용하기 위해 `as_node` 매개변수를 사용하는 것입니다. 이는 다음에 실행을 재개할 때 해당 노드가 행동한 것처럼 재개되는 효과를 줍니다.

## 설정

우리는 호스팅하는 그래프의 전체 코드를 보여주지는 않지만, 원하신다면 [여기](../../how-tos/human_in_the_loop/wait-user-input.ipynb#agent)에서 확인할 수 있습니다. 이 그래프가 호스팅되고 나면, 그래프를 호출하고 사용자 입력을 기다릴 준비가 된 것입니다.

### SDK 초기화

먼저, 호스팅된 그래프와 통신할 수 있도록 클라이언트를 설정해야 합니다:

=== "Python"

```python
from langgraph_sdk import get_client
client = get_client(url=<DEPLOYMENT_URL>)
# "agent"라는 이름으로 배포된 그래프 사용
assistant_id = "agent"
thread = await client.threads.create()
```

=== "자바스크립트"

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
// "agent"라는 이름으로 배포된 그래프 사용
const assistantId = "agent";
const thread = await client.threads.create();
```

=== "CURL"

```bash
curl --request POST \
  --url <DEPLOYMENT_URL>/threads \
  --header 'Content-Type: application/json' \
  --data '{}'
```

## 사용자 입력 대기

### 초기 호출

이제 `ask_human` 노드 전에 중단하여 그래프를 호출합시다:

=== "Python"

```python
input = {
    "messages": [
        {
            "role": "user",
            "content": "사용자에게 그들이 어디에 있는지 물어보는 검색 도구를 사용한 다음, 그곳의 날씨를 확인하세요.",
        }
    ]
}

async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id,
    input=input,
    stream_mode="updates",
    interrupt_before=["ask_human"],
):
    if chunk.data and chunk.event != "metadata": 
        print(chunk.data)
```
=== "자바스크립트"

    ```js
const input = {
  messages: [
    {
      role: "human",
      content: "사용자에게 그들이 어디에 있는지 물어보는 검색 도구를 사용한 다음, 그곳의 날씨를 확인하세요."
    }
  ]
};

const streamResponse = client.runs.stream(
  thread["thread_id"],
  assistantId,
  {
    input: input,
    streamMode: "updates",
    interruptBefore: ["ask_human"]
  }
);

for await (const chunk of streamResponse) {
  if (chunk.data && chunk.event !== "metadata") {
    console.log(chunk.data);
  }
}
```

=== "CURL"

```bash
curl --request POST \
 --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
 --header 'Content-Type: application/json' \
 --data "{
   \"assistant_id\": \"agent\",
   \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"사용자에게 그들이 어디에 있는지 물어보는 검색 도구를 사용한 다음, 그곳의 날씨를 확인하세요.\"}]},
   \"interrupt_before\": [\"ask_human\"],
   \"stream_mode\": [
     \"updates\"
   ]
 }" | \
 sed 's/\r$//' | \
 awk '
 /^event:/ {
     if (data_content != "" && event_type != "metadata") {
         print data_content "\n"
     }
     sub(/^event: /, "", $0)
     event_type = $0
     data_content = ""
 }
 /^data:/ {
     sub(/^data: /, "", $0)
     data_content = $0
 }
 END {
     if (data_content != "" && event_type != "metadata") {
         print data_content "\n"
     }
 }
 '
```

출력:

{'agent': {'messages': [{'content': [{'text': "확실히! 사용자의 위치에 대해 묻기 위해 AskHuman 기능을 사용하고, 그런 다음 그 위치의 날씨를 확인하기 위해 검색 기능을 사용할 것입니다. 사용자가 어디에 있는지 물어보는 것부터 시작하겠습니다.", 'type': 'text'}, {'id': 'toolu_01RFahzYPvnPWTb2USk2RdKR', 'input': {'question': '현재 어디에 있습니까?'}, 'name': 'AskHuman', 'type': 'tool_use'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-a8422215-71d3-4093-afb4-9db141c94ddb', 'example': False, 'tool_calls': [{'name': 'AskHuman', 'args': {'question': '현재 어디에 있습니까?'}, 'id': 'toolu_01RFahzYPvnPWTb2USk2RdKR'}], 'invalid_tool_calls': [], 'usage_metadata': None}]}}

### 사용자 입력을 상태에 추가하기

이제 사용자로부터의 응답으로 이 스레드를 업데이트하고자 합니다. 이후 또 다른 실행을 시작할 수 있습니다.

우리는 이를 도구 호출로 처리하고 있기 때문에, 도구 호출의 응답인 것처럼 상태를 업데이트해야 합니다. 이를 위해, 도구 호출의 ID를 얻기 위해 상태를 확인해야 합니다.


=== "Python"

```python
state = await client.threads.get_state(thread['thread_id'])
tool_call_id = state['values']['messages'][-1]['tool_calls'][0]['id']

# 이제 id와 우리가 원하는 응답으로 도구 호출을 생성합니다.
tool_message = [{"tool_call_id": tool_call_id, "type": "tool", "content": "샌프란시스코"}]

await client.threads.update_state(thread['thread_id'], {"messages": tool_message}, as_node="ask_human")
```

=== "Javascript"

    ```js
const state = await client.threads.getState(thread["thread_id"]);
const toolCallId = state.values.messages[state.values.messages.length - 1].tool_calls[0].id;

// 이제 우리가 원하는 응답과 함께 도구 호출을 생성합니다.
const toolMessage = [
  {
    tool_call_id: toolCallId,
    type: "tool",
    content: "샌프란시스코"
  }
];

await client.threads.updateState(
  thread["thread_id"],
  { values: { messages: toolMessage } },
  { asNode: "ask_human" }
);
```

=== "CURL"

```bash
curl --request GET \
 --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state \
 | jq -r '.values.messages[-1].tool_calls[0].id' \
 | sh -c '
     TOOL_CALL_ID="$1"
     
     # JSON 페이로드 구성
     JSON_PAYLOAD=$(printf "{\"messages\": [{\"tool_call_id\": \"%s\", \"type\": \"tool\", \"content\": \"샌프란시스코\"}], \"as_node\": \"ask_human\"}" "$TOOL_CALL_ID")
     
     # 업데이트된 상태 전송
     curl --request POST \
          --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state \
          --header "Content-Type: application/json" \
          --data "${JSON_PAYLOAD}"
 ' _ 
```

출력:

{'configurable': {'thread_id': 'a9f322ae-4ed1-41ec-942b-38cb3d342c3a',
'checkpoint_ns': '',
'checkpoint_id': '1ef58e97-a623-63dd-8002-39a9a9b20be3'}}

### 인간 입력을 받은 후 호출하기

이제 에이전트에게 계속 진행하라고 지시할 수 있습니다. 추가 입력이 필요 없으므로 그래프에 입력으로 None을 그냥 전달하면 됩니다:

=== "Python"

```python
async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id,
    input=None,
    stream_mode="updates",
):
    if chunk.data and chunk.event != "metadata": 
        print(chunk.data)
```
=== "Javascript"

```js
const streamResponse = client.runs.stream(
  thread["thread_id"],
  assistantId,
  {
    input: null,
    streamMode: "updates"
  }
);

for await (const chunk of streamResponse) {
  if (chunk.data && chunk.event !== "metadata") {
    console.log(chunk.data);
  }
}
```

=== "CURL"

    ```bash
    curl --request POST \                                                                             
     --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"stream_mode\": [
         \"updates\"
       ]
     }"| \ 
     sed 's/\r$//' | \
     awk '
     /^event:/ {
         if (data_content != "" && event_type != "metadata") {
             print data_content "\n"
         }
         sub(/^event: /, "", $0)
         event_type = $0
         data_content = ""
     }
     /^data:/ {
         sub(/^data: /, "", $0)
         data_content = $0
     }
     END {
         if (data_content != "" && event_type != "metadata") {
             print data_content "\n"
         }
     }
     '
    ```

출력:

    {'agent': {'messages': [{'content': [{'text': "샌프란시스코에 계신 걸 알려주셔서 감사합니다. 이제 검색 기능을 사용하여 샌프란시스코의 날씨를 확인하겠습니다.", 'type': 'text'}, {'id': 'toolu_01K57ofmgG2wyJ8tYJjbq5k7', 'input': {'query': '샌프란시스코의 현재 날씨'}, 'name': 'search', 'type': 'tool_use'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-241baed7-db5e-44ce-ac3c-56431705c22b', 'example': False, 'tool_calls': [{'name': 'search', 'args': {'query': '샌프란시스코의 현재 날씨'}, 'id': 'toolu_01K57ofmgG2wyJ8tYJjbq5k7'}], 'invalid_tool_calls': [], 'usage_metadata': None}]}}
    {'action': {'messages': [{'content': '["제가 검색해봤습니다: 샌프란시스코의 현재 날씨. 결과: 샌프란시스코는 맑지만, 쌍둥이자리 사람이라면 조심하세요 😈."]', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'search', 'id': '8b699b95-8546-4557-8e66-14ea71a15ed8', 'tool_call_id': 'toolu_01K57ofmgG2wyJ8tYJjbq5k7'}]}}
    {'agent': {'messages': [{'content': "검색 결과에 따라, 샌프란시스코의 현재 날씨에 대한 정보를 제공해드리겠습니다:\n\n현재 샌프란시스코의 날씨는 맑습니다. 도시에서 아름다운 날입니다! \n\n그러나 검색 결과에는 쌍둥이자리 관련해 이상한 언급이 포함되어 있음을 알려드려야 합니다. 이는 농담이거나 검색 엔진에 의해 추가된 관련 없는 정보일 수 있습니다. 정확하고 자세한 날씨 정보를 원하신다면, 신뢰할 수 있는 날씨 서비스나 앱을 통해 샌프란시스코의 날씨를 확인하시기 바랍니다.\n\n날씨나 샌프란시스코에 대해 더 알고 싶은 것이 있으신가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-b4d7309f-f849-46aa-b6ef-475bcabd2be9', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}

