_한국어로 기계번역됨_

# 빠른 시작: LangGraph Cloud에 배포하기

!!! 주의 "사전 준비사항"

    시작하기 전에 다음 사항을 확인하세요:

    - [GitHub 계정](https://github.com/)
    - [LangSmith 계정](https://smith.langchain.com/)

## GitHub에서 저장소 만들기

**LangGraph Cloud**에 LangGraph 애플리케이션을 배포하려면 애플리케이션 코드가 GitHub 저장소에 있어야 합니다. 공개 및 비공식 저장소 모두 지원됩니다.

모든 [LangGraph 애플리케이션](../concepts/application_structure.md)을 LangGraph Cloud에 배포할 수 있습니다.

이번 가이드는 미리 구축된 Python [**ReAct Agent**](https://github.com/langchain-ai/react-agent) 템플릿을 사용할 것입니다.

??? 주의 "ReAct Agent 템플릿에 필요한 API 키 받기"

    이 **ReAct Agent** 애플리케이션은 [Anthropic](https://console.anthropic.com/)과 [Tavily](https://app.tavily.com/)에서 API 키를 요구합니다. 각 웹사이트에 가입하여 이러한 API 키를 받을 수 있습니다.

    **대안**: API 키가 필요 없는 기본 응용 프로그램을 원하신다면, **ReAct Agent** 템플릿 대신 [**New LangGraph Project**](https://github.com/langchain-ai/new-langgraph-project) 템플릿을 사용하세요.


1. [ReAct Agent](https://github.com/langchain-ai/react-agent) 저장소로 이동합니다.
2. 오른쪽 상단의 `Fork` 버튼을 클릭하여 저장소를 GitHub 계정으로 포크합니다.

## LangGraph Cloud에 배포하기

??? 주의 "1. [LangSmith](https://smith.langchain.com/)에 로그인하기"

    <figure markdown="1">
    [![LangSmith 로그인](deployment/img/01_login.png){: style="max-height:300px"}](deployment/img/01_login.png)
    <figcaption>
    [LangSmith](https://smith.langchain.com/)에 가서 로그인합니다. 계정이 없다면 무료로 가입할 수 있습니다.
    </figcaption>
    </figure>


??? 주의 "2. 왼쪽 사이드바에서 <em>LangGraph Platform</em> 클릭하기"

    <figure markdown="1">
    [![LangGraph 플랫폼](deployment/img/02_langgraph_platform.png){: style="max-height:300px"}](deployment/img/02_langgraph_platform.png)
    <figcaption>
    왼쪽 사이드바에서 **LangGraph Platform**을 선택합니다.
    </figcaption>
    </figure>

??? 주의 "3. 오른쪽 상단의 + New Deployment 클릭하기"

    <figure markdown="1">
    [![배포 페이지](deployment/img/03_deployments_page.png){: style="max-height:300px"}](deployment/img/03_deployments_page.png)
    <figcaption>
    **+ New Deployment**를 클릭하여 새로운 배포를 생성합니다. 이 버튼은 오른쪽 상단 모서리에 위치해 있습니다.
    필요한 필드를 작성할 수 있는 새로운 모달이 열립니다.
    </figcaption>
    </figure>

??? 주의 "4. GitHub에서 가져오기 클릭하기 (첫 사용자)"

    <figure markdown="1">
    [![배포 생성](deployment/img/04_create_new_deployment.png)](deployment/img/04_create_new_deployment.png)
    <figcaption>
    **Import from GitHub**를 클릭하고 GitHub 계정을 연결하기 위한 지침을 따릅니다. 이 단계는 **첫 사용자**를 위한 것이며, 이전에 연결되지 않은 비공식 저장소를 추가할 때 필요합니다.</figcaption>
    </figure>

??? 주의 "5. 저장소 선택, ENV 변수 구성 등"

    <figure markdown="1">
    [![배포 구성](deployment/img/05_configure_deployment.png){: style="max-height:300px"}](deployment/img/05_configure_deployment.png)
    <figcaption>
    <strong>저장소</strong>를 선택하고 환경 변수 및 비밀번호를 추가하고 다른 설정 옵션을 설정합니다.
    </figcaption>
    </figure>

    - **저장소**: 이전에 포크한 저장소(또는 배포하려는 다른 저장소)를 선택합니다.
    - 애플리케이션에서 필요한 비밀번호와 환경 변수를 설정합니다. **ReAct Agent** 템플릿의 경우, 다음 비밀번호를 설정해야 합니다:
        - **ANTHROPIC_API_KEY**: [Anthropic](https://console.anthropic.com/)에서 API 키를 받습니다.
        - **TAVILY_API_KEY**: [Tavily 웹사이트](https://app.tavily.com/)에서 API 키를 받습니다.

??? 주의 "6. Deploy!를 눌러 제출하세요!"

    <figure markdown="1">
    [![이미지](deployment/img/05_configure_deployment.png){: style="max-height:300px"}](deployment/img/05_configure_deployment.png)
    <figcaption>
        이 단계는 완료하는 데 약 15분이 걸릴 수 있습니다. **배포** 보기에서 배포 상태를 확인할 수 있습니다.
        애플리케이션을 배포하려면 오른쪽 상단 모서리에 있는 <strong>Submit</strong> 버튼을 클릭합니다.
    </figcaption>
    </figure>


## LangGraph Studio 웹 UI

애플리케이션이 배포되면 **LangGraph Studio**에서 테스트할 수 있습니다.

??? 주의 "1. 기존 배포 클릭하기"

    <figure markdown="1">
    [![배포 페이지](deployment/img/07_deployments_page.png){: style="max-height:300px"}](deployment/img/07_deployments_page.png)
    <figcaption>
        방금 생성한 배포를 클릭하여 자세한 내용을 확인하세요.
    </figcaption>
    </figure>

??? 노트 "2. LangGraph Studio를 클릭하세요"

    <figure markdown="1">
    [![image](deployment/img/08_deployment_view.png){: style="max-height:300px"}](deployment/img/08_deployment_view.png)
    <figcaption>
        <strong>LangGraph Studio</strong> 버튼을 클릭하여 LangGraph Studio를 엽니다.
    </figcaption>
    </figure>

<figure markdown="1">
[![image](deployment/img/09_langgraph_studio.png){: style="max-height:400px"}](deployment/img/09_langgraph_studio.png)
<figcaption>
    LangGraph Studio에서 실행된 샘플 그래프.
</figcaption>
</figure>

## API 테스트하기

!!! 노트

    아래 API 호출은 **ReAct Agent** 템플릿에 해당합니다. 다른 애플리케이션을 배포하는 경우 API 호출을 적절히 조정해야 할 수 있습니다.

사용하기 전에 LangGraph 배포의 `URL`을 가져와야 합니다. 이는 `Deployment` 뷰에서 확인할 수 있습니다. `URL`을 클릭하여 클립보드에 복사하세요.

또한 LangGraph Cloud에 인증할 수 있도록 API 키를 올바르게 설정했는지 확인해야 합니다.

```shell
export LANGSMITH_API_KEY=...
```

=== "Python SDK (비동기)"

    **LangGraph Python SDK 설치하기**

    ```shell
    pip install langgraph-sdk
    ```

    **비스레드 방식으로 어시스턴트에게 메시지 보내기**

    ```python
    from langgraph_sdk import get_client

    client = get_client(url="your-deployment-url", api_key="your-langsmith-api-key")

    async for chunk in client.runs.stream(
        None,  # 비스레드 실행
        "agent", # 어시스턴트의 이름. langgraph.json에서 정의됨.
        input={
            "messages": [{
                "role": "human",
                "content": "LangGraph란 무엇인가요?",
            }],
        },
        stream_mode="updates",
    ):
        print(f"새 이벤트 수신: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "Python SDK (동기)"

    **LangGraph Python SDK 설치하기**

    ```shell
    pip install langgraph-sdk
    ```

    **비스레드 방식으로 어시스턴트에게 메시지 보내기**

    ```python
    from langgraph_sdk import get_sync_client

    client = get_sync_client(url="your-deployment-url", api_key="your-langsmith-api-key")

    for chunk in client.runs.stream(
        None,  # 비스레드 실행
        "agent", # 어시스턴트의 이름. langgraph.json에서 정의됨.
        input={
            "messages": [{
                "role": "human",
                "content": "LangGraph란 무엇인가요?",
            }],
        },
        stream_mode="updates",
    ):
        print(f"새 이벤트 수신: {chunk.event}...")
        print(chunk.data)
        print("\n\n")
    ```

=== "자바스크립트 SDK"

    **LangGraph JS SDK 설치하기**

```shell
npm install @langchain/langgraph-sdk
```

**어시스턴트에게 메시지 보내기 (스레드리스 실행)**

```js
const { Client } = await import("@langchain/langgraph-sdk");

const client = new Client({ apiUrl: "your-deployment-url", apiKey: "your-langsmith-api-key" });

const streamResponse = client.runs.stream(
    null, // 스레드리스 실행
    "agent", // 어시스턴트 ID
    {
        input: {
            "messages": [
                { "role": "user", "content": "LangGraph란 무엇인가요?"}
            ]
        },
        streamMode: "messages",
    }
);

for await (const chunk of streamResponse) {
    console.log(`새로운 이벤트 수신 중: ${chunk.event}...`);
    console.log(JSON.stringify(chunk.data));
    console.log("\n\n");
}
```

=== "REST API"

```bash
curl -s --request POST \
    --url <DEPLOYMENT_URL> \
    --header 'Content-Type: application/json' \
    --data "{
        \"assistant_id\": \"agent\",
        \"input\": {
            \"messages\": [
                {
                    \"role\": \"human\",
                    \"content\": \"LangGraph란 무엇인가요?\"
                }
            ]
        },
        \"stream_mode\": \"updates\"
    }" 
```

## 다음 단계

축하합니다! 이 튜토리얼을 모두 완료했다면 LangGraph Cloud 전문가가 되는 길에 잘 나아가고 있습니다. 전문가가 되는 길에 도움이 될 다른 리소스를 확인해 보세요:

### LangGraph 프레임워크

- **[LangGraph 튜토리얼](../tutorials/introduction.ipynb)**: LangGraph 프레임워크 시작하기.
- **[LangGraph 개념](../concepts/index.md)**: LangGraph의 기본 개념을 배워보세요.
- **[LangGraph 사용 가이드](../how-tos/index.md)**: LangGraph로 공통 작업을 수행하기 위한 가이드.

### 📚 LangGraph 플랫폼에 대해 더 알아보기

다음 리소스를 통해 지식을 확장하세요:

- **[LangGraph 플랫폼 개념](../concepts/index.md#langgraph-platform)**: LangGraph 플랫폼의 기본 개념 이해하기.
- **[LangGraph 플랫폼 사용 가이드](../how-tos/index.md#langgraph-platform)**: 애플리케이션을 구축하고 배포하기 위한 단계별 가이드.
- **[로컬 LangGraph 서버 시작하기](../tutorials/langgraph-platform/local-server.md)**: 이 빠른 시작 가이드는 **ReAct Agent** 템플릿을 위해 로컬에서 LangGraph 서버를 시작하는 방법을 보여줍니다. 다른 템플릿에 대해서도 비슷한 단계가 적용됩니다.


