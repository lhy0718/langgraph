_한국어로 기계번역됨_

# 상태 비저장 실행

대부분의 경우, LangGraph Cloud에서 구현된 지속 상태를 통해 이전 실행을 추적하기 위해 클라이언트에 `thread_id`를 제공합니다. 그러나 실행을 지속할 필요가 없다면 내장된 지속 상태를 사용할 필요가 없으며 상태 비저장 실행을 생성할 수 있습니다.

## 설정

먼저 클라이언트를 설정해 봅시다:

=== "파이썬"

    ```python
    from langgraph_sdk import get_client

    client = get_client(url=<DEPLOYMENT_URL>)
    # "agent"라는 이름으로 배포된 그래프 사용
    assistant_id = "agent"
    # 스레드 생성
    thread = await client.threads.create()
    ```

=== "자바스크립트"

    ```js
    import { Client } from "@langchain/langgraph-sdk";

    const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
    // "agent"라는 이름으로 배포된 그래프 사용
    const assistantId = "agent";
    // 스레드 생성
    const thread = await client.threads.create();
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <DEPLOYMENT_URL>/assistants/search \
        --header 'Content-Type: application/json' \
        --data '{
            "limit": 10,
            "offset": 0
        }' | jq -c 'map(select(.config == null or .config == {})) | .[0].graph_id' && \
    curl --request POST \
        --url <DEPLOYMENT_URL>/threads \
        --header 'Content-Type: application/json' \
        --data '{}'
    ```

## 상태 비저장 스트리밍

상태가 있는 실행에서 스트리밍하는 방법과 거의 동일한 방식으로 상태 비저장 실행의 결과를 스트리밍할 수 있지만, `thread_id` 매개변수에 값을 전달하는 대신 `None`을 전달합니다:

=== "파이썬"

    ```python
    input = {
        "messages": [
            {"role": "user", "content": "안녕하세요! 제 이름은 바가투르이고, 26세입니다."}
        ]
    }

    async for chunk in client.runs.stream(
        # thread_id를 전달하지 않으면 스트림은 상태 비저장이 됩니다
        None,
        assistant_id,
        input=input,
        stream_mode="updates",
    ):
        if chunk.data and "run_id" not in chunk.data:
            print(chunk.data)
    ```

=== "자바스크립트"

    ```js
    let input = {
      messages: [
        { role: "user", content: "안녕하세요! 제 이름은 바가투르이고, 26세입니다." }
      ]
    };

    const streamResponse = client.runs.stream(
      // thread_id를 전달하지 않으면 스트림은 상태 비저장이 됩니다
      null,
      assistantId,
      {
        input,
        streamMode: "updates"
      }
    );
    for await (const chunk of streamResponse) {
      if (chunk.data && !("run_id" in chunk.data)) {
        console.log(chunk.data);
      }
    }
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <DEPLOYMENT_URL>/runs/stream \
        --header 'Content-Type: application/json' \
        --data "{
            \"assistant_id\": \"agent\",
            \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"안녕하세요! 제 이름은 바가투르이고 26세입니다.\"}]},
            \"stream_mode\": [
                \"updates\"
            ]
        }" | jq -c 'select(.data and (.data | has("run_id") | not)) | .data'
    ```

출력:

    {'agent': {'messages': [{'content': "안녕하세요 바가투르! 만나서 반가워요. 자기소개와 나이를 알려줘서 감사해요. 궁금한 점이나 논의하고 싶은 특정한 주제가 있다면 말씀해 주세요. 제가 도와드릴 수 있는 질문이나 관심 있는 주제에 대해 준비되어 있습니다.", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-489ec573-1645-4ce2-a3b8-91b391d50a71', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}

## 무상태 결과 기다리기

스트리밍 외에도 `.wait` 함수를 사용해 무상태 결과를 기다릴 수 있습니다. 사용 예는 다음과 같습니다:

=== "Python"

    ```python
    stateless_run_result = await client.runs.wait(
        None,
        assistant_id,
        input=input,
    )
    print(stateless_run_result)
    ```

=== "Javascript"

    ```js
    let statelessRunResult = await client.runs.wait(
      null,
      assistantId,
      { input: input }
    );
    console.log(statelessRunResult);
    ```

=== "CURL"

    ```bash
    curl --request POST \
        --url <DEPLOYMENT_URL>/runs/wait \
        --header 'Content-Type: application/json' \
        --data '{
            "assistant_id": <ASSISTANT_IDD>,
        }'
    ```

출력:

    {
        'messages': [
            {
                'content': '안녕하세요! 제 이름은 바가투르이고 26세입니다.',
                'additional_kwargs': {},
                'response_metadata': {},
                'type': 'human',
                'name': None,
                'id': '5e088543-62c2-43de-9d95-6086ad7f8b48',
                'example': False}
            ,
            {
                'content': "안녕하세요 바가투르! 만나서 반가워요. 자기소개와 나이를 알려줘서 감사해요. 궁금한 점이나 논의하고 싶은 특정한 주제가 있다면 말씀해 주세요. 제가 도와드릴 수 있는 질문이나 탐색하고 싶은 주제가 있다면 준비되어 있습니다.",
                'additional_kwargs': {},
                'response_metadata': {},
                'type': 'ai',
                'name': None,
                'id': 'run-d6361e8d-4d4c-45bd-ba47-39520257f773',
                'example': False,
                'tool_calls': [],
                'invalid_tool_calls': [],
                'usage_metadata': None
            }
        ]
    }