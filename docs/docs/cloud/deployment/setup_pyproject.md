_한국어로 기계번역됨_

# LangGraph 애플리케이션 배포 설정 방법

LangGraph 애플리케이션은 LangGraph Cloud에 배포되거나 자체 호스팅되기 위해 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)로 구성되어야 합니다. 이 가이드에서는 패키지의 종속성을 정의하기 위해 `pyproject.toml`을 사용하여 LangGraph 애플리케이션을 배포하기 위한 기본 단계에 대해 설명합니다.

이 walkthrough는 [이 저장소](https://github.com/langchain-ai/langgraph-example-pyproject)를 기반으로 하며, 이를 통해 LangGraph 애플리케이션 배포 설정 방법에 대해 더 배울 수 있습니다.

!!! 팁 "requirements.txt로 설정하기"
   종속성 관리를 위해 `requirements.txt`를 사용하는 것을 선호하는 경우 [이 가이드](./setup.md)를 확인해 보세요.

!!! 팁 "모노레포로 설정하기"
   모노레포 내부에 있는 그래프를 배포하는 데 관심이 있다면 [이](https://github.com/langchain-ai/langgraph-example-monorepo) 저장소를 살펴보며 방법을 알아보세요.

최종 저장소 구조는 다음과 유사하게 생길 것입니다:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드가 여기 있습니다
│   ├── utils # 그래프를 위한 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프를 위한 도구
│   │   ├── nodes.py # 그래프를 위한 노드 함수
│   │   └── state.py # 그래프의 상태 정의
│   ├── __init__.py
│   └── agent.py # 그래프를 구성하는 코드
├── .env # 환경 변수
├── langgraph.json  # LangGraph를 위한 구성 파일
└── pyproject.toml # 프로젝트에 필요한 종속성
```

각 단계 후에 코드를 어떻게 구성할 수 있는지 보여주는 예시 파일 디렉토리가 제공됩니다.

## 종속성 지정

종속성은 다음 파일 중 하나인 `pyproject.toml`, `setup.py`, 또는 `requirements.txt`에 선택적으로 지정할 수 있습니다. 이 파일들 중 어느 것도 생성되지 않으면, 종속성은 [LangGraph API 구성 파일](#create-langgraph-api-config)에서 나중에 지정할 수 있습니다.

아래의 종속성은 이미지에 포함되며, 호환되는 버전 범위 내에서 코드를 사용하여 사용할 수 있습니다:

```
langgraph>=0.2.56,<0.3.0
langgraph-checkpoint>=2.0.5,<3.0
langchain-core>=0.2.38,<0.4.0
langsmith>=0.1.63
orjson>=3.9.7
httpx>=0.25.0
tenacity>=8.0.0
uvicorn>=0.26.0
sse-starlette>=2.1.0
uvloop>=0.18.0
httptools>=0.5.0
jsonschema-rs>=0.16.3
croniter>=1.0.1
structlog>=23.1.0
redis>=5.0.0,<6.0.0
```

예시 `pyproject.toml` 파일:

```toml
[tool.poetry]
name = "my-agent"
version = "0.0.1"
description = "LangGraph 클라우드를 위한 훌륭한 에이전트."
authors = ["Polly the parrot <1223+polly@users.noreply.github.com>"]
license = "MIT"
readme = "README.md"

[tool.poetry.dependencies]
python = ">=3.9.0,<3.13"
langgraph = "^0.2.0"
langchain-fireworks = "^0.1.3"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

예시 파일 디렉토리:

```bash
my-app/
└── pyproject.toml   # 그래프에 필요한 Python 패키지
```

## 환경 변수 지정

환경 변수는 선택적으로 파일(예: `.env`)에 지정할 수 있습니다. 배포를 위한 추가 변수를 구성하려면 [환경 변수 참조](../reference/env_var.md)를 참조하세요.

예시 `.env` 파일:

```
MY_ENV_VAR_1=foo
MY_ENV_VAR_2=bar
FIREWORKS_API_KEY=key
```

예시 파일 디렉토리:

```bash
my-app/
├── .env             # 환경 변수를 포함하는 파일
└── pyproject.toml
```

## 그래프 정의

그래프를 구현하세요! 그래프는 단일 파일 또는 여러 파일로 정의할 수 있습니다. LangGraph 애플리케이션에 포함될 각 [CompiledGraph][langgraph.graph.graph.CompiledGraph]의 변수 이름을 기록해 두세요. 이 변수 이름은 나중에 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)을 생성할 때 사용됩니다.

예시 `agent.py` 파일로, 정의한 다른 모듈에서 가져오는 방법을 보여줍니다 (모듈의 코드는 여기에 표시되지 않으니, [이 리포지토리](https://github.com/langchain-ai/langgraph-example-pyproject)를 참조하여 구현을 확인하세요):

```python
# my_agent/agent.py
from typing import Literal
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, END, START
from my_agent.utils.nodes import call_model, should_continue, tool_node # 노드 가져오기
from my_agent.utils.state import AgentState # 상태 가져오기

# 구성 정의
class GraphConfig(TypedDict):
    model_name: Literal["anthropic", "openai"]

workflow = StateGraph(AgentState, config_schema=GraphConfig)
workflow.add_node("agent", call_model)
workflow.add_node("action", tool_node)
workflow.add_edge(START, "agent")
workflow.add_conditional_edges(
    "agent",
    should_continue,
    {
        "continue": "action",
        "end": END,
    },
)
workflow.add_edge("action", "agent")

graph = workflow.compile()
```

!!! 경고 "변수에 `CompiledGraph` 할당"
    LangGraph Cloud의 빌드 프로세스에서는 Python 모듈의 최상위 수준에 `CompiledGraph` 객체가 변수에 할당되어야 합니다.

예시 파일 디렉토리:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드가 여기에 포함됨
│   ├── utils # 그래프의 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프의 도구
│   │   ├── nodes.py # 그래프의 노드 함수
│   │   └── state.py # 그래프의 상태 정의
│   ├── __init__.py
│   └── agent.py # 그래프를 구성하는 코드
├── .env
└── pyproject.toml
```

## LangGraph API 구성 생성

`langgraph.json`라는 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)을 생성하세요. JSON 객체의 구성 파일 내 각 키에 대한 자세한 설명은 [LangGraph CLI 참조](../reference/cli.md#configuration-file)를 참조하세요.

예시 `langgraph.json` 파일:

```json
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./my_agent/agent.py:graph"
  },
  "env": ".env"
}
```

`CompiledGraph`의 변수 이름이 최상위 `graphs` 키의 각 하위 키 값 끝에 나타나는 점에 유의하세요 (즉, `:<variable_name>`).

!!! 경고 "구성 위치"
    LangGraph API 구성 파일은 컴파일된 그래프와 관련된 종속성이 포함된 Python 파일과 동일한 수준 또는 더 높은 수준의 디렉토리에 있어야 합니다.

예시 파일 디렉토리:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드가 여기에 포함됨
│   ├── utils # 그래프의 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프의 도구
│   │   ├── nodes.py # 그래프의 노드 함수
│   │   └── state.py # 그래프의 상태 정의
│   ├── __init__.py
│   └── agent.py # 그래프를 구성하는 코드
├── .env # 환경 변수
├── langgraph.json  # LangGraph의 구성 파일
└── pyproject.toml # 프로젝트에 대한 종속성
```

## 다음

프로젝트를 설정하고 GitHub 저장소에 배치한 후, [앱을 배포할](/cloud.md) 시간입니다.
