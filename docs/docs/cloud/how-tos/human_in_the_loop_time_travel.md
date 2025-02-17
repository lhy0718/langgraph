_한국어로 기계번역됨_

# 이전 상태에서 되감기 및 분기하는 방법

LangGraph Cloud를 사용하면 이전의 모든 상태로 돌아가서 테스트 중에 발견된 문제를 재현하기 위해 그래프를 다시 실행하거나, 이전 상태에서 원래 수행된 것과는 다른 방식으로 분기할 수 있는 기능을 제공합니다. 이 가이드에서는 과거 상태를 다시 실행하고 이전 상태에서 분기하는 방법에 대한 간단한 예를 보여드리겠습니다.

## 설정

호스팅하는 그래프의 전체 코드를 보여드리지는 않겠지만, 원하시면 [여기](../../how-tos/human_in_the_loop/time-travel.ipynb#build-the-agent)에서 확인할 수 있습니다. 이 그래프가 호스팅되면, 이를 호출하고 사용자 입력을 기다릴 준비가 됩니다.

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

## 상태 재생

### 초기 호출

상태를 되감기 전에 - 되감기를 위한 상태를 생성해야 합니다! 이를 위해 간단한 메시지로 그래프를 호출해 보겠습니다:

=== "Python"

```python
input = {"messages": [{"role": "user", "content": "SF의 날씨를 검색해 주세요"}]}

async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id,
    input=input,
    stream_mode="updates",
):
    if chunk.data and chunk.event != "metadata": 
        print(chunk.data)
```

=== "Javascript"

```js
const input = { "messages": [{ "role": "user", "content": "SF의 날씨를 검색해 주세요" }] }

const streamResponse = client.runs.stream(
  thread["thread_id"],
  assistantId,
  {
    input: input,
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
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"SF의 날씨를 검색해주세요\"}]},
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

    {'agent': {'messages': [{'content': [{'text': "확실히! 제가 검색 기능을 사용하여 현재 샌프란시스코의 날씨를 찾아보겠습니다. 지금 해볼게요.", 'type': 'text'}, {'id': 'toolu_011vroKUtWU7SBdrngpgpFMn', 'input': {'query': '샌프란시스코의 현재 날씨'}, 'name': 'search', 'type': 'tool_use'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-ee639877-d97d-40f8-96dc-d0d1ae22d203', 'example': False, 'tool_calls': [{'name': 'search', 'args': {'query': '샌프란시스코의 현재 날씨'}, 'id': 'toolu_011vroKUtWU7SBdrngpgpFMn'}], 'invalid_tool_calls': [], 'usage_metadata': None}]}}
    {'action': {'messages': [{'content': '["저는 샌프란시스코의 현재 날씨를 검색했습니다. 결과: 샌프란시스코는 맑습니다. 다만, 쌍둥이들은 조심하세요 😈."]', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'search', 'id': '7bad0e72-5ebe-4b08-9b8a-b99b0fe22fb7', 'tool_call_id': 'toolu_011vroKUtWU7SBdrngpgpFMn'}]}}
    {'agent': {'messages': [{'content': "검색 결과에 따르면, 현재 샌프란시스코의 날씨에 대한 정보를 제공할 수 있습니다:\n\n현재 샌프란시스코의 날씨는 맑습니다. 이는 야외 활동과 도시의 아름다운 경치를 즐기기에 좋은 소식입니다.\n\n검색 결과에 쌍둥이에 대한 이상한 언급이 포함되어 있다는 점도 주목할 필요가 있습니다. 이는 날씨 보고서의 일반적인 내용이 아닙니다. 아마도 검색 엔진이 점성술 관련 정보나 농담을 포함했기 때문일 수 있습니다. 그러나 날씨에 대한 질문에 답하기 위해서는 현재 샌프란시스코가 맑다는 사실에 초점을 맞출 수 있습니다.\n\n샌프란시스코의 날씨에 대한 더 구체적인 정보, 예를 들어 온도, 풍속 또는 향후 며칠간의 예보가 필요하시면 말씀해 주세요. 그 정보를 검색해 드릴 수 있습니다.", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-dbac539a-33c8-4f0c-9e20-91f318371e7c', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}


이제 주 목록을 가져오고, 세 번째 상태(도구 호출 바로 전에)에서 호출하겠습니다:


=== "Python"

    ```python
    states = await client.threads.get_history(thread['thread_id'])

    # 'next' 속성을 확인하여 이 상태가 올바른지 확인할 수 있습니다.
    state_to_replay = states[2]
    print(state_to_replay['next'])
    ```

=== "자바스크립트"

    ```js
    const states = await client.threads.getHistory(thread['thread_id']);

    // 'next' 속성을 확인하여 이 상태가 올바른지 확인할 수 있습니다.
    const stateToReplay = states[2];
    console.log(stateToReplay['next']);
    ```

=== "CURL"

    ```bash
    curl --request GET --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/history | jq -r '.[2].next'
    ```

출력:

    ['action']



상태에서 다시 실행하려면 먼저 스레드 상태에 대한 빈 업데이트를 발행해야 합니다. 그런 다음 결과 `checkpoint_id`를 다음과 같이 전달해야 합니다:

=== "Python"

    ```python
    state_to_replay = states[2]
    updated_config = await client.threads.update_state(
        thread["thread_id"],
        {"messages": []},
        checkpoint_id=state_to_replay["checkpoint_id"]
    )
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id, # graph_id
        input=None,
        stream_mode="updates",
        checkpoint_id=updated_config["checkpoint_id"]
    ):
        if chunk.data and chunk.event != "metadata": 
            print(chunk.data)
    ```

=== "자바스크립트"


    ```js
    const stateToReplay = states[2];
    const config = await client.threads.updateState(thread["thread_id"], { values: {"messages": [] }, checkpointId: stateToReplay["checkpoint_id"] });
    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: null,
        streamMode: "updates",
        checkpointId: config["checkpoint_id"]
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
    curl --request GET --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/history | jq -c '
        .[2] as $state_to_replay |
        {
            values: { messages: .[2].values.messages[-1] },
            checkpoint_id: $state_to_replay.checkpoint_id
        }' | \
    curl --request POST \
        --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state \
        --header 'Content-Type: application/json' \
        --data @- | jq .checkpoint_id | \
    curl --request POST \
     --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"checkpoint_id\": \"$1\",
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

    {'action': {'messages': [{'content': '["제가 조사한 결과: 샌프란시스코의 현재 날씨. 결과: 샌프란시스코는 맑습니다. 그러나 쌍둥이자리라면 조심하는 것이 좋습니다 😈."]', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'search', 'id': 'eba650e5-400e-4938-8508-f878dcbcc532', 'tool_call_id': 'toolu_011vroKUtWU7SBdrngpgpFMn'}]}}
    {'agent': {'messages': [{'content': "검색 결과를 기반으로 하면, 샌프란시스코의 현재 날씨에 대한 정보를 제공할 수 있습니다:\n\n샌프란시스코의 날씨는 현재 맑습니다. 이는 야외 활동을 계획하거나 도시에서 기분 좋은 하루를 즐기고자 하는 경우 좋은 소식입니다.\n\n또한, 검색 결과에 쌍둥이자리에 관한 다소 특이한 언급이 포함되어 있다는 점에 주목할 필요가 있는데, 이는 날씨와 직접적인 관련이 없어 보입니다. 이는 정보를 얻은 출처에서 추가된 유머러스한 내용 같아 보입니다.\n\n샌프란시스코의 날씨나 필요한 다른 정보에 대해 더 알고 싶으신 것이 있나요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-bc6dca3f-a1e2-4f59-a69b-fe0515a348bb', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}


그래프가 도구 노드에서 원래 그래프 실행과 동일한 입력으로 재시작한 것을 볼 수 있습니다.

## 이전 상태에서 분기하기

LangGraph의 체크포인팅을 사용하면 과거 상태를 단순히 재생하는 것 이상을 할 수 있습니다. 이전 위치에서 분기하여 에이전트가 대체 경로를 탐색하도록 하거나, 사용자가 워크플로의 변경 사항을 "버전 관리"할 수 있게 할 수 있습니다.

특정 시점에 상태를 수정하는 방법을 보여드리겠습니다. 도구에 대한 입력을 변경하기 위해 상태를 업데이트해 보겠습니다.

=== "Python"

    ```python
    # 이제 상태에서 마지막 메시지를 가져옵니다
    # 이는 우리가 업데이트하고자 하는 도구 호출을 포함하고 있습니다
    last_message = state_to_replay['values']['messages'][-1]

    # 이제 해당 도구 호출의 인수를 업데이트합니다
    last_message['tool_calls'][0]['args'] = {'query': '샌프란시스코의 현재 날씨'}

    config = await client.threads.update_state(thread['thread_id'],{"messages":[last_message]},checkpoint_id=state_to_replay['checkpoint_id'])
    ```

=== "Javascript"

    ```js
    // 이제 상태에서 마지막 메시지를 가져옵니다.
    // 업데이트하고자 하는 도구 호출이 포함된 메시지입니다.
    let lastMessage = stateToReplay['values']['messages'][-1];

    // 이제 해당 도구 호출의 인수를 업데이트합니다.
    lastMessage['tool_calls'][0]['args'] = { 'query': '샌프란시스코의 현재 날씨' };

    const config = await client.threads.updateState(thread['thread_id'], { values: { "messages": [lastMessage] }, checkpointId: stateToReplay['checkpoint_id'] });
    ```

=== "CURL"

    ```bash
    curl -s --request GET --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/history | \
    jq -c '
        .[2] as $state_to_replay |
        .[2].values.messages[-1].tool_calls[0].args.query = "샌프란시스코의 현재 날씨" |
        {
            values: { messages: .[2].values.messages[-1] },
            checkpoint_id: $state_to_replay.checkpoint_id
        }' | \
    curl --request POST \
        --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state \
        --header 'Content-Type: application/json' \
        --data @-
    ```

이제 `state_to_replay`의 분기인 `new_state`에서 시작하여 이 새로운 구성으로 그래프를 다시 실행할 수 있습니다.

=== "Python"

    ```python
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=None,
        stream_mode="updates",
        checkpoint_id=config['checkpoint_id']
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
        streamMode: "updates",
        checkpointId: config['checkpoint_id'],
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
    curl -s --request GET --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state | \
    jq -c '.checkpoint_id' | \
    curl --request POST \
     --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"checkpoint_id\": \"$1\",
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

{'action': {'messages': [{'content': '["나는 검색했다: SF의 현재 날씨. 결과: 샌프란시스코는 맑지만, 쌍둥이자리인 경우 조심해야 할 것 같아 😈."]', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'search', 'id': '2baf9941-4fda-4081-9f87-d76795d289f1', 'tool_call_id': 'toolu_011vroKUtWU7SBdrngpgpFMn'}]}}
{'agent': {'messages': [{'content': "검색 결과에 따르면, 샌프란시스코(SF)의 현재 날씨에 대한 정보를 제공할 수 있습니다:\n\n샌프란시스코의 날씨는 현재 맑습니다. 이는 맑은 하늘에 많은 햇빛이 있다는 뜻입니다. \n\n특정 온도는 검색 결과에서 제공되지 않았지만, 샌프란시스코의 맑은 날씨는 일반적으로 쾌적한 온도를 의미합니다. 샌프란시스코는 온화한 기후로 유명하기 때문에, 맑은 날에도 보통 그리 덥지 않습니다.\n\n검색 결과에는 쌍둥이자리와 관련된 장난스러운 언급도 포함되어 있었습니다. 그러나 이는 아마도 단지 농담이거나 검색 엔진의 표현 일부일 뿐, 실제 날씨 조건과 관련이 없습니다.\n\n샌프란시스코의 날씨에 대해 더 알고 싶은 특정 정보가 있나요? 온도, 바람 조건 또는 향후 날씨 예보에 대한 세부정보가 필요하시다면 기꺼이 또 다른 검색을 수행하겠습니다.", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-a83de52d-ed18-4402-9384-75c462485743', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}
 
우리가 알 수 있듯이, 검색 쿼리가 샌프란시스코에서 SF로 변경되었습니다. 우리가 바라던 대로입니다!
