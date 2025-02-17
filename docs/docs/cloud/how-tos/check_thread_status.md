_한국어로 기계번역됨_

# 스레드 상태 확인하기

## 설정

시작하기 위해, 그래프를 호스팅하고 있는 URL로 클라이언트를 설정할 수 있습니다:

### SDK 초기화

먼저, 호스팅된 그래프와 소통할 수 있도록 클라이언트를 설정해야 합니다:

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

## 유휴 스레드 찾기

다음 명령을 사용하여 모든 실행이 완료된 유휴 상태의 스레드를 찾을 수 있습니다:

=== "Python"

```python
print(await client.threads.search(status="idle",limit=1))
```

=== "Javascript"

```js
console.log(await client.threads.search({ status: "idle", limit: 1 }));
```

=== "CURL"

```bash
curl --request POST \  
--url <DEPLOYMENT_URL>/threads/search \
--header 'Content-Type: application/json' \
--data '{"status": "idle", "limit": 1}'
```

출력:

```json
[{'thread_id': 'cacf79bb-4248-4d01-aabc-938dbd60ed2c',
'created_at': '2024-08-14T17:36:38.921660+00:00',
'updated_at': '2024-08-14T17:36:38.921660+00:00',
'metadata': {'graph_id': 'agent'},
'status': 'idle',
'config': {'configurable': {}}}]
```

## 중단된 스레드 찾기

다음 명령을 사용하여 실행 도중 중단된 스레드를 찾을 수 있습니다. 이는 실행이 끝나기 전에 오류가 발생했거나, 인간 개입 포인트에 도달해 실행이 계속되기를 기다리는 경우를 의미할 수 있습니다:

=== "Python"

```python
print(await client.threads.search(status="interrupted",limit=1))
```

=== "Javascript"

```js
console.log(await client.threads.search({ status: "interrupted", limit: 1 }));
```

=== "CURL"

```bash
curl --request POST \  
--url <DEPLOYMENT_URL>/threads/search \
--header 'Content-Type: application/json' \
--data '{"status": "interrupted", "limit": 1}'
```

출력:

    [{'thread_id': '0d282b22-bbd5-4d95-9c61-04dcc2e302a5',
    'created_at': '2024-08-14T17:41:50.235455+00:00',
    'updated_at': '2024-08-14T17:41:50.235455+00:00',
    'metadata': {'graph_id': 'agent'},
    'status': '중단됨',
    'config': {'configurable': {}}}]
    
## 바쁜 스레드 찾기

다음 명령어를 사용하여 현재 실행을 처리하고 있는 바쁜 스레드를 찾을 수 있습니다:

=== "Python"

    ```python
    print(await client.threads.search(status="busy",limit=1))
    ```

=== "Javascript"

    ```js
    console.log(await client.threads.search({ status: "busy", limit: 1 }));
    ```

=== "CURL"

    ```bash
    curl --request POST \  
    --url <DEPLOYMENT_URL>/threads/search \
    --header 'Content-Type: application/json' \
    --data '{"status": "busy", "limit": 1}'
    ```

출력:

    [{'thread_id': '0d282b22-bbd5-4d95-9c61-04dcc2e302a5',
    'created_at': '2024-08-14T17:41:50.235455+00:00',
    'updated_at': '2024-08-14T17:41:50.235455+00:00',
    'metadata': {'graph_id': 'agent'},
    'status': '바쁨',
    'config': {'configurable': {}}}]

## 특정 스레드 찾기

특정 스레드의 상태를 확인하려는 경우 몇 가지 방법으로 확인할 수 있습니다:

### ID로 찾기

ID를 저장하고 있다면 `get` 함수를 사용하여 특정 스레드의 상태를 찾을 수 있습니다.

=== "Python"

    ```python
    print((await client.threads.get(<THREAD_ID>))['status'])
    ```

=== "Javascript"

    ```js
    console.log((await client.threads.get(<THREAD_ID>)).status);
    ```

=== "CURL"

    ```bash
    curl --request GET \ 
    --url <DEPLOYMENT_URL>/threads/<THREAD_ID> \
    --header 'Content-Type: application/json' | jq -r '.status'
    ```

출력:

    '유휴'

### 메타데이터로 찾기

스레드 검색 엔드포인트는 메타데이터로 필터링할 수 있도록 하여 스레드를 정리하는 데 도움이 될 수 있습니다:

=== "Python"

    ```python
    print((await client.threads.search(metadata={"foo":"bar"},limit=1))[0]['status'])
    ```

=== "Javascript"

    ```js
    console.log((await client.threads.search({ metadata: { "foo": "bar" }, limit: 1 }))[0].status);
    ```

=== "CURL"

    ```bash
    curl --request POST \  
    --url <DEPLOYMENT_URL>/threads/search \
    --header 'Content-Type: application/json' \
    --data '{"metadata": {"foo":"bar"}, "limit": 1}' | jq -r '.[0].status'
    ```

'유휴'