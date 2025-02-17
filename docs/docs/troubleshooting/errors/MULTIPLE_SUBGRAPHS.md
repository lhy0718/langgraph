_한국어로 기계번역됨_

# 다중 서브그래프

체크포인팅이 활성화된 상태에서 하나의 LangGraph 노드 내에서 서브그래프를 여러 번 호출하는 것은 현재 내부 제한으로 인해 허용되지 않습니다.

## 문제 해결

다음 사항이 이 오류를 해결하는 데 도움이 될 수 있습니다:

- 서브그래프에서 중단/재개할 필요가 없다면, 이를 컴파일할 때 `checkpointer=False`를 전달하세요. 예: `.compile(checkpointer=False)`
- 동일한 노드 내에서 그래프를 여러 번 강제로 호출하지 말고, 대신 [`Send`](https://langchain-ai.github.io/langgraph/concepts/low_level/#send) API를 사용하세요.
