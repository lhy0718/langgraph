_한국어로 기계번역됨_

# 롤백

이 가이드는 더블 텍스트가 무엇인지에 대한 지식을 가정하며, 이에 대한 정보는 [더블 텍스트 개념 가이드](../../concepts/double_texting.md)에서 확인할 수 있습니다.

이 가이드는 더블 텍스트와 함께 새로운 그래프 실행을 시작하고 이전 실행을 중단하는 `rollback` 옵션을 다룹니다. 이 옵션은 `interrupt` 옵션과 매우 유사하지만, 이 경우 첫 번째 실행은 데이터베이스에서 완전히 삭제되며 다시 시작할 수 없습니다. 아래는 `rollback` 옵션 사용에 대한 간단한 예시입니다.

## 설정

먼저, JS 및 CURL 모델 출력 결과를 인쇄하는 간단한 헬퍼 함수를 정의하겠습니다 (Python을 사용하는 경우 이 단계는 건너뛸 수 있습니다):

=== "자바스크립트"

```js
function prettyPrint(m) {
  const padded = " " + m['type'] + " ";
  const sepLen = Math.floor((80 - padded.length) / 2);
  const sep = "=".repeat(sepLen);
  const secondSep = sep + (padded.length % 2 ? "=" : "");

  console.log(`${sep}${padded}${secondSep}`);
  console.log("\n\n");
  console.log(m.content);
}
```

=== "CURL"

```bash
# pretty_print.sh라는 이름의 파일에 이 내용을 넣으세요
pretty_print() {
  local type="$1"
  local content="$2"
  local padded=" $type "
  local total_width=80
  local sep_len=$(( (total_width - ${#padded}) / 2 ))
  local sep=$(printf '=%.0s' $(eval "echo {1.."${sep_len}"}"))
  local second_sep=$sep
  if (( (total_width - ${#padded}) % 2 )); then
    second_sep="${second_sep}="
  fi

  echo "${sep}${padded}${second_sep}"
  echo
  echo "$content"
}
```

이제 필요한 패키지를 가져오고 클라이언트, 어시스턴트, 및 스레드를 인스턴스화하겠습니다.

=== "파이썬"

```python
import asyncio

import httpx
from langchain_core.messages import convert_to_messages
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

## 실행 생성

이제 롤백 매개변수로 설정된 스레드를 실행해 보겠습니다.

=== "파이썬"

    ```python
# 첫 번째 실행은 롤백됩니다.
rolled_back_run = await client.runs.create(
    thread["thread_id"],
    assistant_id,
    input={"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]},
)
run = await client.runs.create(
    thread["thread_id"],
    assistant_id,
    input={"messages": [{"role": "user", "content": "뉴욕의 날씨는 어때?"}]},
    multitask_strategy="rollback",
)
# 두 번째 실행이 완료될 때까지 대기합니다.
await client.runs.join(thread["thread_id"], run["run_id"])
```

=== "Javascript"

```js
// 첫 번째 실행은 중단됩니다.
let rolledBackRun = await client.runs.create(
    thread["thread_id"],
    assistantId,
    { input: { messages: [{ role: "human", content: "샌프란시스코의 날씨는 어때?" }] } }
);

let run = await client.runs.create(
    thread["thread_id"],
    assistant_id,
    { 
        input: { messages: [{ role: "human", content: "뉴욕의 날씨는 어때?" }] },
        multitaskStrategy: "rollback" 
    }
);

// 두 번째 실행이 완료될 때까지 대기합니다.
await client.runs.join(thread["thread_id"], run["run_id"]);
```

=== "CURL"

```bash
curl --request POST \
--url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs \
--header 'Content-Type: application/json' \
--data "{
  \"assistant_id\": \"agent\",
  \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어때?\"}]},
}" && curl --request POST \
--url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs \
--header 'Content-Type: application/json' \
--data "{
  \"assistant_id\": \"agent\",
  \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"뉴욕의 날씨는 어때?\"}]},
  \"multitask_strategy\": \"rollback\"
}" && curl --request GET \
--url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/<RUN_ID>/join
```

## 실행 결과 보기

스레드에 두 번째 실행에서만 데이터가 있음을 확인할 수 있습니다.

=== "Python"

```python
state = await client.threads.get_state(thread["thread_id"])

for m in convert_to_messages(state["values"]["messages"]):
    m.pretty_print()
```

=== "Javascript"

```js
const state = await client.threads.getState(thread["thread_id"]);

for (const m of state['values']['messages']) {
    prettyPrint(m);
}
```

=== "CURL"

```bash
source pretty_print.sh && curl --request GET \
--url <DEPLOYMENT_URL>/threads/<THREAD_ID>/state | \
jq -c '.values.messages[]' | while read -r element; do
    type=$(echo "$element" | jq -r '.type')
    content=$(echo "$element" | jq -r '.content | if type == "array" then tostring else . end')
    pretty_print "$type" "$content"
done
```

출력:

```plaintext
======================== 인간 메시지 ================================
샌프란시스코의 날씨는 어때?
```
    현재 뉴욕시의 날씨는 맑고 기온은 약 29°C (85°F)입니다. 바람은 남남동에서 오는 약 2-3 mph의 가벼운 바람이 불고 있습니다. 전반적으로 NYC에서는 화창한 여름 날씨를 보이고 있습니다. 

원래의 롤백된 실행이 삭제되었는지 확인합니다.

=== "파이썬"

```python
try:
    await client.runs.get(thread["thread_id"], rolled_back_run["run_id"])
except httpx.HTTPStatusError as _:
    print("원래 실행이 제대로 삭제되었습니다.")
```

=== "자바스크립트"

```js
try {
  await client.runs.get(thread["thread_id"], rolledBackRun["run_id"]);
} catch (e) {
  console.log("원래 실행이 제대로 삭제되었습니다.");
}
```

출력:

원래 실행이 제대로 삭제되었습니다.

