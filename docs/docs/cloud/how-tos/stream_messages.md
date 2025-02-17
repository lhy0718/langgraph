_한국어로 기계번역됨_

# 그래프에서 메시지를 스트리밍하는 방법

!!! 정보 "사전 요구 사항"
    * [스트리밍](../../concepts/streaming.md)

이 가이드는 그래프에서 메시지를 스트리밍하는 방법을 설명합니다. `stream_mode="messages-tuple"`을 사용하면 그래프 노드 내의 모든 채팅 모델 호출에서 개별 LLM 토큰(즉, 메시지)이 스트리밍됩니다.

## 설정

먼저 클라이언트와 스레드를 설정합시다:

=== "파이썬"

    ```python
    from langgraph_sdk import get_client

    client = get_client(url=<DEPLOYMENT_URL>)
    # "agent"라는 이름으로 배포된 그래프 사용
    assistant_id = "agent"
    # 스레드 생성
    thread = await client.threads.create()
    print(thread)
    ```

=== "자바스크립트"

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
        'thread_id': 'e1431c95-e241-4d1d-a252-27eceb1e5c86',
        'created_at': '2024-06-21T15:48:59.808924+00:00',
        'updated_at': '2024-06-21T15:48:59.808924+00:00',
        'metadata': {},
        'status': 'idle',
        'config': {},
        'values': None
    }

## 메시지 모드에서 그래프 스트리밍

이제 노드 내에서 생성된 모든 메시지에 대해 LLM 토큰을 `(message, metadata)` 형태의 튜플로 스트리밍할 수 있습니다. 메타데이터는 스트리밍 출력을 특정 노드나 LLM으로 필터링하는 데 유용할 수 있는 추가 정보를 포함합니다.

=== "파이썬"

    ```python
    input = {"messages": [{"role": "user", "content": "샌프란시스코 날씨 어때?"}]}
    config = {"configurable": {"model_name": "openai"}}

    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id=assistant_id,
        input=input,
        config=config,
        stream_mode="messages-tuple",
    ):
        print(f"새로운 이벤트 수신 유형: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "자바스크립트"

    ```js
    const input = {
      messages: [
        {
          role: "human",
          content: "샌프란시스코의 날씨는 어때?",
        }
      ]
    };
    const config = { configurable: { model_name: "openai" } };

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input,
        config,
        streamMode: "messages-tuple"
      }
    );
    for await (const chunk of streamResponse) {
      console.log(`새 이벤트 유형 수신: ${chunk.event}...`);
      console.log(chunk.data);
      console.log("\n\n");
    }
    ```

=== "CURL"

    ```bash
    curl --request POST \
     --url <배포_URL>/threads/<스레드_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"에이전트\",
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"로스앤젤레스의 날씨는 어때?\"}]},
       \"stream_mode\": [
         \"messages-tuple\"
       ]
     }" | \
     sed 's/\r$//' | \
     awk '
     /^event:/ {
         if (data_content != "") {
             print data_content "\n"
         }
         sub(/^event: /, "유형의 이벤트 수신: ", $0)
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

    새 이벤트 유형 수신: 메타데이터...
    {"run_id": "1ef971e0-9a84-6154-9047-247b4ce89c4d", "attempt": 1}

    ...

    새 이벤트 유형 수신: 메시지...
    [
      {
        "type": "AIMessageChunk",
        "tool_calls": [
          {
            "name": "tavily_search_results_json",
            "args": {
              "query": "weat"
            },
            "id": "toolu_0114XKXdNtHQEa3ozmY1uDdM",
            "type": "tool_call"
          }
        ],
        ...
      },
      {
        "graph_id": "agent",
        "langgraph_node": "agent",
        ...
      }
    ]



    새 이벤트 유형 수신: 메시지...
    [
      {
        "type": "AIMessageChunk",
        "tool_calls": [
          {
            "name": "tavily_search_results_json",
            "args": {
              "query": "샌프란시스코에서 그녀"
            },
            "id": "toolu_0114XKXdNtHQEa3ozmY1uDdM",
            "type": "tool_call"
          }
        ],
        ...
      },
      {
        "graph_id": "agent",
        "langgraph_node": "agent",
        ...
      }
    ]

    ...

    새로운 메시지 유형 이벤트 수신 중...
    [
      {
        "type": "AIMessageChunk",
        "tool_calls": [
          {
            "name": "tavily_search_results_json",
            "args": {
              "query": "샌프란시스코"
            },
            "id": "toolu_0114XKXdNtHQEa3ozmY1uDdM",
            "type": "tool_call"
          }
        ],
        ...
      },
      {
        "graph_id": "agent",
        "langgraph_node": "agent",
        ...
      }
    ]

    ...

    새로운 메시지 유형 이벤트 수신 중...
    [
      {
        "content": "[{\"url\": \"https://www.weatherapi.com/\", \"content\": \"{'location': {'name': '샌프란시스코', 'region': '캘리포니아', 'country': '미국', 'lat': 37.775, 'lon': -122.4183, 'tz_id': 'America/Los_Angeles', 'localtime_epoch': 1730475777, 'localtime': '2024-11-01 08:42'}, 'current': {'last_updated_epoch': 1730475000, 'last_updated': '2024-11-01 08:30', 'temp_c': 11.1, 'temp_f': 52.0, 'is_day': 1, 'condition': {'text': '부분적으로 흐림', 'icon': '//cdn.weatherapi.com/weather/64x64/day/116.png', 'code': 1003}, 'wind_mph': 2.2, 'wind_kph': 3.6, 'wind_degree': 192, 'wind_dir': 'SSW', 'pressure_mb': 1018.0, 'pressure_in': 30.07, 'precip_mm': 0.0, 'precip_in': 0.0, 'humidity': 89, 'cloud': 75, 'feelslike_c': 11.5, 'feelslike_f': 52.6, 'windchill_c': 10.0, 'windchill_f': 50.1, 'heatindex_c': 10.4, 'heatindex_f': 50.7, 'dewpoint_c': 9.1, 'dewpoint_f': 48.5, 'vis_km': 16.0, 'vis_miles': 9.0, 'uv': 3.0, 'gust_mph': 6.7, 'gust_kph': 10.8}}\"}]",
        "type": "tool",
        "tool_call_id": "toolu_0114XKXdNtHQEa3ozmY1uDdM",
        ...
      },
      {
        "graph_id": "agent",
        "langgraph_node": "action",
        ...
      }
    ]

    ...

    새로운 메시지 유형 이벤트 수신 중...
    [
      {
        "content": [
          {
            "text": "\n\n검색",
            "type": "text",
            "index": 0
          }
        ],
        "type": "AIMessageChunk",
        ...
      },
      {
        "graph_id": "agent",
        "langgraph_node": "agent",
        ...
      }
    ]



    새로운 메시지 유형 이벤트 수신 중...
    [
      {
        "content": [
          {
            "text": " 결과가 제공됩니다.",
            "type": "text",
            "index": 0
          }
        ],
        "type": "AIMessageChunk",
        ...
      },
      {
        "graph_id": "agent",
        "langgraph_node": "에이전트",
        ...
      }
    ]



    새 이벤트 수신 유형: 메시지...
    [
      {
        "content": [
          {
            "text": " 현재 날씨 조건",
            "type": "text",
            "index": 0
          }
        ],
        "type": "AIMessageChunk",
        ...
      },
      {
        "graph_id": "에이전트",
        "langgraph_node": "에이전트",
        ...
      }
    ]



    새 이벤트 수신 유형: 메시지...
    [
      {
        "content": [
          {
            "text": " 샌프란시스코에서.",
            "type": "text",
            "index": 0
          }
        ],
        "type": "AIMessageChunk",
        ...
      },
      {
        "graph_id": "에이전트",
        "langgraph_node": "에이전트",
        ...
      }
    ]

    ...