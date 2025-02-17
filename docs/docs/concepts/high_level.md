_한국어로 기계번역됨_

# 왜 LangGraph인가?

## LLM 애플리케이션

LLM(대형 언어 모델)은 새로운 유형의 애플리케이션에 지능을 통합할 수 있게 합니다. LLM을 사용하는 애플리케이션을 구축하는 데에는 여러 패턴이 있습니다. [워크플로우](https://www.anthropic.com/research/building-effective-agents)는 LLM 호출을 둘러싼 미리 정의된 코드 경로의 골조를 제공합니다. LLM은 이러한 미리 정의된 코드 경로를 통해 제어 흐름을 유도할 수 있으며, 이를 일부는 "[에이전틱 시스템](https://www.anthropic.com/research/building-effective-agents)"이라고 합니다. 다른 경우에는 이러한 골조를 제거하고, [계획](https://huyenchip.com/2025/01/07/agents.html)을 세우고, [도구 호출](https://python.langchain.com/docs/concepts/tool_calling/)을 통해 작업을 수행하며, 자신의 행동에 대한 [피드백](https://research.google/blog/react-synergizing-reasoning-and-acting-in-language-models/)을 바탕으로 추가 작업을 수행하는 자율적인 에이전트를 만들 수 있습니다.

![Agent Workflow](img/agent_workflow.png)

## LangGraph가 제공하는 것

LangGraph는 _어떤_ 워크플로우나 에이전트 아래에서 동작하는 저수준 지원 인프라를 제공합니다. 이는 프롬프트나 아키텍처를 추상화하지 않으며, 세 가지 주요 이점을 제공합니다:

### 지속성

LangGraph는 [지속성 계층](https://langchain-ai.github.io/langgraph/concepts/persistence/)을 제공하여 여러 가지 이점을 제공합니다:

- [메모리](https://langchain-ai.github.io/langgraph/concepts/memory/): LangGraph는 애플리케이션 상태의 임의 부분을 지속적으로 저장할 수 있어, 대화 및 사용자 상호작용 내외의 업데이트를 지원합니다.
- [휴먼 인 더 루프](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/): 상태가 체크포인트 되어 실행이 중단되고 다시 시작될 수 있어, 인간의 입력을 통해 결정, 검증 및 수정이 가능합니다.

### 스트리밍

LangGraph는 애플리케이션 실행 중에 워크플로우/에이전트 상태를 사용자(또는 개발자)에게 스트리밍하는 기능도 제공합니다. LangGraph는 [도구 호출로부터의 피드백](../how-tos/streaming.ipynb#updates)과 [LLM 호출로부터의 토큰](../how-tos/streaming-tokens.ipynb)을 스트리밍하는 것을 모두 지원합니다.

### 디버깅 및 배포

LangGraph는 애플리케이션 테스트, 디버깅 및 배포를 위한 간편한 방법을 제공합니다. [LangGraph 플랫폼](https://langchain-ai.github.io/langgraph/concepts/langgraph_platform/)을 통해 이를 수행할 수 있습니다. 이에는 워크플로우나 에이전트의 시각화, 상호작용 및 디버깅을 가능하게 하는 IDE인 [Studio](https://langchain-ai.github.io/langgraph/concepts/langgraph_studio/)가 포함되어 있습니다. 또한 여러 가지 [배포 옵션](https://langchain-ai.github.io/langgraph/tutorials/deployment/)이 제공됩니다.
