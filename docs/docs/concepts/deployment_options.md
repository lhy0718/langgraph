_한국어로 기계번역됨_

# 배포 옵션

!!! 정보 "사전 요구 사항"

    - [LangGraph 플랫폼](./langgraph_platform.md)
    - [LangGraph 서버](./langgraph_server.md)
    - [LangGraph 플랫폼 요금제](./plans.md)

## 개요

LangGraph 플랫폼에는 4가지 주요 배포 옵션이 있습니다:

1. **[셀프 호스팅 라이트](#self-hosted-lite)**: 모든 요금제에서 제공.

2. **[셀프 호스팅 엔터프라이즈](#self-hosted-enterprise)**: **엔터프라이즈** 요금제에서 제공.

3. **[클라우드 SaaS](#cloud-saas)**: **플러스** 및 **엔터프라이즈** 요금제에서 제공.

4. **[자체 클라우드 사용](#bring-your-own-cloud)**: **엔터프라이즈** 요금제에서만 제공되며 **AWS에서만** 가능합니다.

다양한 요금제에 대한 정보는 [LangGraph 플랫폼 요금제](./plans.md)를 참조하십시오.

아래 가이드는 배포 옵션 간의 차이를 설명합니다.

## 셀프 호스팅 엔터프라이즈

!!! 중요

    셀프 호스팅 엔터프라이즈 버전은 **엔터프라이즈** 요금제에만 제공됩니다.

!!! 경고 "참고"

    LangGraph 플랫폼 배포 보기 기능은 선택적으로 셀프 호스팅 엔터프라이즈 LangGraph 배포에서 사용할 수 있습니다. 한 번의 클릭으로 셀프 호스팅된 LangSmith 인스턴스가 배포된 동일한 Kubernetes 클러스터에 셀프 호스팅된 LangGraph 배포를 배포할 수 있습니다.

셀프 호스팅 엔터프라이즈 배포에서 인프라를 관리할 책임이 있으며, 필요한 데이터베이스 및 Redis 인스턴스를 설정하고 유지해야 합니다.

[LangGraph CLI](./langgraph_cli.md)를 사용하여 Docker 이미지를 빌드한 다음, 이를 자신의 인프라에 배포할 수 있습니다.

자세한 내용은 다음을 참조하십시오:

- [셀프 호스팅 개념 가이드](./self_hosted.md)
- [셀프 호스팅 배포 방법 가이드](../how-tos/deploy-self-hosted.md)

## 셀프 호스팅 라이트

!!! 중요

    셀프 호스팅 라이트 버전은 모든 요금제에서 제공됩니다.

!!! 경고 "참고"

    LangGraph 플랫폼 배포 보기 기능은 선택적으로 셀프 호스팅 라이트 LangGraph 배포에서 사용할 수 있습니다. 한 번의 클릭으로 셀프 호스팅된 LangSmith 인스턴스가 배포된 동일한 Kubernetes 클러스터에 셀프 호스팅된 LangGraph 배포를 배포할 수 있습니다.

셀프 호스팅 라이트 배포 옵션은 매년 100만 노드 실행까지 무료로 제공되는 LangGraph 플랫폼의 제한된 버전으로, 로컬 또는 셀프 호스팅 방식으로 실행할 수 있습니다.

셀프 호스팅 라이트 배포에서 인프라를 관리할 책임이 있으며, 필요한 데이터베이스 및 Redis 인스턴스를 설정하고 유지해야 합니다.

[LangGraph CLI](./langgraph_cli.md)를 사용하여 Docker 이미지를 빌드한 다음, 이를 자신의 인프라에 배포할 수 있습니다.

(Cron 작업)[../cloud/how-tos/cron_jobs.md]은 셀프 호스팅 라이트 배포에 사용할 수 없습니다.

자세한 내용은 다음을 참조하십시오:

- [셀프 호스팅 개념 가이드](./self_hosted.md)
- [셀프 호스팅 배포 방법 가이드](../how-tos/deploy-self-hosted.md)

## 클라우드 SaaS

!!! 중요

    LangGraph 플랫폼의 클라우드 SaaS 버전은 **플러스** 및 **엔터프라이즈** 요금제에만 제공됩니다.

LangGraph 플랫폼의 [클라우드 SaaS](./langgraph_cloud.md) 버전은 [LangSmith](https://smith.langchain.com/)의 일부로 호스팅됩니다.

클라우드 SaaS 버전의 LangGraph 플랫폼은 LangGraph 애플리케이션을 배포하고 관리하는 간단한 방법을 제공합니다.

이 배포 옵션은 LangGraph 플랫폼 UI( LangSmith 내)와 GitHub 통합에 대한 액세스를 제공하여 GitHub의 모든 리포지토리에서 코드를 배포할 수 있게 합니다.

자세한 내용은 다음을 참조하십시오:

- [클라우드 SaaS 개념 가이드](./langgraph_cloud.md)
- [클라우드 SaaS에 배포하는 방법](../cloud/deployment/cloud.md)

## 자체 클라우드 사용

!!! 중요

    LangGraph 플랫폼의 자체 클라우드 사용 버전은 **엔터프라이즈** 요금제에만 제공됩니다.

이 옵션은 클라우드와 셀프 호스팅의 장점을 결합합니다. LangGraph 플랫폼 UI(LangSmith 내)를 통해 배포를 생성하고, 인프라는 저희가 관리하여 귀하는 관리할 필요가 없습니다. 모든 인프라는 귀하의 클라우드 내에서 실행됩니다. 현재 AWS에서만 제공됩니다.

자세한 내용은 다음을 참조하십시오:

- [자체 클라우드 사용 개념 가이드](./bring_your_own_cloud.md)

## 관련

자세한 내용은 다음을 참조하십시오:

- [LangGraph 플랫폼 요금제](./plans.md)
- [LangGraph 플랫폼 가격](https://www.langchain.com/langgraph-platform-pricing)
- [배포 방법 안내](../how-tos/index.md#deployment)
