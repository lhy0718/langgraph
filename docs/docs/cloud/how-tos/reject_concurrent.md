_한국어로 기계번역됨_

# 거부

이 가이드는 더블 텍스트란 무엇인지에 대한 지식을 전제로 하며, 이 내용은 [더블 텍스트 개념 가이드](../../concepts/double_texting.md)에서 배울 수 있습니다.

이 가이드는 더블 텍스트를 위한 `reject` 옵션을 다루며, 이는 그래프의 새로운 실행을 오류를 발생시켜 거부하고, 원래의 실행이 완료될 때까지 계속 진행하게 합니다. 아래는 `reject` 옵션을 사용하는 간단한 예제입니다.

## 설정

먼저, JS와 CURL 모델 출력을 인쇄하는 간단한 도우미 함수를 정의합니다(파이썬을 사용하는 경우 이 단계를 건너뛰어도 됩니다).

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
# 이 내용을 pretty_print.sh라는 파일에 저장하세요.
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

이제 필요한 패키지를 가져오고 클라이언트, 어시스턴트 및 스레드를 인스턴스화해보겠습니다.

=== "파이썬"

```python
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

이제 스레드를 실행하고 "거부" 옵션으로 두 번째 실행을 시도해보겠습니다. 이미 실행이 시작되었으므로 실패해야 합니다: 

=== "파이썬"

    ```bash
# 원본 실행이 완료될 때까지 대기
await client.runs.join(thread["thread_id"], run["run_id"])

상태 = await client.threads.get_state(thread["thread_id"])

for m in convert_to_messages(상태["values"]["messages"]):
    m.pretty_print()
```

=== "자바스크립트"

```js
await client.runs.join(thread["thread_id"], run["run_id"]);

const 상태 = await client.threads.getState(thread["thread_id"]);

for (const m of 상태["values"]["messages"]) {
  prettyPrint(m);
}
```

=== "CURL"

    ```bash
    source pretty_print.sh && curl --request GET \
    --url <배포_URL>/threads/<스레드_ID>/runs/<실행_ID>/join && \
    curl --request GET --url <배포_URL>/threads/<스레드_ID>/state | \
    jq -c '.values.messages[]' | while read -r element; do
        type=$(echo "$element" | jq -r '.type')
        content=$(echo "$element" | jq -r '.content | if type == "array" then tostring else . end')
        pretty_print "$type" "$content"
    done
    ```

출력:

    ================================ 인간 메시지 =================================
    
    샌프란시스코의 날씨는 어떤가요?
    ================================== AI 메시지 ==================================
    
    [{'id': 'toolu_01CyewEifV2Kmi7EFKHbMDr1', 'input': {'query': '샌프란시스코의 날씨'}, 'name': 'tavily_search_results_json', 'type': 'tool_use'}]
    도구 호출:
      tavily_search_results_json (toolu_01CyewEifV2Kmi7EFKHbMDr1)
     호출 ID: toolu_01CyewEifV2Kmi7EFKHbMDr1
      인수:
        쿼리: 샌프란시스코의 날씨
    ================================= 도구 메시지 =================================
    이름: tavily_search_results_json
    
    [{"url": "https://www.accuweather.com/en/us/san-francisco/94103/june-weather/347629", "content": "샌프란시스코, CA의 월별 날씨 예보를 확인하세요. 여기에는 매일의 최고/최저 온도, 역사적 평균이 포함되어 있어 계획을 세우는 데 도움이 됩니다."}]
    ================================== AI 메시지 ==================================
    
    Tavily의 검색 결과에 따르면 샌프란시스코의 현재 날씨는 다음과 같습니다:
    
    6월의 샌프란시스코의 평균 최고 기온은 약 65°F (18°C)이며, 평균 최저 기온은 약 54°F (12°C)입니다. 6월은 여름철에 종종 도시를 덮는 해양 안개 층 때문에 샌프란시스코에서 가장 서늘하고 안개가 많이 끼는 달 중 하나입니다.
    
    6월의 샌프란시스코 전형적인 날씨에 대한 몇 가지 주요 사항:
    
    - 온화한 기온, 최고 기온은 60°F대, 최저 기온은 50°F대
    - 아침에 안개가 자욱하다가 오후에는 맑아지는 경향
    - 강수량이 거의 없으며, 6월은 건조한 계절에 해당
    - 태평양에서 불어오는 바람이 있는 시원한 날씨
    - 변화하는 날씨에 대비해 겹겹이 옷을 입는 것이 추천됨
    
    요약하자면, 올해 이 시기에 샌프란시스코에서는 온화하고 안개가 자욱한 아침이 맑고 서늘한 오후로 이어질 것으로 예상할 수 있습니다. 해양 층은 샌프란시스코 6월의 기온을 캘리포니아 다른 지역에 비해 온화하게 유지합니다.

