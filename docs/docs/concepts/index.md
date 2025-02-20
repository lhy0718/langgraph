_한국어로 기계번역됨_

---
title: 개념
description: LangGraph에 대한 개념적 가이드
---

# 개념적 가이드

이 가이드는 LangGraph 프레임워크와 보다 넓은 AI 애플리케이션의 핵심 개념에 대한 설명을 제공합니다.

개념적 가이드에 들어가기 전에 최소한 [빠른 시작](../tutorials/introduction.ipynb)을 진행할 것을 권장합니다. 이 과정은 이곳에서 논의되는 개념을 이해하는 데 도움이 되는 실제적인 맥락을 제공합니다.

이 개념적 가이드는 단계별 지침이나 특정 구현 예제를 다루지 않습니다. 그런 내용은 [튜토리얼](../tutorials/index.md)과 [사용 방법 가이드](../how-tos/index.md)에서 찾을 수 있습니다. 자세한 참조 자료는 [API 참조](../reference/index.md)를 참조하시기 바랍니다.

## LangGraph

### 전체 개요

- [왜 LangGraph인가?](high_level.md): LangGraph와 그 목표에 대한 고수준 개요입니다.

### 개념

- [LangGraph 용어집](low_level.md): LangGraph 워크플로우는 그래프로 디자인되어 있으며, 노드는 다양한 구성요소를 나타내고 엣지는 그들 간의 정보 흐름을 나타냅니다. 이 가이드는 LangGraph 그래프 기본 요소와 관련된 핵심 개념을 제공합니다.
- [일반적인 에이전트 패턴](agentic_concepts.md): 에이전트는 LLM을 사용하여 더 복잡한 문제를 해결하기 위해 자체적으로 제어 흐름을 선택합니다! 에이전트는 많은 LLM 애플리케이션의 주요 구성 요소입니다. 이 가이는 다양한 종류의 에이전트 아키텍처와 이를 통해 애플리케이션의 흐름을 제어하는 방법을 설명합니다.
- [다중 에이전트 시스템](multi_agent.md): 복잡한 LLM 애플리케이션은 종종 여러 에이전트로 나누어질 수 있으며, 각 에이전트는 애플리케이션의 다른 부분을 책임집니다. 이 가이드는 다중 에이전트 시스템을 구축하기 위한 일반적인 패턴을 설명합니다.
- [중단점](breakpoints.md): 중단점을 사용하면 특정 지점에서 그래프의 실행을 일시 중지할 수 있습니다. 중단점은 디버깅 목적으로 그래프 실행을 단계별로 진행하는 데 도움이 됩니다.
- [인간 개입](human_in_the_loop.md): LangGraph 애플리케이션에 인간의 피드백을 통합하는 다양한 방법을 설명합니다.
- [타임 트래블](time-travel.md): 타임 트래블을 통해 LangGraph 애플리케이션의 과거 작업을 재생하여 대체 경로를 탐색하고 문제를 디버깅할 수 있습니다.
- [영속성](persistence.md): LangGraph에는 체크포인터를 통해 구현된 내장 영속성 계층이 있습니다. 이 영속성 계층은 인간 개입, 메모리, 타임 트래블, 내결함성 같은 강력한 기능을 지원하는 데 도움이 됩니다.
- [메모리](memory.md): AI 애플리케이션의 메모리는 과거 상호작용에서 정보를 처리, 저장 및 효과적으로 회상하는 능력을 말합니다. 메모리를 사용하면 에이전트가 피드백을 학습하고 사용자의 선호도에 따라 적응할 수 있습니다.
- [스트리밍](streaming.md): 스트리밍은 LLM 기반 애플리케이션의 응답성을 향상시키는 데 매우 중요합니다. 전체 응답이 준비되기 전에 출력을 점진적으로 표시하여 LLM의 지연 문제를 처리할 때 사용자 경험(UX)을 크게 향상시킵니다.
- [기능적 API](functional_api.md): `@entrypoint` 및 `@task` 데코레이터를 사용하여 기존 코드베이스에 LangGraph 기능을 추가할 수 있습니다.
- [내구성 있는 실행](durable_execution.md): LangGraph의 내장 [영속성](./persistence.md) 계층은 워크플로우에 대한 내구성 있는 실행을 제공하여 각 실행 단계의 상태를 내구성 있는 저장소에 저장합니다.
- [자주 묻는 질문](faq.md): LangGraph에 대한 자주 묻는 질문입니다.

## LangGraph 플랫폼

LangGraph 플랫폼은 상업적으로 에이전트 기반 애플리케이션을 프로덕션 환경에 배포하기 위한 솔루션으로, 오픈 소스 LangGraph 프레임워크 위에 구축되었습니다.

LangGraph 플랫폼은 [배포 옵션 가이드](./deployment_options.md)에서 설명된 몇 가지 배포 옵션을 제공합니다.

!!! 팁

    * LangGraph는 MIT 라이센스를 가진 오픈 소스 라이브러리로, 우리는 커뮤니티를 위해 유지하고 발전시키는 데 전념하고 있습니다.
    * LangGraph 플랫폼을 사용하지 않고도 오픈 소스 LangGraph 프로젝트를 통해 자체 인프라에 LangGraph 애플리케이션을 배포할 수 있습니다.

### 전체 개요

- [왜 LangGraph 플랫폼인가?](./langgraph_platform.md): LangGraph 플랫폼은 LangGraph 애플리케이션을 배포하고 관리하기 위한 특정한 방법입니다. 이 가이드는 LangGraph 플랫폼의 주요 기능과 개념에 대한 개요를 제공합니다.
- [플랫폼 아키텍처](./platform_architecture.md): LangGraph 플랫폼 아키텍처에 대한 고수준 개요입니다.
- [배포 옵션](./deployment_options.md): LangGraph 플랫폼은 [자체 호스팅 라이트](./self_hosted.md#self-hosted-lite), [자체 호스팅 엔터프라이즈](./self_hosted.md#self-hosted-enterprise), [구름 가져오기 (BYOC)](./bring_your_own_cloud.md), [클라우드 SaaS](./langgraph_cloud.md)라는 네 가지 배포 옵션을 제공합니다. 이 가이드는 이러한 옵션 간의 차이점과 각 계획에서 제공되는 내용을 설명합니다.
- [계획](./plans.md): LangGraph 플랫폼은 개발자, 플러스, 엔터프라이즈라는 세 가지 다른 계획을 제공합니다. 이 가이드는 이러한 옵션 간의 차이점, 각 옵션에 대한 배포 옵션 및 각 옵션에 대한 가입 방법을 설명합니다.
- [템플릿 애플리케이션](./template_applications.md): LangGraph로 신속하게 시작하는 데 도움이 되는 참조 애플리케이션입니다.

### 구성 요소

LangGraph 플랫폼은 LangGraph 애플리케이션의 배포 및 관리를 지원하기 위해 함께 작동하는 여러 구성 요소로 구성됩니다:

- [LangGraph 서버](./langgraph_server.md): LangGraph 서버는 백그라운드 처리부터 실시간 상호작용에 이르기까지 다양한 에이전트 애플리케이션 사용 사례를 지원하도록 설계되었습니다.
- [LangGraph 스튜디오](./langgraph_studio.md): LangGraph 스튜디오는 LangGraph 서버에 연결하여 애플리케이션의 시각화, 상호작용 및 디버깅을 로컬에서 가능하게 하는 전문 IDE입니다.
- [LangGraph CLI](./langgraph_cli.md): LangGraph CLI는 로컬 LangGraph와 상호작용하는 데 도움이 되는 명령줄 인터페이스입니다.
- [Python/JS SDK](./sdk.md): Python/JS SDK는 배포된 LangGraph 애플리케이션과 상호작용하는 프로그래밍 방식의 방법을 제공합니다.
- [원격 그래프](../how-tos/use-remote-graph.md): 원격 그래프를 사용하면 로컬에서 실행되는 것처럼 배포된 LangGraph 애플리케이션과 상호작용할 수 있습니다.

### LangGraph 서버

- [애플리케이션 구조](./application_structure.md): LangGraph 애플리케이션은 하나 이상의 그래프, LangGraph API 구성 파일(`langgraph.json`), 의존성을 지정하는 파일, 환경 변수로 구성됩니다.
- [어시스턴트](./assistants.md): 어시스턴트는 LangGraph 애플리케이션의 다양한 구성을 저장하고 관리하는 방법입니다.
- [웹훅](./langgraph_server.md#webhooks): 웹훅은 실행 중인 LangGraph 애플리케이션이 특정 이벤트에 대해 외부 서비스에 데이터를 전송할 수 있게 해줍니다.
- [크론 작업](./langgraph_server.md#cron-jobs): 크론 작업은 LangGraph 애플리케이션에서 특정 시간에 작업을 실행하도록 예약하는 방법입니다.
- [중복 텍스트](./double_texting.md): 중복 텍스트는 사용자가 그래프 실행이 끝나기 전에 여러 메시지를 보낼 때 발생하는 LLM 애플리케이션에서의 일반적인 문제입니다. 이 가이드는 LangGraph 배포에서 중복 텍스트를 처리하는 방법을 설명합니다.
- [인증 및 접근 제어](./auth.md): LangGraph 플랫폼을 배포할 때 인증 및 접근 제어 옵션에 대해 알아보세요.

### 배포 옵션

- [자체 호스팅 라이트](./self_hosted.md): 무료(연간 100만 노드 실행까지)의 제한된 버전으로, 로컬 또는 자체 호스팅 방식으로 실행할 수 있습니다.
- [클라우드 SaaS](./langgraph_cloud.md): LangSmith의 일부로 호스팅됩니다.
- [구름 가져오기](./bring_your_own_cloud.md): 우리는 인프라를 관리하므로 귀하는 관리하지 않아도 되지만, 모든 인프라는 귀하의 클라우드 내에서 실행됩니다.
- [자체 호스팅 엔터프라이즈](./self_hosted.md): 완전히 귀하가 관리합니다.
