_한국어로 기계번역됨_

# 배포 옵션

!!! info " prerequisites"

    - [LangGraph 플랫폼](./langgraph_platform.md)
    - [LangGraph 서버](./langgraph_server.md)
    - [LangGraph 플랫폼 플랜](./plans.md)

## 개요

LangGraph 플랫폼을 사용한 배포에는 4가지 주요 옵션이 있습니다:

1. **[셀프 호스팅 라이트](#self-hosted-lite)**: 모든 플랜에서 사용 가능합니다.

2. **[셀프 호스팅 엔터프라이즈](#self-hosted-enterprise)**: **엔터프라이즈** 플랜에서 사용 가능합니다.

3. **[클라우드 SaaS](#cloud-saas)**: **플러스** 및 **엔터프라이즈** 플랜에서 사용 가능합니다.

4. **[내 클라우드 사용하기](#bring-your-own-cloud)**: **엔터프라이즈** 플랜에서만 사용 가능하며 **AWS에서만** 가능합니다.

자세한 정보는 [LangGraph 플랫폼 플랜](./plans.md)을 참조하시기 바랍니다.

아래 가이드는 배포 옵션 간의 차이를 설명합니다.

## 셀프 호스팅 엔터프라이즈

!!! 중요

    셀프 호스팅 엔터프라이즈 버전은 **엔터프라이즈** 플랜에서만 사용할 수 있습니다.

!!! 경고 "참고"

    셀프 호스팅 엔터프라이즈 LangGraph 배포의 경우 LangSmith SaaS 및 셀프 호스팅 LangSmith 내의 LangGraph 플랫폼 배포 보기 사용이 불가능합니다. 셀프 호스팅 LangGraph 배포는 LangSmith 외부에서 관리됩니다(예: 이러한 배포를 관리하기 위한 UI가 없습니다).

셀프 호스팅 엔터프라이즈 배포의 경우, 필요한 데이터베이스와 Redis 인스턴스를 설정하고 유지하는 등 인프라 관리에 대한 책임이 있습니다.

당신은 [LangGraph CLI](./langgraph_cli.md)를 사용하여 Docker 이미지를 구축하고, 이를 자신의 인프라에 배포할 수 있습니다.

자세한 내용은 다음을 참조하십시오:

- [셀프 호스팅 개념 가이드](./self_hosted.md)
- [셀프 호스팅 배포 방법 가이드](../how-tos/deploy-self-hosted.md)

## 셀프 호스팅 라이트

!!! 중요

    셀프 호스팅 라이트 버전은 모든 플랜에서 사용 가능합니다.

!!! 경고 "참고"

    셀프 호스팅 라이트 LangGraph 배포의 경우 LangSmith SaaS 및 셀프 호스팅 LangSmith 내의 LangGraph 플랫폼 배포 보기 사용이 불가능합니다. 셀프 호스팅 LangGraph 배포는 LangSmith 외부에서 관리됩니다(예: 이러한 배포를 관리하기 위한 UI가 없습니다).

셀프 호스팅 라이트 배포 옵션은 매년 최대 100만 노드가 실행될 수 있는 무료의 제한된 버전의 LangGraph 플랫폼입니다. 이를 로컬 또는 셀프 호스팅 방식으로 실행할 수 있습니다.

셀프 호스팅 라이트 배포의 경우, 필요한 데이터베이스와 Redis 인스턴스를 설정하고 유지하는 등 인프라 관리에 대한 책임이 있습니다.

당신은 [LangGraph CLI](./langgraph_cli.md)를 사용하여 Docker 이미지를 구축하고, 이를 자신의 인프라에 배포할 수 있습니다.

셀프 호스팅 라이트 배포에서는 [크론 작업](../cloud/how-tos/cron_jobs.md)을 사용할 수 없습니다.

자세한 내용은 다음을 참조하십시오:

- [셀프 호스팅 개념 가이드](./self_hosted.md)
- [셀프 호스팅 배포 방법 가이드](../how-tos/deploy-self-hosted.md)

## 클라우드 SaaS

!!! 중요

    LangGraph 플랫폼의 클라우드 SaaS 버전은 **플러스** 및 **엔터프라이즈** 플랜에서만 사용할 수 있습니다.

LangGraph 플랫폼의 [클라우드 SaaS](./langgraph_cloud.md) 버전은 [LangSmith](https://smith.langchain.com/)의 일부로 호스팅됩니다.

클라우드 SaaS 버전의 LangGraph 플랫폼은 LangGraph 애플리케이션을 배포하고 관리하는 간단한 방법을 제공합니다.

이 배포 옵션은 LangGraph 플랫폼 UI(그룹 내 LangSmith) 및 GitHub와의 통합에 접근할 수 있어, GitHub의 모든 리포지토리에서 코드를 배포할 수 있습니다.

자세한 내용은 다음을 참조하십시오:

- [클라우드 SaaS 개념 가이드](./langgraph_cloud.md)
- [클라우드 SaaS에 배포하는 방법](../cloud/deployment/cloud.md)

## 내 클라우드 사용하기

!!! 중요

    LangGraph 플랫폼의 내 클라우드 사용하기 버전은 **엔터프라이즈** 플랜에서만 사용 가능합니다.

이 옵션은 클라우드와 셀프 호스팅의 두 가지 장점을 결합합니다. LangGraph 플랫폼 UI(그룹 내 LangSmith)를 통해 배포를 생성하고, 인프라는 관리하므로 사용자가 직접 관리할 필요가 없습니다. 모든 인프라는 귀하의 클라우드 내에서 실행됩니다. 현재 AWS에서만 사용 가능합니다.

자세한 내용은 다음을 참조하십시오:

- [내 클라우드 사용하기 개념 가이드](./bring_your_own_cloud.md)

## 관련

자세한 정보는 다음을 참조하십시오:

- [LangGraph 플랫폼 계획](./plans.md)
- [LangGraph 플랫폼 가격](https://www.langchain.com/langgraph-platform-pricing)
- [배포 방법 가이드](../how-tos/index.md#deployment)
