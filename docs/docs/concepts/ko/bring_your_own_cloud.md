# 클라우드 가져오기 (BYOC)

!!! 주의 사전 요건

    - [LangGraph 플랫폼](./langgraph_platform.md)
    - [배포 옵션](./deployment_options.md)

## 아키텍처

제어 평면(호스팅: 우리)과 데이터 평면(호스팅: 당신, 관리: 우리)으로 분리됨.

|                             | 제어 평면                         | 데이터 평면                                |
|-----------------------------|----------------------------------|-------------------------------------------|
| 기능                        | 배포, 수정 관리                   | LangGraph 그래프 실행, 데이터 저장       |
| 호스팅 위치                | LangChain Cloud 계정              | 귀하의 클라우드 계정                      |
| 프로비저닝 및 모니터링 주체 | LangChain                        | LangChain                                 |

LangChain은 귀하의 클라우드 계정에서 생성된 리소스에 직접 접근할 수 없으며, 오직 AWS API를 통해서만 상호 작용할 수 있습니다. 귀하의 데이터는 정지 상태나 전송 중에도 귀하의 클라우드 계정을 벗어나지 않습니다.

![Architecture](img/byoc_architecture.png)

## 요구 사항

- 이미 AWS를 사용하고 있습니다.
- `langgraph-cli` 및/또는 [LangGraph Studio](./langgraph_studio.md) 앱을 사용해 그래프를 로컬에서 테스트합니다.
- `langgraph build` 명령어를 사용하여 이미지를 빌드한 후 AWS ECR 저장소에 푸시합니다 (`docker push`).

## 작동 방식

- 우리의 요구 사항을 설정하기 위해 실행하는 [Terraform 모듈](https://github.com/langchain-ai/terraform/tree/main/modules/langgraph_cloud_setup)을 제공합니다.
    1. AWS 역할을 생성합니다(우리의 제어 평면이 리소스를 프로비저닝하고 모니터링하기 위해 나중에 사용할 역할).
        - https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonVPCReadOnlyAccess.html
            - 하위 네트워크를 찾기 위해 VPC 읽기
        - https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonECS_FullAccess.html
            - LangGraph Cloud 인스턴스를 위한 ECS 리소스 생성/삭제에 사용
        - https://docs.aws.amazon.com/aws-managed-policy/latest/reference/SecretsManagerReadWrite.html
            - ECS 리소스를 위한 비밀 생성
        - https://docs.aws.amazon.com/aws-managed-policy/latest/reference/CloudWatchReadOnlyAccess.html
            - 인스턴스 모니터링 및 배포 로그 푸시를 위해 CloudWatch 메트릭/로그 읽기
        - https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonRDSFullAccess.html
            - LangGraph Cloud 인스턴스를 위한 `RDS` 인스턴스 프로비저닝
            - 기본 `RDS` 인스턴스 대신 외부 관리되는 Postgres 인스턴스를 사용할 수 있습니다. LangChain은 외부 관리되는 Postgres 인스턴스를 모니터링하거나 관리하지 않습니다. [`POSTGRES_URI_CUSTOM` 환경 변수](../cloud/reference/env_var.md#postgres_uri_custom) 관련 세부사항을 참조하십시오.
    2. 다음 중 하나 수행:
        - 기존 VPC/하위 네트워크에 `langgraph-cloud-enabled` 태그 추가
        - 새로운 VPC와 하위 네트워크 생성 후 `langgraph-cloud-enabled` 태그 추가
- 귀하는 `smith.langchain.com`에서 LangGraph Cloud 프로젝트를 생성하며
    - 위 단계에서 생성된 AWS 역할의 ID 제공
    - 서비스 이미지를 가져올 AWS ECR 저장소 제공
- 우리는 위 역할을 사용하여 귀하의 클라우드 계정에 리소스를 프로비저닝합니다.
- 우리는 이러한 리소스를 모니터링하여 가동 시간과 오류 복구를 보장합니다.

[자체 호스팅된 LangSmith](https://docs.smith.langchain.com/self_hosting) 사용 고객을 위한 주의 사항:

- 새로운 LangGraph Cloud 프로젝트와 수정의 생성은 현재 `smith.langchain.com`에서 수행해야 합니다.
- 그러나 원하신다면 프로젝트를 귀하의 자체 호스팅된 LangSmith 인스턴스에 연결할 수 있습니다. [`LANGSMITH_RUNS_ENDPOINTS` 환경 변수](../cloud/reference/env_var.md#langsmith_runs_endpoints) 관련 세부사항을 참조하십시오.
