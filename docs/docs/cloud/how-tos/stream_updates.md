_한국어로 기계번역됨_

# 그래프의 상태 업데이트 스트리밍 방법

!!! 정보 "사전 조건"
    * [스트리밍](../../concepts/streaming.md)

이 가이드는 각 노드가 실행된 후 그래프 상태에 대해 수행된 업데이트를 스트리밍하는 `stream_mode="updates"`를 사용하는 방법을 다룹니다. 이는 `stream_mode="values"`를 사용하는 것과 다릅니다: 각 슈퍼스텝에서 상태의 전체 값을 스트리밍하는 대신, 해당 슈퍼스텝에서 상태를 업데이트한 각 노드의 업데이트만 스트리밍합니다.

## 설정

먼저 클라이언트와 스레드를 설정해 보겠습니다:

=== "파이썬"

    ```python
    from langgraph_sdk import get_client

    client = get_client(url=<DEPLOYMENT_URL>)
    # "agent"라는 이름으로 배포된 그래프를 사용
    assistant_id = "agent"
    # 스레드 생성
    thread = await client.threads.create()
    print(thread)
    ```

=== "자바스크립트"

    ```js
    import { Client } from "@langchain/langgraph-sdk";

    const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
    // "agent"라는 이름으로 배포된 그래프를 사용
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
      'thread_id': '979e3c89-a702-4882-87c2-7a59a250ce16',
      'created_at': '2024-06-21T15:22:07.453100+00:00',
      'updated_at': '2024-06-21T15:22:07.453100+00:00',
      'metadata': {},
      'status': 'idle',
      'config': {},
      'values': None 
    }

## 업데이트 모드에서 그래프 스트리밍

이제 업데이트를 통해 스트리밍할 수 있습니다. 이는 각 노드가 실행된 후 상태에 수행된 업데이트를 출력합니다:


=== "파이썬"

    ```python
    input = {
        "messages": [
            {
                "role": "user",
                "content": "la의 날씨는 어때?"
            }
        ]
    }
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=input,
        stream_mode="updates",
    ):
        print(f"새로운 이벤트 유형 수신: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "자바스크립트"

    ```js
    const input = {
      messages: [
        {
          role: "human",
          content: "LA의 날씨는 어때?"
        }
      ]
    };

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input,
        streamMode: "updates"
      }
    );

    for await (const chunk of streamResponse) {
      console.log(`새로운 이벤트 수신 유형: ${chunk.event}...`);
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
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"LA의 날씨는 어때?\"}]},
       \"stream_mode\": [
         \"updates\"
       ]
     }" | \
     sed 's/\r$//' | \
     awk '
     /^event:/ {
         if (data_content != "") {
             print data_content "\n"
         }
         sub(/^event: /, "이벤트 수신 유형: ", $0)
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

    유형: 메타데이터의 새로운 이벤트 수신...
    {"run_id": "cfc96c16-ed9a-44bd-b5bb-c30e3c0725f0"}



    유형: 업데이트의 새로운 이벤트 수신...
    {
      "agent": {
        "messages": [
          {
            "type": "ai",
            "tool_calls": [
              {
                "name": "tavily_search_results_json",
                "args": {
                  "query": "로스앤젤레스의 날씨"
                },
                "id": "toolu_0148tMmDK51iLQfG1yaNwRHM"
              }
            ],
            ...
          }
        ]
      }
    }



    유형: 업데이트의 새로운 이벤트 수신...
    {
      "action": {
        "messages": [
          {
            "content": [
              {
                "url": "https://www.weatherapi.com/",
                "content": "{\"location\": {\"name\": \"로스앤젤레스\", \"region\": \"캘리포니아\", \"country\": \"미국\", \"lat\": 34.05, \"lon\": -118.24, \"tz_id\": \"America/Los_Angeles\", \"localtime_epoch\": 1716062239, \"localtime\": \"2024-05-18 12:57\"}, \"current\": {\"last_updated_epoch\": 1716061500, \"last_updated\": \"2024-05-18 12:45\", \"temp_c\": 18.9, \"temp_f\": 66.0, \"is_day\": 1, \"condition\": {\"text\": \"흐림\", \"icon\": \"//cdn.weatherapi.com/weather/64x64/day/122.png\", \"code\": 1009}, \"wind_mph\": 2.2, \"wind_kph\": 3.6, \"wind_degree\": 10, \"wind_dir\": \"북\", \"pressure_mb\": 1017.0, \"pressure_in\": 30.02, \"precip_mm\": 0.0, \"precip_in\": 0.0, \"humidity\": 65, \"cloud\": 100, \"feelslike_c\": 18.9, \"feelslike_f\": 66.0, \"vis_km\": 16.0, \"vis_miles\": 9.0, \"uv\": 6.0, \"gust_mph\": 7.5, \"gust_kph\": 12.0}}"
              }
            ],
            "유형": "도구",
            "이름": "tavily_search_results_json",
            "도구_호출_ID": "toolu_0148tMmDK51iLQfG1yaNwRHM",
            ...
          }
        ]
      }
    }



    새로운 업데이트 이벤트 수신 중...
    {
      "대리인": {
        "메시지": [
          {
            "콘텐츠": "현재 로스앤젤레스의 날씨는 흐림이며, 기온은 약 66°F(18.9°C)입니다. 북쪽에서 오는 약 2-3 mph의 가벼운 바람이 불고 있습니다. 습도는 65%이며 가시거리는 9마일로 좋습니다. 전반적으로 LA의 온화한 봄 날씨입니다.",
            "유형": "ai",
            ...
          }
        ]
      }
    }



    종료 이벤트 수신 중...
    없음