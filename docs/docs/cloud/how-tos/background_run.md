_한국어로 기계번역됨_

# 백그라운드 실행 시작하기

이 가이드는 에이전트를 위한 백그라운드 실행을 시작하는 방법을 다룹니다. 이는 장시간 실행되는 작업에 유용할 수 있습니다.

## 설정

먼저 클라이언트와 스레드를 설정해보겠습니다.

=== "Python"

```python
from langgraph_sdk import get_client

client = get_client(url=<DEPLOYMENT_URL>)
# 이름이 "agent"인 그래프 사용
assistant_id = "agent"
# 스레드 생성
thread = await client.threads.create()
print(thread)
```

=== "Javascript"

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
// 이름이 "agent"인 그래프 사용
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

```json
{
    'thread_id': '5cb1e8a1-34b3-4a61-a34e-71a9799bd00d',
    'created_at': '2024-08-30T20:35:52.062934+00:00',
    'updated_at': '2024-08-30T20:35:52.062934+00:00',
    'metadata': {},
    'status': 'idle',
    'config': {},
    'values': None
}
```

## 스레드에서 실행 목록 확인

이 스레드에서 현재 실행 목록을 나열하면 빈 목록이 표시됩니다:

=== "Python"

```python
runs = await client.runs.list(thread["thread_id"])
print(runs)
```

=== "Javascript"

```js
let runs = await client.runs.list(thread['thread_id']);
console.log(runs);
```

=== "CURL"

```bash
curl --request GET \
    --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs
```

출력:

```json
[]
```

## 스레드에서 실행 시작하기

이제 실행을 시작해보겠습니다:

=== "Python"

```python
input = {"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]}
run = await client.runs.create(thread["thread_id"], assistant_id, input=input)
```

=== "Javascript"

    ```js
    let input = {"messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어떻습니까?"}]};
    let run = await client.runs.create(thread["thread_id"], assistantID, { input });
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <ASSISTANT_ID>
        }'
    ```

처음으로 폴링할 때, `status=pending`을 볼 수 있습니다:

=== "Python"

    ```python
    print(await client.runs.get(thread["thread_id"], run["run_id"]))
    ```

=== "Javascript"

    ```js
    console.log(await client.runs.get(thread["thread_id"], run["run_id"]));
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/<RUN_ID>
    ```

출력:

        {
            "run_id": "1ef6a5f8-bd86-6763-bbd6-bff042db7b1b",
            "thread_id": "7885f0cf-94ad-4040-91d7-73f7ba007c8a",
            "assistant_id": "fe096781-5601-53d2-b2f6-0d3403f7e9ca",
            "created_at": "2024-09-04T01:46:47.244887+00:00",
            "updated_at": "2024-09-04T01:46:47.244887+00:00",
            "metadata": {},
            "status": "pending",
            "kwargs": {
                "input": {
                    "messages": [
                        {
                            "role": "user",
                            "content": "샌프란시스코의 날씨는 어떻습니까?"
                        }
                    ]
                },
                "config": {
                    "metadata": {
                        "created_by": "system"
                    },
                    "configurable": {
                        "run_id": "1ef6a5f8-bd86-6763-bbd6-bff042db7b1b",
                        "user_id": "",
                        "graph_id": "agent",
                        "thread_id": "7885f0cf-94ad-4040-91d7-73f7ba007c8a",
                        "assistant_id": "fe096781-5601-53d2-b2f6-0d3403f7e9ca",
                        "checkpoint_id": null
                    }
                },
                "webhook": null,
                "temporary": false,
                "stream_mode": [
                    "values"
                ],
                "feedback_keys": null,
                "interrupt_after": null,
                "interrupt_before": null
            },
            "multitask_strategy": "reject"
        }

이제 실행을 참여하고, 완료될 때까지 기다린 후 다시 상태를 확인할 수 있습니다:

=== "Python"

    ```python
    await client.runs.join(thread["thread_id"], run["run_id"])
    print(await client.runs.get(thread["thread_id"], run["run_id"]))
    ```

=== "Javascript"

    ```js
    await client.runs.join(thread["thread_id"], run["run_id"]);
    console.log(await client.runs.get(thread["thread_id"], run["run_id"]));
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <배포_URL>/threads/<스레드_ID>/runs/<실행_ID>/join &&
    curl --request GET \
        --url <배포_URL>/threads/<스레드_ID>/runs/<실행_ID>
    ```

출력:

    {
        "run_id": "1ef6a5f8-bd86-6763-bbd6-bff042db7b1b",
        "thread_id": "7885f0cf-94ad-4040-91d7-73f7ba007c8a",
        "assistant_id": "fe096781-5601-53d2-b2f6-0d3403f7e9ca",
        "created_at": "2024-09-04T01:46:47.244887+00:00",
        "updated_at": "2024-09-04T01:46:47.244887+00:00",
        "metadata": {},
        "status": "success",
        "kwargs": {
            "input": {
                "messages": [
                    {
                        "role": "user",
                        "content": "샌프란시스코 날씨 어때?"
                    }
                ]
            },
            "config": {
                "metadata": {
                    "created_by": "system"
                },
                "configurable": {
                    "run_id": "1ef6a5f8-bd86-6763-bbd6-bff042db7b1b",
                    "user_id": "",
                    "graph_id": "agent",
                    "thread_id": "7885f0cf-94ad-4040-91d7-73f7ba007c8a",
                    "assistant_id": "fe096781-5601-53d2-b2f6-0d3403f7e9ca",
                    "checkpoint_id": null
                }
            },
            "webhook": null,
            "temporary": false,
            "stream_mode": [
                "values"
            ],
            "feedback_keys": null,
            "interrupt_after": null,
            "interrupt_before": null
        },
        "multitask_strategy": "reject"
    }


완벽합니다! 실행이 예상대로 성공했습니다. 최종 상태를 출력하여 실행이 제대로 되었는지 다시 확인할 수 있습니다:

=== "파이썬"

    ```python
    final_result = await client.threads.get_state(thread["thread_id"])
    print(final_result)
    ```

=== "자바스크립트"

    ```js
    let finalResult = await client.threads.getState(thread["thread_id"]);
    console.log(finalResult);
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <배포_URL>/threads/<스레드_ID>/state
    ```

출력:

    {
        "values": {
            "messages": [
                {
                    "content": "샌프란시스코 날씨 어때?",
                    "additional_kwargs": {},
                    "response_metadata": {},
                    "type": "human",
                    "name": null,
                    "id": "beba31bf-320d-4125-9c37-cadf526ac47a",
                    "example": false
                },
                {
                    "content": [
                        {
                            "id": "toolu_01AaNPSPzqia21v7aAKwbKYm",
                            "input": {},
                            "name": "tavily_search_results_json",
                            "type": "tool_use",
                            "index": 0,
                            "partial_json": "{\"query\": \"샌프란시스코 날씨\"}"
                        }
                    ],
                    "additional_kwargs": {},
                    "response_metadata": {
                        "stop_reason": "tool_use",
                        "stop_sequence": null
                    },
                    "type": "ai",
                    "name": null,
                    "id": "run-f220faf8-1d27-4f73-ad91-6bb3f47e8639",
                    "example": false,
                    "tool_calls": [
                        {
                            "name": "tavily_search_results_json",
                            "args": {
                                "query": "샌프란시스코 날씨"
                            },
                            "id": "toolu_01AaNPSPzqia21v7aAKwbKYm",
                            "type": "tool_call"
                        }
                    ],
                    "invalid_tool_calls": [],
                    "usage_metadata": {
                        "input_tokens": 273,
                        "output_tokens": 61,
                        "total_tokens": 334
                    }
                },
                {
                    "content": "[{\"url\": \"https://www.weatherapi.com/\", \"content\": \"{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1725052131, 'localtime': '2024-08-30 14:08'}, 'current': {'last_updated_epoch': 1725051600, 'last_updated': '2024-08-30 14:00', 'temp_c': 21.1, 'temp_f': 70.0, 'is_day': 1, 'condition': {'text': '부분적으로 흐림', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 11.9, 'wind_kph': 19.1, 'wind_degree': 290, 'wind_dir': '서북서', 'pressure_mb': 1018.0, 'pressure_in': 30.07, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 59, 'cloud': 25, 'feelslike_c': 21.1, 'feelslike_f': 70.0, 'windchill_c': 18.6, 'windchill_f': 65.5, 'heatindex_c': 18.6, 'heatindex_f': 65.5, 'dewpoint_c': 12.2, 'dewpoint_f': 54.0, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 15.0, 'gust_kph': 24.2}}\"}]",
                    "additional_kwargs": {},
                    "response_metadata": {},
                    "type": "tool",
                    "name": "tavily_search_results_json",
                    "id": "686b2487-f332-4e58-9508-89b3a814cd81",
                    "tool_call_id": "toolu_01AaNPSPzqia21v7aAKwbKYm",
                    "artifact": {
                        "query": "샌프란시스코 날씨",
                        "follow_up_questions": null,
                        "answer": null,
                        "images": [],
                        "results": [
                            {
                                "title": "샌프란시스코 날씨",
                                "url": "https://www.weatherapi.com/",
                                "content": "{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.78, 'lon': -122.42, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1725052131, 'localtime': '2024-08-30 14:08'}, 'current': {'last_updated_epoch': 1725051600, 'last_updated': '2024-08-30 14:00', 'temp_c': 21.1, 'temp_f': 70.0, 'is_day': 1, 'condition': {'text': '부분적으로 흐림', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 11.9, 'wind_kph': 19.1, 'wind_degree': 290, 'wind_dir': '서북서', 'pressure_mb': 1018.0, 'pressure_in': 30.07, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 59, 'cloud': 25, 'feelslike_c': 21.1, 'feelslike_f': 70.0, 'windchill_c': 18.6, 'windchill_f': 65.5, 'heatindex_c': 18.6, 'heatindex_f': 65.5, 'dewpoint_c': 12.2, 'dewpoint_f': 54.0, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 5.0, 'gust_mph': 15.0, 'gust_kph': 24.2}}",
                                "score": 0.976148,
                                "raw_content": null
                            }
                        ],
                        "response_time": 3.07
                    },
                    "status": "success"
                },
                {
                    "content": [
                        {
                            "text": "\n\n검색 결과는 샌프란시스코의 현재 날씨 조건을 제공합니다. 데이터에 따르면, 2024년 8월 30일 오후 2시 기준 샌프란시스코의 기온은 70°F(21.1°C)로 부분적으로 흐림 상태입니다. 바람은 서북서에서 시속 약 12마일(19km/h)로 불고 있습니다. 습도는 59%이며 시야는 9마일(16km)입니다. 전반적으로 샌프란시스코의 늦여름에 기온이 쾌적하고 부분적으로 맑은 날씨로 보입니다.",
                            "type": "text",
                            "index": 0
                        }
                    ],
                    "additional_kwargs": {},
                    "response_metadata": {
                        "stop_reason": "end_turn",
                        "stop_sequence": null
                    },
                    "type": "ai",
                    "name": null,
                    "id": "run-8fecc61d-3d9f-4e16-8e8a-92f702be498a",
                    "example": false,
                    "tool_calls": [],
                    "invalid_tool_calls": [],
                    "usage_metadata": {
                        "input_tokens": 837,
                        "output_tokens": 124,
                        "total_tokens": 961
                    }
                }
            ]
        },
        "next": [],
        "tasks": [],
        "metadata": {
            "step": 3,
            "run_id": "1ef67140-eb23-684b-8253-91d4c90bb05e",
            "source": "loop",
            "writes": {
                "agent": {
                    "messages": [
                        {
                            "id": "run-8fecc61d-3d9f-4e16-8e8a-92f702be498a",
                            "name": null,
                            "type": "ai",
                            "content": [
                                {
                                    "text": "\n\n검색 결과는 샌프란시스코의 현재 날씨 조건을 제공합니다. 데이터에 따르면, 2024년 8월 30일 오후 2시 기준 샌프란시스코의 기온은 70°F(21.1°C)로 부분적으로 흐림 상태입니다. 바람은 서북서에서 시속 약 12마일(19km/h)로 불고 있습니다. 습도는 59%이며 시야는 9마일(16km)입니다. 전반적으로 샌프란시스코의 늦여름에 기온이 쾌적하고 부분적으로 맑은 날씨로 보입니다.",
                                    "type": "text",
                                    "index": 0
                                검색 결과는 샌프란시스코의 현재 날씨를 제공합니다. 데이터에 따르면, 2024년 8월 30일 오후 2시 기준으로 샌프란시스코의 기온은 70°F(21.1°C)로 부분적으로 흐린 하늘을 보이고 있습니다. 바람은 서북서쪽에서 시속 약 12마일(19km/h)로 불고 있으며, 습도는 59%이고 가시거리는 9마일(16km)입니다. 전반적으로 샌프란시스코에서는 편안한 기온과 부분적으로 맑은 날씨로 좋은 늦여름 날이 될 것으로 보입니다.