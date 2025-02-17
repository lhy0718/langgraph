_한국어로 기계번역됨_

# 동일 스레드에서 여러 에이전트를 실행하는 방법

LangGraph Cloud에서 스레드는 특정 에이전트와 명시적으로 연결되지 않습니다.  
즉, 동일한 스레드에서 여러 에이전트를 실행할 수 있으며, 이는 다른 에이전트가 초기 에이전트의 진행 상황에서 계속 진행할 수 있게 해줍니다.

이번 예제에서는 두 개의 에이전트를 생성한 다음 두 에이전트를 모두 동일한 스레드에서 호출할 것입니다.  
두 번째 에이전트가 첫 번째 에이전트가 스레드에서 생성한 [체크포인트](https://langchain-ai.github.io/langgraph/concepts/low_level/#checkpointer-state)의 정보를 컨텍스트로 사용하여 응답하는 것을 볼 수 있습니다.

## 설정

=== "Python"

    ```python
    from langgraph_sdk import get_client

    client = get_client(url=<DEPLOYMENT_URL>)

    openai_assistant = await client.assistants.create(
        graph_id="agent", config={"configurable": {"model_name": "openai"}}
    )

    # 항상 구성 없이 기본 보조자가 있어야 합니다.
    assistants = await client.assistants.search()
    default_assistant = [a for a in assistants if not a["config"]][0]
    ```

=== "Javascript"

    ```js
    import { Client } from "@langchain/langgraph-sdk";

    const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
    
    const openAIAssistant = await client.assistants.create(
      { graphId: "agent", config: {"configurable": {"model_name": "openai"}}}
    );

    const assistants = await client.assistants.search();
    const defaultAssistant = assistants.find(a => !a.config);
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <DEPLOYMENT_URL>/assistants \
        --header 'Content-Type: application/json' \
        --data '{
            "graph_id": "agent",
            "config": { "configurable": { "model_name": "openai" } }
        }' && \
    curl --request POST \
        --url <DEPLOYMENT_URL>/assistants/search \
        --header 'Content-Type: application/json' \
        --data '{
            "limit": 10,
            "offset": 0
        }' | jq -c 'map(select(.config == null or .config == {})) | .[0]'
    ```

이 에이전트들이 서로 다름을 확인할 수 있습니다:

=== "Python"

    ```python
    print(openai_assistant)
    ```

=== "Javascript"

    ```js
    console.log(openAIAssistant);
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <DEPLOYMENT_URL>/assistants/<OPENAI_ASSISTANT_ID>
    ```

출력:

    {
        "assistant_id": "db87f39d-b2b1-4da8-ac65-cf81beb3c766",
        "graph_id": "agent",
        "created_at": "2024-08-30T21:18:51.850581+00:00",
        "updated_at": "2024-08-30T21:18:51.850581+00:00",
        "config": {
            "configurable": {
                "model_name": "openai"
            }
        },
        "metadata": {}
    }

=== "Python"

    ```python
    print(default_assistant)
    ```

=== "자바스크립트"

    ```js
    console.log(defaultAssistant);
    ```

=== "CURL"

    ```bash
    curl --request GET \
        --url <DEPLOYMENT_URL>/assistants/<DEFAULT_ASSISTANT_ID>
    ```

출력:

    {
        "assistant_id": "fe096781-5601-53d2-b2f6-0d3403f7e9ca",
        "graph_id": "agent",
        "created_at": "2024-08-08T22:45:24.562906+00:00",
        "updated_at": "2024-08-08T22:45:24.562906+00:00",
        "config": {},
        "metadata": {
            "created_by": "system"
        }
    }

## 스레드에서 어시스턴트 실행

### OpenAI 어시스턴트 실행

이제 스레드에서 OpenAI 어시스턴트를 먼저 실행할 수 있습니다.

=== "파이썬"

    ```python
    thread = await client.threads.create()
    input = {"messages": [{"role": "user", "content": "누가 너를 만들었니?"}]}
    async for event in client.runs.stream(
        thread["thread_id"],
        openai_assistant["assistant_id"],
        input=input,
        stream_mode="updates",
    ):
        print(f"수신한 이벤트 유형: {event.event}")
        print(event.data)
        print("\n\n")
    ```

=== "자바스크립트"

    ```js
    const thread = await client.threads.create();
    let input =  {"messages": [{"role": "user", "content": "누가 너를 만들었니?"}]}

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      openAIAssistant["assistant_id"],
      {
        input,
        streamMode: "updates"
      }
    );
    for await (const event of streamResponse) {
      console.log(`수신한 이벤트 유형: ${event.event}`);
      console.log(event.data);
      console.log("\n\n");
    }
    ```

=== "CURL"

    ```bash
    thread_id=$(curl --request POST \
        --url <배포_URL>/threads \
        --header 'Content-Type: application/json' \
        --data '{}' | jq -r '.thread_id') && \
    curl --request POST \
        --url "<배포_URL>/threads/${thread_id}/runs/stream" \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <OPENAI_ASSISTANT_ID>,
            "input": {
                "messages": [
                    {
                        "role": "user",
                        "content": "누가 너를 만들었니?"
                    }
                ]
            },
            "stream_mode": [
                "업데이트"
            ]
        }' | \
        sed 's/\r$//' | \
        awk '
        /^event:/ {
            if (data_content != "") {
                print data_content "\n"
            }
            sub(/^event: /, "받은 이벤트 유형: ", $0)
            printf "%s...\n", $0
            data_content = ""
        }
        /^data:/ {
            sub(/^data: /, "", $0)
            data_content = $0
        }
        END {
            if (data_content != "") {
                print data_content "\n\n"
            }
        }
    '
    ```

출력:

    받은 이벤트 유형: 메타데이터
    {'run_id': '1ef671c5-fb83-6e70-b698-44dba2d9213e'}


    받은 이벤트 유형: 업데이트
    {'agent': {'messages': [{'content': '저는 OpenAI에 의해 만들어졌습니다. OpenAI는 인공지능 기술을 개발하고 발전시키는 데 초점을 맞춘 연구 기관입니다.', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_157b3831f5'}, 'type': 'ai', 'name': None, 'id': 'run-f5735b86-b80d-4c71-8dc3-4782b5a9c7c8', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}

### 기본 어시스턴트 실행

이제 기본 어시스턴트로 실행하여 두 번째 어시스턴트가 초기 질문을 알고 있으며, "너는?"이라는 질문에 답할 수 있는지 확인해 보겠습니다:

=== "파이썬"

    ```python
    입력 = {"messages": [{"role": "user", "content": "너는?"}]}
    async for event in client.runs.stream(
        thread["thread_id"],
        default_assistant["assistant_id"],
        input=입력,
        stream_mode="업데이트",
    ):
        print(f"받은 이벤트 유형: {event.event}")
        print(event.data)
        print("\n\n")
    ```

=== "자바스크립트"

    ```js
    let input =  {"messages": [{"role": "user", "content": "너는?"}]}

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      defaultAssistant["assistant_id"],
      {
        input,
        streamMode: "업데이트"
      }
    );
    for await (const event of streamResponse) {
      console.log(`받은 이벤트 유형: ${event.event}`);
      console.log(event.data);
      console.log("\n\n");
    }
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <배포_URL>/threads/<스레드_ID>/runs/stream \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <기본_어시스턴트_ID>,
            "input": {
                "messages": [
                    {
                        "role": "user",
                        "content": "그리고 당신은?"
                    }
                ]
            },
            "stream_mode": [
                "업데이트"
            ]
        }' | \
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
                print data_content "\n\n"
            }
        }
    '
    ```

출력:

    유형의 이벤트 수신: 메타데이터
    {'run_id': '1ef6722d-80b3-6fbb-9324-253796b1cd13'}


    유형의 이벤트 수신: 업데이트
    {'agent': {'messages': [{'content': [{'text': '나는 OpenAI가 아닌 Anthropic에 의해 생성된 인공지능입니다. OpenAI가 나를 만들었다고 말해서는 안 되며, 그것은 사실이 아닙니다. Anthropic은 나를 발전된 언어 모델과 AI 기술을 사용하여 개발하고 훈련한 회사입니다. 앞으로 내 기원에 대한 정확한 정보를 제공하는 데 더 주의하겠습니다.', 'type': 'text', 'index': 0}], 'additional_kwargs': {}, 'response_metadata': {'stop_reason': 'end_turn', 'stop_sequence': None}, 'type': 'ai', 'name': None, 'id': 'run-ebaacf62-9dd9-4165-9535-db432e4793ec', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': {'input_tokens': 302, 'output_tokens': 72, 'total_tokens': 374}}]}}



