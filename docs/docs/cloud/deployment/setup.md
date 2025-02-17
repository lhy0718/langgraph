_한국어로 기계번역됨_

# LangGraph 애플리케이션 배포 설정 방법

LangGraph 애플리케이션은 LangGraph Cloud에 배포되거나 자체 호스팅을 위해 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)로 설정되어야 합니다. 이 가이드는 `requirements.txt`를 사용하여 프로젝트 의존성을 지정하면서 LangGraph 애플리케이션을 배포하기 위한 기본적인 설정 단계를 설명합니다.

이 안내는 LangGraph 애플리케이션 배포 설정에 대해 더 배우기 위해 사용해 볼 수 있는 [이 저장소](https://github.com/langchain-ai/langgraph-example)를 기반으로 합니다.

!!! 팁 "pyproject.toml로 설정하기"
    의존성 관리에 poetry를 선호하는 경우, LangGraph Cloud에 대한 `pyproject.toml`을 사용하는 [이 가이드](./setup_pyproject.md)를 확인하세요.

!!! 팁 "모노레포로 설정하기"
    모노레포 안에 있는 그래프를 배포하는 데 관심이 있다면, [이](https://github.com/langchain-ai/langgraph-example-monorepo) 저장소에서 그 예제를 확인하세요.

최종 저장소 구조는 다음과 같이 생길 것입니다:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드가 여기에 위치합니다
│   ├── utils # 그래프에 대한 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프의 도구
│   │   ├── nodes.py # 그래프의 노드 함수
│   │   └── state.py # 그래프의 상태 정의
│   ├── requirements.txt # 패키지 의존성
│   ├── __init__.py
│   └── agent.py # 그래프를 구성하기 위한 코드
├── .env # 환경 변수
└── langgraph.json # LangGraph을 위한 구성 파일
```

각 단계 후에 코드를 어떻게 구성할 수 있는지 보여주는 예제 파일 디렉토리가 제공됩니다.

## 의존성 지정하기

의존성은 선택적으로 다음 파일 중 하나에 지정할 수 있습니다: `pyproject.toml`, `setup.py`, 또는 `requirements.txt`. 이러한 파일이 생성되지 않으면 [LangGraph API 구성 파일](#create-langgraph-api-config)에서 나중에 의존성을 지정할 수 있습니다.

아래의 의존성은 이미지에 포함되며, 호환 가능한 버전 범위를 사용하는 한 코드에서 사용할 수 있습니다:

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

예제 `requirements.txt` 파일:

```
langgraph
langchain_anthropic
tavily-python
langchain_community
langchain_openai
```

예제 파일 디렉토리:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드가 여기에 위치합니다
│   └── requirements.txt # 패키지 의존성
```

## 환경 변수 지정하기

환경 변수는 선택적으로 파일(예: `.env`)에 지정할 수 있습니다. 배포를 위한 추가 변수를 구성하려면 [환경 변수 참고](../reference/env_var.md)를 참조하세요.

예제 `.env` 파일:

```
MY_ENV_VAR_1=foo
MY_ENV_VAR_2=bar
OPENAI_API_KEY=key
```

예제 파일 디렉토리:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드가 여기에 위치합니다
│   └── requirements.txt # 패키지 의존성
└── .env # 환경 변수
```

## 그래프 정의하기

그래프를 구현하세요! 그래프는 단일 파일이나 여러 파일에 정의될 수 있습니다. LangGraph 애플리케이션에 포함될 각 [CompiledGraph][langgraph.graph.graph.CompiledGraph]의 변수 이름을 메모해 두세요. 변수 이름은 나중에 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)을 생성할 때 사용됩니다.

다음은 다른 모듈에서 가져오는 방법을 보여주는 `agent.py` 파일의 예로, 모듈에 대한 코드는 여기에서 보여지지 않으니 [이 저장소](https://github.com/langchain-ai/langgraph-example)에서 구현을 확인해 주세요:

```python
# my_agent/agent.py
from typing import Literal
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, END, START
from my_agent.utils.nodes import call_model, should_continue, tool_node # 노드 임포트
from my_agent.utils.state import AgentState # 상태 임포트

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
    LangGraph Cloud의 빌드 프로세스는 `CompiledGraph` 객체를 Python 모듈의 최상위에서 변수에 할당해야 합니다 (또는 [그래프를 생성하는 함수](./graph_rebuild.md)를 제공할 수 있습니다).

예시 파일 디렉토리:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드는 여기 있습니다
│   ├── utils # 그래프를 위한 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프를 위한 도구
│   │   ├── nodes.py # 그래프를 위한 노드 함수
│   │   └── state.py # 그래프의 상태 정의
│   ├── requirements.txt # 패키지 의존성
│   ├── __init__.py
│   └── agent.py # 그래프 생성을 위한 코드
└── .env # 환경 변수
```

## LangGraph API 구성 생성

`langgraph.json`이라고 불리는 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)을 생성합니다. 구성 파일의 JSON 객체에 있는 각 키에 대한 자세한 설명은 [LangGraph CLI 참조](../reference/cli.md#configuration-file)를 참조하세요.

예시 `langgraph.json` 파일:

```json
{
  "dependencies": ["./my_agent"],
  "graphs": {
    "agent": "./my_agent/agent.py:graph"
  },
  "env": ".env"
}
```

`CompiledGraph`의 변수 이름이 최상위 `graphs` 키의 각 서브키 값 끝에 나타난다는 점에 유의하세요 (즉, `:<variable_name>`).

!!! 경고 "구성 위치"
    LangGraph API 구성 파일은 컴파일된 그래프와 관련된 종속성을 포함한 Python 파일과 동일한 수준 또는 그 이상의 디렉토리에 배치해야 합니다.

예시 파일 디렉토리:

```bash
my-app/
├── my_agent # 모든 프로젝트 코드는 여기 있습니다
│   ├── utils # 그래프를 위한 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프를 위한 도구
│   │   ├── nodes.py # 그래프를 위한 노드 함수
│   │   └── state.py # 그래프의 상태 정의
│   ├── requirements.txt # 패키지 의존성
│   ├── __init__.py
│   └── agent.py # 그래프 생성을 위한 코드
├── .env # 환경 변수
└── langgraph.json # LangGraph를 위한 구성 파일
```

## 다음

프로젝트를 설정하고 GitHub 리포지토리에 배치한 후, [앱을 배포할 시간입니다](./cloud.md).
