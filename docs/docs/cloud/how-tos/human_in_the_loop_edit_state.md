_한국어로 기계번역됨_

# 배포된 그래프의 상태 편집 방법

LangGraph 에이전트를 생성할 때, 인간-루프 구성 요소를 추가하는 것이 종종 유용합니다. 이는 도구에 대한 접근 권한을 제공할 때 도움이 될 수 있습니다. 이러한 상황에서는 계속 진행하기 전에 그래프 상태를 편집하고 싶을 수 있습니다(예: 호출되는 도구나 호출하는 방법을 수정하는 등의 이유로).

이는 여러 가지 방법으로 가능하지만, 기본적으로 지원되는 방법은 노드가 실행되기 전에 "인터럽트"를 추가하는 것입니다. 이 방법은 해당 노드에서 실행을 중단시킵니다. 그런 다음 update_state를 사용하여 상태를 업데이트하고 그 지점에서 다시 이어서 계속할 수 있습니다.

## 설정

호스팅하고 있는 그래프의 전체 코드를 보여주지는 않겠지만, 원하신다면 [여기](../../how-tos/human_in_the_loop/edit-graph-state.ipynb#agent)를 확인하실 수 있습니다. 그래프가 호스팅되면, 이를 호출하고 사용자 입력을 기다릴 준비가 됩니다.

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

=== "Javascript"

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

## 상태 편집

### 초기 호출

이제 `action` 노드 전에 인터럽트를 설정하여 그래프를 호출해보겠습니다.

=== "Python"

```python
input = { 'messages':[{ "role":"user", "content":"search for weather in SF" }] }

async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id,
    input=input,
    stream_mode="updates",
    interrupt_before=["action"],
):
    if chunk.data and chunk.event != "metadata": 
        print(chunk.data)
```

=== "Javascript"

```js
const input = { messages: [{ role: "human", content: "search for weather in SF" }] };

const streamResponse = client.runs.stream(
  thread["thread_id"],
  assistantId,
  {
    input: input,
    streamMode: "updates",
    interruptBefore: ["action"],
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
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코 날씨 검색\"}]},
       \"interrupt_before\": [\"action\"],
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

    {'agent': {'messages': [{'content': [{'text': "확실히! 검색 기능을 사용하여 샌프란시스코의 현재 날씨를 검색해 드리겠습니다. 이렇게 하겠습니다:", 'type': 'text'}, {'id': 'toolu_01KEJMBFozSiZoS4mAcPZeqQ', 'input': {'query': '샌프란시스코 현재 날씨'}, 'name': 'search', 'type': 'tool_use'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-6dbb0167-f8f6-4e2a-ab68-229b2d1fbb64', 'example': False, 'tool_calls': [{'name': 'search', 'args': {'query': '샌프란시스코 현재 날씨'}, 'id': 'toolu_01KEJMBFozSiZoS4mAcPZeqQ'}], 'invalid_tool_calls': [], 'usage_metadata': None}]}}


### 상태 수정

이제 우리가 실제로는 시디 프레즈(Sidi Frej)에서 날씨를 검색하려고 했다고 가정해 보겠습니다. 상태를 수정하여 이를 정확히 반영할 수 있습니다:


=== "파이썬"

    ```python
    # 첫째, 현재 상태를 가져옵니다
    current_state = await client.threads.get_state(thread['thread_id'])

    # 이제 상태에서 마지막 메시지를 가져옵니다
    # 이것이 우리가 업데이트하고자 하는 도구 호출과 함께하는 메시지입니다
    last_message = current_state['values']['messages'][-1]

    # 이제 그 도구 호출을 위한 인수를 업데이트합니다
    last_message['tool_calls'][0]['args'] = {'query': '시디 프레즈 현재 날씨'}

    # 이제 `update_state`를 호출하여 이 메시지를 `messages` 키에 전달합니다
    # 이는 상태의 다른 업데이트처럼 처리됩니다
    # 그것은 `messages` 키의 리듀서 함수에 전달됩니다
    # 그 리듀서 함수는 메시지의 ID를 사용하여 업데이트합니다
    # 올바른 ID가 있는 것이 중요합니다! 그렇지 않으면 새 메시지로 추가됩니다
    await client.threads.update_state(thread['thread_id'], {"messages": last_message})
    ```

=== "자바스크립트"

    ```js
    // 먼저, 현재 상태를 가져옵니다
    const currentState = await client.threads.getState(thread["thread_id"]);

    // 이제 상태에서 마지막 메시지를 가져옵니다
    // 이것이 우리가 업데이트하고자 하는 도구 호출과 함께하는 메시지입니다
    let lastMessage = currentState.values.messages.slice(-1)[0];

    // 이제 그 도구 호출을 위한 인수를 업데이트합니다
    lastMessage.tool_calls[0].args = { query: "시디 프레즈 현재 날씨" };

    // 이제 `update_state`를 호출하여 이 메시지를 `messages` 키에 전달합니다
    // 이는 상태의 다른 업데이트처럼 처리됩니다
    // 그것은 `messages` 키의 리듀서 함수에 전달됩니다
    // 그 리듀서 함수는 메시지의 ID를 사용하여 업데이트합니다
    // 올바른 ID가 있는 것이 중요합니다! 그렇지 않으면 새 메시지로 추가됩니다
    await client.threads.updateState(thread["thread_id"], { values: { messages: lastMessage } });
    ```

=== "CURL"

    ```bash
    curl --request GET --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state | \                                                                                      
    jq '.values.messages[-1] | (.tool_calls[0].args = {"query": "시디 프레즈 현재 날씨"})' | \
    curl --request POST \
      --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state \
      --header 'Content-Type: application/json' \
      --data @-
    ```

    {'configurable': {'thread_id': '9c8f1a43-9dd8-4017-9271-2c53e57cf66a',
      'checkpoint_ns': '',
      'checkpoint_id': '1ef58e7e-3641-649f-8002-8b4305a64858'}}



### 그래프 실행 재개

이제 업데이트된 상태로 그래프 실행을 재개할 수 있습니다:


=== "파이썬"

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
=== "자바스크립트"

    ```js
    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: null,
        streamMode: "updates",
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

    {'action': {'messages': [{'content': '["현재 시디 프레의 날씨를 조사했습니다. 결과: 샌프란시스코는 맑지만, 쌍둥이자리라면 조심하세요 😈."]', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'search', 'id': '1161b8d1-bee4-4188-9be8-698aecb69f10', 'tool_call_id': 'toolu_01KEJMBFozSiZoS4mAcPZeqQ'}]}}
    {'agent': {'messages': [{'content': [{'text': '검색 쿼리에 대한 혼란에 대해 사과드립니다. 검색 기능이 "SF"를 우리가 의도했던 "샌프란시스코"가 아니라 "시디 프레"로 해석한 것 같습니다. 올바른 정보를 얻기 위해 전체 도시 이름으로 다시 검색해 보겠습니다:', 'type': 'text'}, {'id': 'toolu_0111rrwgfAcmurHZn55qjqTR', 'input': {'query': '샌프란시스코의 현재 날씨'}, 'name': 'search', 'type': 'tool_use'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-b8c25779-cfb4-46fc-a421-48553551242f', 'example': False, 'tool_calls': [{'name': 'search', 'args': {'query': '샌프란시스코의 현재 날씨'}, 'id': 'toolu_0111rrwgfAcmurHZn55qjqTR'}], 'invalid_tool_calls': [], 'usage_metadata': None}]}}
    {'action': {'messages': [{'content': '["샌프란시스코의 현재 날씨를 조사했습니다. 결과: 샌프란시스코는 맑지만, 쌍둥이자리라면 조심하세요 😈."]', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'search', 'id': '6bc632ae-5ee6-4d01-9532-79c524a2d443', 'tool_call_id': 'toolu_0111rrwgfAcmurHZn55qjqTR'}]}}
    {'agent': {'messages': [{'content': "이제 검색 결과를 바탕으로 샌프란시스코의 현재 날씨에 대한 정보를 제공할 수 있습니다:\n\n샌프란시스코의 날씨는 현재 맑습니다.\n\n이 검색 결과에는 다소 특이하게 쌍둥이자리와 관련된 언급이 포함되어 있는데, 이는 날씨와 직접적인 관련이 없는 것 같습니다. 이는 검색 엔진이 어떤 점성술 정보나 농담을 결과에 포함했기 때문일 수 있습니다. 하지만 날씨 정보의 목적을 위해서는 지금 샌프란시스코가 맑다는 사실에 집중할 수 있습니다.\n\n샌프란시스코 또는 다른 위치의 날씨에 대해 더 알고 싶은 것이 있으신가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-227a042b-dd97-476e-af32-76a3703af5d8', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}


보시다시피 이제 시디 프레의 현재 날씨를 조사하고 있습니다(비록 우리의 더미 검색 노드가 여전히 SF에 대한 결과를 반환하지만, 이 예제에서는 실제로 검색을 수행하지 않고 "샌프란시스코는 맑다 ..."는 결과를 매번 반환합니다).
