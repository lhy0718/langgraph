---
title: 개념
description: LangGraph를 위한 개념 가이드
---

# 개념 가이드

이 가이드는 LangGraph 프레임워크와 AI 애플리케이션의 핵심 개념에 대한 설명을 제공합니다.

이 개념 가이드를 살펴보기 전에 최소한 [빠른 시작 가이드](../tutorials/introduction.ipynb)를 진행하는 것이 좋습니다. 이 가이드를 통해 실제적인 컨텍스트를 이해할 수 있어 여기에서 다루는 개념들을 더 쉽게 이해할 수 있습니다.

이 개념 가이드는 단계별 지침이나 특정 구현 예제를 다루지 않습니다. 그런 예제들은 [튜토리얼](../tutorials/index.md)과 [사용법 가이드](../how-tos/index.md)에서 찾을 수 있습니다. 자세한 참조 자료는 [API 참조](../reference/index.md)에서 확인하세요.

## LangGraph

### 높은 수준

- [왜 LangGraph인가?](high_level.md): LangGraph의 목표와 개요에 대한 설명.

### 개념들

- [LangGraph 용어집](low_level.md): LangGraph 워크플로우는 그래프 형태로 설계되며, 노드는 다양한 컴포넌트를 나타내고 엣지는 그들 사이의 정보 흐름을 나타냅니다. 이 가이드는 LangGraph 그래프 원칙과 관련된 주요 개념들을 제공합니다.
- [일반적인 에이전트 패턴](agentic_concepts.md): 에이전트는 더 복잡한 문제를 해결하기 위해 LLM을 사용하여 자체적으로 제어 흐름을 선택합니다! 에이전트는 많은 LLM 애플리케이션에서 중요한 구성 요소입니다. 이 가이드는 다양한 에이전트 아키텍처 유형과 이를 애플리케이션 흐름 제어에 어떻게 활용할 수 있는지 설명합니다.
- [다중 에이전트 시스템](multi_agent.md): 복잡한 LLM 애플리케이션은 종종 여러 에이전트로 나눠질 수 있으며, 각 에이전트는 애플리케이션의 다른 부분을 담당합니다. 이 가이드는 다중 에이전트 시스템을 구축하는 일반적인 패턴을 설명합니다.
- [중단점](breakpoints.md): 중단점은 그래프의 특정 지점에서 실행을 일시 중지할 수 있게 해줍니다. 중단점을 사용하면 디버깅을 위해 그래프 실행을 단계별로 실행할 수 있습니다.
- [인간-루프](human_in_the_loop.md): LangGraph 애플리케이션에 인간 피드백을 통합하는 다양한 방법을 설명합니다.
- [타임 트래블](time-travel.md): 타임 트래블을 사용하면 LangGraph 애플리케이션에서 과거의 작업을 재생하여 대체 경로를 탐색하고 문제를 디버깅할 수 있습니다.
- [지속성](persistence.md): LangGraph는 내장된 지속성 계층을 통해 강력한 기능을 지원합니다. 이 계층은 체크포인터를 통해 구현되며, 인간-루프, 메모리, 타임 트래블, 장애 내성 같은 기능을 지원합니다.
- [메모리](memory.md): AI 애플리케이션에서 메모리는 과거 상호작용에서 정보를 처리, 저장 및 효과적으로 기억할 수 있는 능력을 의미합니다. 메모리를 통해 에이전트는 피드백을 학습하고 사용자의 선호도에 맞게 적응할 수 있습니다.
- [스트리밍](streaming.md): 스트리밍은 LLM 기반 애플리케이션의 반응성을 향상시키는 데 중요합니다. 출력 결과를 점진적으로 표시하여, 응답이 완전히 준비되기 전에 사용자 경험(UX)을 크게 개선할 수 있습니다.
- [Functional API (베타)](functional_api.md): LangGraph에서 개발을 위한 [Graph API (StateGraph)](low_level.md#stategraph)의 대안입니다.
- [FAQ](faq.md): LangGraph에 대한 자주 묻는 질문들.

## LangGraph 플랫폼

LangGraph 플랫폼은 오픈 소스 LangGraph 프레임워크를 기반으로 생산 환경에서 에이전트 애플리케이션을 배포하기 위한 상용 솔루션입니다.

LangGraph 플랫폼은 [배포 옵션 가이드](./deployment_options.md)에 설명된 다양한 배포 옵션을 제공합니다.

!!! tip

    * LangGraph는 MIT 라이선스의 오픈 소스 라이브러리로, 커뮤니티를 위해 지속적으로 유지 관리되고 확장되고 있습니다.
    * LangGraph 플랫폼을 사용하지 않고도 오픈 소스 LangGraph 프로젝트를 사용하여 자체 인프라에서 LangGraph 애플리케이션을 배포할 수 있습니다.

### 하이레벨

- [왜 LangGraph 플랫폼인가?](./langgraph_platform.md): LangGraph 플랫폼은 LangGraph 애플리케이션을 배포하고 관리하는 일관된 방법을 제공합니다. 이 가이드는 LangGraph 플랫폼의 주요 기능과 개념을 제공합니다.
- [배포 옵션](./deployment_options.md): LangGraph 플랫폼은 네 가지 배포 옵션을 제공합니다: [Self-Hosted Lite](./self_hosted.md#self-hosted-lite), [Self-Hosted Enterprise](./self_hosted.md#self-hosted-enterprise), [자체 클라우드 사용 (BYOC)](./bring_your_own_cloud.md), 그리고 [Cloud SaaS](./langgraph_cloud.md). 이 가이드는 각 옵션의 차이점과 각 옵션이 제공되는 요금제를 설명합니다.
- [요금제](./plans.md): LangGraph 플랫폼은 세 가지 요금제를 제공합니다: Developer, Plus, Enterprise. 이 가이드는 각 요금제의 차이점, 각 요금제에 제공되는 배포 옵션, 그리고 각 요금제에 가입하는 방법을 설명합니다.
- [템플릿 애플리케이션](./template_applications.md): LangGraph로 빠르게 시작하는 데 도움이 되는 참조 애플리케이션입니다.

### 구성 요소

LangGraph 플랫폼은 LangGraph 애플리케이션의 배포와 관리 지원을 위해 함께 작동하는 여러 구성 요소로 이루어져 있습니다:

- [LangGraph Server](./langgraph_server.md): LangGraph Server는 백그라운드 처리부터 실시간 상호작용까지 다양한 에이전트 애플리케이션 사용 사례를 지원하도록 설계되었습니다.
- [LangGraph Studio](./langgraph_studio.md): LangGraph Studio는 LangGraph Server와 연결하여 애플리케이션을 시각화하고 상호작용하며 로컬에서 디버깅할 수 있는 특화된 IDE입니다.
- [LangGraph CLI](./langgraph_cli.md): LangGraph CLI는 로컬 LangGraph와 상호작용할 수 있도록 도와주는 명령어 기반 인터페이스입니다.
- [Python/JS SDK](./sdk.md): Python/JS SDK는 배포된 LangGraph 애플리케이션과 상호작용하는 프로그래밍 방식의 방법을 제공합니다.
- [Remote Graph](../how-tos/use-remote-graph.md): RemoteGraph는 배포된 LangGraph 애플리케이션을 로컬에서 실행되는 것처럼 상호작용할 수 있게 해줍니다.

### LangGraph Server

- [애플리케이션 구조](./application_structure.md): LangGraph 애플리케이션은 하나 이상의 그래프, LangGraph API 구성 파일(`langgraph.json`), 종속성을 명시하는 파일, 환경 변수로 구성됩니다.
- [어시스턴트](./assistants.md): 어시스턴트는 LangGraph 애플리케이션의 다양한 구성을 저장하고 관리하는 방법입니다.
- [웹훅](./langgraph_server.md#webhooks): 웹훅은 실행 중인 LangGraph 애플리케이션이 특정 이벤트에 따라 외부 서비스에 데이터를 전송할 수 있게 해줍니다.
- [크론 작업](./langgraph_server.md#cron-jobs): 크론 작업은 LangGraph 애플리케이션에서 특정 시간에 작업을 예약할 수 있는 방법입니다.
- [더블 텍스팅](./double_texting.md): 더블 텍스팅은 LLM 애플리케이션에서 사용자가 그래프 실행이 완료되기 전에 여러 메시지를 보낼 수 있는 문제입니다. 이 가이드는 LangGraph Deploy에서 더블 텍스팅을 처리하는 방법을 설명합니다.
- [인증 및 접근 제어](./auth.md): LangGraph 플랫폼을 배포할 때 사용할 수 있는 인증 및 접근 제어 옵션에 대해 알아봅니다.

### 배포 옵션

- [Self-Hosted Lite](./self_hosted.md): 로컬에서 또는 자체 호스팅 방식으로 실행할 수 있는 LangGraph 플랫폼의 제한된 무료 버전(연간 최대 100만 노드 실행).
- [Cloud SaaS](./langgraph_cloud.md): LangSmith의 일부로 호스팅됩니다.
- [Bring Your Own Cloud](./bring_your_own_cloud.md): 인프라를 관리하지만, 모든 인프라는 사용자의 클라우드 내에서 실행됩니다.
- [Self-Hosted Enterprise](./self_hosted.md): 사용자가 완전히 관리하는 방식입니다.