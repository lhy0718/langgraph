_한국어로 기계번역됨_

# 자체 호스팅

!!! 주의 사항 전제 조건

    - [LangGraph 플랫폼](./langgraph_platform.md)
    - [배포 옵션](./deployment_options.md)

## 버전

자체 호스팅 배포에는 [자체 호스팅 기업 버전](./deployment_options.md#self-hosted-enterprise)과 [자체 호스팅 라이트 버전](./deployment_options.md#self-hosted-lite) 두 가지 버전이 있습니다.

### 자체 호스팅 라이트 버전

자체 호스팅 라이트 버전은 로컬이나 자체 호스팅 방식으로 실행할 수 있는 LangGraph 플랫폼의 제한된 버전입니다(연간 100만 노드 실행까지).

자체 호스팅 라이트 버전을 사용할 때는 [LangSmith](https://smith.langchain.com/) API 키로 인증합니다.

### 자체 호스팅 기업 버전

자체 호스팅 기업 버전은 LangGraph 플랫폼의 전체 버전입니다.

자체 호스팅 기업 버전을 사용하려면 Docker 이미지를 실행할 때 전달해야 하는 라이센스 키를 취득해야 합니다. 라이센스 키를 받으려면 sales@langchain.dev로 이메일을 보내주시기 바랍니다.

## 요구 사항

- `langgraph-cli` 및/또는 [LangGraph 스튜디오](./langgraph_studio.md) 앱을 사용하여 그래프를 로컬로 테스트합니다.
- `langgraph build` 명령어를 사용하여 이미지를 빌드합니다.

## 작동 방식

- 귀하의 인프라에서 Redis 및 Postgres 인스턴스를 배포합니다.
- [LangGraph CLI](./langgraph_cli.md)를 사용하여 [LangGraph 서버](./langgraph_server.md)용 Docker 이미지를 빌드합니다.
- Docker 이미지를 실행하고 필요한 환경 변수를 전달하는 웹 서버를 배포합니다.

!!! 경고 "참고"

    LangGraph 플랫폼 배포 보기 기능은 선택적으로 자체 호스팅 LangGraph 배포용으로 제공됩니다. 클릭 한 번으로 자체 호스팅 LangGraph 배포는 자가 호스팅된 LangSmith 인스턴스가 배포된 동일한 Kubernetes 클러스터에 배포될 수 있습니다.

단계별 지침은 [LangGraph의 자체 호스팅 배포 설정 방법](../how-tos/deploy-self-hosted.md)을 참조하시기 바랍니다.

## 헬름 차트

Kubernetes에서 LangGraph Cloud를 배포하려면 이 [헬름 차트](https://github.com/langchain-ai/helm/blob/main/charts/langgraph-cloud/README.md)를 사용할 수 있습니다.

## 관련

- [LangGraph의 자체 호스팅 배포 설정 방법](../how-tos/deploy-self-hosted.md)
