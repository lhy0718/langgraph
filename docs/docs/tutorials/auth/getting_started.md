_한국어로 기계번역됨_

# 사용자 정의 인증 설정 (1부/3부)

!!! 주의 "인증 시리즈의 1부입니다:"

    1. 기본 인증 (여기에 있음) - 봇에 대한 접근 권한을 제어합니다
    2. [자원 권한 부여](resource_auth.md) - 사용자들이 비공식적인 대화를 나눌 수 있도록 합니다
    3. [프로덕션 인증](add_auth_server.md) - 실제 사용자 계정을 추가하고 OAuth2를 사용하여 검증합니다

!!! 팁 "전제 조건"

    이 가이드는 다음 개념에 대한 기본적인 이해를 가정합니다:

      *  [**인증 및 접근 제어**](../../concepts/auth.md)
      *  [**LangGraph 플랫폼**](../../concepts/index.md#langgraph-platform)

!!! 주의 "파이썬 전용"

    현재 우리는 `langgraph-api>=0.0.11`와 함께 파이썬 배포의 사용자 정의 인증 및 권한 부여만 지원합니다. LangGraph.JS에 대한 지원도 곧 추가될 예정입니다.


???+ 주의 "배포 유형별 지원"

    사용자 정의 인증은 **관리되는 LangGraph 클라우드**의 모든 배포와 **기업** 셀프 호스팅 플랜에서 지원됩니다. **라이트** 셀프 호스팅 플랜에서는 지원되지 않습니다.

이번 튜토리얼에서는 특정 사용자만 액세스할 수 있는 챗봇을 구축할 것입니다. LangGraph 템플릿을 시작으로 토큰 기반 보안을 단계별로 추가할 것입니다. 최종적으로는 유효한 토큰을 확인한 후에만 접근을 허용하는 작동하는 챗봇을 갖게 될 것입니다.

## 프로젝트 설정

먼저 LangGraph 스타터 템플릿을 사용하여 새 챗봇을 생성해 봅시다:

```bash
pip install -U "langgraph-cli[inmem]"
langgraph new --template=new-langgraph-project-python custom-auth
cd custom-auth
```

이 템플릿은 LangGraph 앱의 자리 표시자를 제공합니다. 로컬 의존성을 설치하고 개발 서버를 실행하여 이를 시험해 봅시다.
```shell
pip install -e .
langgraph dev
```
모든 것이 잘 작동하면 서버가 시작되고 브라우저에서 스튜디오가 열릴 것입니다.

> - 🚀 API: http://127.0.0.1:2024
> - 🎨 스튜디오 UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
> - 📚 API 문서: http://127.0.0.1:2024/docs
> 
> 이 인메모리 서버는 개발 및 테스트를 위해 설계되었습니다.
> 운영 목적으로는 LangGraph 클라우드를 사용하시기 바랍니다.

그래프가 실행되어야 하며, 만약 이를 공개 인터넷에 셀프 호스팅 한다면 누구나 접근할 수 있을 것입니다!

![No auth](./img/no_auth.png)

이제 기본 LangGraph 앱을 살펴보았으니 이제 인증을 추가해 봅시다! 

???+ 팁 "자리 표시자 토큰"
    
    1부에서는 설명을 위해 하드코딩된 토큰부터 시작합니다.
    기본을 마스터한 후 3부에서 "프로덕션 준비 완료" 인증 체계로 넘어갑니다.


## 인증 추가하기

[`Auth`](../../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.auth.Auth) 객체를 사용하면 LangGraph 플랫폼이 모든 요청에서 실행할 인증 함수를 등록할 수 있습니다. 이 함수는 각 요청을 수신하고 수락할지 거부할지를 결정합니다.

`src/security/auth.py`라는 새 파일을 생성합니다. 이곳에서 사용자가 봇에 접근할 수 있는지 확인하는 코드가 작성됩니다:

```python hl_lines="10 15-16" title="src/security/auth.py"
from langgraph_sdk import Auth

# 이것은 우리의 장난감 사용자 데이터베이스입니다. 운영 환경에서 이렇게 하지 마십시오.
VALID_TOKENS = {
    "user1-token": {"id": "user1", "name": "Alice"},
    "user2-token": {"id": "user2", "name": "Bob"},
}

# "Auth" 객체는 LangGraph가 우리의 인증 함수를 표시하는 데 사용할 컨테이너입니다.
auth = Auth()


# `authenticate` 데코레이터는 LangGraph가 이 함수를 미들웨어로 호출하도록 지시합니다.
# 이는 요청의 허용 여부를 결정하게 됩니다.
@auth.authenticate
async def get_current_user(authorization: str | None) -> Auth.types.MinimalUserDict:
    """사용자의 토큰이 유효한지 확인합니다."""
    assert authorization
    scheme, token = authorization.split()
    assert scheme.lower() == "bearer"
    # 토큰이 유효한지 확인합니다.
    if token not in VALID_TOKENS:
        raise Auth.exceptions.HTTPException(status_code=401, detail="유효하지 않은 토큰")

    # 유효할 경우 사용자 정보를 반환합니다.
    user_data = VALID_TOKENS[token]
    return {
        "identity": user_data["id"],
    }
```

우리의 [인증](../../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.auth.Auth.authenticate) 핸들러가 두 가지 중요한 일을 수행한다는 점에 유의하십시오:

1. 요청의 [Authorization 헤더](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)에 유효한 토큰이 제공되는지 확인합니다.
2. 사용자의 [신원](../../cloud/reference/sdk/python_sdk_ref.md#langgraph_sdk.auth.types.MinimalUserDict)을 반환합니다.

이제 LangGraph에 인증을 사용하도록 하려면 [`langgraph.json`](../../cloud/reference/cli.md#configuration-file) 구성 파일에 다음 내용을 추가하십시오:

```json hl_lines="7-9" title="langgraph.json"
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent/graph.py:graph"
  },
  "env": ".env",
  "auth": {
    "path": "src/security/auth.py:auth"
  }
}
```

## "보안" 봇 테스트하기

모든 것을 테스트하기 위해 서버를 다시 시작합시다!

```bash
langgraph dev --no-browser
```

??? 노트 "스튜디오에서의 사용자 정의 인증"

    만약 `--no-browser`를 추가하지 않았다면, 스튜디오 UI가 브라우저에서 열릴 것입니다. 스튜디오가 여전히 우리의 서버에 연결할 수 있는 이유는 무엇일까요? 기본적으로, 우리는 사용자 정의 인증을 사용할 경우에도 LangGraph 스튜디오의 접근을 허용합니다. 이는 스튜디오에서 봇을 쉽게 개발하고 테스트할 수 있게 합니다. 인증 구성에서 `disable_studio_auth: "true"`로 설정하여 이 대체 인증 옵션을 제거할 수 있습니다:
    ```json
    {
        "auth": {
            "path": "src/security/auth.py:auth",
            "disable_studio_auth": "true"
        }
    }
    ```

이제 우리 봇과 대화해 봅시다. 만약 우리가 인증을 올바르게 구현했다면, 요청 헤더에 유효한 토큰을 제공해야만 봇에 접근할 수 있어야 합니다. 그러나 사용자는 [리소스 권한 부여 핸들러](../../concepts/auth.md#resource-authorization)를 다음 섹션에서 추가할 때까지 서로의 리소스에 계속 접근할 수 있습니다.

![인증, 권한 부여 핸들러 없음](./img/authentication.png)

파일이나 노트북에서 다음 코드를 실행하십시오:

```python
from langgraph_sdk import get_client

# 토큰 없이 시도하기 (실패해야 함)
client = get_client(url="http://localhost:2024")
try:
    thread = await client.threads.create()
    print("❌ 토큰 없이 실패했어야 했습니다!")
except Exception as e:
    print("✅ 접근 차단이 올바르게 이루어졌습니다:", e)

# 유효한 토큰으로 시도하기
client = get_client(
    url="http://localhost:2024", headers={"Authorization": "Bearer user1-token"}
)

# 스레드 생성 및 채팅
thread = await client.threads.create()
print(f"✅ 앨리스로서 스레드를 생성했습니다: {thread['thread_id']}")

response = await client.runs.create(
    thread_id=thread["thread_id"],
    assistant_id="agent",
    input={"messages": [{"role": "user", "content": "안녕하세요!"}]},
)
print("✅ 봇이 응답했습니다:")
print(response)
```

다음과 같은 결과를 보아야 합니다:

1. 유효한 토큰 없이 봇에 접근할 수 없습니다.
2. 유효한 토큰으로 스레드를 만들고 채팅할 수 있습니다.

축하합니다! "인증된" 사용자만 접근할 수 있는 챗봇을 만들었습니다. 이 시스템은 (아직) 운영 준비가 된 보안 체계를 구현하지는 않았지만, 봇에 대한 접근을 제어하는 기본 메커니즘을 배웠습니다. 다음 튜토리얼에서는 각 사용자에게 개인적인 대화를 제공하는 방법에 대해 배울 것입니다.

## 다음 단계는 무엇인가요?

이제 봇에 접근할 수 있는 사람을 제어할 수 있게 되었으니, 다음을 원할 수 있습니다:

1. [대화를 비공개로 만드는 것 (2/3부)](resource_auth.md) 튜토리얼로 계속 진행하여 리소스 권한 부여에 대해 배우기.
2. [인증 개념](../../concepts/auth.md)에 대해 더 읽기.
3. [API 참조](../../cloud/reference/sdk/python_sdk_ref.md)를 확인하여 더 많은 인증 정보를 얻기.