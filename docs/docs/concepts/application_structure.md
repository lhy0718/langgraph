_한국어로 기계번역됨_

# 애플리케이션 구조

!!! 정보 "전제 조건"

    - [LangGraph 서버](./langgraph_server.md)
    - [LangGraph 용어집](./low_level.md)

## 개요

LangGraph 애플리케이션은 하나 이상의 그래프, LangGraph API 구성 파일(`langgraph.json`), 종속성을 지정하는 파일, 그리고 선택적으로 환경 변수를 지정하는 .env 파일로 구성됩니다.

이 가이드는 LangGraph 애플리케이션의 전형적인 구조를 보여주고 LangGraph 플랫폼을 사용하여 LangGraph 애플리케이션을 배포하는 데 필요한 정보를 어떻게 지정하는지 설명합니다.

## 주요 개념

LangGraph 플랫폼을 사용하여 배포하려면 다음 정보를 제공해야 합니다.

1. 애플리케이션에 사용할 종속성, 그래프, 환경 변수를 지정하는 [LangGraph API 구성 파일](#configuration-file) (`langgraph.json`).
2. 애플리케이션의 논리를 구현하는 [그래프](#graphs).
3. 애플리케이션을 실행하는 데 필요한 [종속성](#dependencies)을 지정하는 파일.
4. 애플리케이션이 실행되는 데 필요한 [환경 변수](#environment-variables).

## 파일 구조

아래는 Python과 JavaScript 애플리케이션의 디렉토리 구조 예시입니다:

=== "Python (requirements.txt)"

    ```plaintext
    my-app/
    ├── my_agent # 모든 프로젝트 코드는 여기에 존재
    │   ├── utils # 그래프에 대한 유틸리티
    │   │   ├── __init__.py
    │   │   ├── tools.py # 그래프에 대한 도구
    │   │   ├── nodes.py # 그래프에 대한 노드 함수
    │   │   └── state.py # 그래프의 상태 정의
    │   ├── requirements.txt # 패키지 종속성
    │   ├── __init__.py
    │   └── agent.py # 그래프를 구성하기 위한 코드
    ├── .env # 환경 변수
    └── langgraph.json # LangGraph 구성 파일
    ```

=== "Python (pyproject.toml)"

    ```plaintext
    my-app/
    ├── my_agent # 모든 프로젝트 코드는 여기에 존재
    │   ├── utils # 그래프에 대한 유틸리티
    │   │   ├── __init__.py
    │   │   ├── tools.py # 그래프에 대한 도구
    │   │   ├── nodes.py # 그래프에 대한 노드 함수
    │   │   └── state.py # 그래프의 상태 정의
    │   ├── __init__.py
    │   └── agent.py # 그래프를 구성하기 위한 코드
    ├── .env # 환경 변수
    ├── langgraph.json  # LangGraph 구성 파일
    └── pyproject.toml # 프로젝트의 종속성
    ```

=== "JS (package.json)"

    ```plaintext
    my-app/
    ├── src # 모든 프로젝트 코드는 여기에 존재
    │   ├── utils # 그래프에 대한 선택적 유틸리티
    │   │   ├── tools.ts # 그래프에 대한 도구
    │   │   ├── nodes.ts # 그래프에 대한 노드 함수
    │   │   └── state.ts # 그래프의 상태 정의
    │   └── agent.ts # 그래프를 구성하기 위한 코드
    ├── package.json # 패키지 종속성
    ├── .env # 환경 변수
    └── langgraph.json # LangGraph 구성 파일
    ```

!!! 주의

    LangGraph 애플리케이션의 디렉토리 구조는 프로그래밍 언어와 사용된 패키지 관리자에 따라 다를 수 있습니다.

## 구성 파일

`langgraph.json` 파일은 종속성, 그래프, 환경 변수 및 LangGraph 애플리케이션을 배포하는 데 필요한 기타 설정을 지정하는 JSON 파일입니다.

이 파일은 다음 정보를 지정할 수 있습니다:

| 키                 | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dependencies`     | **필수**. LangGraph API 서버에 대한 종속성 배열. 종속성은 다음 중 하나일 수 있습니다: (1) `"."`, 이는 로컬 Python 패키지를 찾고, (2) 앱 디렉토리 `./local_package`에 있는 `pyproject.toml`, `setup.py` 또는 `requirements.txt`, 또는 (3) 패키지 이름.                                                                                                                                                                                                                               |
| `graphs`           | **필수**. 그래프 ID에서 컴파일된 그래프 또는 그래프를 만드는 함수가 정의된 경로로의 매핑. 예: <ul><li>`./your_package/your_file.py:variable`, 여기서 `variable`은 `langgraph.graph.state.CompiledStateGraph`의 인스턴스</li><li>`./your_package/your_file.py:make_graph`, 여기서 `make_graph`는 구성 사전(`langchain_core.runnables.RunnableConfig`)을 가져와 `langgraph.graph.state.StateGraph` / `langgraph.graph.state.CompiledStateGraph`의 인스턴스를 생성하는 함수.</li></ul> |
| `env`              | `.env` 파일의 경로 또는 환경 변수와 그 값의 매핑.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `python_version`   | `3.11` 또는 `3.12`. 기본값은 `3.11`.                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `pip_config_file`  | `pip` 구성 파일의 경로.                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `dockerfile_lines` | 상위 이미지에서 가져온 후 Dockerfile에 추가할 추가 줄의 배열.                                                                                                                                                                                                                                                                                                                                                                                                                       |

!!! 팁

    LangGraph CLI는 기본적으로 현재 디렉토리의 구성 파일 **langgraph.json**을 사용합니다.

### 예시

=== "파이썬"

- 의존성에는 사용자 정의 로컬 패키지와 `langchain_openai` 패키지가 포함됩니다.
- `variable` 변수로 `./your_package/your_file.py` 파일에서 단일 그래프가 로드됩니다.
- 환경 변수는 `.env` 파일에서 로드됩니다.

```json
{
  "dependencies": ["langchain_openai", "./your_package"],
  "graphs": {
    "my_agent": "./your_package/your_file.py:agent"
  },
  "env": "./.env"
}
```

=== "자바스크립트"

- 의존성은 로컬 디렉터리의 의존성 파일(예: `package.json`)에서 로드됩니다.
- `agent` 함수로 `./your_package/your_file.js` 파일에서 단일 그래프가 로드됩니다.
- 환경 변수 `OPENAI_API_KEY`는 인라인으로 설정됩니다.

```json
{
  "dependencies": ["."],
  "graphs": {
    "my_agent": "./your_package/your_file.js:agent"
  },
  "env": {
    "OPENAI_API_KEY": "secret-key"
  }
}
```

## 의존성

LangGraph 애플리케이션은 다른 파이썬 패키지 또는 자바스크립트 라이브러리에 의존할 수 있습니다(애플리케이션이 작성된 프로그래밍 언어에 따라 다름).

의존성이 올바르게 설정되도록 다음 정보들을 일반적으로 지정해야 합니다:

1. 의존성을 지정하는 디렉터리의 파일(예: `requirements.txt`, `pyproject.toml` 또는 `package.json`).
2. LangGraph 애플리케이션을 실행하는 데 필요한 의존성을 지정하는 [LangGraph 구성 파일](#configuration-file)의 `dependencies` 키.
3. 추가 이진 파일이나 시스템 라이브러리는 [LangGraph 구성 파일](#configuration-file)에서 `dockerfile_lines` 키를 사용하여 지정할 수 있습니다.

## 그래프

배포된 LangGraph 애플리케이션에서 사용할 수 있는 그래프를 지정하려면 [LangGraph 구성 파일](#configuration-file)의 `graphs` 키를 사용하세요.

구성 파일에 하나 이상의 그래프를 지정할 수 있습니다. 각 그래프는 이름(고유해야 함)과 (1) 컴파일된 그래프 또는 (2) 그래프를 정의하는 함수의 경로로 식별됩니다.

## 환경 변수

로컬에서 배포된 LangGraph 애플리케이션과 작업하는 경우, [LangGraph 구성 파일](#configuration-file)의 `env` 키에서 환경 변수를 구성할 수 있습니다.

프로덕션 배포의 경우, 일반적으로 배포 환경에서 환경 변수를 구성하는 것이 좋습니다.

## 관련

자세한 내용은 다음 자료를 참조하세요:

- [애플리케이션 구조](../how-tos/index.md#application-structure)에 대한 가이드.
