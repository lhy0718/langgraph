_한국어로 기계번역됨_

# 큐에 추가하기

이 가이드는 더블 텍스트가 무엇인지에 대한 지식을 가정하며, 이에 대한 정보는 [더블 텍스트 개념 가이드](../../concepts/double_texting.md)에서 확인할 수 있습니다.

이 가이드는 더블 텍스트를 위한 `enqueue` 옵션을 다루며, 이는 중단되는 내용을 큐에 추가하고 클라이언트가 수신한 순서대로 실행합니다. 아래는 `enqueue` 옵션을 사용하는 간단한 예입니다.

## 설정

먼저, JS와 CURL 모델 출력을 인쇄하기 위한 도움 함수 하나를 정의하겠습니다 (Python을 사용하는 경우 이 부분은 건너뛰어도 됩니다):

=== "Javascript"

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
    # 이를 pretty_print.sh라는 파일에 저장합니다.
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

그런 다음, 필요한 패키지를 가져오고 클라이언트, 조수 및 스레드를 인스턴스화하겠습니다.

=== "Python"

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

## 실행 생성

이제 두 개의 실행을 시작하겠습니다. 두 번째 실행이 첫 번째 실행을 중단하며 "enqueue" 멀티태스킹 전략을 사용합니다.

=== "Python"

    ```python
    첫 번째 실행 = await client.runs.create(
        thread["thread_id"],
        assistant_id,
        input={"messages": [{"role": "user", "content": "샌프란시스코 날씨 어때?"}]},
    )
    두 번째 실행 = await client.runs.create(
        thread["thread_id"],
        assistant_id,
        input={"messages": [{"role": "user", "content": "뉴욕 날씨 어때?"}]},
        multitask_strategy="enqueue",
    )
    ```

=== "Javascript"

    ```js
    const 첫 번째 실행 = await client.runs.create(
      thread["thread_id"],
      assistantId,
      input={"messages": [{"role": "user", "content": "샌프란시스코 날씨 어때?"}]},
    )

    const 두 번째 실행 = await client.runs.create(
      thread["thread_id"],
      assistantId,
      input={"messages": [{"role": "user", "content": "뉴욕 날씨 어때?"}]},
      multitask_strategy="enqueue",
    )
    ```

=== "CURL"

    ```bash
    curl --request POST \
    --url <장배치_URL>>/threads/<THREAD_ID>/runs \
    --header 'Content-Type: application/json' \
    --data "{
      \"assistant_id\": \"에이전트\",
      \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코 날씨 어때?\"}]},
    }" && curl --request POST \
    --url <장배치_URL>>/threads/<THREAD_ID>/runs \
    --header 'Content-Type: application/json' \
    --data "{
      \"assistant_id\": \"에이전트\",
      \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"뉴욕 날씨 어때?\"}]},
      \"multitask_strategy\": \"enqueue\"
    }"
    ```

## 실행 결과 보기

스레드에 두 실행의 데이터가 있는지 확인하세요:

=== "Python"

    ```python
    # 두 번째 실행이 완료될 때까지 대기
    await client.runs.join(thread["thread_id"], 두 번째 실행["run_id"])

    상태 = await client.threads.get_state(thread["thread_id"])

    for m in convert_to_messages(상태["values"]["messages"]):
        m.pretty_print()
    ```

=== "Javascript"

    ```js
    await client.runs.join(thread["thread_id"], 두 번째 실행["run_id"]);

    const 상태 = await client.threads.getState(thread["thread_id"]);

    for (const m of 상태["values"]["messages"]) {
      prettyPrint(m);
    }
    ```

=== "CURL"

    ```bash
    source pretty_print.sh && curl --request GET \
    --url <장배치_URL>/threads/<THREAD_ID>/runs/<RUN_ID>/join && \
    curl --request GET --url <장배치_URL>/threads/<THREAD_ID>/state | \
    jq -c '.values.messages[]' | while read -r element; do
        type=$(echo "$element" | jq -r '.type')
        content=$(echo "$element" | jq -r '.content | if type == "array" then tostring else . end')
        pretty_print "$type" "$content"
    done
    ```

출력:
    
    ================================ 인간 메시지 =================================
    
    샌프란시스코 날씨 어때?
    ================================== AI 메시지 ==================================
    
    [{'id': 'toolu_01Dez1sJre4oA2Y7NsKJV6VT', 'input': {'query': '샌프란시스코 날씨'}, 'name': 'tavily_search_results_json', 'type': 'tool_use'}]
    도구 호출:
      현재 샌프란시스코의 날씨 조건은 다음과 같습니다:

- 기온: 57°F (14°C)
- 상황: 대체로 맑음
- 바람: WSW 10 mph
- 습도: 72%

다음 며칠간의 예보는 부분적으로 맑은 날씨와 기온이 50대 후반에서 60대 중반(Fahrenheit, 14-18°C)으로, 최저 기온은 40대 후반에서 50대 초반(Fahrenheit, 9-11°C)으로 예상됩니다. 이 시기에 샌프란시스코에서 흔히 볼 수 있는 온난하고 건조한 날씨입니다.

AccuWeather 예보의 주요 세부 사항은 다음과 같습니다:

- 오늘: 대체로 맑음, 최고 기온 62°F (17°C)
- 오늘 밤: 부분적으로 흐림, 최저 기온 49°F (9°C) 
- 내일: 부분적으로 맑음, 최고 기온 59°F (15°C)
- 토요일: 대체로 맑음, 최고 기온 64°F (18°C)
- 일요일: 부분적으로 맑음, 최고 기온 61°F (16°C)

결론적으로, 샌프란시스코는 다음 며칠간 계절에 알맞은 봄 날씨가 예상되며, 햇빛과 구름이 섞인 날씨 및 기온은 밤에는 40대 후반, 낮에는 60도 초반으로 변동할 것으로 보입니다. 비 예보가 없는 전형적인 건조한 날씨입니다. 

현재 뉴욕시의 날씨 조건과 예보는 다음과 같습니다:

- 기온: 85°F (29°C)
- 상황: 맑음
- 바람: SSE에서 2 mph (4 km/h)
- 습도: 63%
- 체감 온도: 85°F (30°C)

예보에 따르면, 향후 며칠 동안 맑고 따뜻한 날씨가 지속될 것으로 보입니다:

- 오늘: 맑음, 최고 기온 85°F (29°C)
- 오늘 밤: 맑음, 최저 기온 68°F (20°C)
- 내일: 맑음, 최고 기온 88°F (31°C) 
- 목요일: 대체로 맑음, 최고 기온 90°F (32°C)
- 금요일: 부분적으로 흐림, 최고 기온 87°F (31°C)

따라서 뉴욕시는 아름다운 맑은 날씨와 적정 온도가 지속되고 있으며, 80도 중반(Fahrenheit, 약 30°C)의 따뜻한 날씨가 이어질 것으로 보입니다. 습도는 60% 범위로 적당합니다. 전반적으로 외출하기에 이상적인 늦봄/초여름 날씨가 예상됩니다.

