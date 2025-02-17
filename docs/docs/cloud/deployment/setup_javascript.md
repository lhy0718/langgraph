_한국어로 기계번역됨_

# LangGraph.js 애플리케이션 배포 설정 방법

[LangGraph.js](https://langchain-ai.github.io/langgraphjs/) 애플리케이션은 LangGraph Cloud에 배포되거나 자체 호스팅을 위해 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)로 설정되어야 합니다. 이 가이드에서는 `package.json`을 사용하여 프로젝트 종속성을 지정하면서 LangGraph.js 애플리케이션을 배포하도록 설정하는 기본 단계를 설명합니다.

이 과정은 [이 리포지토리](https://github.com/langchain-ai/langgraphjs-studio-starter)를 기반으로 하며, 이 리포지토리를 사용해 LangGraph 애플리케이션을 배포하는 방법을 더 알아볼 수 있습니다.

최종 리포지토리 구조는 다음과 같습니다:

```bash
my-app/
├── src # 모든 프로젝트 코드가 여기에 포함됩니다
│   ├── utils # 그래프를 위한 선택적 유틸리티
│   │   ├── tools.ts # 그래프를 위한 도구
│   │   ├── nodes.ts # 그래프를 위한 노드 함수
│   │   └── state.ts # 그래프의 상태 정의
│   └── agent.ts # 그래프를 구성하는 코드
├── package.json # 패키지 종속성
├── .env # 환경 변수
└── langgraph.json # LangGraph 구성 파일
```

각 단계 후에는 코드를 어떻게 구성할 수 있는지 보여주는 예시 파일 디렉토리가 제공됩니다.

## 종속성 지정

종속성은 `package.json`에 지정할 수 있습니다. 이러한 파일이 생성되지 않은 경우, 이후 [LangGraph API 구성 파일](#create-langgraph-api-config)에서 종속성을 지정할 수 있습니다.

예시 `package.json` 파일:

```json
{
  "name": "langgraphjs-studio-starter",
  "packageManager": "yarn@1.22.22",
  "dependencies": {
    "@langchain/community": "^0.2.31",
    "@langchain/core": "^0.2.31",
    "@langchain/langgraph": "^0.2.0",
    "@langchain/openai": "^0.2.8"
  }
}
```

예시 파일 디렉토리:

```bash
my-app/
└── package.json # 패키지 종속성
```

## 환경 변수 지정

환경 변수는 선택적으로 파일(예: `.env`)에 지정할 수 있습니다. 배포를 위해 추가 변수를 구성하려면 [환경 변수 참조](../reference/env_var.md)를 참조하세요.

예시 `.env` 파일:

```
MY_ENV_VAR_1=foo
MY_ENV_VAR_2=bar
OPENAI_API_KEY=key
TAVILY_API_KEY=key_2
```

예시 파일 디렉토리:

```bash
my-app/
├── package.json
└── .env # 환경 변수
```

## 그래프 정의

그래프를 구현하세요! 그래프는 단일 파일 또는 여러 파일로 정의할 수 있습니다. LangGraph 애플리케이션에 포함될 각 컴파일된 그래프의 변수 이름에 유의하세요. 이 변수 이름은 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)을 만들 때 사용됩니다.

다음은 예시 `agent.ts`입니다:

```ts
import type { AIMessage } from "@langchain/core/messages";
import { TavilySearchResults } from "@langchain/community/tools/tavily_search";
import { ChatOpenAI } from "@langchain/openai";

import { MessagesAnnotation, StateGraph } from "@langchain/langgraph";
import { ToolNode } from "@langchain/langgraph/prebuilt";

const tools = [
  new TavilySearchResults({ maxResults: 3, }),
];

// 모델을 호출하는 함수를 정의합니다.
async function callModel(
  state: typeof MessagesAnnotation.State,
) {
  /**
   * 우리의 에이전트를 지원하는 LLM을 호출합니다.
   * 프롬프트, 모델 및 기타 논리를 자유롭게 사용자 정의하세요!
   */
  const model = new ChatOpenAI({
    model: "gpt-4o",
  }).bindTools(tools);

  const response = await model.invoke([
    {
      role: "system",
      content: `당신은 유용한 조수입니다. 현재 날짜는 ${new Date().getTime()}입니다.`
    },
    ...state.messages
  ]);

  // MessagesAnnotation은 단일 메시지 또는 메시지 배열을 반환하는 것을 지원합니다.
  return { messages: response };
}

// 계속할지 여부를 결정하는 함수를 정의합니다.
function routeModelOutput(state: typeof MessagesAnnotation.State) {
  const messages = state.messages;
  const lastMessage: AIMessage = messages[messages.length - 1];
  // LLM이 도구를 호출 중일 경우, 해당 위치로 라우팅합니다.
  if ((lastMessage?.tool_calls?.length ?? 0) > 0) {
    return "tools";
  }
  // 그렇지 않으면 그래프를 종료합니다.
  return "__end__";
}

// 새로운 그래프를 정의합니다.
// 사용자 정의 그래프 상태 정의에 대한 자세한 내용은 https://langchain-ai.github.io/langgraphjs/how-tos/define-state/#getting-started을 참조하세요.
const workflow = new StateGraph(MessagesAnnotation)
  // 우리가 순환할 두 노드를 정의합니다.
  .addNode("callModel", callModel)
  .addNode("tools", new ToolNode(tools))
  // 진입점을 `callModel`로 설정합니다.
  // 이는 이 노드가 처음 호출되는 노드임을 의미합니다.
  .addEdge("__start__", "callModel")
  .addConditionalEdges(
    // 먼저, 에지의 출발 노드를 정의합니다. `callModel`을 사용합니다.
    // 이는 `callModel` 노드가 호출된 후의 에지들입니다.
    "callModel",
    // 다음으로, 출발 노드가 호출된 후에 호출될 노드(들)를 결정할 함수를 전달합니다.
    routeModelOutput,
    // 조건부 에지가 라우팅할 수 있는 가능한 목적지 목록입니다.
    // 스튜디오에서 그래프가 제대로 렌더링되도록 요구됩니다.
    [
      "tools",
      "__end__"
    ],
  )
  // 이는 `tools`가 호출된 후, 다음에 `callModel` 노드가 호출된다는 의미입니다.
  .addEdge("tools", "callModel");

// 마지막으로, 컴파일합니다!
// 이는 호출하고 배포할 수 있는 그래프로 컴파일합니다.
export const graph = workflow.compile();
```

!!! 정보 "변수에 `CompiledGraph` 할당"
    LangGraph Cloud의 빌드 프로세스는 `CompiledGraph` 객체가 JavaScript 모듈의 최상위에 변수로 할당되어야 합니다 (또는 그래프를 생성하는 [함수를 제공할 수 있습니다](./graph_rebuild.md)).

예시 파일 디렉토리:

```bash
my-app/
├── src # 모든 프로젝트 코드가 여기 있습니다
│   ├── utils # 그래프에 대한 선택적 유틸리티
│   │   ├── tools.ts # 그래프에 대한 도구
│   │   ├── nodes.ts # 그래프에 대한 노드 함수
│   │   └── state.ts # 그래프의 상태 정의
│   └── agent.ts # 그래프를 구성하는 코드
├── package.json # 패키지 의존성
├── .env # 환경 변수
└── langgraph.json # LangGraph 구성 파일
```

## LangGraph API 구성 생성

`langgraph.json`이라는 LangGraph API 구성 파일을 생성합니다. 구성 파일의 JSON 객체에 있는 각 키에 대한 자세한 설명은 LangGraph CLI 참조를 확인하십시오.

예시 `langgraph.json` 파일:

```json
{
  "node_version": "20",
  "dockerfile_lines": [],
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent.ts:graph"
  },
  "env": ".env"
}
```

상위 `graphs` 키의 각 하위 키 값 끝에 `CompiledGraph`의 변수 이름이 나타나야 한다는 점에 유의하세요 (즉, `:<variable_name>`).

!!! 정보 "구성 위치"
    LangGraph API 구성 파일은 컴파일된 그래프 및 관련 종속성을 포함하는 TypeScript 파일과 동일한 수준이거나 그보다 높은 디렉터리에 배치해야 합니다.

## 다음

프로젝트를 설정하고 GitHub 저장소에 배치한 후에는 [앱 배포](./cloud.md)로 이동할 시간입니다.
