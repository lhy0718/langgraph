---
title: 사용 방법 가이드
description: LangGraph에서 일반 작업을 수행하는 방법
---

_한국어로 기계번역됨_

# 사용 방법 가이드

여기서는 "나는 어떻게...?" 유형의 질문에 대한 답을 찾을 수 있습니다. 이 가이드는 **목표 지향적**이고 구체적이며, 특정 작업을 완료하는 데 도움이 되도록 설계되었습니다. 개념적 설명은 [개념 가이드](../concepts/index.md)를 참조하시기 바랍니다. 전체적인 진행 과정을 보시려면 [튜토리얼](../tutorials/index.md)을 확인해 주세요. 모든 클래스와 함수에 대한 포괄적인 설명은 [API 참조](../reference/index.md)를 참조하십시오.

## LangGraph

### 그래프 API 기본

- [노드에서 그래프 상태를 업데이트하는 방법](state-reducers.ipynb)
- [단계의 시퀀스를 생성하는 방법](sequence.ipynb)
- [병렬 실행을 위한 분기를 만드는 방법](branching.ipynb)
- [재귀 한계를 설정하여 루프를 생성하고 제어하는 방법](recursion-limit.ipynb)
- [그래프를 시각화하는 방법](visualization.ipynb)

### 세부 제어

이 가이드는 그래프 실행에 대한 세부 제어를 제공하는 LangGraph 기능을 보여줍니다.

- [병렬 실행을 위한 맵-리듀스 분기를 만드는 방법](map-reduce.ipynb)
- [그래프와 서브그래프에서 상태를 업데이트하고 노드로 점프하는 방법](command.ipynb)
- [그래프에 런타임 구성을 추가하는 방법](configuration.ipynb)
- [노드 재시도를 추가하는 방법](node-retries.ipynb)
- [재귀 한계에 도달하기 전에 상태를 반환하는 방법](return-when-recursion-limit-hits.ipynb)

### 지속성

[LangGraph 지속성](../concepts/persistence.md)은 그래프 실행 간 상태를 지속하는 것을 쉽게 만들어 줍니다 (각 스레드 지속성) 그리고 스레드 간 (크로스 스레드 지속성). 이러한 사용 방법 가이드는 그래프에 지속성을 추가하는 방법을 보여줍니다.

- [그래프에 스레드 수준의 지속성을 추가하는 방법](persistence.ipynb)
- [서브그래프에 스레드 수준의 지속성을 추가하는 방법](subgraph-persistence.ipynb)
- [그래프에 크로스 스레드 지속성을 추가하는 방법](cross-thread-persistence.ipynb)
- [지속성을 위한 Postgres 체크포인터를 사용하는 방법](persistence_postgres.ipynb)
- [지속성을 위한 MongoDB 체크포인터를 사용하는 방법](persistence_mongodb.ipynb)
- [Redis를 사용하여 사용자 정의 체크포인터를 만드는 방법](persistence_redis.ipynb)

[기능적 API](../concepts/functional_api.md)를 사용하여 워크플로에 지속성을 추가하는 방법에 대한 아래 가이드를 참조하세요:

- [스레드 수준의 지속성을 추가하는 방법 (기능적 API)](persistence-functional.ipynb)
- [크로스 스레드 지속성을 추가하는 방법 (기능적 API)](cross-thread-persistence-functional.ipynb)

### 메모리

LangGraph는 그래프에서 대화 [메모리](../concepts/memory.md)를 쉽게 관리할 수 있도록 도와줍니다. 이러한 사용 방법 가이드는 다양한 전략을 구현하는 방법을 보여줍니다.

- [대화 기록 관리하는 방법](memory/manage-conversation-history.ipynb)
- [메시지 삭제하는 방법](memory/delete-messages.ipynb)
- [요약 대화 메모리 추가하는 방법](memory/add-summary-conversation-history.ipynb)
- [장기 메모리 추가하기 (크로스 스레드)](cross-thread-persistence.ipynb)
- [장기 메모리를 위한 의미론적 검색 사용 방법](memory/semantic-search.ipynb)

### 사람 참여

[사람 참여](../concepts/human_in_the_loop.md) 기능은 그래프의 의사 결정 과정에 사람을 포함할 수 있도록 해 줍니다. 이러한 사용 방법 가이드는 그래프에서 사람 참여 워크플로를 구현하는 방법을 보여줍니다.

주요 워크플로:

- [사용자 입력 대기하는 방법](human_in_the_loop/wait-user-input.ipynb): `interrupt` 함수를 사용하여 그래프에서 사람 참여 워크플로를 구현하는 기본 예시입니다.
- [도구 호출 검토하는 방법](human_in_the_loop/review-tool-calls.ipynb): `interrupt` 함수를 사용하여 실행 전에 도구 호출 요청을 검토/편집/수락하는 사람 참여 기능을 통합합니다.

기타 방법:

- [정적 중단점 추가하는 방법](human_in_the_loop/breakpoints.ipynb): 디버깅 목적으로 사용합니다. [**사람 참여**](../concepts/human_in_the_loop.md) 워크플로에는 대신 [`interrupt` 함수][langgraph.types.interrupt]를 권장합니다.
- [그래프 상태 편집하는 방법](human_in_the_loop/edit-graph-state.ipynb): `graph.update_state` 메소드를 사용하여 그래프 상태를 편집합니다. 정적 중단점을 통한 **사람 참여** 워크플로를 구현하는 경우 이 방법을 사용하십시오.
- [`NodeInterrupt`로 동적 중단점 추가하는 방법](human_in_the_loop/dynamic_breakpoints.ipynb): **권장하지 않음**: 대신 [`interrupt` 함수](../concepts/human_in_the_loop.md)를 사용하십시오.

[기능적 API](../concepts/functional_api.md)를 사용하여 사람 참여 워크플로를 구현하는 방법에 대한 아래 가이드를 참조하십시오:

- [사용자 입력 대기하는 방법 (기능적 API)](wait-user-input-functional.ipynb)
- [도구 호출 검토하는 방법 (기능적 API)](review-tool-calls-functional.ipynb)

### 타임 트래블

[타임 트래블](../concepts/time-travel.md)은 LangGraph 애플리케이션에서 지난 행위를 재생하여 대체 경로를 탐색하고 문제를 디버그할 수 있게 해줍니다. 이 사용 방법 가이드는 그래프에서 타임 트래블을 사용하는 방법을 보여줍니다.

- [이전 그래프 상태를 보고 업데이트하는 방법](human_in_the_loop/time-travel.ipynb)

### 스트리밍

[스트리밍](../concepts/streaming.md)은 LLM(대형 언어 모델) 위에 구축된 애플리케이션의 응답성을 향상시키는 데 중요합니다. 출력이 완전한 응답이 준비되기 전에 점진적으로 표시됨으로써 스트리밍은 사용자 경험(UX)을 크게 향상시킵니다, 특히 LLM의 지연 문제를 처리할 때 더욱 그렇습니다.

- [스트리밍하는 방법](streaming.ipynb)
- [LLM 토큰을 스트리밍하는 방법](streaming-tokens.ipynb)
- [특정 노드에서 LLM 토큰을 스트리밍하는 방법](streaming-specific-nodes.ipynb)
- [도구 내부에서 데이터를 스트리밍하는 방법](streaming-events-from-within-tools.ipynb)
- [서브그래프에서 스트리밍하는 방법](streaming-subgraphs.ipynb)
- [지원하지 않는 모델에 대한 스트리밍 비활성화하는 방법](disable-streaming.ipynb)

### 도구 호출

[도구 호출](https://python.langchain.com/docs/concepts/tool_calling/)은 도구 스키마와 메시지를 입력으로 받아들이고, 이러한 도구의 호출을 반환하는 [채팅 모델](https://python.langchain.com/docs/concepts/chat_models/) API의 일종입니다.
출력 메시지의 일부입니다.

이러한 사용 설명서는 LangGraph와 함께 도구 호출을 위한 일반적인 패턴을 보여줍니다:

- [ToolNode를 사용하여 도구 호출하는 방법](tool-calling.ipynb)
- [도구 호출 오류 처리하는 방법](tool-calling-errors.ipynb)
- [도구에 런타임 값 전달하는 방법](pass-run-time-values-to-tools.ipynb)
- [도구에 구성 전달하는 방법](pass-config-to-tools.ipynb)
- [도구로부터 그래프 상태를 업데이트하는 방법](update-state-from-tools.ipynb)
- [많은 수의 도구를 처리하는 방법](many-tools.ipynb)

### 서브그래프

[서브그래프](../concepts/low_level.md#subgraphs)는 다른 그래프에서 기존의 그래프를 재사용할 수 있게 해줍니다. 이러한 사용 설명서는 서브그래프를 사용하는 방법을 보여줍니다:

- [서브그래프 사용 방법](subgraph.ipynb)
- [서브그래프에서 상태를 보고 업데이트하는 방법](subgraphs-manage-state.ipynb)
- [서브그래프의 입력 및 출력 변환하는 방법](subgraph-transform-state.ipynb)

### 다중 에이전트

[다중 에이전트 시스템](../concepts/multi_agent.md)은 복잡한 LLM 애플리케이션을 여러 에이전트로 나누어 각기 다른 부분을 담당하도록 하는 데 유용합니다. 이러한 사용 설명서는 LangGraph에서 다중 에이전트 시스템을 구현하는 방법을 보여줍니다:

- [에이전트 간의 중계 구현 방법](agent-handoffs.ipynb)
- [다중 에이전트 네트워크 구축 방법](multi-agent-network.ipynb)
- [다중 에이전트 애플리케이션에서 다중 턴 대화 추가하는 방법](multi-agent-multi-turn-convo.ipynb)

기타 다중 에이전트 아키텍처의 구현은 [다중 에이전트 튜토리얼](../tutorials/index.md#multi-agent-systems)을 참조하세요.

[Functional API](../concepts/functional_api.md)를 사용하여 다중 에이전트 워크플로우를 구현하는 방법에 대한 가이드는 다음과 같습니다:

- [다중 에이전트 네트워크 구축 (Functional API)](multi-agent-network-functional.ipynb)
- [다중 에이전트 애플리케이션에서 다중 턴 대화 추가 (Functional API)](multi-agent-multi-turn-convo-functional.ipynb)

### 상태 관리

- [Pydantic 모델을 그래프 상태로 사용하는 방법](state-model.ipynb)
- [그래프의 입력/출력 스키마 정의하는 방법](input_output_schema.ipynb)
- [그래프 내부 노드 간에 개인 상태 전달하는 방법](pass_private_state.ipynb)

### 기타

- [그래프를 비동기로 실행하는 방법](async.ipynb)
- [도구 호출 에이전트가 출력을 구조화하도록 강제하는 방법](react-agent-structured-output.ipynb)
- [그래프 실행을 위한 사용자 정의 LangSmith 실행 ID 전달하는 방법](run-id-langsmith.ipynb)
- [LangGraph를 AutoGen, CrewAI 및 기타 프레임워크와 통합하는 방법](autogen-integration.ipynb)

다음 가이드는 [Functional API](../concepts/functional_api.md)를 사용하여 다른 프레임워크와 통합하는 방법을 제공합니다:

- [LangGraph (Functional API)를 AutoGen, CrewAI 및 기타 프레임워크와 통합하는 방법](autogen-integration-functional.ipynb)

### 미리 구축된 ReAct 에이전트

LangGraph의 [미리 구축된 ReAct 에이전트](../reference/prebuilt.md#langgraph.prebuilt.chat_agent_executor.create_react_agent)는 [도구 호출 에이전트](../concepts/agentic_concepts.md#tool-calling-agent)의 미리 구축된 구현입니다.

LangGraph의 큰 장점 중 하나는 자신만의 에이전트 아키텍처를 쉽게 만들 수 있다는 점입니다. 따라서 여기서 시작하여 에이전트를 빠르게 구축하는 것도 좋지만, LangGraph의 모든 이점을 누리기 위해서는 자신만의 에이전트를 구축하는 방법을 배우는 것을 강력히 권장합니다.

이 가이드는 미리 구축된 ReAct 에이전트를 사용하는 방법을 보여줍니다:

- [미리 구축된 ReAct 에이전트 사용 방법](create-react-agent.ipynb)
- [ReAct 에이전트에 스레드 수준의 메모리 추가하는 방법](create-react-agent-memory.ipynb)
- [ReAct 에이전트에 사용자 정의 시스템 프롬프트 추가하는 방법](create-react-agent-system-prompt.ipynb)
- [ReAct 에이전트에 인간이 개입하는 프로세스 추가하는 방법](create-react-agent-hitl.ipynb)
- [ReAct 에이전트에서 구조화된 출력을 반환하는 방법](create-react-agent-structured-output.ipynb)
- [ReAct 에이전트에 장기 메모리를 위한 의미 검색 추가하는 방법](memory/semantic-search.ipynb#using-in-create-react-agent)

ReAct 에이전트를 더욱 맞춤화하는 데 관심이 있나요? 이 가이드는 자신만의 필요에 맞게 커스터마이즈하는 데 도움이 되는 기본 구현 개요를 제공합니다:

- [무에서 시작하여 미리 구축된 ReAct 에이전트를 만드는 방법](react-agent-from-scratch.ipynb)

다음 가이드는 [Functional API](../concepts/functional_api.md)를 사용하여 ReAct 에이전트를 구축하는 방법을 제공합니다:

- [무에서 시작하여 ReAct 에이전트 만들기 (Functional API)](react-agent-from-scratch-functional.ipynb)

## LangGraph 플랫폼

이 섹션은 LangGraph 플랫폼에 대한 사용 설명서를 포함합니다.

LangGraph 플랫폼은 오픈 소스 LangGraph 프레임워크를 기반으로 하는 상용 솔루션으로, 에이전트 애플리케이션을 생산 환경에 배포하는 데 사용됩니다.

LangGraph 플랫폼은 [배포 옵션 가이드](../concepts/deployment_options.md)에 설명된 몇 가지 다양한 배포 옵션을 제공합니다.

!!! 팁

    * LangGraph는 MIT 라이센스가 부여된 오픈 소스 라이브러리로, 커뮤니티를 위해 유지 및 성장시키는 데 전념하고 있습니다.
    * LangGraph 플랫폼을 사용하지 않고도 오픈 소스 LangGraph 프로젝트를 사용하여 자신의 인프라에 LangGraph 애플리케이션을 배포할 수 있습니다.

### 애플리케이션 구조

LangGraph 플랫폼에 배포할 애플리케이션을 설정하는 방법을 배우세요:

- [배포를 위한 애플리케이션 설정 방법 (requirements.txt)](../cloud/deployment/setup.md)
- [배포를 위한 애플리케이션 설정 방법 (pyproject.toml)](../cloud/deployment/setup_pyproject.md)
- [배포를 위한 애플리케이션 설정 방법 (JavaScript)](../cloud/deployment/setup_javascript.md)
- [의미 검색 추가하는 방법](../cloud/deployment/semantic_search.md)
- [Dockerfile 사용자 정의하는 방법](../cloud/deployment/custom_docker.md)
- [로컬에서 테스트하는 방법](../cloud/deployment/test_locally.md)
- [런타임에 그래프를 재구성하는 방법](../cloud/deployment/graph_rebuild.md)
- [CrewAI, AutoGen 및 기타 프레임워크를 배포하기 위해 LangGraph 플랫폼 사용 방법](autogen-langgraph-platform.ipynb)
- [LangGraph를 React 애플리케이션에 통합하는 방법](../cloud/how-tos/use_stream_react.md)

### 배포

LangGraph 애플리케이션은 LangGraph Cloud를 사용하여 배포할 수 있으며, 이는 애플리케이션을 배포, 관리 및 확장하는 데 도움을 주는 다양한 서비스를 제공합니다.

- [LangGraph 클라우드에 배포하는 방법](../cloud/deployment/cloud.md)
- [자체 호스팅 환경에 배포하는 방법](./deploy-self-hosted.md)
- [RemoteGraph를 사용하여 배포와 상호작용하는 방법](./use-remote-graph.md)

### 인증 및 접근 제어

- [사용자 지정 인증 추가하는 방법](./auth/custom_auth.md)
- [OpenAPI 사양의 보안 스키마 업데이트하는 방법](./auth/openapi_security.md)

### 어시스턴트

[어시스턴트](../concepts/assistants.md)는 템플릿의 구성된 인스턴스입니다.

지원되는 엔드포인트 및 기타 세부정보는 [SDK 참조](../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.client.AssistantsClient)에서 확인하십시오.

- [에이전트 구성하는 방법](../cloud/how-tos/configuration_cloud.md)
- [어시스턴트 버전 관리하는 방법](../cloud/how-tos/assistant_versioning.md)

### 스레드

지원되는 엔드포인트 및 기타 세부정보는 [SDK 참조](../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.client.ThreadsClient)에서 확인하십시오.

- [스레드 복사하는 방법](../cloud/how-tos/copy_threads.md)
- [스레드 상태 확인하는 방법](../cloud/how-tos/check_thread_status.md)

### 실행

LangGraph 플랫폼은 스트리밍 실행 외에도 여러 가지 유형의 실행을 지원합니다.

- [백그라운드에서 에이전트 실행하는 방법](../cloud/how-tos/background_run.md)
- [같은 스레드에서 여러 에이전트 실행하는 방법](../cloud/how-tos/same-thread.md)
- [크론 작업 만드는 방법](../cloud/how-tos/cron_jobs.md)
- [무상태 실행 만드는 방법](../cloud/how-tos/stateless_runs.md)

### 스트리밍

LLM 애플리케이션의 결과를 스트리밍하는 것은 사용자 경험을 보장하는 데 중요합니다. 특히 그래프가 여러 모델을 호출하고 실행이 완료되기까지 오랜 시간이 걸릴 수 있는 경우 더욱 그렇습니다. 다음 가이드에서 그래프에서 값을 스트리밍하는 방법에 대해 읽어보십시오.

- [값 스트리밍하는 방법](../cloud/how-tos/stream_values.md)
- [업데이트 스트리밍하는 방법](../cloud/how-tos/stream_updates.md)
- [메시지 스트리밍하는 방법](../cloud/how-tos/stream_messages.md)
- [이벤트 스트리밍하는 방법](../cloud/how-tos/stream_events.md)
- [디버그 모드에서 스트리밍하는 방법](../cloud/how-tos/stream_debug.md)
- [여러 모드로 스트리밍하는 방법](../cloud/how-tos/stream_multiple.md)

### 인간-인-루프

복잡한 그래프를 설계할 때 LLM에 의존하여 의사 결정을 내리는 것은 위험할 수 있습니다. 특히 파일, API 또는 데이터베이스와 상호작용하는 도구가 포함된 경우 더욱 그렇습니다. 이러한 상호작용은 사용 사례에 따라 의도치 않은 데이터 접근 또는 수정을 초래할 수 있습니다. 이러한 위험을 완화하기 위해 LangGraph는 인간-인-루프 행동을 통합할 수 있으며, 이는 LLM 애플리케이션이 의도한 대로 작동하고 바람직하지 않은 결과를 방지하도록 합니다.

- [중단점 추가하는 방법](../cloud/how-tos/human_in_the_loop_breakpoint.md)
- [사용자 입력 대기하는 방법](../cloud/how-tos/human_in_the_loop_user_input.md)
- [그래프 상태 수정하는 방법](../cloud/how-tos/human_in_the_loop_edit_state.md)
- [이전 상태로부터 재생 및 가지치기하는 방법](../cloud/how-tos/human_in_the_loop_time_travel.md)
- [도구 호출 검토하는 방법](../cloud/how-tos/human_in_the_loop_review_tool_calls.md)

### 더블 텍스트

그래프 실행은 시간이 걸릴 수 있으며, 사용자가 원래 입력이 완료되기 전에 보내고자 했던 입력에 대해 마음을 바꿀 수 있습니다. 예를 들어, 사용자가 원래 요청에서 오타를 발견하고 프롬프트를 수정하여 재전송할 수 있습니다. 이러한 경우에 어떻게 대응하느냐는 원활한 사용자 경험을 보장하고 그래프가 예상치 못한 방식으로 작동하는 것을 방지하는 데 중요합니다.

- [인터럽트 옵션 사용하는 방법](../cloud/how-tos/interrupt_concurrent.md)
- [롤백 옵션 사용하는 방법](../cloud/how-tos/rollback_concurrent.md)
- [거부 옵션 사용하는 방법](../cloud/how-tos/reject_concurrent.md)
- [대기열 옵션 사용하는 방법](../cloud/how-tos/enqueue_concurrent.md)

### 웹훅

- [웹훅 통합하는 방법](../cloud/how-tos/webhooks.md)

### 크론 작업

- [크론 작업 만드는 방법](../cloud/how-tos/cron_jobs.md)

### LangGraph Studio

LangGraph Studio는 에이전트를 시각화하고 테스트하며 디버깅하기 위한 내장 UI입니다.

- [LangGraph Cloud 배포에 연결하는 방법](../cloud/how-tos/test_deployment.md)
- [로컬 개발 서버에 연결하는 방법](../how-tos/local-studio.md)
- [로컬 배포(Docker)에 연결하는 방법](../cloud/how-tos/test_local_deployment.md)
- [LangGraph Studio에서 그래프 테스트하는 방법 (MacOS 전용)](../cloud/how-tos/invoke_studio.md)
- [LangGraph Studio에서 스레드와 상호작용하는 방법](../cloud/how-tos/threads_studio.md)
- [LangGraph Studio에서 데이터 예제로 노드 추가하는 방법](../cloud/how-tos/datasets_studio.md)
- [LangGraph Studio에서 프롬프트를 엔지니어링하는 방법](../cloud/how-tos/iterate_graph_studio.md)

## 문제 해결

다음은 LangGraph를 사용하여 구축하는 동안 발견할 수 있는 일반 오류를 해결하기 위한 가이드입니다. 아래에서 언급된 오류는 코드에서 발생할 때 아래 코드 중 하나에 해당하는 `lc_error_code` 속성을 갖습니다.

- [GRAPH_RECURSION_LIMIT](../troubleshooting/errors/GRAPH_RECURSION_LIMIT.md)
- [INVALID_CONCURRENT_GRAPH_UPDATE](../troubleshooting/errors/INVALID_CONCURRENT_GRAPH_UPDATE.md)
- [잘못된 그래프 노드 반환 값](../troubleshooting/errors/INVALID_GRAPH_NODE_RETURN_VALUE.md)
- [다중 서브그래프](../troubleshooting/errors/MULTIPLE_SUBGRAPHS.md)
- [잘못된 채팅 기록](../troubleshooting/errors/INVALID_CHAT_HISTORY.md)

### LangGraph 플랫폼 문제 해결

이 가이드는 LangGraph 플랫폼에 특화된 오류에 대한 문제 해결 정보를 제공합니다.

- [잘못된 라이선스](../troubleshooting/errors/INVALID_LICENSE.md)
