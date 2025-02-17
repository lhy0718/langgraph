_한국어로 기계번역됨_

# 크론 잡

때때로 사용자의 상호작용에 따라 그래프를 실행하고 싶지 않을 때가 있습니다. 대신 그래프를 일정에 따라 실행되도록 예약하고 싶을 수 있습니다. 예를 들어 팀의 할 일 목록을 주간 이메일로 작성하고 보내고 싶을 때입니다. LangGraph Cloud는 `Crons` 클라이언트를 사용하여 자신의 스크립트를 작성할 필요 없이 이를 가능하게 합니다. 그래프 작업을 예약하려면 그래프를 언제 실행할지 클라이언트에 알리기 위해 [크론 표현식](https://crontab.cronhub.io/)을 전달해야 합니다. `Cron` 잡은 백그라운드에서 실행되며 그래프의 정상적인 호출에 간섭하지 않습니다.

## 설정

먼저 SDK 클라이언트, 어시스턴트 및 스레드를 설정해 보겠습니다:

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
const assistantId = "agent";
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
    }' | jq -c 'map(select(.config == null or .config == {})) | .[0].graph_id' && \
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

## 스레드에서의 크론 잡 

특정 스레드와 연결된 크론 잡을 만들려면 다음과 같이 작성할 수 있습니다:

=== "Python"

```python
# 매일 15:27 (오후 3:27)에 실행되도록 작업을 예약합니다
cron_job = await client.crons.create_for_thread(
    thread["thread_id"],
    assistant_id,
    schedule="27 15 * * *",
    input={"messages": [{"role": "user", "content": "지금 몇 시죠?"}]},
)
```

=== "Javascript"

```js
// 매일 15:27 (오후 3:27)에 실행되도록 작업을 예약합니다
const cronJob = await client.crons.create_for_thread(
  thread["thread_id"],
  assistantId,
  {
    schedule: "27 15 * * *",
    input: { messages: [{ role: "user", content: "지금 몇 시죠?" }] }
  }
);
```

=== "CURL"

    ```bash
    curl --request POST \
        --url <배포_URL>/threads/<THREAD_ID>/runs/crons \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <ASSISTANT_ID>,
        }'
    ```

`Cron` 작업이 더 이상 유용하지 않을 경우 삭제하는 것이 **매우** 중요합니다. 그렇지 않으면 원치 않는 API 요금이 발생할 수 있습니다! 다음 코드를 사용하여 `Cron` 작업을 삭제할 수 있습니다:

=== "Python"

    ```python
    await client.crons.delete(cron_job["cron_id"])
    ```

=== "Javascript"

    ```js
    await client.crons.delete(cronJob["cron_id"]);
    ```

=== "CURL"

    ```bash
    curl --request DELETE \
        --url <배포_URL>/runs/crons/<CRON_ID>
    ```

## Cron 작업 무상태

다음 코드를 사용하여 무상태 cron 작업을 생성할 수도 있습니다:

=== "Python"

    ```python
    # 이 코드는 매일 15:27(오후 3:27)에 작업을 실행하도록 예약합니다
    cron_job_stateless = await client.crons.create(
        assistant_id,
        schedule="27 15 * * *",
        input={"messages": [{"role": "user", "content": "지금 몇 시인가요?"}]},
    )
    ```

=== "Javascript"

    ```js
    // 이 코드는 매일 15:27(오후 3:27)에 작업을 실행하도록 예약합니다
    const cronJobStateless = await client.crons.create(
      assistantId,
      {
        schedule: "27 15 * * *",
        input: { messages: [{ role: "user", content: "지금 몇 시인가요?" }] }
      }
    );
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <배포_URL>/runs/crons \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <ASSISTANT_ID>,
        }'
    ```

다시 한 번, 작업이 완료되면 삭제하는 것을 잊지 마세요!

=== "Python"

    ```python
    await client.crons.delete(cron_job_stateless["cron_id"])
    ```

=== "Javascript"

    ```js
    await client.crons.delete(cronJobStateless["cron_id"]);
    ```

=== "CURL"

    ```bash
    curl --request DELETE \
        --url <배포_URL>/runs/crons/<CRON_ID>
    ```
