_한국어로 기계번역됨_

# LangGraph 파이썬 SDK

이 저장소는 LangGraph Cloud REST API와 상호작용하기 위한 파이썬 SDK를 포함하고 있습니다.

## 빠른 시작

파이썬 SDK를 시작하려면 [패키지를 설치하세요](https://pypi.org/project/langgraph-sdk/).

```bash
pip install -U langgraph-sdk
```

LangGraph API 서버가 실행 중이어야 합니다. `langgraph-cli`를 사용하여 로컬에서 서버를 실행하는 경우, SDK는 자동으로 `http://localhost:8123`를 가리키며, 그렇지 않은 경우 클라이언트를 생성할 때 서버 URL을 지정해야 합니다.

```python
from langgraph_sdk import get_client

# 원격 서버를 사용 중인 경우 `get_client(url=REMOTE_URL)`로 클라이언트를 초기화하세요.
client = get_client()

# 모든 어시스턴트 목록 조회
assistants = await client.assistants.search()

# 등록한 각 그래프에 대해 어시스턴트를 자동으로 생성합니다.
agent = assistants[0]

# 새 스레드 시작
thread = await client.threads.create()

# 스트리밍 실행 시작
input = {"messages": [{"role": "human", "content": "로스앤젤레스의 날씨는 어때?"}]}
async for chunk in client.runs.stream(thread['thread_id'], agent['assistant_id'], input=input):
    print(chunk)
```
