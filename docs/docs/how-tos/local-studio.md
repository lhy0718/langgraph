_한국어로 기계번역됨_

# 로컬 에이전트를 LangGraph Studio에 연결하는 방법

이 가이드는 시각화, 상호작용 및 디버깅을 위해 로컬 에이전트를 [LangGraph Studio](../concepts/langgraph_studio.md)에 연결하는 방법을 보여줍니다.

## 연결 옵션

로컬 에이전트를 LangGraph Studio에 연결하는 방법은 두 가지가 있습니다:

- [개발 서버](../concepts/langgraph_studio.md#development-server-with-web-ui): Python 패키지, 모든 플랫폼, Docker 필요 없음
- [LangGraph Desktop](../concepts/langgraph_studio.md#desktop-app): 애플리케이션, Mac 전용, Docker 필요

이 가이드에서는 일반적으로 더 쉽고 나은 경험을 제공하는 개발 서버를 사용하는 방법을 다룹니다.

## 애플리케이션 설정하기

먼저, 애플리케이션을 적절한 형식으로 설정해야 합니다.
이는 에이전트에 대한 경로를 포함하는 `langgraph.json` 파일을 정의하는 것을 의미합니다.
어떻게 해야 하는지에 대한 정보는 [이 가이드](../concepts/application_structure.md)를 참조하세요.

## langgraph-cli 설치하기

[`langgraph-cli`](../cloud/reference/cli.md#langgraph-cli) (버전 `0.1.55` 이상)를 설치해야 합니다.
`inmem` 추가 항목을 설치해야 합니다.

???+ 주의 "최소 버전"

  `langgraph-cli`와 함께 `inmem` 추가 항목을 사용하기 위한 최소 버전은 `0.1.55`입니다.
  Python 3.11 이상이 필요합니다.
  

```shell
pip install -U "langgraph-cli[inmem]"
```

## 개발 서버 실행하기

1. 프로젝트 디렉터리로 이동합니다 (여기서는 `langgraph.json`이 위치한 곳입니다).

2. 서버를 시작합니다:
   ```bash
   langgraph dev
   ```

이 명령은 현재 디렉터리에서 `langgraph.json` 파일을 찾습니다. 
그러면 그 안에서 그래프의 경로를 찾아 시작합니다.
그 후 자동으로 클라우드 호스팅 스튜디오에 연결됩니다.

## 스튜디오 사용하기

스튜디오에 연결하면 브라우저 창이 자동으로 팝업됩니다.
이것은 클라우드 호스팅 스튜디오 UI를 사용하여 로컬 개발 서버에 연결합니다.
그래프는 여전히 로컬에서 실행 중이며, UI는 로컬에서 정의된 에이전트와 스레드를 시각화합니다.

그래프는 항상 최신 코드를 사용하므로 기본 코드를 변경하고 스튜디오에 자동으로 반영됩니다.
이는 워크플로우 디버깅에 유용합니다.
UI에서 그래프를 실행하다가 문제가 생기면 코드를 변경한 다음, 실패한 노드에서 다시 실행할 수 있습니다.

# (옵션) 디버거 연결하기

중단점 및 변수 검사를 통한 단계별 디버깅을 위해:

```bash
# debugpy 패키지 설치
pip install debugpy

# 디버깅을 활성화하여 서버 시작
langgraph dev --debug-port 5678
```

그 후 선호하는 디버거를 연결합니다:

=== "VS Code"
    `launch.json`에 다음 구성을 추가하세요:
    ```json
    {
      "name": "LangGraph에 연결",
      "type": "debugpy",
      "request": "attach",
      "connect": {
        "host": "0.0.0.0",
        "port": 5678
      }
    }
    ```
    이전 단계에서 선택한 포트 번호를 지정합니다.

=== "PyCharm"
    1. 실행 → 구성 편집으로 이동합니다.
    2. +를 클릭하고 "Python Debug Server"를 선택합니다.
    3. IDE 호스트 이름을 설정합니다: `localhost`
    4. 포트를 설정합니다: `5678` (또는 이전 단계에서 선택한 포트 번호)
    5. "확인"을 클릭하고 디버깅을 시작합니다.