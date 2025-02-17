_한국어로 기계번역됨_

# 디버그 이벤트 스트리밍 방법

!!! 정보 "전제 조건"
    * [스트리밍](../../concepts/streaming.md)

이 가이드는 그래프에서 디버그 이벤트를 스트리밍하는 방법(`stream_mode="debug"`)을 다룹니다. 디버그 이벤트 스트리밍은 `type` 및 `timestamp` 키를 포함하는 응답을 생성합니다. 디버그 이벤트는 그래프의 실행 중 서로 다른 단계를 나타내며, 세 가지 종류의 단계가 스트리밍됩니다:

- `checkpoint`: 그래프가 상태를 저장할 때마다 스트리밍되는 이벤트로, 이는 매 슈퍼 스텝 후에 발생합니다. 체크포인트에 대한 자세한 내용은 [여기](https://langchain-ai.github.io/langgraph/concepts/low_level/#checkpointer)를 참조하세요.
- `task`: 이 이벤트는 각 슈퍼 스텝 전에 스트리밍되며, 단일 작업에 대한 정보를 포함합니다. 각 슈퍼 스텝은 특정 노드와 입력에 해당하는 작업 목록을 실행함으로써 작동하며, 아래에서 이러한 작업의 형식을 자세히 논의할 것입니다.
- `task_result`: 각 `task` 이벤트 후에 해당 작업의 결과에 대한 정보를 포함하는 `task_result` 이벤트를 볼 수 있습니다. 이러한 이벤트의 정확한 구조에 대해 더 알아보려면 아래로 스크롤하세요.

## 설정

먼저 클라이언트와 스레드를 설정해 봅시다:

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
      --url <DEPLOYMENT_URL>/threads \
      --header 'Content-Type: application/json' \
      --data '{}'
    ```


출력:

    {
        'thread_id': 'd0cbe9ad-f11c-443a-9f6f-dca0ae5a0dd3',
        'created_at': '2024-06-21T22:10:27.696862+00:00',
        'updated_at': '2024-06-21T22:10:27.696862+00:00',
        'metadata': {},
        'status': 'idle',
        'config': {},
        'values': None
    }

## 디버그 모드에서 그래프 스트리밍

=== "Python"

    ```python
    # 입력 생성
    input = {
        "messages": [
            {
                "role": "user",
                "content": "샌프란시스코의 날씨는 어떤가요?",
            }
        ]
    }

    # 디버그 스트리밍
    async for chunk in client.runs.stream(
        thread_id=thread["thread_id"],
        assistant_id=assistant_id,
        input=input,
        stream_mode="debug",
    ):
        print(f"유형의 새로운 이벤트 수신 중: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "Javascript"

    ```js
    // 입력 생성
    const input = {
      messages: [
        {
          role: "human",
          content: "샌프란시스코의 날씨는 어때요?",
        }
      ]
    };

    // 스트림 디버그
    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input,
        streamMode: "debug"
      }
    );

    for await (const chunk of streamResponse) {
      console.log(`타입: ${chunk.event} 의 새로운 이벤트 수신 중...`);
      console.log(chunk.data);
      console.log("\n\n");
    }
```

=== "CURL"

```bash
curl --request POST \
 --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
 --header 'Content-Type: application/json' \
 --data "{
   \"assistant_id\": \"agent\",
   \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨는 어때요?\"}]},
   \"stream_mode\": [
     \"debug\"
   ]
 }" | \
 sed 's/\r$//' | \
 awk '
 /^event:/ {
     if (data_content != "") {
         print data_content "\n"
     }
     sub(/^event: /, "타입: ", $0)
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

타입: metadata...
{'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2'}

타입: debug...
{'type': 'checkpoint', 'timestamp': '2024-08-28T23:16:28.134680+00:00', 'step': -1, 'payload': {'config': {'tags': [], 'metadata': {'created_by': 'system', 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'callbacks': [None], 'recursion_limit': 25, 'configurable': {'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'checkpoint_id': '1ef65938-d8f3-6b25-bfff-30a8ed6460bd', 'checkpoint_ns': ''}, 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2'}, 'values': {'messages': [], 'search_results': []}, 'metadata': {'source': 'input', 'writes': {'messages': [{'role': 'human', 'content': "샌프란시스코의 날씨는 어때요?"}]}, 'step': -1}, 'next': ['__start__'], 'tasks': [{'id': 'b40d2c90-dc1e-52db-82d6-08751b769c55', 'name': '__start__', 'interrupts': []}]}}

타입: debug...
{'type': 'checkpoint', 'timestamp': '2024-08-28T23:16:28.139821+00:00', 'step': 0, 'payload': {'config': {'tags': [], 'metadata': {'created_by': 'system', 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'callbacks': [None], 'recursion_limit': 25, 'configurable': {'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'checkpoint_id': '1ef65938-d900-63f1-8000-70fe53e0da5c', 'checkpoint_ns': ''}, 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2'}, 'values': {'messages': [{'content': "샌프란시스코의 날씨는 어때요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}], 'search_results': []}, 'metadata': {'source': 'loop', 'writes': None, 'step': 0}, 'next': ['call_model'], 'tasks': [{'id': '685d89f6-542b-5e11-8cff-2963e7f4ea63', 'name': 'call_model', 'interrupts': []}]}}

타입: debug...
{'type': 'task', 'timestamp': '2024-08-28T23:16:28.139928+00:00', 'step': 1, 'payload': {'id': '600a6ff3-7ff1-570a-b626-f887e9a70f1c', 'name': 'call_model', 'input': {'messages': [{'content': "샌프란시스코의 날씨는 어때요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}], 'search_results': [], 'final_answer': None}, 'triggers': ['start:call_model']}}

타입: debug...
{'type': 'task_result', 'timestamp': '2024-08-28T23:16:28.584833+00:00', 'step': 1, 'payload': {'id': '600a6ff3-7ff1-570a-b626-f887e9a70f1c', 'name': 'call_model', 'error': None, 'result': [['messages', {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]], 'interrupts': []}}

타입: debug...
{'type': 'checkpoint', 'timestamp': '2024-08-28T23:16:28.584991+00:00', 'step': 1, 'payload': {'config': {'tags': [], 'metadata': {'created_by': 'system', 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'callbacks': [None], 'recursion_limit': 25, 'configurable': {'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'checkpoint_id': '1ef65938-dd3f-616f-8001-ce1c6f31e130', 'checkpoint_ns': ''}, 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2'}, 'values': {'messages': [{'content': "샌프란시스코의 날씨는 어때요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}, {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}], 'search_results': []}, 'metadata': {'source': 'loop', 'writes': {'call_model': {'messages': {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}}}, 'step': 1}, 'next': ['exa_search', 'tavily_search'], 'tasks': [{'id': '43865935-be38-5f6e-8d38-d44ef369c278', 'name': 'exa_search', 'interrupts': []}, {'id': 'dc220677-2720-56c7-a524-caaff60fce2c', 'name': 'tavily_search', 'interrupts': []}]}}

타입: debug...
{'type': 'task', 'timestamp': '2024-08-28T23:16:28.585219+00:00', 'step': 2, 'payload': {'id': '870b5854-2f84-533d-8e7d-87158ee948fc', 'name': 'exa_search', 'input': {'messages': [{'content': "샌프란시스코의 날씨는 어때요?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}, {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}], 'search_results': [], 'final_answer': None}, 'triggers': ['call_model']}}



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'task', 'timestamp': '2024-08-28T23:16:28.585219+00:00', 'step': 2, 'payload': {'id': '7589abfc-04df-58c6-8835-be172f84a7ff', 'name': 'tavily_search', 'input': {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}, {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}], 'search_results': [], 'final_answer': None}, 'triggers': ['call_model']}}



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'task_result', 'timestamp': '2024-08-28T23:16:32.422243+00:00', 'step': 2, 'payload': {'id': '7589abfc-04df-58c6-8835-be172f84a7ff', 'name': 'tavily_search', 'error': None, 'result': [['search_results', ["{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1724886988, 'localtime': '2024-08-28 16:16'}, 'current': {'last_updated_epoch': 1724886900, 'last_updated': '2024-08-28 16:15', 'temp_c': 22.2, 'temp_f': 72.0, 'is_day': 1, 'condition': {'text': '부분 구름', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 16.1, 'wind_kph': 25.9, 'wind_degree': 300, 'wind_dir': 'WNW', 'pressure_mb': 1013.0, 'pressure_in': 29.91, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 61, 'cloud': 25, 'feelslike_c': 24.6, 'feelslike_f': 76.4, 'windchill_c': 19.6, 'windchill_f': 67.2, 'heatindex_c': 19.7, 'heatindex_f': 67.4, 'dewpoint_c': 13.0, 'dewpoint_f': 55.5, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 18.7, 'gust_kph': 30.0}}"]]], 'interrupts': []}}



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'task_result', 'timestamp': '2024-08-28T23:16:34.750124+00:00', 'step': 2, 'payload': {'id': '870b5854-2f84-533d-8e7d-87158ee948fc', 'name': 'exa_search', 'error': None, 'result': [['search_results', ['태양이 수평선 아래로 6도 이하인 기간. 수평선은 명확하게 정의되어야 하며, 좋은 대기 조건에서 가장 밝은 별들이 보일 수 있어야 합니다(즉, 달빛이나 다른 조명 없이). 여전히 정상적인 야외 활동을 계속할 수 있어야 합니다. 태양이 수평선 아래로 6도에서 12도 사이인 기간. 수평선은 잘 정의되어 있으며, 인공 조명 없이도 물체의 윤곽을 볼 수 있을 수 있습니다. 이때는 추가 조명 없이는 일반적인 야외 활동이 불가능합니다. 태양이 수평선 아래로 12도에서 18도 사이인 기간. 이때 태양은 아침에 이 시점 이후나 저녁에 이 시점 이후 하늘을 밝히지 않습니다. 아침의 천문학적 황혼 시작과 저녁의 천문학적 황혼 끝에서는 하늘의 조명이 매우 미미하며 감지되지 않을 수도 있습니다. 시민 일몰 시간에서 시민 일출 시간을 뺀 시간. 실제 일몰 시간에서 실제 일출 시간을 뺀 시간. 오늘과 내일의 일조 시간 변화도 가능한 경우 나열됩니다.']]], 'interrupts': []} 



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'checkpoint', 'timestamp': '2024-08-28T23:16:34.750266+00:00', 'step': 2, 'payload': {'config': {'tags': [], 'metadata': {'created_by': 'system', 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'callbacks': [None], 'recursion_limit': 25, 'configurable': {'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'checkpoint_id': '1ef65939-180b-6087-8002-f969296f8e3d', 'checkpoint_ns': ''}, 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2'}, 'values': {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}, {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}], 'search_results': ['태양이 수평선 아래로 6도 이하인 기간. 수평선은 명확하게 정의되어야 하며, 가장 밝은 별들이 좋은 대기 조건에서 보일 수 있어야 합니다(즉, 달빛이나 다른 조명 없이). 여전히 정상적인 야외 활동을 계속할 수 있어야 합니다. 태양이 수평선 아래로 6도에서 12도 사이인 기간. 수평선은 잘 정의되어 있으며, 인공 조명 없이 물체의 윤곽을 볼 수 있습니다. 이때는 추가 조명 없이는 일반적인 야외 활동이 불가능합니다. 태양이 수평선 아래로 12도에서 18도 사이인 기간. 이때 태양은 아침에 이 시점 이후나 저녁에 이 시점 이후 하늘을 밝히지 않습니다. 아침의 천문학적 황혼 시작과 저녁의 천문학적 황혼 끝에서는 하늘의 조명이 매우 미미하며 감지되지 않을 수도 있습니다. 시민 일몰 시간에서 시민 일출 시간을 뺀 시간. 실제 일몰 시간에서 실제 일출 시간을 뺀 시간. 오늘과 내일의 일조 시간 변화도 가능한 경우 나열됩니다.', "{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1724886988, 'localtime': '2024-08-28 16:16'}, 'current': {'last_updated_epoch': 1724886900, 'last_updated': '2024-08-28 16:15', 'temp_c': 22.2, 'temp_f': 72.0, 'is_day': 1, 'condition': {'text': '부분 구름', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 16.1, 'wind_kph': 25.9, 'wind_degree': 300, 'wind_dir': 'WNW', 'pressure_mb': 1013.0, 'pressure_in': 29.91, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 61, 'cloud': 25, 'feelslike_c': 24.6, 'feelslike_f': 76.4, 'windchill_c': 19.6, 'windchill_f': 67.2, 'heatindex_c': 19.7, 'heatindex_f': 67.4, 'dewpoint_c': 13.0, 'dewpoint_f': 55.5, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 18.7, 'gust_kph': 30.0}}"]}, 'metadata': {'source': 'loop', 'writes': {'exa_search': {'search_results': ['태양이 수평선 아래로 6도 이하인 기간. 수평선은 명확하게 정의되어야 하며, 가장 밝은 별들이 좋은 대기 조건에서 보일 수 있어야 합니다(즉, 달빛이나 다른 조명 없이). 여전히 정상적인 야외 활동을 계속할 수 있어야 합니다. 태양이 수평선 아래로 6도에서 12도 사이인 기간. 수평선은 잘 정의되어 있으며, 인공 조명 없이 물체의 윤곽을 볼 수 있습니다. 이때는 추가 조명 없이는 일반적인 야외 활동이 불가능합니다. 태양이 수평선 아래로 12도에서 18도 사이인 기간. 이때 태양은 아침에 이 시점 이후나 저녁에 이 시점 이후 하늘을 밝히지 않습니다. 아침의 천문학적 황혼 시작과 저녁의 천문학적 황혼 끝에서는 하늘의 조명이 매우 미미하며 감지되지 않을 수도 있습니다. 시민 일몰 시간에서 시민 일출 시간을 뺀 시간. 실제 일몰 시간에서 실제 일출 시간을 뺀 시간. 오늘과 내일의 일조 시간 변화도 가능한 경우 나열됩니다.']}, 'tavily_search': {'search_results': ["{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1724886988, 'localtime': '2024-08-28 16:16'}, 'current': {'last_updated_epoch': 1724886900, 'last_updated': '2024-08-28 16:15', 'temp_c': 22.2, 'temp_f': 72.0, 'is_day': 1, 'condition': {'text': '부분 구름', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 16.1, 'wind_kph': 25.9, 'wind_degree': 300, 'wind_dir': 'WNW', 'pressure_mb': 1013.0, 'pressure_in': 29.91, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 61, 'cloud': 25, 'feelslike_c': 24.6, 'feelslike_f': 76.4, 'windchill_c': 19.6, 'windchill_f': 67.2, 'heatindex_c': 19.7, 'heatindex_f': 67.4, 'dewpoint_c': 13.0, 'dewpoint_f': 55.5, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 18.7, 'gust_kph': 30.0}}"]}}, 'step': 2}, 'next': ['summarize_search_results'], 'tasks': [{'id': '7263c738-516d-5708-b318-2c8ef54d4a33', 'name': 'summarize_search_results', 'interrupts': []}]} 



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'task', 'timestamp': '2024-08-28T23:16:34.750394+00:00', 'step': 3, 'payload': {'id': '5beaa05d-57d4-5acd-95c1-c7093990910f', 'name': 'summarize_search_results', 'input': {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}, {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}], 'search_results': ['태양이 수평선 아래로 6도 이하인 기간. 수평선은 명확하게 정의되어야 하며, 가장 밝은 별들이 좋은 대기 조건에서 보일 수 있어야 합니다(즉, 달빛이나 다른 조명 없이). 여전히 정상적인 야외 활동을 계속할 수 있어야 합니다. 태양이 수평선 아래로 6도에서 12도 사이인 기간. 수평선은 잘 정의되어 있으며, 인공 조명 없이 물체의 윤곽을 볼 수 있습니다. 이때는 추가 조명 없이는 일반적인 야외 활동이 불가능합니다. 태양이 수평선 아래로 12도에서 18도 사이인 기간. 이때 태양은 아침에 이 시점 이후나 저녁에 이 시점 이후 하늘을 밝히지 않습니다. 아침의 천문학적 황 twilight 시작과 저녁의 천문학적 황 twilight 끝에서는 하늘의 조명이 매우 미미하며 감지되지 않을 수도 있습니다. 시민 일몰 시간에서 시민 일출 시간을 뺀 시간. 실제 일몰 시간에서 실제 일출 시간을 뺀 시간. 오늘과 내일의 일조 시간 변화도 가능한 경우 나열됩니다.', "{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1724886988, 'localtime': '2024-08-28 16:16'}, 'current': {'last_updated_epoch': 1724886900, 'last_updated': '2024-08-28 16:15', 'temp_c': 22.2, 'temp_f': 72.0, 'is_day': 1, 'condition': {'text': '부분 구름', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 16.1, 'wind_kph': 25.9, 'wind_degree': 300, 'wind_dir': 'WNW', 'pressure_mb': 1013.0, 'pressure_in': 29.91, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 61, 'cloud': 25, 'feelslike_c': 24.6, 'feelslike_f': 76.4, 'windchill_c': 19.6, 'windchill_f': 67.2, 'heatindex_c': 19.7, 'heatindex_f': 67.4, 'dewpoint_c': 13.0, 'dewpoint_f': 55.5, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 18.7, 'gust_kph': 30.0}}"], 'final_answer': None}, 'triggers': ['exa_search', 'tavily_search']} 



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'task_result', 'timestamp': '2024-08-28T23:16:35.851058+00:00', 'step': 3, 'payload': {'id': '5beaa05d-57d4-5acd-95c1-c7093990910f', 'name': 'summarize_search_results', 'error': None, 'result': [['final_answer', {'content': "제공된 데이터는 태양의 위치에 따라 다양한 황혼 기간을 상세하게 설명하며, 2024년 8월 28일 현재 샌프란시스코, 캘리포니아의 날씨 정보를 포함합니다. 날씨는 부분 구름이며 기온은 22.2°C(72.0°F)로, 서북서 방향에서 시속 16.1마일의 온화한 바람이 불고 있으며, UV 지수는 5입니다.", 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_157b3831f5'}, 'type': 'ai', 'name': None, 'id': 'run-928c997b-9d85-4664-bd20-97ade4cc655e', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]], 'interrupts': []} 



    새로운 '디버그' 이벤트 수신 중...
    {'type': 'checkpoint', 'timestamp': '2024-08-28T23:16:35.851194+00:00', 'step': 3, 'payload': {'config': {'tags': [], 'metadata': {'created_by': 'system', 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'callbacks': [None], 'recursion_limit': 25, 'configurable': {'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2', 'user_id': '', 'graph_id': 'agent', 'thread_id': 'be4fd54d-ff22-4e9e-8876-d5cccc0e8048', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'checkpoint_id': '1ef65939-228a-6d93-8003-8b06d7483024', 'checkpoint_ns': ''}, 'run_id': '1ef65938-d7c7-68db-b786-011aa1cb3cd2'}, 'values': {'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '4123a12c-46cb-4815-bdcc-32537af0cb5b', 'example': False}, {'content': '샌프란시스코의 현재 날씨', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_a2ff031fb5'}, 'type': 'ai', 'name': None, 'id': 'run-0407bff9-3692-4ab5-9e57-2e9f396a3ee4', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}], 'search_results': ['태양이 수평선 아래로 6도 이하인 기간. 수평선은 명확하게 정의되어야 하며, 가장 밝은 별들이 좋은 대기 조건에서 보일 수 있어야 합니다(즉, 달빛이나 다른 조명 없이). 여전히 정상적인 야외 활동을 계속할 수 있어야 합니다. 태양이 수평선 아래로 6도에서 12도 사이인 기간. 수평선은 잘 정의되어 있으며, 인공 조명 없이 물체의 윤곽을 볼 수 있어야 합니다. 이때는 추가 조명 없이는 일반적인 야외 활동이 불가능합니다. 태양이 수평선 아래로 12도에서 18도 사이인 기간. 이때 태양은 아침에 이 시점 이후나 저녁에 이 시점 이후 하늘을 밝히지 않습니다. 아침의 천문학적 황혼 시작과 저녁의 천문학적 황혼 끝에서는 하늘의 조명이 매우 미미하며 감지되지 않을 수도 있습니다. 시민 일몰 시간에서 시민 일출 시간을 뺀 시간. 실제 일몰 시간에서 실제 일출 시간을 뺀 시간. 오늘과 내일의 일조 시간 변화도 가능한 경우 나열됩니다.', "{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1724886988, 'localtime': '2024-08-28 16:16'}, 'current': {'last_updated_epoch': 1724886900, 'last_updated': '2024-08-28 16:15', 'temp_c': 22.2, 'temp_f': 72.0, 'is_day': 1, 'condition': {'text': '부분 구름', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 16.1, 'wind_kph': 25.9, 'wind_degree': 300, 'wind_dir': 'WNW', 'pressure_mb': 1013.0, 'pressure_in': 29.91, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 61, 'cloud': 25, 'feelslike_c': 24.6, 'feelslike_f': 76.4, 'windchill_c': 19.6, 'windchill_f': 67.2, 'heatindex_c': 19.7, 'heatindex_f': 67.4, 'dewpoint_c': 13.0, 'dewpoint_f': 55.5, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 18.7, 'gust_kph': 30.0}}"], 'final_answer': {'content': "제공된 데이터는 태양의 위치에 따라 다양한 황혼 기간을 상세하게 설명하며, 2024년 8월 28일 현재 샌프란시스코, 캘리포니아의 날씨 정보를 포함합니다. 날씨는 부분 구름이며 기온은 22.2°C(72.0°F)로, 서북서 방향에서 시속 16.1마일의 온화한 바람이 불고 있으며, UV 지수는 5입니다.", 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_157b3831f5'}, 'type': 'ai', 'name': None, 'id': 'run-928c997b-9d85-4664-bd20-97ade4cc655e', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}}, 'metadata': {'source': 'loop', 'writes': {'summarize_search_results': {'final_answer': {'content': "제공된 데이터는 태양의 위치에 따라 다양한 황혼 기간을 상세하게 설명하며, 2024년 8월 28일 현재 샌프란시스코, 캘리포니아의 날씨 정보를 포함합니다. 날씨는 부분 구름이며 기온은 22.2°C(72.0°F)로, 서북서 방향에서 시속 16.1마일의 온화한 바람이 불고 있으며, UV 지수는 5입니다.", 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_157b3831f5'}, 'type': 'ai', 'name': None, 'id': 'run-928c997b-9d85-4664-bd20-97ade4cc655e', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}}}, 'step': 3}, 'next': [], 'tasks': []} 



이제 우리의 디버그 이벤트는 첫 번째 '체크포인트' 이벤트와 그래프가 생성된 후의 체크포인트 이벤트로 시작됩니다. 그 다음에 'call_model' 노드와 관련된 단일 'task'와 해당 'task_result'가 있으며, 이는 호출이 끝나면 전체 슈퍼 단계가 완료되고, 그래프가 저장됩니다. 

다음 슈퍼 단계에서는 각기 다른 두 검색 노드가 실행됩니다 - 하나는 Exa 검색을 수행하고, 다른 하나는 Tavily를 사용합니다. 이러한 노드를 병렬로 동일한 슈퍼 단계에서 실행함으로써 2개의 'task' 이벤트와 그에 따른 2개의 'task_result' 이벤트가 생성됩니다. 두 'task_result' 이벤트를 수신한 후, 우리는 또 다른 'checkpoint' 이벤트를 보게 됩니다.

마지막으로, 우리는 'summarize_search_results' 노드와 관련된 최종 'task' 및 'task_result' 쌍을 확인할 수 있으며, 이는 우리 그래프의 마지막 노드입니다. 이 슈퍼 단계가 완료되면 우리는 이번 실행의 마지막 체크포인트에 해당하는 한 번의 최종 'checkpoint' 이벤트를 보게 됩니다.
    

