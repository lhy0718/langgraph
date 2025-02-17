_한국어로 기계번역됨_

# RemoteGraph를 사용하여 배포와 상호작용하는 방법

!!! 정보 "전제 조건"
    - [LangGraph 플랫폼](../concepts/langgraph_platform.md)
    - [LangGraph 서버](../concepts/langgraph_server.md)

`RemoteGraph`는 LangGraph 플랫폼 배포와 상호작용할 수 있는 인터페이스로, 마치 일반적으로 정의된 로컬 LangGraph 그래프(예: `CompiledGraph`)처럼 사용할 수 있습니다. 이 가이드는 `RemoteGraph`를 초기화하고 상호작용하는 방법을 보여줍니다.

## 그래프 초기화

`RemoteGraph`를 초기화할 때에는 항상 다음을 지정해야 합니다:

- `name`: 상호작용할 그래프의 이름. 이는 배포를 위한 `langgraph.json` 구성 파일에서 사용하는 그래프 이름과 동일합니다.
- `api_key`: 유효한 LangSmith API 키. 환경 변수(`LANGSMITH_API_KEY`)로 설정하거나 `api_key` 인수를 통해 직접 전달할 수 있습니다. API 키는 `LangGraphClient` / `SyncLangGraphClient`가 `api_key` 인수로 초기화된 경우 `client` / `sync_client` 인수를 통해서도 제공될 수 있습니다.

추가로 다음 중 하나를 제공해야 합니다:

- `url`: 상호작용할 배포의 URL. `url` 인수를 전달하면 제공된 URL, 헤더(제공된 경우) 및 기본 구성 값(예: 타임아웃 등)을 사용하여 비동기 및 동기 클라이언트가 생성됩니다.
- `client`: 비동기적으로 배포와 상호작용하기 위한 `LangGraphClient` 인스턴스(예: `.astream()`, `.ainvoke()`, `.aget_state()`, `.aupdate_state()` 등의 메서드를 사용).
- `sync_client`: 동기적으로 배포와 상호작용하기 위한 `SyncLangGraphClient` 인스턴스(예: `.stream()`, `.invoke()`, `.get_state()`, `.update_state()` 등의 메서드를 사용).

!!! 주의

    `client` 또는 `sync_client`와 `url` 인수를 모두 제공하는 경우, `url` 인수보다 우선합니다. `client` / `sync_client` / `url` 인수가 모두 제공되지 않으면, `RemoteGraph`는 런타임에 `ValueError`를 발생시킵니다.


### URL 사용하기

=== "Python"

    ```python
    from langgraph.pregel.remote import RemoteGraph

    url = <DEPLOYMENT_URL>
    graph_name = "agent"
    remote_graph = RemoteGraph(graph_name, url=url)
    ```

=== "JavaScript"

    ```ts
    import { RemoteGraph } from "@langchain/langgraph/remote";

    const url = `<DEPLOYMENT_URL>`;
    const graphName = "agent";
    const remoteGraph = new RemoteGraph({ graphId: graphName, url });
    ```

### 클라이언트 사용하기

=== "Python"

    ```python
    from langgraph_sdk import get_client, get_sync_client
    from langgraph.pregel.remote import RemoteGraph

    url = <DEPLOYMENT_URL>
    graph_name = "agent"
    client = get_client(url=url)
    sync_client = get_sync_client(url=url)
    remote_graph = RemoteGraph(graph_name, client=client, sync_client=sync_client)
    ```

=== "JavaScript"

    ```ts
    import { Client } from "@langchain/langgraph-sdk";
    import { RemoteGraph } from "@langchain/langgraph/remote";

    const client = new Client({ apiUrl: `<DEPLOYMENT_URL>` });
    const graphName = "agent";
    const remoteGraph = new RemoteGraph({ graphId: graphName, client });
    ```

## 그래프 호출하기

`RemoteGraph`는 `Runnable`이며 `CompiledGraph`와 동일한 메서드를 구현하므로, 보통의 컴파일된 그래프와 동일한 방식으로 상호작용할 수 있습니다. 즉, `.invoke()`, `.stream()`, `.get_state()`, `.update_state()` 등을 호출하여 상호작용할 수 있습니다(비동기 버전도 마찬가지입니다).

### 비동기적으로

!!! 주의

    그래프를 비동기적으로 사용하려면 `RemoteGraph`를 초기화할 때 `url` 또는 `client`를 제공해야 합니다.

=== "Python"

    ```python
    # 그래프 호출
    result = await remote_graph.ainvoke({
        "messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]
    })

    # 그래프로부터 출력 스트리밍
    async for chunk in remote_graph.astream({
        "messages": [{"role": "user", "content": "로스앤젤레스의 날씨는 어때?"}]
    }):
        print(chunk)
    ```

=== "JavaScript"

    ```ts
    // 그래프 호출
    const result = await remoteGraph.invoke({
        messages: [{role: "user", content: "샌프란시스코의 날씨는 어때?"}]
    })

    // 그래프에서 출력 스트리밍
    for await (const chunk of await remoteGraph.stream({
        messages: [{role: "user", content: "로스앤젤레스의 날씨는 어때?"}]
    })):
        console.log(chunk)
    ```

### 동기적으로

!!! 주의

    그래프를 동기적으로 사용하려면 `RemoteGraph` 초기화 시 `url` 또는 `sync_client`를 제공해야 합니다.

=== "파이썬"

    ```python
    # 그래프 호출
    result = remote_graph.invoke({
        "messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]
    })

    # 그래프에서 출력 스트리밍
    for chunk in remote_graph.stream({
        "messages": [{"role": "user", "content": "로스앤젤레스의 날씨는 어때?"}]
    }):
        print(chunk)
    ```

## 스레드 수준의 지속성

기본적으로 그래프 실행 (즉, `.invoke()` 또는 `.stream()` 호출)은 무상태이며 - 체크포인트와 그래프의 최종 상태가 지속되지 않습니다. 그래프 실행의 출력을 지속하고 싶다면 (예: 사람의 개입 기능을 활성화하기 위해), 스레드를 생성하고 `config` 인자를 통해 스레드 ID를 제공해야 합니다. 일반적으로 컴파일된 그래프와 동일합니다:

=== "파이썬"

    ```python
    from langgraph_sdk import get_sync_client
    url = <DEPLOYMENT_URL>
    graph_name = "agent"
    sync_client = get_sync_client(url=url)
    remote_graph = RemoteGraph(graph_name, url=url)

    # 스레드 생성 (또는 기존 스레드 사용)
    thread = sync_client.threads.create()

    # 스레드 구성으로 그래프 호출
    config = {"configurable": {"thread_id": thread["thread_id"]}}
    result = remote_graph.invoke({
        "messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]
    }, config=config)

    # 상태가 스레드에 지속되었는지 검증
    thread_state = remote_graph.get_state(config)
    print(thread_state)
    ```

=== "자바스크립트"

    ```ts
    import { Client } from "@langchain/langgraph-sdk";
    import { RemoteGraph } from "@langchain/langgraph/remote";

    const url = `<DEPLOYMENT_URL>`;
    const graphName = "agent";
    const client = new Client({ apiUrl: url });
    const remoteGraph = new RemoteGraph({ graphId: graphName, url });

    // 스레드 생성 (또는 기존 스레드 사용)
    const thread = await client.threads.create();

    // 스레드 구성으로 그래프 호출
    const config = { configurable: { thread_id: thread.thread_id }};
    const result = await remoteGraph.invoke({
      messages: [{ role: "user", content: "샌프란시스코의 날씨는 어때?" }],
    }, config);

    // 상태가 스레드에 지속되었는지 검증
    const threadState = await remoteGraph.getState(config);
    console.log(threadState);
    ```

## 서브그래프로 사용하기

!!! 주의

    `RemoteGraph` 서브그래프 노드가 있는 그래프에서 `checkpointer`를 사용해야 하는 경우, 스레드 ID로 UUID를 사용해야 합니다.

`RemoteGraph`는 일반 `CompiledGraph`와 같은 방식으로 동작하므로, 다른 그래프의 서브그래프로도 사용할 수 있습니다. 예를 들어:

=== "파이썬"

    ```python
    from langgraph_sdk import get_sync_client
    from langgraph.graph import StateGraph, MessagesState, START
    from typing import TypedDict

    url = <배포_URL>
    graph_name = "에이전트"
    remote_graph = RemoteGraph(graph_name, url=url)

    # 부모 그래프 정의
    builder = StateGraph(MessagesState)
    # 원격 그래프를 노드로 직접 추가
    builder.add_node("자식", remote_graph)
    builder.add_edge(START, "자식")
    graph = builder.compile()

    # 부모 그래프 호출
    result = graph.invoke({
        "messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]
    })
    print(result)

    # 부모 그래프와 하위 그래프의 출력 스트리밍
    for chunk in graph.stream({
        "messages": [{"role": "user", "content": "샌프란시스코의 날씨는 어때?"}]
    }, subgraphs=True):
        print(chunk)
    ```

=== "JavaScript"

    ```ts
    import { MessagesAnnotation, StateGraph, START } from "@langchain/langgraph";
    import { RemoteGraph } from "@langchain/langgraph/remote";

    const url = `<배포_URL>`;
    const graphName = "에이전트";
    const remoteGraph = new RemoteGraph({ graphId: graphName, url });

    // 부모 그래프 정의 및 원격 그래프를 노드로 직접 추가
    const graph = new StateGraph(MessagesAnnotation)
      .addNode("자식", remoteGraph)
      .addEdge(START, "자식")
      .compile()

    // 부모 그래프 호출
    const result = await graph.invoke({
      messages: [{ role: "user", content: "샌프란시스코의 날씨는 어때?" }]
    });
    console.log(result);

    // 부모 그래프와 하위 그래프의 출력 스트리밍
    for await (const chunk of await graph.stream({
      messages: [{ role: "user", content: "로스앤젤레스의 날씨는 어때?" }]
    }, { subgraphs: true })) {
      console.log(chunk);
    }
    ```