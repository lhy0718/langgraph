_한국어로 기계번역됨_

# 대화 내용을 비공개로 만들기 (2부/3부)

!!! 주의 "인증 시리즈의 2부입니다:" 

    1. [기본 인증](getting_started.md) - 봇에 접근할 수 있는 사람 제어
    2. 리소스 권한 부여 (여기에 있습니다) - 사용자에게 개인 대화 제공
    3. [생산 인증](add_auth_server.md) - 실제 사용자 계정 추가 및 OAuth2를 사용한 검증

이번 튜토리얼에서는 각 사용자에게 개인 대화를 제공하기 위해 챗봇을 확장합니다. 우리는 사용자가 자신의 스레드만 볼 수 있도록 [리소스 수준 접근 제어](../../concepts/auth.md#resource-level-access-control)를 추가할 것입니다.

![권한 부여 핸들러](./img/authorization.png)

???+ 팁 "플레이스홀더 토큰"
    
    [1부](getting_started.md)에서 했던 것처럼, 이 섹션에서는 설명 목적으로 하드코딩된 토큰을 사용할 것입니다.
    기본을 마스터한 후 3부에서 "생산 준비 완료" 인증 방식에 대해 다룰 것입니다.

## 리소스 권한 부여 이해하기

지난 튜토리얼에서는 누가 봇에 접근할 수 있는지를 제어했습니다. 하지만 현재, 인증된 모든 사용자가 다른 사람의 대화를 볼 수 있습니다! 이를 해결하기 위해 [리소스 권한 부여](../../concepts/auth.md#resource-authorization)를 추가하겠습니다.

먼저, [기본 인증](getting_started.md) 튜토리얼을 완료했으며, 보안 봇이 오류 없이 실행될 수 있는지 확인하세요:

```bash
cd custom-auth
pip install -e .
langgraph dev --no-browser
```

> - 🚀 API: http://127.0.0.1:2024
> - 🎨 스튜디오 UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
> - 📚 API 문서: http://127.0.0.1:2024/docs

## 리소스 권한 부여 추가하기

지난 튜토리얼을 기억해보세요. [`Auth`](../../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.auth.Auth) 객체를 사용하여 [인증 함수](../../concepts/auth.md#authentication)를 등록했습니다. LangGraph 플랫폼은 이를 사용하여 수신 요청의 베어러 토큰을 검증합니다. 이제 우리는 **권한 부여** 핸들러를 등록하는 데 사용할 것입니다.

권한 부여 핸들러는 인증이 성공한 **후**에 실행되는 함수입니다. 이러한 핸들러는 리소스에 [메타데이터](../../concepts/auth.md#resource-metadata)를 추가하고 사용자가 볼 수 있는 항목을 필터링할 수 있습니다.

이제 `src/security/auth.py`를 업데이트하고 매 요청마다 실행되는 하나의 권한 부여 핸들러를 추가하겠습니다:

```python hl_lines="29-39" title="src/security/auth.py"
from langgraph_sdk import Auth

# 이전 튜토리얼에서 사용한 테스트 사용자들
VALID_TOKENS = {
    "user1-token": {"id": "user1", "name": "Alice"},
    "user2-token": {"id": "user2", "name": "Bob"},
}

auth = Auth()


@auth.authenticate
async def get_current_user(authorization: str | None) -> Auth.types.MinimalUserDict:
    """이전 튜토리얼의 인증 핸들러입니다."""
    assert authorization
    scheme, token = authorization.split()
    assert scheme.lower() == "bearer"

    if token not in VALID_TOKENS:
        raise Auth.exceptions.HTTPException(status_code=401, detail="잘못된 토큰입니다.")

    user_data = VALID_TOKENS[token]
    return {
        "identity": user_data["id"],
    }


@auth.on
async def add_owner(
    ctx: Auth.types.AuthContext,  # 현재 사용자에 대한 정보를 포함합니다.
    value: dict,  # 생성되거나 접근되는 리소스
):
    """리소스를 생성자에게만 비공개로 만듭니다."""
    # 예시:
    # ctx: AuthContext(
    #     permissions=[],
    #     user=ProxyUser(
    #         identity='user1',
    #         is_authenticated=True,
    #         display_name='user1'
    #     ),
    #     resource='threads',
    #     action='create_run'
    # )
    # value: 
    # {
    #     'thread_id': UUID('1e1b2733-303f-4dcd-9620-02d370287d72'),
    #     'assistant_id': UUID('fe096781-5601-53d2-b2f6-0d3403f7e9ca'),
    #     'run_id': UUID('1efbe268-1627-66d4-aa8d-b956b0f02a41'),
    #     'status': 'pending',
    #     'metadata': {},
    #     'prevent_insert_if_inflight': True,
    #     'multitask_strategy': 'reject',
    #     'if_not_exists': 'reject',
    #     'after_seconds': 0,
    #     'kwargs': {
    #         'input': {'messages': [{'role': 'user', 'content': '안녕하세요!'}]},
    #         'command': None,
    #         'config': {
    #             'configurable': {
    #                 'langgraph_auth_user': ... 사용자 객체...
    #                 'langgraph_auth_user_id': 'user1'
    #             }
    #         },
    #         'stream_mode': ['values'],
    #         'interrupt_before': None,
    #         'interrupt_after': None,
    #         'webhook': None,
    #         'feedback_keys': None,
    #         'temporary': False,
    #         'subgraphs': False
    #     }
    # }

    # 두 가지 작업을 수행합니다:
    # 1. 사용자의 ID를 리소스의 메타데이터에 추가합니다. 각 LangGraph 리소스에는 리소스와 함께 지속되는 `metadata` dict가 있습니다.
    # 이 메타데이터는 읽기 및 업데이트 작업에서 필터링하는 데 유용합니다.
    # 2. 사용자가 자신의 리소스만 볼 수 있도록 필터를 반환합니다.
    filters = {"owner": ctx.user.identity}
    metadata = value.setdefault("metadata", {})
    metadata.update(filters)

    # 사용자가 자신의 리소스만 볼 수 있도록 합니다.
    return filters
```

핸들러는 두 개의 매개변수를 받습니다:

1. `ctx` ([AuthContext](../../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.auth.types.AuthContext)): 현재 `user`, 사용자의 `permissions`, `resource` ("threads", "crons", "assistants") 및 수행 중인 `action` ("create", "read", "update", "delete", "search", "create_run")에 대한 정보를 포함합니다.
2. `value` (`dict`): 생성되거나 접근되고 있는 데이터. 이 dict의 내용은 접근 중인 리소스와 작업에 따라 다릅니다. 더 세밀한 범위의 접근 제어에 대한 정보는 [스코프가 있는 인증 핸들러 추가](#scoped-authorization)에서 확인하세요.

우리의 간단한 핸들러는 두 가지 작업을 수행합니다:

1. 사용자의 ID를 리소스의 메타데이터에 추가합니다.
2. 사용자가 자신이 소유한 리소스만 볼 수 있도록 메타데이터 필터를 반환합니다.

## 개인 대화 테스트하기

우리의 인증을 테스트해봅시다. 모든 ✅ 메시지를 볼 수 있어야 합니다. 개발 서버가 실행 중인지 확인하세요 ( `langgraph dev` 명령을 실행하세요):

```python
from langgraph_sdk import get_client

# 두 사용자를 위한 클라이언트 생성
alice = get_client(
    url="http://localhost:2024",
    headers={"Authorization": "Bearer user1-token"}
)

bob = get_client(
    url="http://localhost:2024",
    headers={"Authorization": "Bearer user2-token"}
)

# 앨리스가 비서 생성
alice_assistant = await alice.assistants.create()
print(f"✅ 앨리스가 비서 생성: {alice_assistant['assistant_id']}")

# 앨리스가 스레드 생성 및 채팅
alice_thread = await alice.threads.create()
print(f"✅ 앨리스가 스레드 생성: {alice_thread['thread_id']}")

await alice.runs.create(
    thread_id=alice_thread["thread_id"],
    assistant_id="agent",
    input={"messages": [{"role": "user", "content": "안녕하세요, 이것은 앨리스의 개인 채팅입니다"}]}
)

# 밥이 앨리스의 스레드에 접근 시도
try:
    await bob.threads.get(alice_thread["thread_id"])
    print("❌ 밥은 앨리스의 스레드를 봐서는 안됩니다!")
except Exception as e:
    print("✅ 밥의 접근이 올바르게 거부됨:", e)

# 밥이 자신의 스레드 생성
bob_thread = await bob.threads.create()
await bob.runs.create(
    thread_id=bob_thread["thread_id"],
    assistant_id="agent",
    input={"messages": [{"role": "user", "content": "안녕하세요, 이것은 밥의 개인 채팅입니다"}]}
)
print(f"✅ 밥이 자신의 스레드 생성: {bob_thread['thread_id']}")

# 스레드 목록 - 각 사용자는 자신의 스레드만 본다
alice_threads = await alice.threads.search()
bob_threads = await bob.threads.search()
print(f"✅ 앨리스는 {len(alice_threads)} 스레드를 봅니다")
print(f"✅ 밥은 {len(bob_threads)} 스레드를 봅니다")
```

테스트 코드를 실행하면 다음과 같은 출력이 표시됩니다:

```bash
✅ 앨리스가 비서 생성: fc50fb08-78da-45a9-93cc-1d3928a3fc37
✅ 앨리스가 스레드 생성: 533179b7-05bc-4d48-b47a-a83cbdb5781d
✅ 밥의 접근이 올바르게 거부됨: 클라이언트 오류 '404 찾을 수 없음' URL 'http://localhost:2024/threads/533179b7-05bc-4d48-b47a-a83cbdb5781d'
자세한 정보는 다음을 참조하세요: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404
✅ 밥이 자신의 스레드 생성: 437c36ed-dd45-4a1e-b484-28ba6eca8819
✅ 앨리스는 1 스레드를 봅니다
✅ 밥은 1 스레드를 봅니다
```

이는 다음을 의미합니다:

1. 각 사용자는 자신의 스레드를 생성하고 채팅할 수 있습니다.
2. 사용자는 서로의 스레드를 볼 수 없습니다.
3. 스레드 목록은 자신의 것만 표시됩니다.

## 범위가 지정된 권한 처리 추가 {#scoped-authorization}

광범위한 `@auth.on` 핸들러는 모든 [인증 이벤트](../../concepts/auth.md#authorization-events)에 해당합니다. 이는 간결하지만 `value` 딕셔너리의 내용이 잘 범위가 지정되지 않으며, 모든 리소스에 대해 동일한 사용자 수준의 접근 제어가 적용됨을 의미합니다. 보다 세분화된 접근 제어를 원한다면 특정 리소스에 대한 특정 작업을 제어할 수도 있습니다.

특정 리소스 유형에 대한 핸들러를 추가하기 위해 `src/security/auth.py`를 업데이트하세요:

```python
# 이전 핸들러 유지...

from langgraph_sdk import Auth

@auth.on.threads.create
async def on_thread_create(
    ctx: Auth.types.AuthContext,
    value: Auth.types.on.threads.create.value,
):
    """스레드를 생성할 때 소유자를 추가합니다.
    
    이 핸들러는 새로운 스레드를 생성할 때 실행되며 두 가지를 수행합니다:
    1. 소유권을 추적하기 위해 생성 중인 스레드에 메타데이터를 설정합니다.
    2. 생성자만 접근할 수 있도록 보장하는 필터를 반환합니다.
    """
    # 예제 값:
    #  {'thread_id': UUID('99b045bc-b90b-41a8-b882-dabc541cf740'), 'metadata': {}, 'if_exists': 'raise'}

    # 생성 중인 스레드에 소유자 메타데이터 추가
    # 이 메타데이터는 스레드와 함께 저장되고 지속됩니다.
    metadata = value.setdefault("metadata", {})
    metadata["owner"] = ctx.user.identity
    
    
    # 생성자만 접근할 수 있도록 제한하는 필터 반환
    return {"owner": ctx.user.identity}

@auth.on.threads.read
async def on_thread_read(
    ctx: Auth.types.AuthContext,
    value: Auth.types.on.threads.read.value,
):
    """사용자가 자신의 스레드만 읽을 수 있도록 합니다.
    
    이 핸들러는 읽기 작업에서 실행됩니다. 스레드는 이미 존재하므로
    메타데이터를 설정할 필요는 없으며, 사용자가 자신의 스레드만 볼 수 있도록
    필터를 반환하면 됩니다.
    """
    return {"owner": ctx.user.identity}

@auth.on.assistants
async def on_assistants(
    ctx: Auth.types.AuthContext,
    value: Auth.types.on.assistants.value,
):
    # 설명을 위해 모든 요청을 거부합니다.
    # 어시스턴트 리소스에 관련된 요청
    # 예제 값:
    # {
    #     'assistant_id': UUID('63ba56c3-b074-4212-96e2-cc333bbc4eb4'),
    #     'graph_id': 'agent',
    #     'config': {},
    #     'metadata': {},
    #     'name': 'Untitled'
    # }
    raise Auth.exceptions.HTTPException(
        status_code=403,
        detail="사용자에게 필요한 권한이 없습니다.",
    )

# 사용자 정보를 (user_id, resource_type, resource_id)와 같은 형식으로 저장한다고 가정
@auth.on.store()
async def authorize_store(ctx: Auth.types.AuthContext, value: dict):
    # 각 스토어 항목의 "namespace" 필드는 항목의 디렉토리처럼 생각할 수 있는 튜플입니다.
    namespace: tuple = value["namespace"]
    assert namespace[0] == ctx.user.identity, "권한이 없습니다."

```

위 코드에서 하나의 전역 핸들러 대신, 이제 각각의 리소스에 대한 특정 핸들러가 있습니다:

1. 스레드 생성
2. 스레드 읽기
3. 어시스턴트 접근

위 세 가지는 각각의 리소스에 대한 특정 **동작**에 해당하며 (자세한 내용은 [리소스 동작](../../concepts/auth.md#resource-actions) 참조), 마지막 하나(`@auth.on.assistants`)는 `assistants` 리소스의 _모든_ 동작에 해당합니다. 각 요청에 대해 LangGraph는 접근하려는 리소스 및 동작에 가장 구체적인 핸들러를 실행합니다. 이는 네 개의 핸들러가 실행되며, 광범위하게 범위가 지정된 "`@auth.on`" 핸들러는 실행되지 않음을 의미합니다.

테스트 파일에 다음 테스트 코드를 추가하세요:

```python
# ... 이전과 동일
# 어시스턴트 생성 시도. 이 작업이 실패해야 합니다.
try:
    await alice.assistants.create("agent")
    print("❌ 앨리스는 어시스턴트를 생성할 수 없습니다!")
except Exception as e:
    print("✅ 앨리스가 접근이 거부되었습니다:", e)

# 어시스턴트 검색 시도. 이것도 실패해야 합니다.
try:
    await alice.assistants.search()
    print("❌ 앨리스는 어시스턴트를 검색할 수 없습니다!")
except Exception as e:
    print("✅ 앨리스가 어시스턴트 검색에 대한 접근이 거부되었습니다:", e)

# 앨리스는 여전히 스레드를 생성할 수 있습니다.
alice_thread = await alice.threads.create()
print(f"✅ 앨리스가 스레드를 생성했습니다: {alice_thread['thread_id']}")
```

그리고 테스트 코드를 다시 실행합니다:

```bash
✅ 앨리스가 스레드를 생성했습니다: dcea5cd8-eb70-4a01-a4b6-643b14e8f754
✅ 밥이 접근을 올바르게 거부했습니다: 클라이언트 오류 '404 Not Found' URL 'http://localhost:2024/threads/dcea5cd8-eb70-4a01-a4b6-643b14e8f754'에 대한
자세한 정보는 다음 링크를 확인하십시오: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404
✅ 밥이 자신의 스레드를 생성했습니다: 400f8d41-e946-429f-8f93-4fe395bc3eed
✅ 앨리스는 1개의 스레드를 봅니다
✅ 밥은 1개의 스레드를 봅니다
✅ 앨리스가 접근을 올바르게 거부했습니다:
자세한 정보는 다음 링크를 확인하십시오: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500
✅ 앨리스가 검색 도우미에 대한 접근을 올바르게 거부했습니다:
```

축하합니다! 여러분은 각 사용자가 자신의 개인 대화를 가지는 챗봇을 만들었습니다. 이 시스템은 간단한 토큰 기반 인증을 사용하고 있지만, 우리가 배운 인증 패턴은 실제 인증 시스템 구현에도 적용될 것입니다. 다음 튜토리얼에서는 테스트 사용자를 실제 사용자 계정으로 대체하여 OAuth2를 사용할 것입니다.

## 다음 단계는 무엇인가요?

이제 리소스에 대한 접근을 제어할 수 있으므로, 다음으로 진행할 수 있습니다:

1. 실제 사용자 계정을 추가하기 위해 [프로덕션 인증](add_auth_server.md)으로 이동하기
2. [인증 패턴](../../concepts/auth.md#authorization)에 대해 더 읽기
3. 이 튜토리얼에서 사용된 인터페이스 및 메서드에 대한 자세한 정보는 [API 레퍼런스](../../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.auth.Auth) 확인하기
