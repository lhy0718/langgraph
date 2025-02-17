_한국어로 기계번역됨_

# 중단

이 가이드는 더블 텍스트의 개념에 대한 지식을 전제로 하며, 더블 텍스트에 대한 개념 가이드는 [여기](../../concepts/double_texting.md)에서 확인할 수 있습니다.

이 가이드는 이전 그래프 실행을 중단하고 더블 텍스트로 새로운 실행을 시작하는 '중단' 옵션에 대해 다룹니다. 이 옵션은 첫 번째 실행을 삭제하지 않고 데이터베이스에 유지하며 상태를 '중단됨'으로 설정합니다. 아래는 '중단' 옵션을 사용하는 간단한 예시입니다.

## 설정

먼저, JS와 CURL 모델 출력을 인쇄하기 위한 간단한 도우미 함수를 정의하겠습니다 (Python을 사용하는 경우 생략 가능):

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
# 이 코드를 pretty_print.sh라는 파일에 저장하세요
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

이제 필요한 패키지를 가져오고 클라이언트, 어시스턴트, 스레드를 인스턴스화하겠습니다.

=== "파이썬"

```python
import asyncio

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

이제 두 개의 실행을 시작하고 두 번째 실행이 완료될 때까지 기다리겠습니다:

=== "파이썬"

    ```python
    # 첫 번째 실행이 중단됩니다.
    interrupted_run = await client.runs.create(
        thread["thread_id"],
        assistant_id,
        input={"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때요?"}]},
    )
    # 첫 번째 실행에서 부분 출력을 얻기 위해 잠시 기다립니다.
    await asyncio.sleep(2)
    run = await client.runs.create(
        thread["thread_id"],
        assistant_id,
        input={"messages": [{"role": "user", "content": "뉴욕의 날씨는 어때요?"}]},
        multitask_strategy="interrupt",
    )
    # 두 번째 실행이 완료될 때까지 기다립니다.
    await client.runs.join(thread["thread_id"], run["run_id"])
```

=== "Javascript"

```js
// 첫 번째 실행이 중단됩니다.
let interruptedRun = await client.runs.create(
  thread["thread_id"],
  assistantId,
  { input: { messages: [{ role: "human", content: "샌프란시스코의 날씨는 어때요?" }] } }
);
// 첫 번째 실행에서 부분 출력을 얻기 위해 잠시 기다립니다.
await new Promise(resolve => setTimeout(resolve, 2000)); 

let run = await client.runs.create(
  thread["thread_id"],
  assistantId,
  { 
    input: { messages: [{ role: "human", content: "뉴욕의 날씨는 어때요?" }] },
    multitaskStrategy: "interrupt" 
  }
);

// 두 번째 실행이 완료될 때까지 기다립니다.
await client.runs.join(thread["thread_id"], run["run_id"]);
```

=== "CURL"

```bash
curl --request POST \
--url <DEPLOY<ENT_URL>>/threads/<THREAD_ID>/runs \
--header 'Content-Type: application/json' \
--data "{
  \"assistant_id\": \"agent\",
  \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어때요?\"}]},
}" && sleep 2 && curl --request POST \
--url <DEPLOY<ENT_URL>>/threads/<THREAD_ID>/runs \
--header 'Content-Type: application/json' \
--data "{
  \"assistant_id\": \"agent\",
  \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"뉴욕의 날씨는 어때요?\"}]},
  \"multitask_strategy\": \"interrupt\"
}" && curl --request GET \
--url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/<RUN_ID>/join
```

## 실행 결과 보기

우리는 스레드가 첫 번째 실행의 부분 데이터와 두 번째 실행의 데이터를 가지고 있음을 알 수 있습니다.

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

=============================== 인간 메시지 =================================

샌프란시스코 날씨 어때?
================================== AI 메시지 ==================================

[{'id': 'toolu_01MjNtVJwEcpujRGrf3x6Pih', 'input': {'query': '샌프란시스코 날씨'}, 'name': 'tavily_search_results_json', 'type': 'tool_use'}]
도구 호출:
  tavily_search_results_json (toolu_01MjNtVJwEcpujRGrf3x6Pih)
 호출 ID: toolu_01MjNtVJwEcpujRGrf3x6Pih
  인수:
    query: 샌프란시스코 날씨
================================= 도구 메시지 =================================
이름: tavily_search_results_json

[{"url": "https://www.wunderground.com/hourly/us/ca/san-francisco/KCASANFR2002/date/2024-6-18", "content": "최고 64도. 바람은 시속 10~20마일. 가끔 구름이 있음. 최저 49도. 바람은 시속 10~20마일. 샌프란시스코 날씨 예보. Weather Underground는 지역 및 장기 날씨 ..."}]
=============================== 인간 메시지 =================================

뉴욕시 날씨 어때?
================================== AI 메시지 ==================================

[{'id': 'toolu_01KtE1m1ifPLQAx4fQLyZL9Q', 'input': {'query': '뉴욕시 날씨'}, 'name': 'tavily_search_results_json', 'type': 'tool_use'}]
도구 호출:
  tavily_search_results_json (toolu_01KtE1m1ifPLQAx4fQLyZL9Q)
 호출 ID: toolu_01KtE1m1ifPLQAx4fQLyZL9Q
  인수:
    query: 뉴욕시 날씨
================================= 도구 메시지 =================================
이름: tavily_search_results_json

[{"url": "https://www.accuweather.com/en/us/new-york/10021/june-weather/349727", "content": "뉴욕, NY의 월간 날씨 예보를 받아보세요. 일일 최고/최저 기온 및 역사적 평균을 포함하여 미리 계획하는 데 도움이 됩니다."}]
================================== AI 메시지 ==================================

검색 결과는 뉴욕시의 날씨 예보와 정보를 제공합니다. AccuWeather의 상위 결과에 기반하여 뉴욕시의 날씨에 대한 주요 세부 정보는 다음과 같습니다:

- 이는 뉴욕시의 6월 월간 날씨 예보입니다.
- 미리 계획하는 데 도움이 되는 일일 최고 및 최저 기온을 포함합니다.
- NYC의 6월에 대한 역사적 평균도 참조 점으로 제공됩니다.
- 강수 확률, 습도, 바람 등과 함께 보다 자세한 일간 또는 시간대별 예보는 AccuWeather 페이지를 방문하여 확인할 수 있습니다.

따라서 요약하자면, 이 검색은 뉴욕시에서 다음 달 동안 예상되는 날씨 조건에 대한 유용한 개요를 제공하여 여행하거나 계획을 세울 때 준비해야 할 사항을 안내합니다. 추가 세부 정보가 필요하시면 알려주세요!

원래의 중단된 실행이 중단되었는지 확인하십시오.

=== "파이썬"

```python
print((await client.runs.get(thread["thread_id"], interrupted_run["run_id"]))["status"])
```

=== "자바스크립트"

```js
console.log((await client.runs.get(thread['thread_id'], interruptedRun["run_id"]))["status"])
```

출력:

'interrupted'

