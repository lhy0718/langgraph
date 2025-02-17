_한국어로 기계번역됨_

# 구성으로 에이전트 생성하기

LangGraph API의 이점 중 하나는 다양한 구성으로 에이전트를 생성할 수 있는 기능입니다. 이는 다음과 같은 경우에 유용합니다:

- 인지 아키텍처를 LangGraph로 한 번 정의하기
- 시스템 메시지나 사용할 LLM과 같은 일부 속성을 통해 해당 LangGraph가 구성 가능하도록 설정하기
- 사용자가 임의의 구성으로 에이전트를 생성하고 저장한 후, 이를 나중에 사용할 수 있도록 하기

이번 가이드에서는 기본적으로 구축된 에이전트에 대해 이를 수행하는 방법을 보여줍니다.

정의한 에이전트를 살펴보면, `call_model` 노드 내에서 일부 구성을 기반으로 모델을 생성한 것을 볼 수 있습니다. 해당 노드는 다음과 같습니다:

=== "Python"

```python
def call_model(state, config):
    messages = state["messages"]
    model_name = config.get('configurable', {}).get("model_name", "anthropic")
    model = _get_model(model_name)
    response = model.invoke(messages)
    # 기존 목록에 추가될 것이므로 리스트를 반환합니다.
    return {"messages": [response]}
```

=== "Javascript"

```js
function callModel(state: State, config: RunnableConfig) {
  const messages = state.messages;
  const modelName = config.configurable?.model_name ?? "anthropic";
  const model = _getModel(modelName);
  const response = model.invoke(messages);
  // 기존 목록에 추가될 것이므로 리스트를 반환합니다.
  return { messages: [response] };
}
```

우리는 config 내에서 `model_name` 매개변수를 찾고 있습니다(찾지 못할 경우 기본값은 `anthropic`입니다). 이는 기본적으로 Anthropic을 모델 제공자로 사용하고 있음을 의미합니다. 이 예제에서는 OpenAI를 사용하도록 구성된 에이전트를 생성하는 방법을 살펴보겠습니다.

먼저 클라이언트와 스레드를 설정해 보겠습니다:

=== "Python"

```python
from langgraph_sdk import get_client

client = get_client(url=<DEPLOYMENT_URL>)
# 구성되지 않은 도우미 선택
assistants = await client.assistants.search()
assistant = [a for a in assistants if not a["config"]][0]
```

=== "Javascript"

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
// 구성되지 않은 도우미 선택
const assistants = await client.assistants.search();
const assistant = assistants.find(a => !a.config);
```

=== "CURL"

```bash
curl --request POST \
    --url <DEPLOYMENT_URL>/assistants/search \
    --header 'Content-Type: application/json' \
    --data '{
        "limit": 10,
        "offset": 0
    }' | jq -c 'map(select(.config == null or .config == {})) | .[0]'
```

이제 `.get_schemas`를 호출하여 이 그래프와 관련된 스키마를 가져올 수 있습니다:

=== "Python"

```python
schemas = await client.assistants.get_schemas(
    assistant_id=assistant["assistant_id"]
)
# 여러 유형의 스키마가 있습니다.
# 구성 가능한 매개변수를 보기 위해 `config_schema`를 가져올 수 있습니다.
print(schemas["config_schema"])
```

=== "Javascript"

```js
const schemas = await client.assistants.getSchemas(
  assistant["assistant_id"]
);
// 여러 유형의 스키마가 있습니다.
// 구성 가능한 매개변수를 보기 위해 `config_schema`를 가져올 수 있습니다.
console.log(schemas.config_schema);
```

=== "CURL"

```bash
curl --request GET \
    --url <DEPLOYMENT_URL>/assistants/<ASSISTANT_ID>/schemas | jq -r '.config_schema'
```

출력:

```json
{
    'model_name': 
        {
            'title': '모델 이름',
            'enum': ['anthropic', 'openai'],
            'type': 'string'
        }
}
```

이제 구성으로 어시스턴트를 초기화할 수 있습니다:

=== "파이썬"

```python
openai_assistant = await client.assistants.create(
    # "agent"는 우리가 배포한 그래프의 이름입니다
    "agent", config={"configurable": {"model_name": "openai"}}
)

print(openai_assistant)
```

=== "자바스크립트"

```js
let openAIAssistant = await client.assistants.create(
  // "agent"는 우리가 배포한 그래프의 이름입니다
  "agent", { "configurable": { "model_name": "openai" } }
);

console.log(openAIAssistant);
```

=== "CURL"

```bash
curl --request POST \
    --url <DEPLOYMENT_URL>/assistants \
    --header 'Content-Type: application/json' \
    --data '{"graph_id":"agent","config":{"configurable":{"model_name":"open_ai"}}}'
```

출력:

```json
{
    "assistant_id": "62e209ca-9154-432a-b9e9-2d75c7a9219b",
    "graph_id": "agent",
    "created_at": "2024-08-31T03:09:10.230718+00:00",
    "updated_at": "2024-08-31T03:09:10.230718+00:00",
    "config": {
        "configurable": {
            "model_name": "open_ai"
        }
    },
    "metadata": {}
}
```

구성이 실제로 적용되었는지 확인할 수 있습니다:

=== "파이썬"

```python
thread = await client.threads.create()
input = {"messages": [{"role": "user", "content": "누가 너를 만들었니?"}]}
async for event in client.runs.stream(
    thread["thread_id"],
    openai_assistant["assistant_id"],
    input=input,
    stream_mode="updates",
):
    print(f"수신된 이벤트 유형: {event.event}")
    print(event.data)
    print("\n\n")
```

=== "자바스크립트"

    ```js
    const thread = await client.threads.create();
    let input = { "messages": [{ "role": "user", "content": "누가 당신을 만들었나요?" }] };

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      openAIAssistant["assistant_id"],
      {
        input,
        streamMode: "updates"
      }
    );

    for await (const event of streamResponse) {
      console.log(`이벤트 유형 수신: ${event.event}`);
      console.log(event.data);
      console.log("\n\n");
    }
    ```

=== "CURL"

    ```bash
    thread_id=$(curl --request POST \
        --url <DEPLOYMENT_URL>/threads \
        --header 'Content-Type: application/json' \
        --data '{}' | jq -r '.thread_id') && \
    curl --request POST \
        --url "<DEPLOYMENT_URL>/threads/${thread_id}/runs/stream" \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <OPENAI_ASSISTANT_ID>,
            "input": {
                "messages": [
                    {
                        "role": "user",
                        "content": "누가 당신을 만들었나요?"
                    }
                ]
            },
            "stream_mode": [
                "updates"
            ]
        }' | \
        sed 's/\r$//' | \
        awk '
        /^event:/ {
            if (data_content != "") {
                print data_content "\n"
            }
            sub(/^event: /, "이벤트 유형 수신: ", $0)
            printf "%s...\n", $0
            data_content = ""
        }
        /^data:/ {
            sub(/^data: /, "", $0)
            data_content = $0
        }
        END {
            if (data_content != "") {
                print data_content "\n\n"
            }
        }
    '
    ```

출력:

    이벤트 유형 수신: metadata
    {'run_id': '1ef6746e-5893-67b1-978a-0f1cd4060e16'}



    이벤트 유형 수신: updates
    {'agent': {'messages': [{'content': '나는 OpenAI에 의해 만들어졌습니다. OpenAI는 인공지능 기술을 개발하고 발전시키는 데 중점을 둔 연구 기관입니다.', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_157b3831f5'}, 'type': 'ai', 'name': None, 'id': 'run-e1a6b25c-8416-41f2-9981-f9cfe043f414', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}



