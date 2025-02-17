_한국어로 기계번역됨_

# 중단점 추가 방법

LangGraph 에이전트를 생성할 때, 종종 인간의 개입이 필요한 요소를 추가하는 것이 유용할 수 있습니다. 이는 도구에 접근할 수 있게 할 때 도움이 될 수 있습니다. 이러한 상황에서는 작업을 실행하기 전에 수동으로 승인하고 싶을 수 있습니다.

이는 여러 가지 방법으로 이루어질 수 있지만, 주로 지원되는 방법은 노드가 실행되기 전에 "중단(interrupt)"을 추가하는 것입니다. 이는 해당 노드에서 실행을 중단시킵니다. 그 자리를 계속 진행할 수 있습니다.

## 설정

### 그래프 코드

이 방법에서는 간단한 ReAct 스타일의 호스팅된 그래프를 사용합니다(정의에 대한 전체 코드는 [여기](../../how-tos/human_in_the_loop/breakpoints.ipynb)에서 볼 수 있습니다). 중요한 것은 두 개의 노드가 있어야 한다는 것입니다(하나는 LLM을 호출하는 `agent`, 다른 하나는 도구를 호출하는 `action`), 그리고 `agent`에서 `action`을 호출할지 아니면 그래프 실행을 종료할지를 결정하는 라우팅 함수가 있어야 합니다(`action` 노드는 실행 후 항상 `agent` 노드를 호출합니다).

### SDK 초기화

=== "파이썬"

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

## 중단점 추가

이제 그래프 실행에 중단점을 추가하고자 하며, 이는 도구가 호출되기 전에 수행됩니다. `interrupt_before=["action"]`를 추가하여 action 노드를 호출하기 전에 중단하라는 지시를 할 수 있습니다. 이는 그래프를 컴파일 할 때 또는 실행을 시작할 때 수행할 수 있습니다. 여기서는 실행을 시작할 때 수행할 것이며, 컴파일 시 추가하고 싶다면 그래프가 정의된 파이썬 파일을 수정하고 `.compile`을 호출할 때 `interrupt_before` 매개변수를 추가해야 합니다.

먼저 SDK를 통해 호스팅된 LangGraph 인스턴스에 접근해 보겠습니다:

그리고 이제 도구 노드 앞에 중단점을 추가하여 컴파일해 보겠습니다:

=== "파이썬"

```python
input = {"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때요?"}]}
async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id,
    input=input,
    stream_mode="updates",
    interrupt_before=["action"],
):
    print(f"새로운 타입의 이벤트 수신: {chunk.event}...")
    print(chunk.data)
    print("\n\n")
```

=== "자바스크립트"

```js
const input = { messages: [{ role: "human", content: "샌프란시스코의 날씨는 어때요?" }] };

const streamResponse = client.runs.stream(
  thread["thread_id"],
  assistantId,
  {
    input: input,
    streamMode: "updates",
    interruptBefore: ["action"]
  }
);

for await (const chunk of streamResponse) {
  console.log(`새로운 타입의 이벤트 수신: ${chunk.event}...`);
  console.log(chunk.data);
  console.log("\n\n");
}
```

=== "CURL"

    ```bash
    curl --request POST \
     --url <배포_URL>/threads/<스레드_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어때?\"}]},
       \"interrupt_before\": [\"action\"],
       \"stream_mode\": [
         \"messages\"
       ]
     }" | \
     sed 's/\r$//' | \
     awk '
     /^event:/ {
         if (data_content != "") {
             print data_content "\n"
         }
         sub(/^event: /, "수신된 이벤트 유형: ", $0)
         printf "%s...\n", $0
         data_content = ""
     }
     /^data:/ {
         sub(/^data: /, "", $0)
         data_content = $0
     }
     END {
         if (data_content != "") {
             print data_content "\n"
         }
     }
     '
    ```

출력:

    수신된 새 이벤트 유형: metadata...
    {'run_id': '3b77ef83-687a-4840-8858-0371f91a92c3'}
    
    수신된 새 이벤트 유형: data...
    {'agent': {'messages': [{'content': [{'id': 'toolu_01HwZqM1ptX6E15A5LAmyZTB', 'input': {'query': '샌프란시스코의 날씨'}, 'name': 'tavily_search_results_json', 'type': 'tool_use'}], 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-e5d17791-4d37-4ad2-815f-a0c4cba62585', 'example': False, 'tool_calls': [{'name': 'tavily_search_results_json', 'args': {'query': '샌프란시스코의 날씨'}, 'id': 'toolu_01HwZqM1ptX6E15A5LAmyZTB'}], 'invalid_tool_calls': []}]}}
    
    수신된 새 이벤트 유형: end...
    None
    
    
    

