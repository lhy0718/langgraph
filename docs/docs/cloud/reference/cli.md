_한국어로 기계번역됨_

# LangGraph CLI

LangGraph 명령줄 인터페이스는 [Docker](https://www.docker.com/)에서 로컬로 LangGraph Cloud API 서버를 구축하고 실행하는 명령을 포함합니다. 개발 및 테스트를 위해 CLI를 사용하여 [Studio 데스크탑 앱](../../concepts/langgraph_studio.md)의 대안으로 로컬 API 서버를 배포할 수 있습니다.

## 설치

1. Docker가 설치되어 있는지 확인하세요 (예: `docker --version`).
2. CLI 패키지를 설치합니다:

    === "Python"
        ```bash
        pip install langgraph-cli

        # Homebrew를 통해 설치
        brew install langgraph-cli
        ```

    === "JS"
        ```bash
        npx @langchain/langgraph-cli

        # 전역 설치, `langgraphjs`로 사용 가능
        npm install -g @langchain/langgraph-cli
        ```

3. `langgraph --help` 또는 `npx @langchain/langgraph-cli --help` 명령을 실행하여 CLI가 제대로 작동하는지 확인합니다.

## 구성 파일 {#configuration-file}

LangGraph CLI는 다음 키를 포함하는 JSON 구성 파일이 필요합니다:

<div class="admonition tip">
    <p class="admonition-title">참고</p>
    <p>
        LangGraph CLI는 기본적으로 현재 디렉토리에서 <strong>langgraph.json</strong> 구성 파일을 사용합니다.
    </p>
</div>

=== "Python"

    | 키                                                          | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
    | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | <span style="white-space: nowrap;">`dependencies`</span>     | **필수**. LangGraph Cloud API 서버를 위한 종속성 배열. 종속성은 다음 중 하나일 수 있습니다: (1) `"."`, 로컬 Python 패키지를 찾습니다, (2) 앱 디렉토리 `"./local_package"`의 `pyproject.toml`, `setup.py` 또는 `requirements.txt`, 또는 (3) 패키지 이름.                                                                                                                                                                                                                                                                                     |
    | <span style="white-space: nowrap;">`graphs`</span>           | **필수**. 그래프 ID에서 컴파일된 그래프 또는 그래프를 생성하는 함수가 정의된 경로로의 매핑. 예: <ul><li>`./your_package/your_file.py:variable`, 여기서 `variable`은 `langgraph.graph.state.CompiledStateGraph`의 인스턴스입니다.</li><li>`./your_package/your_file.py:make_graph`, 여기서 `make_graph`는 구성 딕셔너리 (`langchain_core.runnables.RunnableConfig`)를 받아 `langgraph.graph.state.StateGraph` / `langgraph.graph.state.CompiledStateGraph`의 인스턴스를 생성하는 함수입니다.</li></ul>                                    |
    | <span style="white-space: nowrap;">`auth`</span>             | _(v0.0.11에서 추가됨)_ 인증 핸들러의 경로를 포함하는 인증 구성. 예: `./your_package/auth.py:auth`, 여기서 `auth`는 `langgraph_sdk.Auth`의 인스턴스입니다. 자세한 내용은 [인증 가이드](../../concepts/auth.md)를 참조하세요.                                                                                                                                                                                                                                                                                                                        |
    | <span style="white-space: nowrap;">`env`</span>              | `.env` 파일의 경로 또는 환경 변수와 그 값을 매핑합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
    | <span style="white-space: nowrap;">`store`</span>            | BaseStore에 의미론적 검색을 추가하기 위한 구성. 다음 필드를 포함합니다: <ul><li>`index`: 다음 필드를 포함하는 의미론적 검색 인덱싱을 위한 구성:<ul><li>`embed`: 임베딩 공급자 (예: "openai:text-embedding-3-small") 또는 사용자 정의 임베딩 함수의 경로</li><li>`dims`: 임베딩 모델의 차원 크기. 벡터 테이블을 초기화하는 데 사용됩니다.</li><li>`fields` (선택 사항): 인덱싱할 필드 목록. 기본값은 `["$"]`, 즉 전체 문서를 인덱싱합니다. 특정 필드는 `["text", "summary", "some.value"]`와 같이 설정할 수 있습니다.</li></ul></li></ul> |
    | <span style="white-space: nowrap;">`python_version`</span>   | `3.11` 또는 `3.12`. 기본값은 `3.11`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
    | <span style="white-space: nowrap;">`node_version`</span>     | LangGraph.js를 사용하기 위해 `node_version: 20`를 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
    | <span style="white-space: nowrap;">`pip_config_file`</span>  | `pip` 구성 파일의 경로입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
    | <span style="white-space: nowrap;">`dockerfile_lines`</span> | 상위 이미지에서 가져온 후 Dockerfile에 추가할 추가 행의 배열입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

=== "JS"

    | 키                                                          | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
    | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | <span style="white-space: nowrap;">`graphs`</span>           | **필수**. 그래프 ID에서 컴파일된 그래프 또는 그래프를 생성하는 함수가 정의된 경로로의 매핑. 예: <ul><li>`./src/graph.ts:variable`, 여기서 `variable`은 `CompiledStateGraph`의 인스턴스입니다.</li><li>`./src/graph.ts:makeGraph`, 여기서 `makeGraph`는 구성 딕셔너리 (`LangGraphRunnableConfig`)를 받아 `StateGraph` / `CompiledStateGraph`의 인스턴스를 생성하는 함수입니다.</li></ul>                                    |
    | <span style="white-space: nowrap;">`env`</span>              | `.env` 파일의 경로 또는 환경 변수와 그 값을 매핑합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
    | <span style="white-space: nowrap;">`store`</span>            | BaseStore에 의미론적 검색을 추가하기 위한 구성. 다음 필드를 포함합니다: <ul><li>`index`: 다음 필드를 포함하는 의미론적 검색 인덱싱을 위한 구성:<ul><li>`embed`: 임베딩 공급자 (예: "openai:text-embedding-3-small") 또는 사용자 정의 임베딩 함수의 경로</li><li>`dims`: 임베딩 모델의 차원 크기. 벡터 테이블을 초기화하는 데 사용됩니다.</li><li>`fields` (선택 사항): 인덱싱할 필드 목록. 기본값은 `["$"]`, 즉 전체 문서를 인덱싱합니다. 특정 필드는 `["text", "summary", "some.value"]`와 같이 설정할 수 있습니다.</li></ul></li></ul> |
    | <span style="white-space: nowrap;">`node_version`</span>     | LangGraph.js를 사용하기 위해 `node_version: 20`를 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
    | <span style="white-space: nowrap;">`dockerfile_lines`</span> | 상위 이미지에서 가져온 후 Dockerfile에 추가할 추가 행의 배열입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

### 예시

=== "Python"
    
    #### 기본 구성

    ```json
    {
      "dependencies": ["."],
      "graphs": {
        "chat": "./chat/graph.py:graph"
      }
    }
    ```

    #### 스토어에 의미론적 검색 추가

    모든 배포에는 DB 지원 BaseStore가 포함됩니다. `langgraph.json`에 "index" 구성을 추가하면 배포의 BaseStore 내에서 [의미론적 검색](../deployment/semantic_search.md)이 활성화됩니다.

    `fields` 구성은 문서의 어떤 부분을 임베딩할지를 결정합니다:

    - 생략되거나 `["$"]`로 설정하면 전체 문서가 임베딩됩니다.
    - 특정 필드를 임베딩하려면 JSON 경로 표기법을 사용하십시오: `["metadata.title", "content.text"]`
    - 지정된 필드가 없는 문서는 여전히 저장되지만 해당 필드에 대한 임베딩이 없습니다.
    - 특정 항목의 `put` 시점에서 `index` 매개변수를 사용하여 임베딩할 필드를 재정의할 수 있습니다.

    ```json
    {
      "dependencies": ["."],
      "graphs": {
        "memory_agent": "./agent/graph.py:graph"
      },
      "store": {
        "index": {
          "embed": "openai:text-embedding-3-small",
          "dims": 1536,
          "fields": ["$"]
        }
      }
    }
    ```

    !!! 주의 "일반적인 모델 차원" 
        - `openai:text-embedding-3-large`: 3072 
        - `openai:text-embedding-3-small`: 1536 
        - `openai:text-embedding-ada-002`: 1536 
        - `cohere:embed-english-v3.0`: 1024 
        - `cohere:embed-english-light-v3.0`: 384 
        - `cohere:embed-multilingual-v3.0`: 1024 
        - `cohere:embed-multilingual-light-v3.0`: 384

    #### 사용자 정의 임베딩 함수로 의미 기반 검색

    사용자 정의 임베딩 함수를 사용하여 의미 기반 검색을 수행하려면 사용자 정의 임베딩 함수의 경로를 전달할 수 있습니다:

    ```json
    {
      "dependencies": ["."],
      "graphs": {
        "memory_agent": "./agent/graph.py:graph"
      },
      "store": {
        "index": {
          "embed": "./embeddings.py:embed_texts",
          "dims": 768,
          "fields": ["text", "summary"]
        }
      }
    }
    ```

    저장소 구성의 `embed` 필드는 문자열 목록을 받아 임베딩 목록을 반환하는 사용자 정의 함수를 참조할 수 있습니다. 예시 구현:

    ```python
    # embeddings.py
    def embed_texts(texts: list[str]) -> list[list[float]]:
        """의미 기반 검색을 위한 사용자 정의 임베딩 함수."""
        # 선호하는 임베딩 모델을 사용한 구현
        return [[0.1, 0.2, ...] for _ in texts]  # 차원-차원의 벡터
    ```

    #### 사용자 정의 인증 추가

    ```json
    {
      "dependencies": ["."],
      "graphs": {
        "chat": "./chat/graph.py:graph"
      },
      "auth": {
        "path": "./auth.py:auth",
        "openapi": {
          "securitySchemes": {
            "apiKeyAuth": {
              "type": "apiKey",
              "in": "header",
              "name": "X-API-Key"
            }
          },
          "security": [{ "apiKeyAuth": [] }]
        },
        "disable_studio_auth": false
      }
    }
    ```

    자세한 내용은 [인증 개념 가이드](../../concepts/auth.md)를 참조하고, 프로세스의 실용적인 안내는 [사용자 정의 인증 설정](../../tutorials/auth/getting_started.md) 가이드를 참조하십시오.


=== "JS"
    
    #### 기본 구성

    ```json
    {
      "graphs": {
        "chat": "./src/graph.ts:graph"
      }
    }
    ```


## 명령

**사용법**

=== "Python"

    LangGraph CLI의 기본 명령은 `langgraph`입니다.

    ```
    langgraph [OPTIONS] COMMAND [ARGS]
    ```
=== "JS"

    LangGraph.js CLI의 기본 명령은 `langgraphjs`입니다.

    ```
    npx @langchain/langgraph-cli [OPTIONS] COMMAND [ARGS]
    ```

    항상 최신 버전의 CLI를 사용하기 위해 `npx` 사용을 권장합니다.

### `dev`

=== "Python"

    핫 리로딩 및 디버깅 기능이 있는 개발 모드에서 LangGraph API 서버를 실행합니다. 이 경량 서버는 Docker 설치가 필요 없으며 개발 및 테스트에 적합합니다. 상태는 로컬 디렉토리에 유지됩니다.

    !!! 주의

        현재 CLI는 Python >= 3.11만 지원합니다.

    **설치**

    이 명령은 "inmem" 추가 패키지가 설치되어 있어야 합니다:

    ```bash
    pip install -U "langgraph-cli[inmem]"
    ```

    **사용법**

    ```
    langgraph dev [OPTIONS]
    ```

    **옵션**

    | 옵션                          | 기본값            | 설명                                                                                |
    | ----------------------------- | ----------------- | ---------------------------------------------------------------------------------- |
    | `-c, --config FILE`           | `langgraph.json`  | 종속성, 그래프 및 환경 변수를 선언하는 구성 파일의 경로                         |
    | `--host TEXT`                 | `127.0.0.1`       | 서버에 바인딩할 호스트                                                             |
    | `--port INTEGER`              | `2024`            | 서버에 바인딩할 포트                                                               |
    | `--no-reload`                 |                   | 자동 리로드 비활성화                                                               |
    | `--n-jobs-per-worker INTEGER` |                   | 작업자당 작업 수. 기본값은 10                                                       |
    | `--debug-port INTEGER`        |                   | 디버거가 수신할 포트                                                               |
    | `--help`                      |                   | 명령 문서 표시                                                                     |

=== "JS"

    핫 리로딩 기능이 있는 개발 모드에서 LangGraph API 서버를 실행합니다. 이 경량 서버는 Docker 설치가 필요 없으며 개발 및 테스트에 적합합니다. 상태는 로컬 디렉토리에 유지됩니다.

    **사용법**

    ```
    npx @langchain/langgraph-cli dev [OPTIONS]
    ```

    **옵션**

    | 옵션                          | 기본값            | 설명                                                                                |
    | ----------------------------- | ----------------- | ---------------------------------------------------------------------------------- |
    | `-c, --config FILE`           | `langgraph.json`  | 종속성, 그래프 및 환경 변수를 선언하는 구성 파일의 경로                         |
    | `--host TEXT`                 | `127.0.0.1`       | 서버에 바인딩할 호스트                                                             |
    | `--port INTEGER`              | `2024`            | 서버에 바인딩할 포트                                                               |
    | `--no-reload`                 |                   | 자동 리로드 비활성화                                                               |
    | `--n-jobs-per-worker INTEGER` |                   | 작업자당 작업 수. 기본값은 10                                                       |
    | `--debug-port INTEGER`        |                   | 디버거가 수신할 포트                                                               |
    | `--help`                      |                   | 명령 문서 표시                                                                     |

### `build`

=== "Python"

    LangGraph Cloud API 서버 Docker 이미지를 빌드합니다.

    **사용법**

    ```
    langgraph build [OPTIONS]
    ```

    **옵션**

    | 옵션                   | 기본값            | 설명                                                                                                                |
    | ---------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------- |
    | `--platform TEXT`      |                   | Docker 이미지를 빌드할 대상 플랫폼. 예: `langgraph build --platform linux/amd64,linux/arm64`                     |
    | `-t, --tag TEXT`       |                   | **필수**. Docker 이미지의 태그. 예: `langgraph build -t my-image`                                               |
    | `--pull / --no-pull`   | `--pull`          | 최신 원격 Docker 이미지로 빌드합니다. `--no-pull`을 사용하여 로컬에서 빌드된 이미지로 LangGraph Cloud API 서버 실행. |
    | `-c, --config FILE`    | `langgraph.json`  | 종속성, 그래프 및 환경 변수를 선언하는 구성 파일의 경로.                                                            |
    | `--help`               |                   | 명령 문서 표시.                                                                                                     | 

=== "JS"

    LangGraph Cloud API 서버 Docker 이미지 빌드합니다.

    **사용법**

    ```
    npx @langchain/langgraph-cli build [옵션]
    ```

    **옵션**

    | 옵션                  | 기본값          | 설명                                                                                                                  |
    | --------------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------- |
    | `--platform TEXT`     |                  | Docker 이미지를 빌드할 대상 플랫폼. 예: `langgraph build --platform linux/amd64,linux/arm64`                       |
    | `-t, --tag TEXT`      |                  | **필수**. Docker 이미지 태그. 예: `langgraph build -t my-image`                                                    |
    | `--no-pull`           |                  | 로컬에 빌드된 이미지를 사용합니다. 기본값은 최신 원격 Docker 이미지로 빌드하는 `false`입니다.                    |
    | `-c, --config FILE`   | `langgraph.json` | 의존성, 그래프 및 환경 변수를 선언하는 구성 파일의 경로입니다.                                                       |
    | `--help`              |                  | 명령 문서 표시.                                                                                                     |


### `up`

=== "Python"

    LangGraph API 서버를 시작합니다. 로컬 테스트를 위해 LangSmith API 키가 필요하며 LangGraph Cloud 폐쇄 베타에 대한 접근이 필요합니다. 생산 사용을 위해 라이선스 키가 필요합니다.

    **사용법**

    ```
    langgraph up [옵션]
    ```

    **옵션**

    | 옵션                             | 기본값                    | 설명                                                                                                                 |
    | -------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------- |
    | `--wait`                         |                           | 서비스가 시작될 때까지 기다립니다. --detach를 의미합니다.                                                          |
    | `--postgres-uri TEXT`            | 로컬 데이터베이스         | 데이터베이스에 사용할 Postgres URI입니다.                                                                           |
    | `--watch`                        |                           | 파일 변경 시 재시작                                                                                                 |
    | `--debugger-base-url TEXT`       | `http://127.0.0.1:[포트]` | 디버거가 LangGraph API에 접근하는 데 사용할 URL입니다.                                                             |
    | `--debugger-port INTEGER`        |                           | 디버거 이미지를 로컬에서 가져오고 지정된 포트에서 UI를 제공합니다.                                                |
    | `--verbose`                      |                           | 서버 로그에서 더 많은 출력을 표시합니다.                                                                          |
    | `-c, --config FILE`              | `langgraph.json`          | 의존성, 그래프 및 환경 변수를 선언하는 구성 파일의 경로입니다.                                                    |
    | `-d, --docker-compose FILE`      |                           | 추가 서비스를 시작하기 위한 docker-compose.yml 파일의 경로입니다.                                                 |
    | `-p, --port INTEGER`             | `8123`                    | 노출할 포트입니다. 예: `langgraph up --port 8000`                                                                   |
    | `--pull / --no-pull`             | `pull`                    | 최신 이미지를 끌어옵니다. 로컬에서 빌드된 이미지를 사용하기 위해 `--no-pull`를 사용합니다. 예: `langgraph up --no-pull` |
    | `--recreate / --no-recreate`     | `no-recreate`             | 구성 및 이미지가 변경되지 않은 경우에도 컨테이너를 재생성합니다.                                                 |
    | `--help`                         |                           | 명령 문서 표시.                                                                                                     |

=== "JS"

    LangGraph API 서버를 시작합니다. 로컬 테스트를 위해 LangSmith API 키가 필요하며 LangGraph Cloud 폐쇄 베타에 대한 접근이 필요합니다. 생산 사용을 위해 라이선스 키가 필요합니다.

    **사용법**

    ```
    npx @langchain/langgraph-cli up [옵션]
    ```

    **옵션**

    | 옵션                                                                 | 기본값                    | 설명                                                                                                                 |
    | ---------------------------------------------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------- |
    | <span style="white-space: nowrap;">`--wait`</span>                     |                           | 서비스가 시작될 때까지 기다립니다. --detach를 의미합니다.                                                          |
    | <span style="white-space: nowrap;">`--postgres-uri TEXT`</span>        | 로컬 데이터베이스         | 데이터베이스에 사용할 Postgres URI입니다.                                                                           |
    | <span style="white-space: nowrap;">`--watch`</span>                    |                           | 파일 변경 시 재시작                                                                                                 |
    | <span style="white-space: nowrap;">`-c, --config FILE`</span>          | `langgraph.json`          | 의존성, 그래프 및 환경 변수를 선언하는 구성 파일의 경로입니다.                                                    |
    | <span style="white-space: nowrap;">`-d, --docker-compose FILE`</span>  |                           | 추가 서비스를 시작하기 위한 docker-compose.yml 파일의 경로입니다.                                                 |
    | <span style="white-space: nowrap;">`-p, --port INTEGER`</span>         | `8123`                    | 노출할 포트입니다. 예: `langgraph up --port 8000`                                                                    |
    | <span style="white-space: nowrap;">`--no-pull`</span>                  |                           | 로컬에 빌드된 이미지를 사용합니다. 기본값은 최신 원격 Docker 이미지로 빌드하는 `false`입니다.                  |
    | <span style="white-space: nowrap;">`--recreate`</span>                 |                           | 구성 및 이미지가 변경되지 않은 경우에도 컨테이너를 재생성합니다.                                                 |
    | <span style="white-space: nowrap;">`--help`</span>                     |                           | 명령 문서 표시.                                                                                                     |

### `dockerfile`

=== "Python"

    LangGraph Cloud API 서버 Docker 이미지를 빌드하기 위한 Dockerfile을 생성합니다.

    **사용법**

    ```
    langgraph dockerfile [옵션] 저장 경로
    ```

    **옵션**

    | 옵션                     | 기본값          | 설명                                                                                                   |
    | ------------------------ | ---------------- | ------------------------------------------------------------------------------------------------------- |
    | `-c, --config FILE`      | `langgraph.json` | 의존성, 그래프 및 환경 변수를 선언하는 [구성 파일](#configuration-file)의 경로입니다.            |
    | `--help`                 |                  | 이 메시지를 표시하고 종료합니다.                                                                          |

    예시:

    ```bash
    langgraph dockerfile -c langgraph.json Dockerfile
    ```

    이렇게 생성된 Dockerfile은 다음과 유사하게 보입니다:

    ```dockerfile
    FROM langchain/langgraph-api:3.11

    ADD ./pipconf.txt /pipconfig.txt

    RUN PIP_CONFIG_FILE=/pipconfig.txt PYTHONDONTWRITEBYTECODE=1 pip install --no-cache-dir -c /api/constraints.txt langchain_community langchain_anthropic langchain_openai wikipedia scikit-learn

    ADD ./graphs /deps/__outer_graphs/src
    RUN set -ex && \
        for line in '[project]' \
                    'name = "graphs"' \
                    'version = "0.1"' \
                    '[tool.setuptools.package-data]' \
                    '"*" = ["**/*"]'; do \
            echo "$line" >> /deps/__outer_graphs/pyproject.toml; \
        done

    RUN PIP_CONFIG_FILE=/pipconfig.txt PYTHONDONTWRITEBYTECODE=1 pip install --no-cache-dir -c /api/constraints.txt -e /deps/*

    ENV LANGSERVE_GRAPHS='{"agent": "/deps/__outer_graphs/src/agent.py:graph", "storm": "/deps/__outer_graphs/src/storm.py:graph"}'
    ```

    ???+ 주의 "langgraph.json 파일 업데이트하기"
         `langgraph dockerfile` 명령은 `langgraph.json` 파일의 모든 설정을 Dockerfile 명령으로 변환합니다. 이 명령을 사용할 때는 `langgraph.json` 파일을 업데이트할 때마다 다시 실행해야 합니다. 그렇지 않으면 Dockerfile을 빌드하거나 실행할 때 변경 사항이 반영되지 않습니다.

=== "JS"

    LangGraph Cloud API 서버 Docker 이미지를 빌드하기 위한 Dockerfile을 생성합니다.

    **사용법**

    ```
    npx @langchain/langgraph-cli dockerfile [OPTIONS] SAVE_PATH
    ```

    **옵션**

    | 옵션                | 기본값          | 설명                                                                                                   |
    | ------------------- | ---------------- | ------------------------------------------------------------------------------------------------------- |
    | `-c, --config FILE` | `langgraph.json` | 의존성, 그래프 및 환경 변수를 선언하는 [구성 파일](#configuration-file) 경로입니다.              |
    | `--help`            |                  | 이 메시지를 표시하고 종료합니다.                                                                         |

    예제:

    ```bash
    npx @langchain/langgraph-cli dockerfile -c langgraph.json Dockerfile
    ```

    이는 다음과 유사한 Dockerfile을 생성합니다:

    ```dockerfile
    FROM langchain/langgraphjs-api:20
    
    ADD . /deps/agent
    
    RUN cd /deps/agent && yarn install
    
    ENV LANGSERVE_GRAPHS='{"agent":"./src/react_agent/graph.ts:graph"}'
    
    WORKDIR /deps/agent
    
    RUN (test ! -f /api/langgraph_api/js/build.mts && echo "사전 빌드 스크립트가 없습니다, 건너뜁니다") || tsx /api/langgraph_api/js/build.mts
    ```

    ???+ 주의 "langgraph.json 파일 업데이트하기"
         `npx @langchain/langgraph-cli dockerfile` 명령은 `langgraph.json` 파일의 모든 설정을 Dockerfile 명령으로 변환합니다. 이 명령을 사용할 때는 `langgraph.json` 파일을 업데이트할 때마다 다시 실행해야 합니다. 그렇지 않으면 Dockerfile을 빌드하거나 실행할 때 변경 사항이 반영되지 않습니다.
