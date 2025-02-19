_한국어로 기계번역됨_

# 빠른 시작: 로컬 LangGraph 서버 실행하기

이것은 여러분이 LangGraph 앱을 로컬에서 실행할 수 있도록 돕는 빠른 시작 가이드입니다.

!!! 정보 "요구 사항"

    - Python >= 3.11
    - [LangGraph CLI](https://langchain-ai.github.io/langgraph/cloud/reference/cli/): langchain-cli[inmem] >= 0.1.58 필요

## LangGraph CLI 설치하기

```bash
pip install --upgrade "langgraph-cli[inmem]"
```

## 🌱 LangGraph 앱 생성하기

`react-agent` 템플릿에서 새 앱을 생성합니다. 이 템플릿은 여러 도구로 유연하게 확장할 수 있는 간단한 에이전트입니다.

=== "파이썬 서버"

    ```shell
    langgraph new path/to/your/app --template react-agent-python 
    ```

=== "노드 서버"

    ```shell
    langgraph new path/to/your/app --template react-agent-js
    ```

!!! 팁 "추가 템플릿"

    `langgraph new`를 템플릿을 지정하지 않고 사용하면 사용 가능한 템플릿 목록에서 선택할 수 있는 인터랙티브 메뉴가 제공됩니다.

## 종속성 설치하기

새 LangGraph 앱의 루트에서 서버에 로컬 변경 사항이 사용되도록 `edit` 모드에서 종속성을 설치합니다:

```shell
pip install -e .
```

## `.env` 파일 생성하기

새 LangGraph 앱의 루트에서 `.env.example` 파일을 찾을 수 있습니다. 이 파일의 내용을 복사하여 필요한 API 키를 입력한 후 `.env` 파일을 새 LangGraph 앱의 루트에 생성합니다:

```bash
LANGSMITH_API_KEY=lsv2...
TAVILY_API_KEY=tvly-...
ANTHROPIC_API_KEY=sk-
OPENAI_API_KEY=sk-...
```

??? 메모 "API 키 얻기"

    - **LANGSMITH_API_KEY**: [LangSmith 설정 페이지](https://smith.langchain.com/settings)로 가세요. 그런 다음 **API 키 생성**을 클릭하세요.
    - **ANTHROPIC_API_KEY**: [Anthropic](https://console.anthropic.com/)에서 API 키를 받으세요.
    - **OPENAI_API_KEY**: [OpenAI](https://openai.com/)에서 API 키를 받으세요.
    - **TAVILY_API_KEY**: [Tavily 웹사이트](https://app.tavily.com/)에서 API 키를 받으세요.

## 🚀 LangGraph 서버 실행하기

```shell
langgraph dev
```

이 명령어는 로컬에서 LangGraph API 서버를 시작합니다. 성공적으로 실행되면 다음과 같은 메시지가 표시됩니다:

>    준비 완료!
> 
>    - API: [http://localhost:2024](http://localhost:2024/)
>     
>    - 문서: http://localhost:2024/docs
>     
>    - LangGraph 스튜디오 웹 UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024

!!! 메모 "인메모리 모드"

    `langgraph dev` 명령어는 LangGraph 서버를 인메모리 모드에서 시작합니다. 이 모드는 개발 및 테스트 용도로 적합합니다. 프로덕션 사용의 경우, 지속적인 저장소 백엔드에 접근할 수 있는 LangGraph 서버를 배포해야 합니다.

    지속적인 저장소 백엔드로 애플리케이션을 테스트하고 싶다면 `langgraph dev` 대신 `langgraph up` 명령어를 사용할 수 있습니다. 이 명령을 사용하려면 머신에 `docker`가 설치되어 있어야 합니다.

## LangGraph 스튜디오 웹 UI

LangGraph 스튜디오 웹은 LangGraph API 서버에 연결하여 로컬에서 애플리케이션을 시각화, 상호작용 및 디버깅할 수 있는 전문 UI입니다. `langgraph dev` 명령어의 출력에 제공된 URL을 방문하여 LangGraph 스튜디오 웹 UI에서 그래프를 테스트하세요.

>    - LangGraph 스튜디오 웹 UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024

!!! 정보 "사용자 지정 호스트/포트로 서버에 연결하기"

    사용자 지정 호스트/포트로 LangGraph API 서버를 실행하는 경우, `baseUrl` URL 매개변수를 변경하여 스튜디오 웹 UI를 이를 가리키도록 할 수 있습니다. 예를 들어, 서버를 포트 8000에서 실행하는 경우 위의 URL을 다음과 같이 변경할 수 있습니다:

    ```
    https://smith.langchain.com/studio/baseUrl=http://127.0.0.1:8000
    ```


!!! 경고 "Safari 호환성"

현재 LangGraph Studio 웹은 로컬에서 서버를 실행할 때 Safari를 지원하지 않습니다.

## API 테스트

=== "파이썬 SDK (비동기)"

**LangGraph 파이썬 SDK 설치**

```shell
pip install langgraph-sdk
```

**도움말에게 메시지 보내기 (스레드 없는 실행)**

```python
from langgraph_sdk import get_client

client = get_client(url="http://localhost:2024")

async for chunk in client.runs.stream(
    None,  # 스레드 없는 실행
    "agent", # 도움말의 이름. langgraph.json에 정의됨.
    input={
        "messages": [{
            "role": "human",
            "content": "LangGraph란 무엇인가요?",
        }],
    },
    stream_mode="updates",
):
    print(f"유형: {chunk.event}의 새로운 이벤트 수신 중...")
    print(chunk.data)
    print("\n\n")
```

=== "파이썬 SDK (동기)"

**LangGraph 파이썬 SDK 설치**

```shell
pip install langgraph-sdk
```

**도움말에게 메시지 보내기 (스레드 없는 실행)**

```python
from langgraph_sdk import get_sync_client

client = get_sync_client(url="http://localhost:2024")

for chunk in client.runs.stream(
    None,  # 스레드 없는 실행
    "agent", # 도움말의 이름. langgraph.json에 정의됨.
    input={
        "messages": [{
            "role": "human",
            "content": "LangGraph란 무엇인가요?",
        }],
    },
    stream_mode="updates",
):
    print(f"유형: {chunk.event}의 새로운 이벤트 수신 중...")
    print(chunk.data)
    print("\n\n")
```

=== "자바스크립트 SDK"

**LangGraph JS SDK 설치**

```shell
npm install @langchain/langgraph-sdk
```

**도움말에게 메시지 보내기 (스레드 없는 실행)**

    ```js
    const { Client } = await import("@langchain/langgraph-sdk");

    // langgraph dev를 호출할 때 기본 포트를 변경한 경우에만 apiUrl을 설정합니다.
    const client = new Client({ apiUrl: "http://localhost:2024"});

    const streamResponse = client.runs.stream(
        null, // 스레드리스 실행
        "agent", // 어시스턴트 ID
        {
            input: {
                "messages": [
                    { "role": "user", "content": "LangGraph이란 무엇인가요?"}
                ]
            },
            streamMode: "messages",
        }
    );

    for await (const chunk of streamResponse) {
        console.log(`수신된 이벤트 유형: ${chunk.event}...`);
        console.log(JSON.stringify(chunk.data));
        console.log("\n\n");
    }
```

=== "REST API"

```bash
    curl -s --request POST \
        --url "http://localhost:2024/runs/stream" \
        --header 'Content-Type: application/json' \
        --data "{
            \"assistant_id\": \"agent\",
            \"input\": {
                \"messages\": [
                    {
                        \"role\": \"human\",
                        \"content\": \"LangGraph이란 무엇인가요?\"
                    }
                ]
            },
            \"stream_mode\": \"updates\"
        }" 
```

!!! tip "인증"

원격 서버에 연결하는 경우 LangSmith API 키를 제공해야 합니다. 자세한 내용은 클라이언트의 API 참조를 참조하세요.

## 다음 단계

이제 로컬에서 LangGraph 앱이 실행되고 있으니, 배포 및 고급 기능을 탐색하여 여정을 확장하세요:

### 🌐 LangGraph Cloud에 배포

- **[LangGraph Cloud 빠른 시작](../../cloud/quick_start.md)**: LangGraph Cloud를 사용하여 LangGraph 앱을 배포하세요.

### 📚 LangGraph 플랫폼에 대해 더 알아보기

다음 리소스를 통해 지식을 확장하세요:

- **[LangGraph 플랫폼 개념](../../concepts/index.md#langgraph-platform)**: LangGraph 플랫폼의 기본 개념을 이해하세요.  
- **[LangGraph 플랫폼 사용 방법 가이드](../../how-tos/index.md#langgraph-platform)**: 애플리케이션을 구축하고 배포하기 위한 단계별 가이드를 찾아보세요.

### 🛠️ 개발자 참조

개발 및 API 사용을 위한 자세한 문서를 확인하세요:

- **[LangGraph 서버 API 참조](../../cloud/reference/api/api_ref.html)**: LangGraph 서버 API 문서를 탐색하세요.  
- **[Python SDK 참조](../../cloud/reference/sdk/python_sdk_ref.md)**: Python SDK API 참조를 탐색하세요.
- **[JS/TS SDK 참조](../../cloud/reference/sdk/js_ts_sdk_ref.md)**: JS/TS SDK API 참조를 탐색하세요.
