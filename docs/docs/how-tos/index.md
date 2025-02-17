_한국어로 기계번역됨_

---
title: 사용 방법 안내서
description: LangGraph에서 일반 작업을 수행하는 방법
---

# 사용 방법 안내서

여기에서 "어떻게...?" 유형의 질문에 대한 답변을 찾을 수 있습니다. 이 가이드는 **목표 지향적**이며 구체적입니다. 특정 작업을 완료하는 데 도움을 주기 위해 고안되었습니다. 개념적 설명은 [개념 가이드](../concepts/index.md)를 참조하십시오. 엔드 투 엔드 절차에 대한 자세한 내용은 [튜토리얼](../tutorials/index.md)을 참조하세요. 모든 클래스와 기능에 대한 포괄적인 설명은 [API 참조](../reference/index.md)를 참조하십시오.

## LangGraph

### 그래프 API 기초

- [노드에서 그래프 상태 업데이트하는 방법](state-reducers.md)
- [단계의 순서를 만드는 방법](sequence.md)
- [병렬 실행을 위한 분기 생성 방법](branching.md)
- [재귀 제한을 가진 루프를 생성하고 제어하는 방법](recursion-limit.ipynb)
- [그래프를 시각화하는 방법](visualization.ipynb)

### 세밀한 제어

이 가이드는 그래프 실행에 대한 세밀한 제어를 제공하는 LangGraph 기능을 보여줍니다.

- [병렬 실행을 위한 맵-리듀스 분기 생성 방법](map-reduce.ipynb)
- [그래프와 서브그래프의 상태를 업데이트하고 노드로 점프하는 방법](command.ipynb)
- [그래프에 런타임 구성 추가하는 방법](configuration.ipynb)
- [노드 재시도 추가하는 방법](node-retries.ipynb)
- [재귀 제한에 도달하기 전에 상태를 반환하는 방법](return-when-recursion-limit-hits.ipynb)

### 지속성

[LangGraph 지속성](../concepts/persistence.md)은 그래프 실행 간 상태를 지속(스레드별 지속성)하고 스레드 간에 상태를 지속(크로스 스레드 지속성)하기 쉽게 만들어 줍니다. 이 사용 방법 안내서는 그래프에 지속성을 추가하는 방법을 보여줍니다.

- [그래프에 스레드 수준의 지속성 추가 방법](persistence.ipynb)
- [서브그래프에 스레드 수준의 지속성 추가 방법](subgraph-persistence.ipynb)
- [그래프에 크로스 스레드 지속성 추가 방법](cross-thread-persistence.ipynb)
- [지속성을 위한 Postgres 체크포인터 사용 방법](persistence_postgres.ipynb)
- [지속성을 위한 MongoDB 체크포인터 사용 방법](persistence_mongodb.ipynb)
- [Redis를 사용하여 사용자 지정 체크포인터 생성 방법](persistence_redis.ipynb)

(베타) [함수 API](../concepts/functional_api.md)를 사용하여 작업 흐름에 지속성을 추가하는 방법에 대한 아래 가이드를 참조하십시오:

- [스레드 수준의 지속성 추가 방법 (함수 API)](persistence-functional.ipynb)
- [크로스 스레드 지속성 추가 방법 (함수 API)](cross-thread-persistence-functional.ipynb)

### 메모리

LangGraph는 그래프에서 대화 [메모리](../concepts/memory.md)를 관리하는 것을 쉽게 만들어 줍니다. 이 사용 방법 안내서는 다양한 전략을 구현하는 방법을 보여줍니다.

- [대화 기록 관리 방법](memory/manage-conversation-history.ipynb)
- [메시지 삭제 방법](memory/delete-messages.ipynb)
- [요약 대화 메모리 추가 방법](memory/add-summary-conversation-history.ipynb)
- [장기 메모리 추가 방법 (크로스 스레드)](cross-thread-persistence.ipynb)
- [장기 메모리를 위한 의미적 검색 사용 방법](memory/semantic-search.ipynb)

### 사람 개입

[사람 개입](../concepts/human_in_the_loop.md) 기능을 통해 그래프의 의사 결정 과정에 사람을 포함할 수 있습니다. 이 사용 방법 안내서는 그래프에서 사람 개입 작업 흐름을 구현하는 방법을 보여줍니다.

주요 작업 흐름:

- [사용자 입력 대기하는 방법](human_in_the_loop/wait-user-input.ipynb): `interrupt` 함수를 사용하여 그래프에서 사람 개입 작업 흐름을 구현하는 방법을 보여주는 기본 예시입니다.
- [도구 호출 검토하는 방법](human_in_the_loop/review-tool-calls.ipynb): `interrupt` 함수를 사용하여 실행되기 전에 도구 호출 요청을 검토/편집/수락하기 위해 사람 개입을 통합하는 방법입니다.

기타 방법:

- [정적 중단점 추가 방법](human_in_the_loop/breakpoints.ipynb): 디버깅 목적으로 사용합니다. [**사람 개입**](../concepts/human_in_the_loop.md) 작업 흐름에는 [`interrupt` 함수][langgraph.types.interrupt]를 권장합니다.
- [그래프 상태 편집하는 방법](human_in_the_loop/edit-graph-state.ipynb): `graph.update_state` 메서드를 사용하여 그래프 상태를 편집합니다. 정적 중단점을 통한 **사람 개입** 작업 흐름을 구현하는 경우에 사용하십시오.
- [`NodeInterrupt`로 동적 중단점 추가하는 방법](human_in_the_loop/dynamic_breakpoints.ipynb): **추천하지 않습니다**: 대신 [`interrupt` 함수](../concepts/human_in_the_loop.md)를 사용하십시오.

(베타) [함수 API](../concepts/functional_api.md)를 사용하여 사람 개입 작업 흐름을 구현하는 방법에 대한 아래 가이드를 참조하십시오:

- [사용자 입력 대기하는 방법 (함수 API)](wait-user-input-functional.ipynb)
- [도구 호출 검토하는 방법 (함수 API)](review-tool-calls-functional.ipynb)

### 타임 트래블

[타임 트래블](../concepts/time-travel.md)은 LangGraph 애플리케이션에서 과거 작업을 재생하여 대안 경로를 탐색하고 문제를 디버깅할 수 있게 해줍니다. 이 사용 방법 안내서는 그래프에서 타임 트래블을 사용하는 방법을 보여줍니다.

- [과거 그래프 상태 보기 및 업데이트하는 방법](human_in_the_loop/time-travel.ipynb)

### 스트리밍

[스트리밍](../concepts/streaming.md)은 LLM 기반 애플리케이션의 응답성을 향상시키는 데 중요합니다. 출력이 완전한 응답이 준비되기 전에 점진적으로 표시되므로 사용 경험(UX)이 크게 향상되며, 특히 LLM의 대기 시간 처리 시에 그렇습니다.

- [스트리밍하는 방법](streaming.ipynb)
- [LLM 토큰 스트리밍하는 방법](streaming-tokens.ipynb)
- [특정 노드에서 LLM 토큰 스트리밍하는 방법](streaming-specific-nodes.ipynb)
- [도구 내에서 데이터 스트리밍하는 방법](streaming-events-from-within-tools.ipynb)
- [서브그래프에서 스트리밍하는 방법](streaming-subgraphs.ipynb)
- [지원하지 않는 모델에 대해 스트리밍 비활성화하는 방법](disable-streaming.ipynb)

### 도구 호출

[도구 호출](https://python.langchain.com/docs/concepts/tool_calling/)은 메시지를 입력으로 받아 도구 스키마를 수용하고, 출력 메시지의 일부로서 해당 도구의 호출을 반환하는 [챗 모델](https://python.langchain.com/docs/concepts/chat_models/) API의 일종입니다.

다음은 LangGraph를 사용한 도구 호출의 일반적인 패턴을 보여주는 안내서입니다:

- [ToolNode를 사용하여 도구 호출하는 방법](tool-calling.ipynb)
- [도구 호출 오류 처리하는 방법](tool-calling-errors.ipynb)
- [런타임 값을 도구에 전달하는 방법](pass-run-time-values-to-tools.ipynb)
- [도구에 구성 전달하는 방법](pass-config-to-tools.ipynb)
- [도구에서 그래프 상태 업데이트하는 방법](update-state-from-tools.ipynb)
- [대량의 도구를 처리하는 방법](many-tools.ipynb)

### 서브그래프

[서브그래프](../concepts/low_level.md#subgraphs)는 다른 그래프의 기존 그래프를 재사용할 수 있게 해줍니다. 서브그래프 사용 방법에 대한 안내는 다음과 같습니다:

- [서브그래프를 사용하는 방법](subgraph.ipynb)
- [서브그래프에서 상태를 보고 업데이트하는 방법](subgraphs-manage-state.ipynb)
- [서브그래프의 입력과 출력을 변환하는 방법](subgraph-transform-state.ipynb)

### 다중 에이전트

[다중 에이전트 시스템](../concepts/multi_agent.md)은 복잡한 LLM 애플리케이션을 여러 에이전트로 분해하여 각자가 애플리케이션의 다른 부분을 책임지도록 유용합니다. LangGraph에서 다중 에이전트 시스템을 구현하는 방법은 다음과 같습니다:

- [에이전트 간 핸드오프 구현하는 방법](agent-handoffs.ipynb)
- [다중 에이전트 네트워크 구축하는 방법](multi-agent-network.ipynb)
- [다중 에이전트 애플리케이션에서 다중 턴 대화 추가하는 방법](multi-agent-multi-turn-convo.ipynb)

기타 다중 에이전트 아키텍처의 구현을 보려면 [다중 에이전트 튜토리얼](../tutorials/index.md#multi-agent-systems)을 참조하세요.

(베타) [기능적 API](../concepts/functional_api.md)를 사용하여 다중 에이전트 워크플로를 구현하는 방법에 대한 안내는 다음과 같습니다:

- [다중 에이전트 네트워크 구축하기 (기능적 API)](multi-agent-network-functional.ipynb)
- [다중 에이전트 애플리케이션에서 다중 턴 대화 추가하기 (기능적 API)](multi-agent-multi-turn-convo-functional.ipynb)

### 상태 관리

- [Pydantic 모델을 그래프 상태로 사용하는 방법](state-model.ipynb)
- [그래프의 입력/출력 스키마 정의하는 방법](input_output_schema.ipynb)
- [그래프 내의 노드 간 개인 상태 전달하는 방법](pass_private_state.ipynb)

### 기타

- [그래프를 비동기적으로 실행하는 방법](async.ipynb)
- [도구 호출 에이전트가 출력을 구조화하도록 강제하는 방법](react-agent-structured-output.ipynb)
- [그래프 실행을 위해 사용자 정의 LangSmith 실행 ID를 전달하는 방법](run-id-langsmith.ipynb)
- [LangGraph를 AutoGen, CrewAI 및 기타 프레임워크와 통합하는 방법](autogen-integration.ipynb)

(베타) [기능적 API](../concepts/functional_api.md)를 사용하여 다른 프레임워크와 통합하는 방법에 대한 안내는 다음과 같습니다:

- [LangGraph (기능적 API)를 AutoGen, CrewAI 및 기타 프레임워크와 통합하는 방법](autogen-integration-functional.ipynb)

### 사전 구축된 ReAct 에이전트

LangGraph [사전 구축된 ReAct 에이전트](../reference/prebuilt.md#langgraph.prebuilt.chat_agent_executor.create_react_agent)는 [도구 호출 에이전트](../concepts/agentic_concepts.md#tool-calling-agent)의 사전 구축된 구현입니다.

LangGraph의 큰 장점 중 하나는 자신만의 에이전트 아키텍처를 쉽게 만들 수 있다는 것입니다. 따라서 에이전트를 빠르게 구축하기 위해 여기에서 시작하는 것도 좋지만, LangGraph를 최대한 활용할 수 있도록 자신만의 에이전트 구축 방법을 배우는 것을 강력히 추천합니다.

사전 구축된 ReAct 에이전트를 사용하는 방법에 대한 안내는 다음과 같습니다:

- [사전 구축된 ReAct 에이전트 사용하는 방법](create-react-agent.md)
- [ReAct 에이전트에 스레드 수준 메모리 추가하는 방법](create-react-agent-memory.ipynb)
- [ReAct 에이전트에 사용자 정의 시스템 프롬프트 추가하는 방법](create-react-agent-system-prompt.ipynb)
- [ReAct 에이전트에 인간 관여 프로세스 추가하는 방법](create-react-agent-hitl.ipynb)
- [ReAct 에이전트에서 구조화된 출력 반환하는 방법](create-react-agent-structured-output.ipynb)
- [ReAct 에이전트에 장기 기억을 위한 의미론적 검색 추가하는 방법](memory/semantic-search.ipynb#using-in-create-react-agent)

ReAct 에이전트를 추가로 사용자 지정하고 싶으신가요? 이 가이드는 사용자 맞춤형 기능을 위해 ReAct 에이전트의 기본 구현에 대한 개요를 제공합니다:

- [사전 구축된 ReAct 에이전트를 처음부터 만드는 방법](react-agent-from-scratch.ipynb)

(베타) [기능적 API](../concepts/functional_api.md)를 사용하여 ReAct 에이전트를 구축하는 방법에 대한 안내는 다음과 같습니다:

- [기능적 API로부터 ReAct 에이전트를 처음부터 만드는 방법](react-agent-from-scratch-functional.ipynb)

## LangGraph 플랫폼

이 섹션에는 LangGraph 플랫폼에 대한 안내서가 포함되어 있습니다.

LangGraph 플랫폼은 오픈 소스 LangGraph 프레임워크를 기반으로 한 에이전틱 애플리케이션을 프로덕션 환경에 배포하기 위한 상업적 솔루션입니다.

LangGraph 플랫폼은 [배포 옵션 가이드](../concepts/deployment_options.md)에 설명된 몇 가지 다양한 배포 옵션을 제공합니다.

!!! 팁

    * LangGraph는 MIT 라이센스의 오픈 소스 라이브러리로, 커뮤니티를 위해 유지하고 성장시키는 데 전념하고 있습니다.
    * LangGraph 플랫폼을 사용하지 않고도 오픈 소스 LangGraph 프로젝트를 사용하여 자체 인프라에서 LangGraph 애플리케이션을 배포할 수 있습니다.

### 애플리케이션 구조

LangGraph 플랫폼에 배포하기 위해 애플리케이션을 설정하는 방법을 알아봅니다:

- [배포를 위한 애플리케이션 설정하는 방법 (requirements.txt)](../cloud/deployment/setup.md)
- [배포를 위한 앱 설정 방법 (pyproject.toml)](../cloud/deployment/setup_pyproject.md)
- [배포를 위한 앱 설정 방법 (JavaScript)](../cloud/deployment/setup_javascript.md)
- [의미 기반 검색 추가 방법](../cloud/deployment/semantic_search.md)
- [Dockerfile 사용자 정의 방법](../cloud/deployment/custom_docker.md)
- [로컬 테스트 방법](../cloud/deployment/test_locally.md)
- [런타임에서 그래프 재구성 방법](../cloud/deployment/graph_rebuild.md)
- [LangGraph 플랫폼을 사용하여 CrewAI, AutoGen 및 기타 프레임워크 배포하는 방법](autogen-langgraph-platform.ipynb)
- [LangGraph를 React 애플리케이션에 통합하는 방법](../cloud/how-tos/use_stream_react.md)

### 배포

LangGraph 애플리케이션은 LangGraph Cloud를 사용하여 배포할 수 있으며, 이를 통해 애플리케이션을 배포, 관리 및 확장하는 데 도움을 주는 다양한 서비스를 제공합니다.

- [LangGraph 클라우드에 배포하는 방법](../cloud/deployment/cloud.md)
- [자가 호스팅 환경에 배포하는 방법](./deploy-self-hosted.md)
- [RemoteGraph를 사용하여 배포와 상호작용하는 방법](./use-remote-graph.md)

### 인증 및 접근 제어

- [사용자 정의 인증 추가 방법](./auth/custom_auth.md)
- [OpenAPI 명세의 보안 스키마 업데이트 방법](./auth/openapi_security.md)

### 어시스턴트

[어시스턴트](../concepts/assistants.md)는 템플릿의 구성된 인스턴스입니다.

지원되는 엔드포인트 및 기타 세부정보는 [SDK 참조](../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.client.AssistantsClient)를 참조하세요.

- [에이전트 구성하는 방법](../cloud/how-tos/configuration_cloud.md)
- [어시스턴트 버전 관리 방법](../cloud/how-tos/assistant_versioning.md)

### 스레드

지원되는 엔드포인트 및 기타 세부정보는 [SDK 참조](../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.client.ThreadsClient)를 참조하세요.

- [스레드 복사 방법](../cloud/how-tos/copy_threads.md)
- [스레드 상태 확인 방법](../cloud/how-tos/check_thread_status.md)

### 실행

LangGraph 플랫폼은 스트리밍 실행 외에 여러 유형의 실행을 지원합니다.

- [에이전트를 백그라운드에서 실행하는 방법](../cloud/how-tos/background_run.md)
- [같은 스레드에서 여러 에이전트를 실행하는 방법](../cloud/how-tos/same-thread.md)
- [크론 작업 생성 방법](../cloud/how-tos/cron_jobs.md)
- [상태 비저장 실행 생성 방법](../cloud/how-tos/stateless_runs.md)

### 스트리밍

LLM 애플리케이션의 결과를 스트리밍하는 것은 사용자 경험을 보장하는 데 중요합니다. 특히 그래프가 여러 모델을 호출하고 실행이 완료되는 데 긴 시간이 걸릴 수 있는 경우 더욱 그렇습니다. 그래프에서 값을 스트리밍하는 방법에 대한 가이드는 다음과 같습니다:

- [값 스트리밍 방법](../cloud/how-tos/stream_values.md)
- [업데이트 스트리밍 방법](../cloud/how-tos/stream_updates.md)
- [메시지 스트리밍 방법](../cloud/how-tos/stream_messages.md)
- [이벤트 스트리밍 방법](../cloud/how-tos/stream_events.md)
- [디버그 모드에서 스트리밍하는 방법](../cloud/how-tos/stream_debug.md)
- [여러 모드 스트리밍 방법](../cloud/how-tos/stream_multiple.md)

### 사람을 포함한 루프

복잡한 그래프를 설계할 때 LLM에 전적으로 의존하는 것은 위험할 수 있으며, 특히 파일, API 또는 데이터베이스와 상호작용하는 도구를 포함할 경우 더욱 그렇습니다. 이러한 상호작용은 사용 사례에 따라 의도하지 않은 데이터 접근이나 수정을 초래할 수 있습니다. 이러한 위험을 완화하기 위해 LangGraph는 사람을 포함한 루프 행동을 통합하도록 허용하여 LLM 애플리케이션이 의도한 대로 작동하도록 보장합니다.

- [중단점 추가 방법](../cloud/how-tos/human_in_the_loop_breakpoint.md)
- [사용자 입력 대기 방법](../cloud/how-tos/human_in_the_loop_user_input.md)
- [그래프 상태 편집 방법](../cloud/how-tos/human_in_the_loop_edit_state.md)
- [이전 상태에서 재생 및 분기 방법](../cloud/how-tos/human_in_the_loop_time_travel.md)
- [도구 호출 검토 방법](../cloud/how-tos/human_in_the_loop_review_tool_calls.md)

### 이중 텍스트

그래프 실행은 시간이 걸릴 수 있으며, 때때로 사용자가 원래 입력이 완료되기 전에 보내고자 했던 입력에 대한 생각을 바꿀 수 있습니다. 예를 들어, 사용자가 원래 요청에서 오타를 발견하고 프롬프트를 수정한 후 다시 보내는 경우가 있습니다. 이러한 경우에 어떻게 결정할지는 원활한 사용자 경험을 보장하고 그래프가 예기치 않게 작동하지 않도록 하는 데 중요합니다.

- [중단 옵션 사용 방법](../cloud/how-tos/interrupt_concurrent.md)
- [롤백 옵션 사용 방법](../cloud/how-tos/rollback_concurrent.md)
- [거부 옵션 사용 방법](../cloud/how-tos/reject_concurrent.md)
- [큐에 추가 옵션 사용 방법](../cloud/how-tos/enqueue_concurrent.md)

### 웹훅

- [웹훅 통합 방법](../cloud/how-tos/webhooks.md)

### 크론 작업

- [크론 작업 생성 방법](../cloud/how-tos/cron_jobs.md)

### LangGraph 스튜디오

LangGraph Studio는 에이전트를 시각화, 테스트 및 디버깅하기 위한 내장 UI입니다.

- [LangGraph Cloud 배포에 연결하는 방법](../cloud/how-tos/test_deployment.md)
- [로컬 개발 서버에 연결하는 방법](../how-tos/local-studio.md)
- [로컬 배포(Docker)에 연결하는 방법](../cloud/how-tos/test_local_deployment.md)
- [LangGraph Studio에서 그래프를 테스트하는 방법 (MacOS 전용)](../cloud/how-tos/invoke_studio.md)
- [LangGraph Studio에서 스레드와 상호작용하는 방법](../cloud/how-tos/threads_studio.md)
- [LangGraph Studio에서 데이터셋 예제로 노드 추가하는 방법](../cloud/how-tos/datasets_studio.md)

## 문제 해결

다음은 LangGraph로 빌드할 때 발생할 수 있는 일반적인 오류를 해결하기 위한 가이드입니다. 아래에서 언급된 오류들은 코드에서 발생할 때 `lc_error_code` 속성이 아래 코드 중 하나에 해당합니다.

- [GRAPH_RECURSION_LIMIT](../troubleshooting/errors/GRAPH_RECURSION_LIMIT.md)
- [INVALID_CONCURRENT_GRAPH_UPDATE](../troubleshooting/errors/INVALID_CONCURRENT_GRAPH_UPDATE.md)
- [INVALID_GRAPH_NODE_RETURN_VALUE](../troubleshooting/errors/INVALID_GRAPH_NODE_RETURN_VALUE.md)
- [MULTIPLE_SUBGRAPHS](../troubleshooting/errors/MULTIPLE_SUBGRAPHS.md)
- [INVALID_CHAT_HISTORY](../troubleshooting/errors/INVALID_CHAT_HISTORY.md)

### LangGraph 플랫폼 문제 해결

이 가이드는 LangGraph 플랫폼에 특정한 오류에 대한 문제 해결 정보를 제공합니다.

- [INVALID_LICENSE](../troubleshooting/errors/INVALID_LICENSE.md)