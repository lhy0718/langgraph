_한국어로 기계번역됨_

# 리뷰 툴 호출

사람이 참여하는 (HIL) 상호작용은 [주체 시스템](https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/#human-in-the-loop)에서 매우 중요합니다. 일반적인 패턴은 특정 툴 호출 후에 사람을 포함하는 단계(HIL)를 추가하는 것입니다. 이러한 툴 호출은 종종 함수 호출이나 일부 정보 저장으로 이어집니다. 예를 들면 다음과 같습니다.

- SQL을 실행하기 위한 툴 호출, 그 후 툴에 의해 실행됩니다
- 요약을 생성하기 위한 툴 호출, 그 후 그래프의 상태에 저장됩니다

툴 호출을 사용하는 것은 **실제로 툴을 호출하든지 하지 않든지 간에** 일반적입니다.

여기서 수행하고 싶은 몇 가지 다른 상호작용이 일반적으로 있습니다:

1. 툴 호출을 승인하고 계속 진행
2. 툴 호출을 수동으로 수정한 후 계속 진행
3. 자연어 피드백을 제공한 후 이를 에이전트에 전달하고 계속 진행하지 않음

이 기능은 [중단점](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/breakpoints/)을 사용하여 LangGraph에서 구현할 수 있습니다: 중단점을 사용하면 특정 단계 전에 그래프 실행을 중단할 수 있습니다. 이 중단점에서 위의 세 가지 옵션 중 하나를 선택하고 그래프 상태를 수동으로 업데이트할 수 있습니다.

## 설정

우리는 호스팅 중인 그래프의 전체 코드를 보여주지는 않을 것이지만, 원하는 경우 [여기](../../how-tos/human_in_the_loop/review-tool-calls.ipynb#simple-usage)에서 확인할 수 있습니다. 그래프가 호스팅되면 이를 호출하고 사용자 입력을 기다릴 준비가 됩니다.

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

## 리뷰가 필요 없는 예시

리뷰가 필요하지 않은 예시를 살펴보겠습니다 (툴 호출이 없기 때문에)

=== "Python"

```python
input = { 'messages':[{ "role":"user", "content":"hi!" }] }

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
const input = { "messages": [{ "role": "user", "content": "hi!" }] };

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
   \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"안녕하세요!\"}]},
   \"stream_mode\": [
     \"updates\"
   ],
   \"interrupt_before\": [\"action\"]
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

    {'messages': [{'content': '안녕하세요!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '39c51f14-2d5c-4690-883a-d940854b1845', 'example': False}]}
    {'messages': [{'content': '안녕하세요!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '39c51f14-2d5c-4690-883a-d940854b1845', 'example': False}, {'content': [{'text': "안녕하세요! 환영합니다. 오늘 무엇을 도와드릴까요? 알고 싶거나 찾고 계신 정보가 있나요?", 'type': 'text', 'index': 0}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'end_turn', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-d65e07fb-43ff-4d98-ab6b-6316191b9c8b', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 355, 'output_tokens': 31, 'total_tokens': 386}}]}

상태를 확인하면 완료된 것을 볼 수 있습니다.

=== "Python"

    ```python
    state = await client.threads.get_state(thread["thread_id"])

    print(state['next'])
    ```

=== "Javascript"

    ```js
    const state = await client.threads.getState(thread["thread_id"]);

    console.log(state.next);
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state | jq -c '.next'
    ```

출력:

    []

## 도구 승인 예시

이제 도구 호출을 승인하는 모습이 어떤지 살펴보겠습니다. 스트리밍 호출에 인터럽트를 전달할 필요가 없음을 주목해주세요. 왜냐하면 그래프(여기서 정의됨 [여기](../../how-tos/human_in_the_loop/review-tool-calls.ipynb#simple-usage))가 이미 `human_review_node` 이전에 인터럽트로 컴파일되었기 때문입니다.

=== "Python"

    ```python
    input = {"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어떨까요?"}]}

    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=input,
    ):
        if chunk.data and chunk.event != "metadata": 
            print(chunk.data)
    ```

=== "Javascript"

    ```js
    const input = { "messages": [{ "role": "user", "content": "샌프란시스코의 날씨는 어때?" }] };

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: input,
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
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어때?\"}]}
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

    {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '54e19d6e-89fa-44fb-b92c-12e7dd4ddf08', 'example': False}]}
    {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '54e19d6e-89fa-44fb-b92c-12e7dd4ddf08', 'example': False}, {'content': [{'text': "확실히! 샌프란시스코의 날씨를 확인할 수 있도록 도와드리겠습니다. 이 정보를 얻기 위해 날씨 검색 기능을 사용할 것입니다. 바로 진행하겠습니다.", 'type': 'text', 'index': 0}, {'id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-45a6b6c3-ac69-42a4-8957-d982203d6392', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 90, 'total_tokens': 450}}]}


지금 확인해 보면, 인간 검토를 기다리고 있는 것을 알 수 있습니다.

=== "Python"

    ```python
    state = await client.threads.get_state(thread["thread_id"]);

    print(state['next'])
    ```

=== "Javascript"

    ```js
    const state = await client.threads.getState(thread["thread_id"]);

    console.log(state.next);
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <DELPOYMENT_URL>/threads/<THREAD_ID>/state | jq -c '.next'
    ```

출력:

    ['human_review_node']

도구 호출을 승인하기 위해, 수정 없이 스레드를 계속 진행할 수 있습니다. 이를 위해 입력 없이 새 실행을 생성하면 됩니다.

=== "Python"

    ```python
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=None,
        stream_mode="values",
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
        streamMode: "values",
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
       \"assistant_id\": \"agent\"
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

    {'messages': [{'content': "샌프란시스코의 날씨는 어떤가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '54e19d6e-89fa-44fb-b92c-12e7dd4ddf08', 'example': False}, {'content': [{'text': "물론이죠! 샌프란시스코의 날씨를 확인해드릴 수 있습니다. 이 정보를 얻기 위해, 저는 날씨 검색 기능을 사용할 것입니다. 지금 바로 해드리겠습니다.", 'type': 'text', 'index': 0}, {'id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'input': {}, 'name': '날씨_검색', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-45a6b6c3-ac69-42a4-8957-d982203d6392', 'example': False, 'tool_calls': [{'name': '날씨_검색', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 90, 'total_tokens': 450}}, {'content': '맑음!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': '날씨_검색', 'id': '826cd0f2-9cc6-46f0-b7df-daa6a05d13d2', 'tool_call_id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'artifact': None, 'status': 'success'}]}
    {'messages': [{'content': "샌프란시스코의 날씨는 어떤가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '54e19d6e-89fa-44fb-b92c-12e7dd4ddf08', 'example': False}, {'content': [{'text': "물론이죠! 샌프란시스코의 날씨를 확인해드릴 수 있습니다. 이 정보를 얻기 위해, 저는 날씨 검색 기능을 사용할 것입니다. 지금 바로 해드리겠습니다.", 'type': 'text', 'index': 0}, {'id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'input': {}, 'name': '날씨_검색', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-45a6b6c3-ac69-42a4-8957-d982203d6392', 'example': False, 'tool_calls': [{'name': '날씨_검색', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 90, 'total_tokens': 450}}, {'content': '맑음!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': '날씨_검색', 'id': '826cd0f2-9cc6-46f0-b7df-daa6a05d13d2', 'tool_call_id': 'toolu_015yrR3GMDXe6X8m2p9CsEDN', 'artifact': None, 'status': 'success'}, {'content': [{'text': "\n\n좋은 소식입니다! 오늘 샌프란시스코의 날씨는 맑습니다. 만리포의 아름다운 하루입니다. 날씨나 다른 정보에 대해 더 알고 싶은 것이 있나요?", 'type': 'text', 'index': 0}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'end_turn', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-5d5fd0f1-a939-447e-801a-9aaa812322d3', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 464, 'output_tokens': 50, 'total_tokens': 514}}]}

## 도구 호출 수정

이제 도구 호출을 수정하고 싶다고 가정해봅시다. 예를 들어, 일부 매개변수(또는 호출할 도구 자체)를 변경한 후 해당 도구를 실행합니다.

=== "Python"

    ```python
    input = {"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어떤가요?"}]}

    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=input,
        stream_mode="values",
    ):
        if chunk.data and chunk.event != "metadata": 
            print(chunk.data)
    ```

=== "Javascript"

    ```js
    const input = { "messages": [{ "role": "user", "content": "샌프란시스코의 날씨는 어떤가요?" }] };

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: input,
        streamMode: "values",
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
     --url <배포_URL>/threads/<스레드_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어떤가요?\"}]}
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

결과:

    {'messages': [{'content': "샌프란시스코의 날씨는 어떤가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': 'cec11391-84da-464b-bd2a-bd4f0d93b9ee', 'example': False}]}
    {'messages': [{'content': "샌프란시스코의 날씨는 어떤가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': 'cec11391-84da-464b-bd2a-bd4f0d93b9ee', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해 weather_search 기능을 사용할 수 있습니다. 제가 그렇게 해드리겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_01SunSpDurNfcnXppWLPrtjC', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "San Francisco"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-6326da9f-6061-4e12-8586-482e32ab4cab', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': 'San Francisco'}, 'id': 'toolu_01SunSpDurNfcnXppWLPrtjC', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 80, 'total_tokens': 440}}]}


이를 수행하기 위해 먼저 상태를 업데이트해야 합니다. 이를 위해 덮어쓰려는 메시지와 **같은** ID를 가진 메시지를 전달하면 됩니다. 이것은 그 오래된 메시지를 **대체**하는 효과를 가집니다. 이것은 제공하는 **리듀서** 덕분에 가능하며, 같은 ID의 메시지를 교체합니다 - 이에 대한 자세한 내용은 [여기](https://langchain-ai.github.io/langgraph/concepts/low_level/#working-with-messages-in-graph-state)를 읽어보세요.


=== "Python"

    ```python
    # 우리가 교체하려는 메시지의 ID를 얻으려면 현재 상태를 가져와서 거기에서 찾아야 합니다.
    state = await client.threads.get_state(thread['thread_id'])
    print("현재 상태:")
    print(state['values'])
    print("\n현재 도구 호출 ID:")
    current_content = state['values']['messages'][-1]['content']
    current_id = state['values']['messages'][-1]['id']
    tool_call_id = state['values']['messages'][-1]['tool_calls'][0]['id']
    print(tool_call_id)

    # 이제 교체할 도구 호출을 구성해야 합니다.
    # 우리는 인수를 `샌프란시스코, 미국`으로 변경할 것입니다.
    # 우리는任意의 인수나 도구 이름을 변경할 수 있으며, 유효해야 합니다.
    new_message = {
        "role": "assistant", 
        "content": current_content,
        "tool_calls": [
            {
                "id": tool_call_id,
                "name": "weather_search",
                "args": {"city": "샌프란시스코, 미국"}
            }
        ],
        # 이 부분은 중요합니다 - 이 ID는 교체하려는 메시지와 동일해야 합니다!
        # 그렇지 않으면 별도의 메시지로 표시됩니다.
        "id": current_id
    }
    await client.threads.update_state(
        # 이 부분은 이 스레드를 나타내는 구성입니다.
        thread['thread_id'], 
        # 업데이트된 값을 푸시합니다.
        {"messages": [new_message]}, 
        # 우리는 이 업데이트를 우리의 human_review_node로서 푸시합니다.
        as_node="human_review_node"
    )

    print("\n실행 재개")
    # 이제 여기서부터 실행을 계속하겠습니다.
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=None,
    ):
        if chunk.data and chunk.event != "metadata": 
            print(chunk.data)
    ```

=== "자바스크립트"

    ```js
const state = await client.threads.getState(thread.thread_id);
console.log("현재 상태:");
console.log(state.values);

console.log("\n현재 도구 호출 ID:");
const lastMessage = state.values.messages[state.values.messages.length - 1];
const currentContent = lastMessage.content;
const currentId = lastMessage.id;
const toolCallId = lastMessage.tool_calls[0].id;
console.log(toolCallId);

// 대체 도구 호출 구성
const newMessage = {
  role: "assistant",
  content: currentContent,
  tool_calls: [
    {
      id: toolCallId,
      name: "weather_search",
      args: { city: "San Francisco, USA" }
    }
  ],
  // 교체할 메시지와 ID가 동일하도록 보장
  id: currentId
};

await client.threads.updateState(
  thread.thread_id,  // 스레드 ID
  {
    values: { "messages": [newMessage] },  // 업데이트된 메시지
    asNode: "human_review_node"
  }  // human_review_node로 작동
);

console.log("\n실행 재개");
// 여기서부터 실행을 계속합니다
const streamResponseResumed = client.runs.stream(
  thread["thread_id"],
  assistantId,
  {
    input: null,
  }
);

for await (const chunk of streamResponseResumed) {
  if (chunk.data && chunk.event !== "metadata") {
    console.log(chunk.data);
  }
}
```

    ```bash
    curl --request POST \
    --url <배포_URL>/threads/<스레드_ID>/state \
    --header 'Content-Type: application/json' \
    --data "{
        \"values\": { \"messages\": [$(curl --request GET \
            --url <배포_URL>/threads/<스레드_ID>/state |
            jq -c '{
            role: "assistant",
            content: .values.messages[-1].content,
            tool_calls: [
                {
                id: .values.messages[-1].tool_calls[0].id,
                name: "weather_search",
                args: { city: "샌프란시스코, 미국" }
                }
            ],
            id: .values.messages[-1].id
            }')
        ]},
        \"as_node\": \"human_review_node\"
    }" && echo "실행 재개" && curl --request POST \
    --url <배포_URL>/threads/<스레드_ID>/runs/stream \
    --header 'Content-Type: application/json' \
    --data '{
    "assistant_id": "agent"
    }' | \
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

    현재 상태:
    {'messages': [{'content': "샌프란시스코 날씨 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '8713d1fa-9b26-4eab-b768-dafdaac70590', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해 weather_search 기능을 사용할 수 있습니다. 제가 도와드릴게요.', 'type': 'text', 'index': 0}, {'id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-ede13f26-daf5-4d8f-817a-7611075bbcf1', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 80, 'total_tokens': 440}}]}

    현재 도구 호출 ID:
    toolu_01VzagzsUGZsNMwW1wHkcw7h

    실행 재개
    {'messages': [{'content': "샌프란시스코 날씨 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '8713d1fa-9b26-4eab-b768-dafdaac70590', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해 weather_search 기능을 사용할 수 있습니다. 제가 도와드릴게요.', 'type': 'text', 'index': 0}, {'id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-ede13f26-daf5-4d8f-817a-7611075bbcf1', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': None}, {'content': '맑습니다!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'weather_search', 'id': '7fc7d463-66bf-4555-9929-6af483de169b', 'tool_call_id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'artifact': None, 'status': 'success'}]}
    {'messages': [{'content': "샌프란시스코 날씨 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '8713d1fa-9b26-4eab-b768-dafdaac70590', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해 weather_search 기능을 사용할 수 있습니다. 제가 도와드릴게요.', 'type': 'text', 'index': 0}, {'id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-ede13f26-daf5-4d8f-817a-7611075bbcf1', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': None}, {'content': '맑습니다!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'weather_search', 'id': '7fc7d463-66bf-4555-9929-6af483de169b', 'tool_call_id': 'toolu_01VzagzsUGZsNMwW1wHkcw7h', 'artifact': None, 'status': 'success'}, {'content': [{'text': "\n\n검색 결과에 따르면, 샌프란시스코의 날씨는 맑습니다! 만의 날씨가 좋네요! 날씨나 다른 정보에 대해 더 알고 싶으신 것이 있나요?", 'type': 'text', 'index': 0}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'end_turn', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-d90ce97a-39f9-4330-985e-67c5f351a0c5', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 455, 'output_tokens': 52, 'total_tokens': 507}}]}

## 도구 호출에 대한 피드백 제공

때때로, 도구 호출을 실행하고 싶지 않지만 사용자에게 수동으로 도구 호출을 수정하라고 요청하고 싶지 않을 수 있습니다. 이 경우 사용자로부터 자연어 피드백을 받는 것이 더 나을 수 있습니다. 그런 다음 이러한 피드백을 도구 호출의 모의 **결과**로 삽입할 수 있습니다.

이를 수행하는 여러 가지 방법이 있습니다:

상태에 새로운 메시지를 추가할 수 있습니다 (도구 호출의 "결과"를 나타냄)
상태에 두 개의 새로운 메시지를 추가할 수 있습니다 - 하나는 도구 호출의 "오류"를 나타내고 다른 하나는 피드백을 나타내는 HumanMessage
두 가지 모두 상태에 메시지를 추가하는 것과 비슷합니다. 주요 차이점은 `human_node` 이후의 로직과 다양한 유형의 메시지를 처리하는 방법에 있습니다.

이 예제에서는 피드를 나타내는 단일 도구 호출만 추가하겠습니다. 실행해 보겠습니다!

=== "Python"

    ```python
    input = {"messages": [{"role": "user", "content": "샌프란시스코 날씨 어때?"}]}

    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=input,
    ):
        if chunk.data and chunk.event != "metadata": 
            print(chunk.data)
    ```

=== "Javascript"


    ```js
    const input = { "messages": [{ "role": "user", "content": "샌프란시스코의 날씨는 어때?" }] };

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: input,
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
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어때?\"}]}
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

Output:

    {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': 'c80f13d0-674d-4233-b6a0-3940509d3cf3', 'example': False}]}
    {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': 'c80f13d0-674d-4233-b6a0-3940509d3cf3', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 가져오기 위해, weather_search 함수를 사용할 수 있습니다. 그렇게 해드리겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_016XyTdFA8NuPWeLyZPSzoM3', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-4911ac27-3d7c-4edf-a3ca-c2908e3922eb', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_016XyTdFA8NuPWeLyZPSzoM3', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 80, 'total_tokens': 440}}]}

이를 수행하기 위해, 먼저 상태를 업데이트해야 합니다. 원하는 도구 호출의 **도구 호출 ID**로 메시지를 전달하여 이를 수행할 수 있습니다. 위의 ID와는 **다른** ID라는 점에 유의하세요.

=== "Python"

    ```python
    # 교체하고자 하는 메시지의 ID를 얻기 위해 현재 상태를 가져와서 찾아야 합니다.
    state = await client.threads.get_state(thread['thread_id'])
    print("현재 상태:")
    print(state['values'])
    print("\n현재 도구 호출 ID:")
    tool_call_id = state['values']['messages'][-1]['tool_calls'][0]['id']
    print(tool_call_id)

    # 이제 교체할 도구 호출을 구성해야 합니다.
    # 인수를 '샌프란시스코, 미국'으로 변경할 것입니다.
    # 인수나 도구 이름을 여러 개 변경할 수 있지만, 유효한 것이어야 합니다.
    new_message = {
        "role": "tool", 
        # 이것이 우리의 자연어 피드백입니다.
        "content": "사용자가 요청한 변경 사항: 국가도 전달해 주세요",
        "name": "weather_search",
        "tool_call_id": tool_call_id
    }
    await client.threads.update_state(
        # 이 스레드를 나타내는 구성입니다.
        thread['thread_id'], 
        # 우리가 밀어넣고자 하는 업데이트된 값입니다.
        {"messages": [new_message]}, 
        # 이 업데이트를 인간 검토 노드로서 밀어넣습니다.
        as_node="human_review_node"
    )

    print("\n실행 재개")
    # 이제 여기서부터 실행을 계속하겠습니다.
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=None,
        stream_mode="values",
    ):
        if chunk.data and chunk.event != "metadata": 
            print(chunk.data)
    ```

=== "Javascript"

    ```js
    const state = await client.threads.getState(thread.thread_id);
    console.log("현재 상태:");
    console.log(state.values);

    console.log("\n현재 도구 호출 ID:");
    const lastMessage = state.values.messages[state.values.messages.length - 1];
    const toolCallId = lastMessage.tool_calls[0].id;
    console.log(toolCallId);

    // 대체 도구 호출 생성
    const newMessage = {
      role: "tool",
      content: "사용자가 요청한 변경 사항: 국가도 전달해 주세요",
      name: "weather_search",
      tool_call_id: toolCallId,
    };

    await client.threads.updateState(
      thread.thread_id,  // 스레드 ID
      {
        values: { "messages": [newMessage] },  // 업데이트된 메시지
        asNode: "human_review_node"
      }  // human_review_node로 작동
    );

    console.log("\n실행 재개");
    // 여기서부터 계속 실행
    const streamResponseEdited = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: null,
        streamMode: "values",
        interruptBefore: ["action"],
      }
    );

    for await (const chunk of streamResponseEdited) {
      if (chunk.data && chunk.event !== "metadata") {
        console.log(chunk.data);
      }
    }
    ```

=== "CURL"

    ```bash
    curl --request POST \
    --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state \
    --header 'Content-Type: application/json' \
    --data "{
        \"values\": { \"messages\": [$(curl --request GET \
            --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state |
            jq -c '{
            role: "tool",
            content: "사용자가 요청한 변경 사항: 국가도 전달해 주세요",
            name: "get_weather",
            tool_call_id: .values.messages[-1].id.tool_calls[0].id
            }')
        ]},
        \"as_node\": \"human_review_node\"
    }" && echo "실행 재개" && curl --request POST \
    --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
    --header 'Content-Type: application/json' \
    --data '{
    "assistant_id": "agent"
    }' | \
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

    현재 상태:
    {'messages': [{'content': "샌프란시스코의 날씨가 어떤가요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '3b2bbc38-d11b-49eb-80c0-c24a40dab5a8', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해서 weather_search 함수를 사용할 수 있습니다. 제가 그 작업을 해드리겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-c5a50900-abf5-4885-9cdb-da2bf0d892ac', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 80, 'total_tokens': 440}}]}

    현재 도구 호출 ID:
    toolu_01NNw18j57GEGPZvsa9f1wvX

    실행 재개
{'messages': [{'content': "샌프란시스코 날씨 어때요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '3b2bbc38-d11b-49eb-80c0-c24a40dab5a8', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해 weather_search 함수를 사용할 수 있습니다. 그렇게 해드리겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-c5a50900-abf5-4885-9cdb-da2bf0d892ac', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 80, 'total_tokens': 440}}, {'content': '사용자가 요청한 변경사항: 국가도 함께 전달하세요', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'weather_search', 'id': '787288be-213c-4fd3-8503-4a009bdb1b00', 'tool_call_id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'artifact': None, 'status': 'success'}, {'content': [{'text': '\n\n잘못된 점 사과드립니다. 함수가 추가 정보가 필요한 것 같습니다. 좀 더 구체적인 요청으로 다시 시도해보겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_01YAbLBoKozJyRQnB8LUMpXC', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코, 미국"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-5c355a56-cfe3-4046-b49f-f5b09fc397ef', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코, 미국'}, 'id': 'toolu_01YAbLBoKozJyRQnB8LUMpXC', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 461, 'output_tokens': 83, 'total_tokens': 544}}, {'content': '맑음!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'weather_search', 'id': '3b857482-bca2-4a73-a9ab-1f35a3e43e5f', 'tool_call_id': 'toolu_01YAbLBoKozJyRQnB8LUMpXC', 'artifact': None, 'status': 'success'}]}
{'messages': [{'content': "샌프란시스코 날씨 어때요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '3b2bbc38-d11b-49eb-80c0-c24a40dab5a8', 'example': False}, {'content': [{'text': '샌프란시스코의 날씨 정보를 얻기 위해 weather_search 함수를 사용할 수 있습니다. 그렇게 해드리겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-c5a50900-abf5-4885-9cdb-da2bf0d892ac', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코'}, 'id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 360, 'output_tokens': 80, 'total_tokens': 440}}, {'content': '사용자가 요청한 변경사항: 국가도 함께 전달하세요', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'weather_search', 'id': '787288be-213c-4fd3-8503-4a009bdb1b00', 'tool_call_id': 'toolu_01NNw18j57GEGPZvsa9f1wvX', 'artifact': None, 'status': 'success'}, {'content': [{'text': '\n\n잘못된 점 사과드립니다. 함수가 추가 정보가 필요한 것 같습니다. 좀 더 구체적인 요청으로 다시 시도해보겠습니다.', 'type': 'text', 'index': 0}, {'id': 'toolu_01YAbLBoKozJyRQnB8LUMpXC', 'input': {}, 'name': 'weather_search', 'type': 'tool_use', 'index': 1, 'partial_json': '{"city": "샌프란시스코, 미국"}'}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'tool_use', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-5c355a56-cfe3-4046-b49f-f5b09fc397ef', 'example': False, 'tool_calls': [{'name': 'weather_search', 'args': {'city': '샌프란시스코, 미국'}, 'id': 'toolu_01YAbLBoKozJyRQnB8LUMpXC', 'type': 'tool_call'}], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 461, 'output_tokens': 83, 'total_tokens': 544}}, {'content': '맑음!', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': 'weather_search', 'id': '3b857482-bca2-4a73-a9ab-1f35a3e43e5f', 'tool_call_id': 'toolu_01YAbLBoKozJyRQnB8LUMpXC', 'artifact': None, 'status': 'success'}, {'content': [{'text': "\n\n좋은 소식입니다! 오늘 샌프란시스코의 날씨는 맑습니다. 날씨나 다른 정보에 대해 더 알고 싶은 것이 있나요?", 'type': 'text', 'index': 0}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'end_turn', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-6a857bb1-f65b-4b86-93d6-c025e003c777', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 557, 'output_tokens': 38, 'total_tokens': 595}}]}
