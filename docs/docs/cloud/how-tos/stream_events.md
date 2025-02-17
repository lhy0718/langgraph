_한국어로 기계번역됨_

# 이벤트 스트리밍 방법

!!! 정보 "전제 조건"
    * [스트리밍](../../concepts/streaming.md#streaming-llm-tokens-and-events-astream_events)

이 가이드는 그래프에서 이벤트를 스트리밍하는 방법을 다룹니다(`stream_mode="events"`). 사용 사례와 LangGraph 애플리케이션의 사용자 경험에 따라 애플리케이션은 이벤트 유형을 다르게 처리할 수 있습니다.

## 설정

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
        'thread_id': '3f4c64e0-f792-4a5e-aa07-a4404e06e0bd',
        'created_at': '2024-06-24T22:16:29.301522+00:00',
        'updated_at': '2024-06-24T22:16:29.301522+00:00',
        'metadata': {},
        'status': 'idle',
        'config': {},
        'values': None
    }

## 이벤트 모드로 그래프 스트리밍

이벤트를 스트리밍하면 `data`와 같은 다른 키와 함께 `event` 키가 포함된 응답이 생성됩니다. 모든 이벤트 유형에 대한 LangChain [`Runnable.astream_events()` 참고 문서](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream_events)를 참조하세요.

=== "파이썬"

    ```python
    # 입력 생성
    input = {
        "messages": [
            {
                "role": "user",
                "content": "샌프란시스코의 날씨는 어때?",
            }
        ]
    }

    # 이벤트 스트리밍
    async for chunk in client.runs.stream(
        thread_id=thread["thread_id"],
        assistant_id=assistant_id,
        input=input,
        stream_mode="events",
    ):
        print(f"새로운 이벤트 유형 수신: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "자바스크립트"

    ```js
    // 입력 생성
    const input = {
      "messages": [
        {
          "role": "user",
          "content": "샌프란시스코의 날씨 어때요?",
        }
      ]
    }

    // 이벤트 스트리밍
    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantID,
      {
        input,
        streamMode: "events"
      }
    );
    for await (const chunk of streamResponse) {
      console.log(`새로운 이벤트 수신: ${chunk.event}...`);
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
   \"assistant_id\": \"agent\",
   \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코의 날씨 어때요?\"}]},
   \"stream_mode\": [
     \"events\"
   ]
 }" | \
 sed 's/\r$//' | \
 awk '
 /^event:/ {
     if (data_content != "") {
         print data_content "\n"
     }
     sub(/^event: /, "새로운 이벤트 수신: ", $0)
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

    새로운 이벤트 수신: 메타데이터...
    {'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8'}
    
    새로운 이벤트 수신: 이벤트...
    {'event': 'on_chain_start', 'data': {'input': {'messages': [{'role': 'human', 'content': "샌프란시스코의 날씨 어때요?"}]}}, 'name': 'LangGraph', 'tags': [], 'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', 'metadata': {'graph_id': 'agent', 'created_by': 'system', 'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', 'user_id': '', 'thread_id': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'parent_ids': []}
    
    새로운 이벤트 수신: 이벤트...
    {'event': 'on_chain_start', 'data': {}, 'name': 'agent', 'tags': ['graph:step:6'], 'run_id': '7bb08493-d507-4e28-b9e6-4a5eda9d04f0', 'metadata': {'graph_id': 'agent', 'created_by': 'system', 'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', 'user_id': '', 'thread_id': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0}, 'parent_ids': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']}
    
    ...
    (중략)
    ...
    
    새로운 이벤트 수신: 이벤트...
    {'event': 'on_chat_model_stream', 'data': {'chunk': {'content': 'g', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'AIMessageChunk', 'name': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None, 'tool_call_chunks': []}}, 'run_id': 'cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', 'name': 'FakeListChatModel', 'tags': ['seq:step:1'], 'metadata': {'graph_id': 'agent', 'created_by': 'system', 'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', 'user_id': '', 'thread_id': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0}, 'parent_ids': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '7bb08493-d507-4e28-b9e6-4a5eda9d04f0']}
    
    ...
    {'이벤트': 'on_chat_model_stream', '데이터': {'청크': {'내용': 'i', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'AIMessageChunk', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None, '툴_호출_청크': []}}, '런_ID': 'cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '7bb08493-d507-4e28-b9e6-4a5eda9d04f0']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_stream', '데이터': {'청크': {'내용': 'n', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'AIMessageChunk', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None, '툴_호출_청크': []}}, '런_ID': 'cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '7bb08493-d507-4e28-b9e6-4a5eda9d04f0']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_end', '데이터': {'출력': {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, '입력': {'메시지': [[{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}]]}}, '런_ID': 'cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '7bb08493-d507-4e28-b9e6-4a5eda9d04f0']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_start', '데이터': {'입력': {'메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}}, '이름': 'should_continue', '태그': ['seq:step:3'], '런_ID': 'c7fe4d2d-3fb8-4e53-946d-03de13527853', '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '7bb08493-d507-4e28-b9e6-4a5eda9d04f0']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_end', '데이터': {'출력': 'tool', '입력': {'메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}}, '런_ID': 'c7fe4d2d-3fb8-4e53-946d-03de13527853', '이름': 'should_continue', '태그': ['seq:step:3'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '7bb08493-d507-4e28-b9e6-4a5eda9d04f0']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_stream', '런_ID': '7bb08493-d507-4e28-b9e6-4a5eda9d04f0', '이름': 'agent', '태그': ['graph:step:6'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0}, '데이터': {'청크': {'메시지': [{'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_end', '데이터': {'출력': {'메시지': [{'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}, '입력': {'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}, '메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}], '수면': None}}, '런_ID': '7bb08493-d507-4e28-b9e6-4a5eda9d04f0', '이름': 'agent', '태그': ['graph:step:6'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 6, 'langgraph_node': 'agent', 'langgraph_triggers': ['start:agent'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_start', '데이터': {}, '이름': 'tool', '태그': ['graph:step:7'], '런_ID': 'f044fd3d-7271-488f-b8aa-e01572ff9112', '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 7, 'langgraph_node': 'tool', 'langgraph_triggers': ['branch:agent:should_continue:tool'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_stream', '런_ID': 'f044fd3d-7271-488f-b8aa-e01572ff9112', '이름': 'tool', '태그': ['graph:step:7'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 7, 'langgraph_node': 'tool', 'langgraph_triggers': ['branch:agent:should_continue:tool'], 'langgraph_task_idx': 0}, '데이터': {'청크': {'메시지': [{'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': None, '툴_호출_ID': 'tool_call_id'}]}}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_end', '데이터': {'출력': {'메시지': [{'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}]}, '입력': {'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}, '메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], '수면': None}}, '런_ID': 'f044fd3d-7271-488f-b8aa-e01572ff9112', '이름': 'tool', '태그': ['graph:step:7'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 7, 'langgraph_node': 'tool', 'langgraph_triggers': ['branch:agent:should_continue:tool'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_start', '데이터': {}, '이름': 'agent', '태그': ['graph:step:8'], '런_ID': '1f4f95d0-0ce1-4061-85d4-946446bbd3e5', '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_start', '데이터': {'입력': {'메시지': [[{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}]]}}, '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '런_ID': '028a68fb-6435-4b46-a156-c3326f73985c', '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_stream', '데이터': {'청크': {'내용': 'e', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'AIMessageChunk', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None, '툴_호출_청크': []}}, '런_ID': '028a68fb-6435-4b46-a156-c3326f73985c', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_stream', '데이터': {'청크': {'내용': 'n', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'AIMessageChunk', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None, '툴_호출_청크': []}}, '런_ID': '028a68fb-6435-4b46-a156-c3326f73985c', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_stream', '데이터': {'청크': {'내용': 'd', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'AIMessageChunk', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None, '툴_호출_청크': []}}, '런_ID': '028a68fb-6435-4b46-a156-c3326f73985c', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chat_model_end', '데이터': {'출력': {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, '입력': {'메시지': [[{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}]]}}, '런_ID': '028a68fb-6435-4b46-a156-c3326f73985c', '이름': 'FakeListChatModel', '태그': ['seq:step:1'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0, 'ls_model_type': 'chat'}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_start', '데이터': {'입력': {'메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}}, '이름': 'should_continue', '태그': ['seq:step:3'], '런_ID': 'f2b2dfaf-475d-422b-8bf5-02a31bcc7d1a', '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_end', '데이터': {'출력': '__end__', '입력': {'메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}}, '런_ID': 'f2b2dfaf-475d-422b-8bf5-02a31bcc7d1a', '이름': 'should_continue', '태그': ['seq:step:3'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '1f4f95d0-0ce1-4061-85d4-946446bbd3e5']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_stream', '런_ID': '1f4f95d0-0ce1-4061-85d4-946446bbd3e5', '이름': 'agent', '태그': ['graph:step:8'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0}, '데이터': {'청크': {'메시지': [{'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']} 

새로운 이벤트 수신 중: events...
{'이벤트': 'on_chain_end', '데이터': {'출력': {'메시지': [{'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}], 'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}}, '입력': {'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}, '메시지': [{'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', '툴_호출_ID': 'tool_call_id'}, {'내용': 'end', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': "What's the weather in SF?", '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'human', '이름': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', '예시': False}, {'내용': 'begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'ai', '이름': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', '예시': False, '툴_호출': [], '잘못된_툴_호출': [], '사용_메타데이터': None}, {'내용': 'tool_call__begin', '추가_키워드': {}, '응답_메타데이터': {}, '유형': 'tool', '이름': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', '툴_호출_ID': 'tool_call_id'}], '수면': None}}, '런_ID': '1f4f95d0-0ce1-4061-85d4-946446bbd3e5', '이름': 'agent', '태그': ['graph:step:8'], '메타데이터': {'그래프_ID': 'agent', '생성자': 'system', '런_ID': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', '사용자_ID': '', '스레드_ID': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', '어시스턴트_ID': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca', 'langgraph_step': 8, 'langgraph_node': 'agent', 'langgraph_triggers': ['tool'], 'langgraph_task_idx': 0}, '부모_IDs': ['1ef301a5-b867-67de-9e9e-a32e53c5b1f8']}
    
    
    
    새로운 이벤트 유형 수신: 이벤트...
    {'event': 'on_chain_end', 'data': {'output': {'some_bytes': 'c29tZV9ieXRlcw==', 'some_byte_array': 'c29tZV9ieXRlX2FycmF5', 'dict_with_bytes': {'more_bytes': 'bW9yZV9ieXRlcw=='}, 'messages': [{'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': '51f2874d-f8c7-4040-8b3b-8f15429a56ae', 'example': False}, {'content': '시작', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-5f556aa0-26ea-42e2-b9e4-7ece3a00974e', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}, {'content': 'tool_call__begin', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': None, 'id': '1faf5dd0-ae97-4235-963f-5075083a027a', 'tool_call_id': 'tool_call_id'}, {'content': '끝', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-ae383611-6a42-475a-912a-09d5972e9e94', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}, {'content': "샌프란시스코의 날씨는 어때?", 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'human', 'name': None, 'id': 'c67e08e6-e7af-4c4a-aa5e-50c8340ae341', 'example': False}, {'content': '시작', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-cb1b98c1-c9e2-4a30-9d7a-38fa1f6224bd', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}, {'content': 'tool_call__begin', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'tool', 'name': None, 'id': '1c9a16d2-5f0a-4eba-a0d2-240484a4ce7e', 'tool_call_id': 'tool_call_id'}, {'content': '끝', 'additional_kwargs': {}, 'response_metadata': {}, 'type': 'ai', 'name': None, 'id': 'run-028a68fb-6435-4b46-a156-c3326f73985c', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}, 'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', 'name': 'LangGraph', 'tags': [], 'metadata': {'graph_id': 'agent', 'created_by': 'system', 'run_id': '1ef301a5-b867-67de-9e9e-a32e53c5b1f8', 'user_id': '', 'thread_id': '7196a3aa-763c-4a8d-bfda-12fbfe1cd727', 'assistant_id': 'fe096781-5601-53d2-b2f6-0d3403f7e9ca'}, 'parent_ids': []}
    
새로운 이벤트 유형 수신: 종료...
    없음