# 템플릿 애플리케이션

템플릿은 LangGraph로 빠르게 시작할 수 있도록 도와주는 오픈 소스 참조 애플리케이션입니다. 이 템플릿은 일반적인 에이전틱 워크플로우의 작동 예제를 제공하며, 필요에 맞게 커스터마이즈할 수 있습니다.

LangGraph CLI를 사용하여 템플릿에서 애플리케이션을 만들 수 있습니다.

!!! 정보 "요구 사항"

    - Python >= 3.11
    - [LangGraph CLI](https://langchain-ai.github.io/langgraph/cloud/reference/cli/): langchain-cli[inmem] >= 0.1.58 이상 필요

## LangGraph CLI 설치

```bash
pip install "langgraph-cli[inmem]" --upgrade
```

## 사용 가능한 템플릿

| 템플릿                        | 설명                                                                                  | Python                                                             | JS/TS                                                               |
|-------------------------------|--------------------------------------------------------------------------------------|--------------------------------------------------------------------|---------------------------------------------------------------------|
| **새 LangGraph 프로젝트**        | 메모리가 있는 간단한 최소화된 챗봇.                                                    | [Repo](https://github.com/langchain-ai/new-langgraph-project)      | [Repo](https://github.com/langchain-ai/new-langgraphjs-project)     |
| **ReAct 에이전트**               | 다양한 도구로 확장할 수 있는 간단한 에이전트.                                            | [Repo](https://github.com/langchain-ai/react-agent)                | [Repo](https://github.com/langchain-ai/react-agent-js)              |
| **메모리 에이전트**              | ReAct 스타일의 에이전트로, 스레드 간에 사용할 수 있는 메모리 저장 도구가 추가된 에이전트. | [Repo](https://github.com/langchain-ai/memory-agent)               | [Repo](https://github.com/langchain-ai/memory-agent-js)             |
| **검색 기반 에이전트**           | 검색 기반의 질문-답변 시스템을 포함하는 에이전트.                                         | [Repo](https://github.com/langchain-ai/retrieval-agent-template)   | [Repo](https://github.com/langchain-ai/retrieval-agent-template-js) |
| **데이터 강화 에이전트**         | 웹 검색을 수행하고 그 결과를 구조화된 형식으로 정리하는 에이전트.                       | [Repo](https://github.com/langchain-ai/data-enrichment)            | [Repo](https://github.com/langchain-ai/data-enrichment-js)          |

## 🌱 LangGraph 앱 만들기

새로운 앱을 템플릿에서 생성하려면 `langgraph new` 명령어를 사용하세요.

```bash
langgraph new
```

## 다음 단계

새로 만든 LangGraph 앱의 루트에 있는 `README.md` 파일을 확인하여 템플릿에 대한 정보와 커스터마이징 방법을 확인하세요.

앱을 적절히 구성하고 API 키를 추가한 후, LangGraph CLI를 사용하여 앱을 시작할 수 있습니다:

```bash
langgraph dev 
```

앱을 배포하는 방법에 대한 추가 정보는 아래의 가이드를 참조하세요:

- **[로컬 LangGraph 서버 시작](../tutorials/langgraph-platform/local-server.md)**: 이 빠른 시작 가이드는 **ReAct 에이전트** 템플릿에 대해 LangGraph 서버를 로컬에서 시작하는 방법을 설명합니다. 다른 템플릿에도 유사한 절차가 적용됩니다.
- **[LangGraph Cloud에 배포](../cloud/quick_start.md)**: LangGraph Cloud를 사용하여 LangGraph 앱을 배포하세요.
 
### LangGraph 프레임워크

- **[LangGraph 개념](../concepts/index.md)**: LangGraph의 기본 개념을 배우세요.
- **[LangGraph 사용법 가이드](../how-tos/index.md)**: LangGraph에서 자주 사용하는 작업에 대한 가이드입니다.

### 📚 LangGraph 플랫폼에 대해 더 배우기

다음 리소스를 통해 더 많은 정보를 확인하세요:

- **[LangGraph 플랫폼 개념](../concepts/index.md#langgraph-platform)**: LangGraph 플랫폼의 기본 개념을 이해하세요.
- **[LangGraph 플랫폼 사용법 가이드](../how-tos/index.md#langgraph-platform)**: 애플리케이션을 구축하고 배포하는 단계별 가이드를 확인하세요.