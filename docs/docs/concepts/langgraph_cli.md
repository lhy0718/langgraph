_한국어로 기계번역됨_

# LangGraph CLI

!!! 정보 "사전 요구 사항" - [LangGraph 플랫폼](./langgraph_platform.md) - [LangGraph 서버](./langgraph_server.md)

LangGraph CLI는 [LangGraph API 서버](./langgraph_server.md)를 로컬에서 빌드하고 실행하기 위한 다중 플랫폼 명령줄 도구입니다. 이는 모든 주요 운영 체제(리눅스, 윈도우, 맥OS)에서 에이전트를 개발하고 테스트하기 위해 [LangGraph Studio 데스크톱 앱](./langgraph_studio.md)의 대안으로 제공됩니다. 결과 서버에는 그래프의 실행, 스레드, 어시스턴트 등을 위한 모든 API 엔드포인트와 체크포인팅 및 저장을 위한 관리형 데이터베이스를 포함하여 에이전트를 실행하는 데 필요한 기타 서비스가 포함됩니다.

## 설치

LangGraph CLI는 Homebrew(맥OS) 또는 pip를 통해 설치할 수 있습니다.

=== "Homebrew"
`bash
    brew install langgraph-cli
    `

=== "pip"
`bash
    pip install langgraph-cli
    `

## 명령어

CLI는 다음과 같은 핵심 기능을 제공합니다:

### `build`

`langgraph build` 명령은 직접 배포할 수 있는 [LangGraph API 서버](./langgraph_server.md)용 Docker 이미지를 빌드합니다.

### `dev`

!!! 주의 "버전 0.1.55의 새로운 기능"
`langgraph dev` 명령은 langgraph-cli 버전 0.1.55에서 도입되었습니다.

!!! 주의 "파이썬 전용"

    현재 CLI는 Python >= 3.11만 지원합니다.
    JS 지원은 곧 제공될 예정입니다.

`langgraph dev` 명령은 Docker 설치가 필요 없는 경량 개발 서버를 시작합니다. 이 서버는 빠른 개발 및 테스트에 이상적이며, 다음과 같은 기능을 제공합니다:

- 핫 리로딩: 코드의 변경 사항이 자동으로 감지되고 다시 로드됨
- 디버거 지원: IDE의 디버거를 연결하여 라인별 디버깅
- 로컬 지속성을 가진 인메모리 상태: 서버 상태는 속도를 위해 메모리에 저장되지만 재시작 간에 로컬에 지속됨

이 명령어를 사용하려면 "inmem" 추가 기능과 함께 CLI를 설치해야 합니다:

```bash
pip install -U "langgraph-cli[inmem]"
```

**참고**: 이 명령은 로컬 개발 및 테스트 전용으로 설계되었습니다. 프로덕션 사용에는 권장되지 않습니다. Docker를 사용하지 않기 때문에 프로젝트의 종속성을 관리하기 위해 가상 환경을 사용하는 것이 좋습니다.

### `up`

`langgraph up` 명령은 로컬에서 Docker 컨테이너 내에 [LangGraph API 서버](./langgraph_server.md)의 인스턴스를 시작합니다. 이는 로컬에서 Docker 서버가 실행 중이어야 하며, 로컬 개발을 위해 LangSmith API 키나 프로덕션 사용을 위한 라이센스 키가 필요합니다.

서버에는 그래프의 실행, 스레드, 어시스턴트 등을 위한 모든 API 엔드포인트와 체크포인팅 및 저장을 위한 관리형 데이터베이스를 포함하여 에이전트를 실행하는 데 필요한 기타 서비스가 포함됩니다.

### `dockerfile`

`langgraph dockerfile` 명령은 [LangGraph API 서버](./langgraph_server.md)의 이미지 빌드 및 인스턴스 배포에 사용할 수 있는 [Dockerfile](https://docs.docker.com/reference/dockerfile/)을 생성합니다. 이는 Dockerfile을 추가로 사용자 정의하거나 보다 맞춤형 방식으로 배포하려는 경우에 유용합니다.

??? 주의 "langgraph.json 파일 업데이트"
`langgraph dockerfile` 명령은 `langgraph.json` 파일의 모든 구성을 Dockerfile 명령으로 변환합니다. 이 명령을 사용할 때 `langgraph.json` 파일을 업데이트할 때마다 재실행해야 합니다. 그렇지 않으면 Dockerfile을 빌드하거나 실행할 때 변경 사항이 반영되지 않습니다.

## 관련

- [LangGraph CLI API 참조](../cloud/reference/cli.md)
