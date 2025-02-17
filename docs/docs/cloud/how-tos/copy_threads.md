_한국어로 기계번역됨_

# 스레드 복사하기

기존 스레드의 기록을 유지하고 원래 스레드에 영향을 주지 않는 독립적인 실행을 생성하기 위해 기존 스레드를 복사(즉, "포크")하고 싶을 수 있습니다. 이 가이드는 이를 수행하는 방법을 보여줍니다.

## 설정

이 코드는 복사할 스레드가 이미 존재한다고 가정합니다. 스레드가 무엇인지에 대한 내용은 [여기](../../concepts/langgraph_server.md#threads)에서 읽을 수 있으며, 스레드에서 실행을 스트리밍하는 방법은 [이 사용 안내서](../../how-tos/index.md#streaming_1)에서 배울 수 있습니다.

### SDK 초기화

먼저, 호스팅된 그래프와 통신할 수 있도록 클라이언트를 설정해야 합니다:

=== "Python"

```python
from langgraph_sdk import get_client
client = get_client(url="<DEPLOYMENT_URL>")
assistant_id = "agent"
thread = await client.threads.create()
```

=== "Javascript"

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>" });
const assistantId = "agent";
const thread = await client.threads.create();
```

=== "CURL"

```bash
curl --request POST \
  --url <DEPLOYMENT_URL>/threads \
  --header 'Content-Type: application/json' \
  --data '{
    "metadata": {}
  }'
```

## 스레드 복사하기

다음 코드는 복사할 스레드가 이미 존재한다고 가정합니다.

스레드를 복사하면 기존 스레드와 동일한 기록을 가진 새로운 스레드가 생성되며, 이후 실행을 계속 진행할 수 있습니다.

### 복사 생성

=== "Python"

```python
copied_thread = await client.threads.copy(<THREAD_ID>)
```

=== "Javascript"

```js
let copiedThread = await client.threads.copy(<THREAD_ID>);
```

=== "CURL"

```bash
curl --request POST --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/copy \
--header 'Content-Type: application/json'
```

### 복사 확인

이전 스레드의 기록이 정확하게 복사되었는지 확인할 수 있습니다:

=== "Python"

```python
def remove_thread_id(d):
  if 'metadata' in d and 'thread_id' in d['metadata']:
      del d['metadata']['thread_id']
  return d

original_thread_history = list(map(remove_thread_id, await client.threads.get_history(<THREAD_ID>)))
copied_thread_history = list(map(remove_thread_id, await client.threads.get_history(copied_thread['thread_id'])))

# 두 기록 비교하기
assert original_thread_history == copied_thread_history
# 여기까지 온다면 단언이 통과했습니다!
print("기록이 동일합니다.")
```

=== "Javascript"

    ```js
    function removeThreadId(d) {
      if (d.metadata && d.metadata.thread_id) {
        delete d.metadata.thread_id;
      }
      return d;
    }

    // `client.threads.getHistory(threadId)`가 비동기 함수이며 dict 목록을 반환한다고 가정
    async function compareThreadHistories(threadId, copiedThreadId) {
      const originalThreadHistory = (await client.threads.getHistory(threadId)).map(removeThreadId);
      const copiedThreadHistory = (await client.threads.getHistory(copiedThreadId)).map(removeThreadId);

      // 두 히스토리 비교
      console.assert(JSON.stringify(originalThreadHistory) === JSON.stringify(copiedThreadHistory));
      // 여기까지 오면 assertion이 통과한 것임!
      console.log("두 히스토리는 같습니다.");
    }

    // 사용 예시
    compareThreadHistories(<THREAD_ID>, copiedThread.thread_id);
    ```

=== "CURL"

    ```bash
    if diff <(
        curl --request GET --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/history | jq -S 'map(del(.metadata.thread_id))'
    ) <(
        curl --request GET --url <DEPLOYMENT_URL>/threads/<COPIED_THREAD_ID>/history | jq -S 'map(del(.metadata.thread_id))'
    ) >/dev/null; then
        echo "두 히스토리는 같습니다."
    else
        echo "두 히스토리는 다릅니다."
    fi
    ```

출력:

    두 히스토리는 같습니다.