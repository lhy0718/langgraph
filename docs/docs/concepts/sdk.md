_한국어로 기계번역됨_

# LangGraph SDK

!!! 정보 "사전 요구 사항"
    - [LangGraph 플랫폼](./langgraph_platform.md)
    - [LangGraph 서버](./langgraph_server.md)

LangGraph 플랫폼은 [LangGraph 서버 API](./langgraph_server.md)와 상호작용하기 위한 Python 및 JS SDK를 제공합니다.

## 설치

적절한 패키지 관리자를 사용하여 패키지를 설치할 수 있습니다.

=== "Python"
    ```bash
    pip install langgraph-sdk
    ```

=== "JS"
    ```bash
    yarn add @langchain/langgraph-sdk
    ```

## API 참조

SDK의 API 참조를 여기에서 찾을 수 있습니다:

- [Python SDK 참조](../cloud/reference/sdk/python_sdk_ref.md)
- [JS/TS SDK 참조](../cloud/reference/sdk/js_ts_sdk_ref.md)

## Python 동기 vs. 비동기

Python SDK는 LangGraph 서버 API와 상호작용하기 위한 동기(`get_sync_client`) 및 비동기(`get_client`) 클라이언트를 제공합니다.

=== "비동기"
    ```python
    from langgraph_sdk import get_client

    client = get_client(url=..., api_key=...)
    await client.assistants.search()
    ```

=== "동기"

    ```python
    from langgraph_sdk import get_sync_client

    client = get_sync_client(url=..., api_key=...)
    client.assistants.search()
    ```

## 관련 자료

- [LangGraph CLI API 참조](../cloud/reference/cli.md)
- [Python SDK 참조](../cloud/reference/sdk/python_sdk_ref.md)
- [JS/TS SDK 참조](../cloud/reference/sdk/js_ts_sdk_ref.md)