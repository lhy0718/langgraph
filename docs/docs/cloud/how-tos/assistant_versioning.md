_한국어로 기계번역됨_

# 어시스턴트 버전 관리 방법

이 사용 안내서에서는 다양한 어시스턴트 버전을 생성하고 관리하는 방법을 설명합니다. 아직 읽지 않았다면, 어시스턴트 버전 관리의 개념을 더 잘 이해하기 위해 [이 문서](../../concepts/assistants.md#versioning-assistants)를 읽어보십시오. 이 사용 안내서는 구성 가능한 그래프가 있다고 가정하며, 이는 구성 스키마를 정의하고 다음과 같이 그래프에 전달했음을 의미합니다:

=== "Python"

```python
class Config(BaseModel):
    model_name: Literal["anthropic", "openai"] = "anthropic"
    system_prompt: str

agent = StateGraph(State, config_schema=Config)
```

=== "Javascript"

```js
const ConfigAnnotation = Annotation.Root({
    modelName: Annotation<z.enum(["openai", "anthropic"])>({
        default: () => "anthropic",
    }),
    systemPrompt: Annotation<String>
});

// 나머지 코드

const agent = new StateGraph(StateAnnotation, ConfigAnnotation);
```

## 설정

우선 클라이언트와 스레드를 설정하겠습니다. 스튜디오를 사용 중이라면 "agent"라는 이름의 그래프가 있는 애플리케이션을 열기만 하면 됩니다. cURL을 사용하는 경우에는 배포 URL과 사용하려는 그래프의 이름만 복사하면 됩니다.

=== "Python"

```python
from langgraph_sdk import get_client

client = get_client(url=<DEPLOYMENT_URL>)
# 이름이 "agent"인 그래프 사용
graph_name = "agent"
```

=== "Javascript"

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: <DEPLOYMENT_URL> });
// 이름이 "agent"인 그래프 사용
const graphName = "agent";
```

## 어시스턴트 생성

이 예제에서는 그래프에서 사용되는 모델 이름을 수정하여 어시스턴트를 생성합니다. "openai_assistant"라는 이름의 새로운 어시스턴트를 생성할 수 있습니다:

=== "Python"

```python
openai_assistant = await client.assistants.create(graph_name, config={"configurable": {"model_name": "openai"}}, name="openai_assistant")
```

=== "Javascript"

```js
const openaiAssistant = await client.assistants.create({graphId: graphName, config: { configurable: {"modelName": "openai"}}, name: "openaiAssistant"});
```

=== "CURL"

```bash
curl --request POST \
--url <DEPOLYMENT_URL>/assistants \
--header 'Content-Type: application/json' \
--data '{
"graph_id": "agent",
"config": {"model_name": "openai"},
"name": "openai_assistant"
}'
```

### 스튜디오 사용하기

스튜디오를 사용하여 어시스턴트를 생성하려면 다음 단계를 따르십시오:

1. "새 어시스턴트 생성" 버튼을 클릭합니다:

    ![click create](./img/click_create_assistant.png)

2. 어시스턴트 생성 창에서 생성할 어시스턴트를 위한 정보를 입력한 후 생성 버튼을 클릭합니다:

    ![create](./img/create_assistant.png)

3. 어시스턴트가 생성되었으며 스튜디오에 표시되는 것을 확인합니다:

    ![view create](./img/create_assistant_view.png)

4. 선택한 어시스턴트 옆의 편집 버튼을 클릭하여 생성한 어시스턴트를 관리합니다:

    ![생성 편집](./img/edit_created_assistant.png)

## 당신의 도우미를 위한 새로운 버전 생성하기

이제 도우미에 시스템 프롬프트를 추가하고 싶다고 가정해봅시다. `update` 엔드포인트를 사용하여 이를 수행할 수 있습니다. 전체 구성(config)과 메타데이터(사용하는 경우)를 반드시 전달해야 한다는 점에 유의하시기 바랍니다. 업데이트 엔드포인트는 이전에 입력된 구성 없이 완전히 새롭게 새로운 버전을 생성합니다. 이 경우, 도우미에게 "openai"를 모델로 사용하도록 계속 지시해야 합니다.

=== "Python"

```python
openai_assistant_v2 = await client.assistants.update(openai_assistant['assistant_id'], config={"configurable": {"model_name": "openai", "system_prompt": "당신은 유용한 도우미입니다!"}})
```

=== "Javascript"

```js
const openaiAssistantV2 = await client.assistants.update(openaiAssistant['assistant_id'], {config: { configurable: {"modelName": "openai", "systemPrompt": "당신은 유용한 도우미입니다!"}}});
```

=== "CURL"

```bash
curl --request PATCH \
--url <배포_URL>/assistants/<ASSISTANT_ID> \
--header 'Content-Type: application/json' \
--data '{
"config": {"model_name": "openai", "system_prompt": "당신은 유용한 도우미입니다!"}
}'
```

### 스튜디오 사용하기

1. 먼저, `openai_assistant` 옆의 편집 버튼을 클릭합니다. 그러고 나서 시스템 프롬프트를 추가하고 "새 버전 저장"을 클릭합니다:

    ![새 버전 생성](./img/create_new_version.png)

2. 그러면 도우미 드롭다운에서 선택된 것을 볼 수 있습니다:

    ![버전 드롭다운 보기](./img/see_new_version.png)

3. 그리고 도우미의 편집 창에서 모든 버전 기록을 볼 수 있습니다:

    ![버전 보기](./img/see_version_history.png)

## 도우미를 다른 버전으로 가리키기

여러 버전을 생성한 후, SDK와 스튜디오 모두를 사용하여 도우미가 가리키는 버전을 변경할 수 있습니다. 이 경우, 방금 두 개의 버전을 생성한 `openai_assistant`를 첫 번째 버전으로 되돌릴 것입니다. 새 버전을 생성하면( `update` 엔드포인트를 사용하여) 도우미는 자동으로 새로 생성된 버전을 가리키게 되므로, 위의 코드를 따르면 우리 `openai_assistant`는 두 번째 버전을 가리키고 있습니다. 여기서 첫 번째 버전을 가리키도록 변경할 것입니다:

=== "Python"

```python
await client.assistants.set_latest(openai_assistant['assistant_id'], 1)
```

=== "Javascript"

```js
await client.assistants.setLatest(openaiAssistant['assistant_id'], 1);
```

=== "CURL"

```bash
curl --request POST \
--url <배포_URL>/assistants/<ASSISTANT_ID>/latest \
--header 'Content-Type: application/json' \
--data '{
"version": 1
}'
```

### 스튜디오 사용하기

버전을 변경하려면, 도우미의 편집 창에 들어가서 변경하고자 하는 버전을 선택한 다음 "현재 버전으로 설정" 버튼을 클릭하기만 하면 됩니다.

![버전 설정](./img/select_different_version.png)

## 도우미 버전 사용하기

비즈니스 사용자인지 코드를 작성하지 않고 반복하는 개발자인지에 관계없이, 도우미 버전 관리는 통제된 환경에서 다양한 에이전트를 빠르게 테스트할 수 있게 해주며, 빠르게 반복하기 쉽게 만듭니다. 도우미의 모든 버전은 일반 도우미처럼 사용할 수 있으며, 이러한 도우미의 출력을 스트리밍하는 방법에 대해 [이 가이드](https://langchain-ai.github.io/langgraph/cloud/how-tos/#streaming)를 읽거나 스튜디오를 사용하는 경우 [이 가이드](https://langchain-ai.github.io/langgraph/cloud/how-tos/invoke_studio/)를 읽어볼 수 있습니다.

!!! 경고 "도우미 삭제하기"
    도우미를 삭제하면 모든 버전이 삭제됩니다. 그들이 모두 같은 도우미 ID를 가리키기 때문입니다. 현재 단일 버전만 삭제하는 방법은 없지만 도우미를 올바른 버전으로 가리키게 함으로써 사용하고 싶지 않은 버전을 건너뛸 수 있습니다.