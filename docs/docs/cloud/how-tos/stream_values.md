_한국어로 기계번역됨_

# 그래프의 전체 상태 스트리밍 방법

!!! 정보 "선행 조건"
    * [스트리밍](../../concepts/streaming.md)

이 가이드는 각 슈퍼스텝에서 상태의 값을 스트리밍하는 `stream_mode="values"`를 사용하는 방법을 다룹니다. 이는 각 노드에서 상태에 대한 업데이트만 스트리밍하는 `stream_mode="updates"`와 다릅니다. 대신 각 슈퍼스텝에서 전체 그래프 상태를 스트리밍합니다.

## 설정

먼저 클라이언트와 스레드를 설정해 보겠습니다:

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
      'thread_id': 'bfc68029-1f7b-400f-beab-6f9032a52da4',
      'created_at': '2024-06-24T21:30:07.980789+00:00',
      'updated_at': '2024-06-24T21:30:07.980789+00:00',
      'metadata': {},
      'status': 'idle',
      'config': {},
      'values': None
    }

## 값 모드에서 그래프 스트리밍

이제 값에 따라 스트리밍할 수 있으며, 이는 각 노드 실행이 완료된 후 그래프의 전체 상태를 스트리밍합니다:

=== "Python"

    ```python
    input = {"messages": [{"role": "user", "content": "la 날씨가 어때?"}]}

    # 값 스트리밍
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id, 
        input=input,
        stream_mode="values"
    ):
        print(f"새로운 이벤트 유형 수신: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "Javascript"

    ```js
    const input = {"messages": [{"role": "user", "content": "la 날씨가 어때?"}]}

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input,
        streamMode: "values"
      }
    );
    for await (const chunk of streamResponse) {
      console.log(`새로운 이벤트 유형 수신: ${chunk.event}...`);
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
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"로스앤젤레스의 날씨는 어떤가요\"}]},
       \"stream_mode\": [
         \"values\"
       ]
     }" | \
     sed 's/\r$//' | \
     awk '
     /^event:/ {
         if (data_content != "") {
             print data_content "\n"
         }
         sub(/^event: /, "수신된 이벤트의 유형: ", $0)
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

    수신된 새로운 이벤트의 유형: 메타데이터...
    {"run_id": "f08791ce-0a3d-44e0-836c-ff62cd2e2786"}



    수신된 새로운 이벤트의 유형: 값...
    {
      "messages": [
        {
          "role": "human",
          "content": "로스앤젤레스의 날씨는 어떤가요"
        }
      ]
    }



    수신된 새로운 이벤트의 유형: 값...
    {
      "messages": [
        {
          "content": "로스앤젤레스의 날씨는 어떤가요",
          "type": "human",
          ...
        },
        {
          "content": "",
          "type": "ai",
          "tool_calls": [
            {
              "name": "tavily_search_results_json",
              "args": {
                "query": "로스앤젤레스의 날씨"
              },
              "id": "toolu_01E5mSaZWm5rWJnCqmt63v4g"
            }
          ],
          ...
        }
      ]
    }

    ...

    수신된 새로운 이벤트의 유형: 값...
    {
      "messages": [
        {
          "content": "로스앤젤레스의 날씨는 어떤가요",
          "type": "human",
          ...
        },
        {
          "content": "",
          "type": "ai",
          "tool_calls": [
            {
              "name": "tavily_search_results_json",
              "args": {
                "query": "로스앤젤레스의 날씨"
              },
              "id": "toolu_01E5mSaZWm5rWJnCqmt63v4g"
            }
          ],
          ...
        }
        {
          "content": [
            {
              "url": "https://www.weatherapi.com/",
              "content": "{\"location\": {\"name\": \"로스앤젤레스\", \"region\": \"캘리포니아\", \"country\": \"미국\", \"lat\": 34.05, \"lon\": -118.24, \"tz_id\": \"America/Los_Angeles\", \"localtime_epoch\": 1716310320, \"localtime\": \"2024-05-21 9:52\"}, \"current\": {\"last_updated_epoch\": 1716309900, \"last_updated\": \"2024-05-21 09:45\", \"temp_c\": 16.7, \"temp_f\": 62.1, \"is_day\": 1, \"condition\": {\"text\": \"흐림\", \"icon\": \"//cdn.weatherapi.com/weather/64x64/day/122.png\", \"code\": 1009}, \"wind_mph\": 8.1, \"wind_kph\": 13.0, \"wind_degree\": 250, \"wind_dir\": \"서남서\", \"pressure_mb\": 1015.0, \"pressure_in\": 29.97, \"precip_mm\": 0.0, \"precip_in\": 0.0, \"humidity\": 65, \"cloud\": 100, \"feelslike_c\": 16.7, \"feelslike_f\": 62.1, \"vis_km\": 16.0, \"vis_miles\": 9.0, \"uv\": 5.0, \"gust_mph\": 12.5, \"gust_kph\": 20.2}}"
            }
          ],
          "type": "tool",
          "name": "tavily_search_results_json",
          "tool_call_id": "toolu_01E5mSaZWm5rWJnCqmt63v4g"
          ...
        },
        {
          "content": "날씨 API 결과를 바탕으로 현재 로스앤젤레스의 날씨는 흐림이며, 온도는 약 62°F (17°C)입니다. 서남서 방향에서 약 8-13 mph의 가벼운 바람이 불고 있습니다. 습도는 65%이며 시정은 양호하여 9마일입니다. 전반적으로 LA의 온화한 봄 날씨 조건입니다.",
          "type": "ai",
          ...
        }
      ]
    }



    새로운 이벤트 수신: 종료...
    없음
    
    
    


최종 결과만 알고 싶다면 이 엔드포인트를 사용하고 받은 마지막 값을 추적할 수 있습니다.


=== "Python"

    ```python
    final_answer = None
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=input,
        stream_mode="values"
    ):
        if chunk.event == "values":
            final_answer = chunk.data
    ```

=== "Javascript"

    ```js
    let finalAnswer;
    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input,
        streamMode: "values"
      }
    );
    for await (const chunk of streamResponse) {
      finalAnswer = chunk.data;
    }
    ```

=== "CURL"

    ```bash
    curl --request POST \
     --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"로스앤젤레스의 날씨는 어떤가요?\"}]},
       \"stream_mode\": [
         \"values\"
       ]
     }" | \
     sed 's/\r$//' | \
     awk '
     /^data:/ { 
         sub(/^data: /, "", $0)   
         data_content = $0          
     }    
     END {                                               
         if (data_content != "") {
             print data_content
         }
     }         
     '
    ```


출력:

    {
      "messages": [
        {
          "content": "로스앤젤레스의 날씨는 어떤가요",
          "type": "human",
          ...
        },
        {
          "type": "ai",
          "tool_calls": [
            {
              "name": "tavily_search_results_json",
              "args": {
                "query": "로스앤젤레스 날씨"
              },
              "id": "toolu_01E5mSaZWm5rWJnCqmt63v4g"
            }
          ],
          ...
        }
        {
          "content": [
            {
              "url": "https://www.weatherapi.com/",
              "content": "{\"location\": {\"name\": \"로스앤젤레스\", \"region\": \"캘리포니아\", \"country\": \"미국\", \"lat\": 34.05, \"lon\": -118.24, \"tz_id\": \"America/Los_Angeles\", \"localtime_epoch\": 1716310320, \"localtime\": \"2024-05-21 9:52\"}, \"current\": {\"last_updated_epoch\": 1716309900, \"last_updated\": \"2024-05-21 09:45\", \"temp_c\": 16.7, \"temp_f\": 62.1, \"is_day\": 1, \"condition\": {\"text\": \"흐림\", \"icon\": \"//cdn.weatherapi.com/weather/64x64/day/122.png\", \"code\": 1009}, \"wind_mph\": 8.1, \"wind_kph\": 13.0, \"wind_degree\": 250, \"wind_dir\": \"서남서\", \"pressure_mb\": 1015.0, \"pressure_in\": 29.97, \"precip_mm\": 0.0, \"precip_in\": 0.0, \"humidity\": 65, \"cloud\": 100, \"feelslike_c\": 16.7, \"feelslike_f\": 62.1, \"vis_km\": 16.0, \"vis_miles\": 9.0, \"uv\": 5.0, \"gust_mph\": 12.5, \"gust_kph\": 20.2}}"
            }
          ],
          "type": "tool",
          "name": "tavily_search_results_json",
          "tool_call_id": "toolu_01E5mSaZWm5rWJnCqmt63v4g"
          ...
        },
        {
          "content": "날씨 API 결과에 따르면 로스앤젤레스의 현재 날씨는 흐림이며, 기온은 약 17도(62도 화씨)입니다. 서남서 방향에서 불어오는 약한 바람이 시속 8-13마일 정도입니다. 습도는 65%이며, 가시거리는 9마일로 양호합니다. 전체적으로 로스앤젤레스의 온화한 봄 날씨입니다.",
          "type": "ai",
          ...
        }
      ]
    }