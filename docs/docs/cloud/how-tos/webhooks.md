_한국어로 기계번역됨_

# 웹훅 사용하기

클라이언트에서 웹훅을 사용하고 싶을 수 있습니다. 특히 비동기 스트림을 사용할 때, LangGraph Cloud에 대한 API 호출이 완료된 후 서비스에서 무언가를 업데이트하고 싶을 경우입니다. 이를 위해서는 POST 요청을 받을 수 있는 엔드포인트를 노출해야 하며, "webhook" 매개변수에 API 요청으로 전달해야 합니다.

현재 SDK는 이 엔드포인트를 노출하지 않았지만, curl 명령어를 통해 접근할 수 있습니다.

다음 엔드포인트가 `webhook`을 매개변수로 허용합니다:

- 실행 생성 -> POST /thread/{thread_id}/runs
- 스레드 크론 생성 -> POST /thread/{thread_id}/runs/crons
- 스트림 실행 -> POST /thread/{thread_id}/runs/stream
- 실행 대기 -> POST /thread/{thread_id}/runs/wait
- 크론 생성 -> POST /runs/crons
- 무상태 스트림 실행 -> POST /runs/stream
- 무상태 실행 대기 -> POST /runs/wait

이번 예제에서는 스트리밍 실행 후 웹훅을 호출하는 방법을 보여줍니다.

## 설정

먼저, 우리 도우미와 스레드를 설정해보겠습니다:

=== "Python"

```python
from langgraph_sdk import get_client

client = get_client(url=<DEPLOYMENT_URL>)
# "agent"라는 이름으로 배포된 그래프 사용
assistant_id = "agent"
# 스레드 생성
thread = await client.threads.create()
print(thread)
```

=== "Javascript"

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
// "agent"라는 이름으로 배포된 그래프 사용
const assistantID = "agent";
// 스레드 생성
const thread = await client.threads.create();
console.log(thread);
```

=== "CURL"

```bash
curl --request POST \
    --url <DEPLOYMENT_URL>/assistants/search \
    --header 'Content-Type: application/json' \
    --data '{
        "limit": 10,
        "offset": 0
    }' | jq -c 'map(select(.config == null or .config == {})) | .[0]' && \
curl --request POST \
    --url <DEPLOYMENT_URL>/threads \
    --header 'Content-Type: application/json' \
    --data '{}'
```

출력:

```json
{
    'thread_id': '9dde5490-2b67-47c8-aa14-4bfec88af217', 
    'created_at': '2024-08-30T23:07:38.242730+00:00', 
    'updated_at': '2024-08-30T23:07:38.242730+00:00', 
    'metadata': {}, 
    'status': 'idle', 
    'config': {}, 
    'values': None
}
```

## 웹훅을 사용한 그래프 활용

웹훅을 사용하여 실행을 호출하기 위해, 실행을 생성할 때 원하는 엔드포인트와 함께 `webhook` 매개변수를 지정합니다. 웹훅 요청은 실행이 종료된 후에 트리거됩니다.

예를 들어 `https://my-server.app/my-webhook-endpoint`에서 요청을 받을 수 있다면, 이를 `stream`에 전달할 수 있습니다:

=== "Python"

```python
# 입력 생성
input = { "messages": [{ "role": "user", "content": "Hello!" }] }

async for chunk in client.runs.stream(
    thread_id=thread["thread_id"],
    assistant_id=assistant_id,
    input=input,
    stream_mode="events",
    webhook="https://my-server.app/my-webhook-endpoint"
):
    # 스트림 출력으로 무언가 수행
    pass
```

=== "Javascript"

    ```js
    // 입력 생성
    const input = { messages: [{ role: "human", content: "안녕하세요!" }] };

    // 스트림 이벤트
    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input: input,
        webhook: "https://my-server.app/my-webhook-endpoint"
      }
    );
    for await (const chunk of streamResponse) {
      // 스트림 출력으로 무언가를 수행합니다
    }
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <ASSISTANT_ID>,
            "input" : {"messages":[{"role": "user", "content": "안녕하세요!"}]},
            "webhook": "https://my-server.app/my-webhook-endpoint"
        }'
    ```

`my-webhook-endpoint`에 전송되는 페이로드의 스키마는 [run](../../concepts/langgraph_server.md/#runs)입니다. 자세한 내용은 [API 참조](https://langchain-ai.github.io/langgraph/cloud/reference/api/api_ref.html#model/run)를 참조하세요. 실행 입력, 구성 등이 `kwargs` 필드에 포함된다는 점에 유의하세요.

### 웹훅 요청 서명하기

웹훅 요청에 서명하려면 웹훅 URL에 토큰 매개변수를 지정할 수 있습니다. 예를 들어,
```
https://my-server.app/my-webhook-endpoint?token=...
```

서버는 요청의 매개변수에서 토큰을 추출한 다음 페이로드를 처리하기 전에 이를 검증해야 합니다.
