_한국어로 기계번역됨_

# 셀프 호스팅

!!! 참고 선결 조건

    - [LangGraph 플랫폼](./langgraph_platform.md)
    - [배포 옵션](./deployment_options.md)

## 버전

셀프 호스팅 배포에는 두 가지 버전이 있습니다: [Self-Hosted Enterprise](./deployment_options.md#self-hosted-enterprise) 및 [Self-Hosted Lite](./deployment_options.md#self-hosted-lite).

### Self-Hosted Lite

Self-Hosted Lite 버전은 로컬 또는 셀프 호스팅 방식으로 실행할 수 있는 LangGraph 플랫폼의 제한된 버전입니다(연간 100만 개 노드 실행까지).

Self-Hosted Lite 버전을 사용할 때는 [LangSmith](https://smith.langchain.com/) API 키로 인증합니다.

### Self-Hosted Enterprise

Self-Hosted Enterprise 버전은 LangGraph 플랫폼의 전체 버전입니다.

Self-Hosted Enterprise 버전을 사용하려면 Docker 이미지를 실행할 때 전달해야 하는 라이센스 키를 획득해야 합니다. 라이센스 키를 얻으려면 sales@langchain.dev로 이메일을 보내 주세요.

## 요구 사항

- 그래프를 로컬에서 테스트하려면 `langgraph-cli` 및/또는 [LangGraph Studio](./langgraph_studio.md) 앱을 사용합니다.
- `langgraph build` 명령을 사용하여 이미지를 빌드합니다.

## 작동 방식

- 자체 인프라에서 Redis 및 Postgres 인스턴스를 배포합니다.
- [LangGraph CLI](./langgraph_cli.md)를 사용하여 [LangGraph Server](./langgraph_server.md)용 Docker 이미지를 빌드합니다.
- Docker 이미지를 실행하고 필요한 환경 변수를 전달하는 웹 서버를 배포합니다.

!!! 경고 "참고"

    LangGraph 플랫폼 배포 보기(이제까지 LangSmith SaaS 및 자체 호스팅 LangSmith 내)는 Self-Hosted Lite 또는 Self-Hosted Enterprise LangGraph 배포에서 사용할 수 없습니다. 자체 호스팅 LangGraph 배포는 LangSmith 외부에서 관리됩니다(예: 이러한 배포를 관리하기 위한 UI가 없습니다).

단계별 지침은 [LangGraph 셀프 호스팅 배포 설정 방법](../how-tos/deploy-self-hosted.md)을 참조하세요.

## 헬름 차트

Kubernetes에 LangGraph Cloud를 배포하려면 이 [헬름 차트](https://github.com/langchain-ai/helm/blob/main/charts/langgraph-cloud/README.md)를 사용할 수 있습니다.

## 관련 정보

- [LangGraph 셀프 호스팅 배포 설정 방법](../how-tos/deploy-self-hosted.md).
