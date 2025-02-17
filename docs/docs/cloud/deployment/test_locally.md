_한국어로 기계번역됨_

# LangGraph 앱 로컬 테스트 방법

이 가이드는 LangGraph 앱이 적절한 구성 파일과 해당하는 컴파일된 그래프를 가지고 올바르게 설정되어 있으며, 적절한 LangChain API 키를 가지고 있다고 가정합니다.

로컬 테스트를 수행하면 Python 종속성과 관련된 오류나 충돌이 없음을 확인하고 구성 파일이 올바르게 지정되었는지 확인할 수 있습니다.

## 설정

LangGraph CLI 패키지를 설치합니다:

```bash
pip install -U "langgraph-cli[inmem]"
```

API 키가 필요한데, 이는 [LangSmith UI](https://smith.langchain.com) (설정 > API 키)에서 생성할 수 있습니다. 이는 LangGraph Cloud 접근을 인증하기 위해 필요합니다. 키를 안전한 곳에 저장한 후, `.env` 파일에 다음 줄을 추가합니다:

```python
LANGSMITH_API_KEY = *********
```

## API 서버 시작

CLI를 설치한 후, 다음 명령어를 실행하여 로컬 테스트를 위한 API 서버를 시작할 수 있습니다:

```shell
langgraph dev
```

이 명령어가 성공적으로 실행되면 다음과 같은 메시지를 볼 수 있습니다:

>    준비 완료!
> 
>    - API: [http://localhost:2024](http://localhost:2024/)
>     
>    - 문서: http://localhost:2024/docs
>     
>    - LangGraph 스튜디오 웹 UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024

!!! 주의 "인메모리 모드"

    `langgraph dev` 명령어는 LangGraph 서버를 인메모리 모드로 시작합니다. 이 모드는 개발 및 테스트 용도로 적합합니다. 프로덕션 환경에서는 지속적인 저장소 백엔드에 접근할 수 있도록 LangGraph 서버를 배포해야 합니다.

    지속적인 저장소 백엔드에서 애플리케이션을 테스트하고 싶다면 `langgraph dev` 대신 `langgraph up` 명령어를 사용할 수 있습니다. 이 명령어를 사용하려면 머신에 `docker`가 설치되어 있어야 합니다.


### 서버와 상호작용하기

이제 LangGraph SDK를 사용하여 API 서버와 상호작용할 수 있습니다. 먼저 클라이언트를 시작하고, 우리 보조가 사용할 그래프(여기서는 "agent"라는 그래프)를 선택해야 합니다. 테스트할 보조를 올바르게 선택하세요.

인증 정보를 전달하거나 환경 변수를 설정하여 초기화할 수 있습니다.

#### 인증으로 초기화

=== "Python"

    ```python
    from langgraph_sdk import get_client

    # langgraph dev를 호출할 때 기본 포트를 변경한 경우에만 url 인자를 get_client()에 전달하세요.
    client = get_client(url=<DEPLOYMENT_URL>, api_key=<LANGSMITH_API_KEY>)
    # 이름이 "agent"인 그래프를 사용합니다.
    assistant_id = "agent"
    thread = await client.threads.create()
    ```

=== "Javascript"

    ```js
    import { Client } from "@langchain/langgraph-sdk";

    // langgraph dev를 호출할 때 기본 포트를 변경한 경우에만 apiUrl을 설정하세요.
    const client = new Client({ apiUrl: <DEPLOYMENT_URL>, apiKey: <LANGSMITH_API_KEY> });
    // 이름이 "agent"인 그래프를 사용합니다.
    const assistantId = "agent";
    const thread = await client.threads.create();
    ```

=== "CURL"

    ```bash
    curl --request POST \
      --url <DEPLOYMENT_URL>/threads \
      --header 'Content-Type: application/json' \
      --header 'x-api-key: <LANGSMITH_API_KEY>'
    ```

#### 환경 변수로 초기화

환경에 `LANGSMITH_API_KEY`가 설정되어 있는 경우, 클라이언트에 인증 정보를 명시적으로 전달할 필요가 없습니다.

=== "Python"

    ```python
    from langgraph_sdk import get_client

    # langgraph dev 호출 시 기본 포트를 변경하지 않았다면 url 인자만 get_client()에 전달합니다.
    client = get_client()
    # "agent"라는 이름으로 배포된 그래프 사용
    assistant_id = "agent"
    thread = await client.threads.create()
    ```

=== "Javascript"

    ```js
    import { Client } from "@langchain/langgraph-sdk";

    // langgraph dev 호출 시 기본 포트를 변경하지 않았다면 apiUrl을 설정하지 마세요.
    const client = new Client();
    // "agent"라는 이름으로 배포된 그래프 사용
    const assistantId = "agent";
    const thread = await client.threads.create();
    ```

=== "CURL"

    ```bash
    curl --request POST \
      --url <DEPLOYMENT_URL>/threads \
      --header 'Content-Type: application/json'
    ```

이제 그래프를 호출하여 제대로 작동하는지 확인할 수 있습니다. 입력을 그래프의 올바른 스키마에 맞게 변경하는 것을 잊지 마세요.

=== "Python"

    ```python
    input = {"messages": [{"role": "user", "content": "샌프란시스코 날씨 어때?"}]}
    async for chunk in client.runs.stream(
        thread["thread_id"],
        assistant_id,
        input=input,
        stream_mode="updates",
    ):
        print(f"새로운 이벤트 수신: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```
=== "Javascript"

    ```js
    const input = { "messages": [{ "role": "user", "content": "샌프란시스코 날씨 어때?"}] }

    const streamResponse = client.runs.stream(
      thread["thread_id"],
      assistantId,
      {
        input: input,
        streamMode: "updates",
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
     --url <DEPLOYMENT_URL>/threads/<THREAD_ID>/runs/stream \
     --header 'Content-Type: application/json' \
     --data "{
       \"assistant_id\": \"agent\",
       \"input\": {\"messages\": [{\"role\": \"human\", \"content\": \"샌프란시스코 날씨 어때?\"}]},
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
         sub(/^event: /, "이벤트 수신: ", $0)
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

그래프가 제대로 작동하면 콘솔에 그래프 출력이 표시되어야 합니다. 물론 그래프를 테스트해야 할 수 있는 방법은 훨씬 더 많습니다. SDK로 보낼 수 있는 명령어의 전체 목록은 [Python](https://langchain-ai.github.io/langgraph/cloud/reference/sdk/python_sdk_ref/) 및 [JS/TS](https://langchain-ai.github.io/langgraph/cloud/reference/sdk/js_ts_sdk_ref/) 참조를 확인하세요.
