_한국어로 기계번역됨_

# 새로운 LangGraph.js 프로젝트

[![CI](https://github.com/langchain-ai/new-langgraphjs-project/actions/workflows/unit-tests.yml/badge.svg)](https://github.com/langchain-ai/new-langgraphjs-project/actions/workflows/unit-tests.yml)
[![Integration Tests](https://github.com/langchain-ai/new-langgraphjs-project/actions/workflows/integration-tests.yml/badge.svg)](https://github.com/langchain-ai/new-langgraphjs-project/actions/workflows/integration-tests.yml)
[![Open in - LangGraph Studio](https://img.shields.io/badge/Open_in-LangGraph_Studio-00324d.svg?logo=data:image/svg%2bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI4NS4zMzMiIGhlaWdodD0iODUuMzMzIiB2ZXJzaW9uPSIxLjAiIHZpZXdCb3g9IjAgMCA2NCA2NCI+PHBhdGggZD0iTTEzIDcuOGMtNi4zIDMuMS03LjEgNi4zLTYuOCAyNS43LjQgMjQuNi4zIDI0LjUgMjUuOSAyNC41QzU3LjUgNTggNTggNTcuNSA1OCAzMi4zIDU4IDcuMyA1Ni43IDYgMzIgNmMtMTIuOCAwLTE2LjEuMy0xOSAxLjhtMzcuNiAxNi42YzIuOCAyLjggMy40IDQuMiAzLjQgNy42cy0uNiA0LjgtMy40IDcuNkw0Ny4yIDQzSDE2LjhsLTMuNC0zLjRjLTQuOC00LjgtNC44LTEwLjQgMC0xNS4ybDMuNC0zLjRoMzAuNHoiLz48cGF0aCBkPSJNMTguOSAyNS42Yy0xLjEgMS4zLTEgMS43LjQgMi41LjkuNiAxLjcgMS44IDEuNyAyLjcgMCAxIC43IDIuOCAxLjYgNC4xIDEuNCAxLjkgMS40IDIuNS4zIDMuMi0xIC42LS42LjkgMS40LjkgMS41IDAgMi43LS41IDIuNy0xIDAtLjYgMS4xLS44IDIuNi0uNGwyLjYuNy0xLjgtMi45Yy01LjktOS4zLTkuNC0xMi4zLTExLjUtOS44TTM5IDI2YzAgMS4xLS45IDIuNS0yIDMuMi0yLjQgMS41LTIuNiAzLjQtLjUgNC4yLjguMyAyIDEuNyAyLjUgMy4xLjYgMS41IDEuNCAyLjMgMiAyIDEuNS0uOSAxLjItMy41LS40LTMuNS0yLjEgMC0yLjgtMi44LS44LTMuMyAxLjYtLjQgMS42LS41IDAtLjYtMS4xLS4xLTEuNS0uNi0xLjItMS42LjctMS43IDMuMy0yLjEgMy41LS41LjEuNS4yIDEuNi4zIDIuMiAwIC43LjkgMS40IDEuOSAxLjYgMi4xLjQgMi4zLTIuMy4yLTMuMi0uOC0uMy0yLTEuNy0yLjUtMy4xLTEuMS0zLTMtMy4zLTMtLjUiLz48L3N2Zz4=)](https://langgraph-studio.vercel.app/templates/open?githubUrl=https://github.com/langchain-ai/new-langgraphjs-project)

이 템플릿은 [LangGraph.js](https://github.com/langchain-ai/langgraphjs)를 사용하여 구현된 간단한 챗봇을 보여줍니다. 이 챗봇은 [LangGraph Studio](https://github.com/langchain-ai/langgraph-studio)를 위해 설계되었으며, 지속적인 채팅 메모리를 유지하여 여러 상호 작용에 걸쳐 일관된 대화를 가능하게 합니다.

![LangGraph 스튜디오 UI의 그래프 뷰](./static/studio.png)

핵심 로직은 `src/agent/graph.ts`에 정의되어 있으며, 사용자의 질문에 답변하면서 이전 메시지의 맥락을 유지하는 간단한 챗봇을 보여줍니다.

## 기능

간단한 챗봇은 다음과 같은 기능을 제공합니다:

1. 사용자 **메시지**를 입력으로 받습니다.
2. 대화의 히스토리를 유지합니다.
3. 자리 표시자 응답을 반환하며, 대화 히스토리를 업데이트합니다.

이 템플릿은 더 복잡한 대화형 에이전트를 만들기 위해 쉽게 사용자 지정하고 확장할 수 있는 기반을 제공합니다.

## 시작하기

이미 [LangGraph Studio](https://github.com/langchain-ai/langgraph-studio?tab=readme-ov-file#download)를 설치했다고 가정할 때, 설정하려면:

1. `.env` 파일을 생성합니다. 이 템플릿은 기본적으로 환경 변수를 요구하지 않지만, 사용자 지정할 때 여러 개를 추가하고 싶을 수 있습니다.

```bash
cp .env.example .env
```

<!--
설정 지침은 `langgraph template lock`에 의해 자동 생성되었습니다. 수동으로 수정하지 마세요.
-->

2. LangGraph Studio에서 폴더를 엽니다!
3. 필요한 대로 코드를 사용자 지정합니다.

## 사용자 지정 방법

1. **LLM 호출 추가**: [LangChain.js 생태계](https://js.langchain.com/docs/integrations/chat/)에서 채팅 모델 래퍼를 선택하고 설치하거나 LangChain.js 없이 LangGraph.js를 사용할 수 있습니다.
2. **그래프 확장**: 챗봇의 핵심 로직은 [graph.ts](./src/agent/graph.ts)에 정의되어 있습니다. 이 파일을 수정하여 새로운 노드, 엣지를 추가하거나 대화의 흐름을 변경할 수 있습니다.

다음과 같은 방식으로 이 템플릿을 확장할 수 있습니다:

- 챗봇의 기능을 향상시키기 위해 [맞춤 도구 또는 함수](https://js.langchain.com/docs/how_to/tool_calling)를 추가합니다.
- 특정 유형의 사용자 질문이나 작업을 처리하기 위한 추가 논리를 구현합니다.
- [외부 API 또는 데이터베이스](https://langchain-ai.github.io/langgraphjs/tutorials/rag/langgraph_agentic_rag/)를 통합하여 검색 보강 생성(RAG) 기능을 추가하여 더 맞춤화된 응답을 제공합니다.

## 개발

그래프를 반복 작업하는 동안, 이전 상태를 수정하고 이전 상태에서 앱을 재실행하여 특정 노드를 디버그할 수 있습니다. 로컬 변경 사항은 핫 리로드를 통해 자동으로 적용됩니다. 다음과 같은 실험을 해보세요:

- 시스템 프롬프트를 수정하여 챗봇에 독특한 성격을 부여합니다.
- 더 복잡한 대화 흐름을 위해 그래프에 새로운 노드를 추가합니다.
- 다양한 유형의 사용자 입력을 처리하기 위한 조건부 논리를 구현합니다.

후속 요청은 동일한 스레드에 추가됩니다. 이전 히스토리를 지우고 완전히 새로운 스레드를 만들려면 우측 상단의 `+` 버튼을 사용합니다.

더 고급 기능 및 예제에 대한 내용은 [LangGraph.js 문서](https://github.com/langchain-ai/langgraphjs)를 참조하세요. 이러한 자원은 이 템플릿을 특정 용도에 맞게 조정하고 더 정교한 대화형 에이전트를 구축하는 데 도움이 될 것입니다.

LangGraph Studio는 [LangSmith](https://smith.langchain.com/)와도 통합되어 있어, 팀원들과의 더 깊이 있는 추적 및 협업을 통해 챗봇의 성능을 분석하고 최적화할 수 있습니다.

<!--
구성은 `langgraph template lock`에 의해 자동 생성되었습니다. 수동으로 수정하지 마세요.
{
  "config_schemas": {
    "agent": {
      "type": "object",
      "properties": {}
    }
  }
}
-->
