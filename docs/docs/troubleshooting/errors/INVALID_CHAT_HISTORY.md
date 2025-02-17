_한국어로 기계번역됨_

# 잘못된 채팅 기록

이 오류는 미리 만들어진 [create_react_agent][langgraph.prebuilt.chat_agent_executor.create_react_agent]에서 `call_model` 그래프 노드가 잘못 형성된 메시지 목록을 받을 때 발생합니다. 구체적으로, 도구 호출이 있는 `AIMessages`(LLM이 도구를 호출하도록 요청)가 해당 `ToolMessage`(LLM에 반환할 도구 호출 결과) 없이 존재할 때 잘못 형성된 것입니다.

이 오류가 발생하는 몇 가지 이유가 있을 수 있습니다:

1. 그래프를 호출할 때 잘못된 메시지 목록을 수동으로 전달했습니다. 예: `graph.invoke({'messages': [AIMessage(..., tool_calls=[...])]})`
2. 그래프가 `tools` 노드에서 업데이트를 받기 전에 중단되었고 (즉, ToolMessages 목록) None이 아닌 입력 또는 ToolMessage로 호출했습니다. 예: `graph.invoke({'messages': [HumanMessage(...)]}, config)`.
   이 중단은 다음 방법 중 하나로 발생했을 수 있습니다:
   - `create_react_agent`에서 `interrupt_before = ['tools']`를 수동으로 설정했습니다.
   - 도구 중 하나가 [ToolNode][langgraph.prebuilt.tool_node.ToolNode]에서 처리되지 않은 오류를 발생시켰습니다 (`"tools"`).

## 문제 해결

이 문제를 해결하려면 다음 중 하나를 수행할 수 있습니다:

1. 잘못된 메시지 목록으로 그래프를 호출하지 마십시오.
2. 중단이 발생한 경우(수동 또는 오류로 인한) 다음을 수행할 수 있습니다:

    - 기존 도구 호출과 일치하는 ToolMessages를 제공하고 `graph.invoke({'messages': [ToolMessage(...)]})`를 호출하십시오.
    **참고**: 이는 메시지를 기록에 추가하고 그래프를 START 노드에서 실행합니다.
    - 상태를 수동으로 업데이트하고 중단된 지점에서 그래프를 재개하십시오:

        1. `graph.get_state(config)`로 그래프 상태에서 가장 최근 메시지 목록을 가져옵니다.
        2. AIMessages에서 응답이 없는 도구 호출을 제거하거나 응답이 없는 도구 호출과 일치하는 tool_call_ids가 있는 ToolMessages를 추가하여 메시지 목록을 수정합니다.
        3. 수정된 메시지 목록으로 `graph.update_state(config, {'messages': ...})`를 호출합니다.
        4. 그래프를 재개합니다. 예: `graph.invoke(None, config)` 호출.
