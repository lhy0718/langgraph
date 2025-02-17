_한국어로 기계번역됨_

# 사용자 정의 인증 추가 방법

!!! 팁 "사전 요건"

    이 가이드는 다음 개념에 대한 친숙함을 가정합니다:

      *  [**인증 및 접근 제어**](../../concepts/auth.md)
      *  [**LangGraph 플랫폼**](../../concepts/index.md#langgraph-platform)
    
    보다 자세한 안내를 원하신다면 [**사용자 정의 인증 설정**](../../tutorials/auth/getting_started.md) 튜토리얼을 참조하시기 바랍니다.

???+ 노트 "Python 전용"
  
    현재 우리는 `langgraph-api>=0.0.11`가 설치된 Python 배포에서만 사용자 정의 인증 및 권한 부여를 지원합니다. LangGraph.JS에 대한 지원은 곧 추가될 예정입니다.

???+ 노트 "배포 유형별 지원"

    사용자 정의 인증은 **관리되는 LangGraph Cloud**의 모든 배포와 **Enterprise** 자가 호스팅 요금제에서 지원됩니다. **Lite** 자가 호스팅 요금제에서는 지원되지 않습니다.

이 가이드는 LangGraph 플랫폼 응용 프로그램에 사용자 정의 인증을 추가하는 방법을 보여줍니다. 이 가이드는 LangGraph Cloud, BYOC 및 자가 호스팅 배포 모두에 적용됩니다. 당신이 자신의 사용자 정의 서버에서 LangGraph 오픈 소스 라이브러리를 독립적으로 사용하는 경우에는 적용되지 않습니다.

## 1. 인증 구현

```python
from langgraph_sdk import Auth

my_auth = Auth()

@my_auth.authenticate
async def authenticate(authorization: str) -> str:
    token = authorization.split(" ", 1)[-1] # "Bearer <token>"
    try:
        # 인증 제공자를 통해 토큰 확인
        user_id = await verify_token(token)
        return user_id
    except Exception:
        raise Auth.exceptions.HTTPException(
            status_code=401,
            detail="유효하지 않은 토큰"
        )

# 실제로 자원에 대한 접근을 제어하는 권한 규칙 추가
@my_auth.on
async def add_owner(
    ctx: Auth.types.AuthContext,
    value: dict,
):
    """소유자를 자원 메타데이터에 추가하고 소유자를 기준으로 필터링합니다."""
    filters = {"owner": ctx.user.identity}
    metadata = value.setdefault("metadata", {})
    metadata.update(filters)
    return filters

# 사용자 정보를 저장소에 (user_id, resource_type, resource_id) 형식으로 정리한다고 가정합니다.
@my_auth.on.store()
async def authorize_store(ctx: Auth.types.AuthContext, value: dict):
    namespace: tuple = value["namespace"]
    assert namespace[0] == ctx.user.identity, "권한 없음"

```

## 2. 구성 업데이트

`langgraph.json` 파일에 인증 파일 경로를 추가합니다:

```json hl_lines="7-9"
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./agent.py:graph"
  },
  "env": ".env",
  "auth": {
    "path": "./auth.py:my_auth"
  }
}
```

## 3. 클라이언트에서 연결

서버에서 인증을 설정한 후, 요청에는 선택한 스킴에 따라 필수 인증 정보가 포함되어야 합니다.
JWT 토큰 인증을 사용한다고 가정하면, 다음 방법 중 하나를 사용하여 배포에 접근할 수 있습니다:

=== "Python 클라이언트"

    ```python
    from langgraph_sdk import get_client

    my_token = "your-token" # 실제로는 인증 제공자가 서명한 토큰을 생성해야 합니다.
    client = get_client(
        url="http://localhost:2024",
        headers={"Authorization": f"Bearer {my_token}"}
    )
    threads = await client.threads.search()
    ```

=== "Python RemoteGraph"

    ```python
    from langgraph.pregel.remote import RemoteGraph
    
    my_token = "your-token" # 실제로는 인증 제공자로 서명된 토큰을 생성합니다.
    remote_graph = RemoteGraph(
        "agent",
        url="http://localhost:2024",
        headers={"Authorization": f"Bearer {my_token}"}
    )
    threads = await remote_graph.ainvoke(...)
    ```

=== "JavaScript 클라이언트"

    ```javascript
    import { Client } from "@langchain/langgraph-sdk";

    const my_token = "your-token"; // 실제로는 인증 제공자로 서명된 토큰을 생성합니다.
    const client = new Client({
      apiUrl: "http://localhost:2024",
      headers: { Authorization: `Bearer ${my_token}` },
    });
    const threads = await client.threads.search();
    ```

=== "JavaScript RemoteGraph"

    ```javascript
    import { RemoteGraph } from "@langchain/langgraph/remote";

    const my_token = "your-token"; // 실제로는 인증 제공자로 서명된 토큰을 생성합니다.
    const remoteGraph = new RemoteGraph({
      graphId: "agent",
      url: "http://localhost:2024",
      headers: { Authorization: `Bearer ${my_token}` },
    });
    const threads = await remoteGraph.invoke(...);
    ```

=== "CURL"

    ```bash
    curl -H "Authorization: Bearer ${your-token}" http://localhost:2024/threads
    ```
