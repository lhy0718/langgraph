_한국어로 기계번역됨_

---
title: 튜토리얼
---

# 튜토리얼

LangGraph 또는 LLM 애플리케이션 개발이 처음이신가요? 이 자료를 읽고 첫 번째 애플리케이션을 구축하는 데 도움을 받으세요.

## 시작하기 🚀 {#quick-start}

- [LangGraph 빠른 시작](introduction.ipynb): 도구를 사용할 수 있는 챗봇을 구축하고 대화 기록을 유지합니다. 인간 개입 기능을 추가하고 시간 여행이 어떻게 작동하는지 탐구합니다.
- [일반적인 워크플로우](workflows/index.md): LangGraph를 사용하여 구현된 가장 일반적인 LLM 워크플로우의 개요입니다.
- [LangGraph 서버 빠른 시작](langgraph-platform/local-server.md): 로컬에서 LangGraph 서버를 시작하고 REST API 및 LangGraph 스튜디오 웹 UI를 사용하여 상호작용합니다.
- [LangGraph 템플릿 빠른 시작](../concepts/template_applications.md): 템플릿 애플리케이션을 사용하여 LangGraph 플랫폼으로 구축을 시작합니다.
- [LangGraph 클라우드로 배포하기](../cloud/quick_start.md): LangGraph 클라우드를 사용하여 LangGraph 애플리케이션을 배포합니다.

## 사용 사례 🛠️ {#use-cases}

특정 시나리오에 맞춘 실용적인 구현을 탐구해 보세요:

### 챗봇

- [고객 지원](customer-support/customer-support.ipynb): 항공편, 호텔 및 자동차 대여를 위한 다기능 지원 봇을 구축합니다.
- [사용자 요구에 따른 프롬프트 생성](chatbots/information-gather-prompting.ipynb): 정보 수집 챗봇을 구축합니다.
- [코드 도우미](code_assistant/langgraph_code_assistant.ipynb): 코드 분석 및 생성 도우미를 구축합니다.

### RAG

- [Agentic RAG](rag/langgraph_agentic_rag.ipynb): 사용자의 질문에 답하기 위해 가장 관련성 높은 정보를 검색하는 방법을 알아내는 에이전트를 사용합니다.
- [Adaptive RAG](rag/langgraph_adaptive_rag.ipynb): Adaptive RAG는 (1) 쿼리 분석과 (2) 능동적/자기 수정 RAG를 통합하는 RAG 전략입니다. 구현: https://arxiv.org/abs/2403.14403
    - 로컬 LLM을 사용하는 버전: [로컬 LLM을 이용한 Adaptive RAG](rag/langgraph_adaptive_rag_local.ipynb)
- [Corrective RAG](rag/langgraph_crag.ipynb): 주어진 출처에서 검색된 정보의 품질을 평가하기 위해 LLM을 사용하고 품질이 낮으면 다른 출처에서 정보를 검색하려고 시도합니다. 구현: https://arxiv.org/pdf/2401.15884.pdf 
    - 로컬 LLM을 사용하는 버전: [로컬 LLM을 이용한 Corrective RAG](rag/langgraph_crag_local.ipynb)
- [Self-RAG](rag/langgraph_self_rag.ipynb): Self-RAG는 검색된 문서와 생성물에 대해 자기 반성/자체 평가를 포함하는 RAG 전략입니다. 구현: https://arxiv.org/abs/2310.11511.
    - 로컬 LLM을 사용하는 버전: [로컬 LLM을 이용한 Self-RAG](rag/langgraph_self_rag_local.ipynb) 
- [SQL 에이전트](sql-agent.ipynb): SQL 데이터베이스에 대한 질문에 답할 수 있는 SQL 에이전트를 구축합니다.

### 에이전트 아키텍처

#### 다중 에이전트 시스템

- [네트워크](multi_agent/multi-agent-collaboration.ipynb): 두 개 이상의 에이전트가 작업에 협력할 수 있도록 합니다.
- [감독](multi_agent/agent_supervisor.ipynb): LLM을 사용하여 개별 에이전트에게 지휘하고 위임합니다.
- [계층 팀](multi_agent/hierarchical_agent_teams.ipynb): 문제를 해결하기 위해 에이전트의 중첩 팀을 조정합니다.

#### 계획 에이전트

- [계획 및 실행](plan-and-execute/plan-and-execute.ipynb): 기본 계획 및 실행 에이전트를 구현합니다.
- [관찰 없는 추론](rewoo/rewoo.ipynb): 관찰을 변수로 저장하여 재계획을 줄입니다.
- [LLM 컴파일러](llm-compiler/LLMCompiler.ipynb): 기획자로부터 작업의 DAG를 스트리밍하고 적극적으로 실행합니다.

#### 반성 및 비판 

- [기본 반성](reflection/reflection.ipynb): 에이전트에게 자신의 출력을 반성하고 수정하도록 요청합니다.
- [반성](reflexion/reflexion.ipynb): 다음 단계를 안내하기 위해 빠진 세부 사항 및 불필요한 세부 사항을 비판합니다.
- [사고의 나무](tot/tot.ipynb): 점수가 매겨진 나무를 사용하여 문제에 대한 후보 솔루션을 검색합니다.
- [언어 에이전트 트리 검색](lats/lats.ipynb): 반성과 보상을 사용하여 에이전트에 대한 몬테카를로 트리 검색을 진행합니다.
- [자기 발견 에이전트](self-discover/self-discover.ipynb): 자신의 능력에 대해 배우는 에이전트를 분석합니다.

### 평가

- [에이전트 기반](chatbot-simulation-evaluation/agent-simulation-evaluation.ipynb): 시뮬레이터 사용자 상호작용을 통해 챗봇을 평가합니다.
- [LangSmith에서](chatbot-simulation-evaluation/langsmith-agent-simulation-evaluation.ipynb): 대화 데이터 세트에서 LangSmith의 챗봇을 평가합니다.

### 실험적

- [웹 연구 (STORM)](storm/storm.ipynb): 연구 및 다각적 QA를 통해 위키피디아와 유사한 기사를 생성합니다.
- [TNT-LLM](tnt-llm/tnt-llm.ipynb): Microsoft의 Bing Copilot 애플리케이션을 위해 개발된 분류 시스템을 사용하여 사용자 의도의 풍부하고 해석 가능한 분류를 구축합니다.
- [웹 내비게이션](web-navigation/web_voyager.ipynb): 웹사이트를 탐색하고 상호작용할 수 있는 에이전트를 구축합니다.
- [경쟁 프로그래밍](usaco/usaco.ipynb): USA 컴퓨팅 올림피아드의 문제를 해결하기 위해 몇 가지 "일화적 기억"과 인간 개입 협력을 가진 에이전트를 구축합니다. 이는 Shi, Tang, Narasimhan, Yao의 ["언어 모델이 올림피아드 프로그래밍을 해결할 수 있을까요?"](https://arxiv.org/abs/2404.10952v1) 논문에 기반하여 조정되었습니다.
- [복잡한 데이터 추출](extraction/retries.ipynb): 복잡한 추출 작업을 수행하기 위해 함수 호출을 사용할 수 있는 에이전트를 구축합니다.

## LangGraph 플랫폼 🧱 {#platform}

### 인증 및 접근 제어

다음 세 부분으로 구성된 가이드를 통해 기존 LangGraph 플랫폼 배포에 사용자 정의 인증 및 권한 부여를 추가합니다:

1. [사용자 정의 인증 설정](auth/getting_started.md): 배포에서 사용자 인증을 허용하기 위해 OAuth2 인증을 구현합니다.
2. [자원 권한 부여](auth/resource_auth.md): 사용자에게 비공식 대화를 허용합니다.
3. [인증 제공자 연결](auth/add_auth_server.md): 실제 사용자 계정을 추가하고 OAuth2를 사용하여 유효성을 검사합니다.