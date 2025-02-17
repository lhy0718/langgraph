_한국어로 기계번역됨_

# LangGraph 배포에 의미 검색 추가하는 방법

이 가이드는 의미 검색을 LangGraph 배포의 스레드 간 [저장소](../../concepts/persistence.md#memory-store)에 추가하는 방법을 설명합니다. 이를 통해 에이전트가 기억과 다른 문서를 의미적 유사성으로 검색할 수 있습니다.

## 전제 조건

- LangGraph 배포가 필요합니다(배포하는 방법은 [여기](setup_pyproject.md)를 참조).
- 임베딩 제공자의 API 키가 필요합니다(이 경우 OpenAI).
- `langchain >= 0.3.8` (아래 문자열 형식을 지정할 경우)

## 단계

1. `langgraph.json` 구성 파일을 업데이트하여 저장소 구성을 포함합니다:

```json
{
    ...
    "store": {
        "index": {
            "embed": "openai:text-embeddings-3-small",
            "dims": 1536,
            "fields": ["$"]
        }
    }
}
```

이 구성의 의미:

- OpenAI의 text-embeddings-3-small 모델을 사용하여 임베딩을 생성합니다.
- 임베딩 차원을 1536으로 설정합니다(모델의 출력에 맞춤).
- 저장된 데이터의 모든 필드를 인덱싱합니다(`["$"]`는 모든 것을 인덱싱함을 의미하며, 특정 필드를 지정하려면 `["text", "metadata.title"]`와 같이 합니다).

2. 위의 문자열 임베딩 형식을 사용하려면 종속성이 `langchain >= 0.3.8`이 포함되어 있는지 확인합니다:

```toml
# pyproject.toml에서
[project]
dependencies = [
    "langchain>=0.3.8"
]
```

또는 requirements.txt를 사용하는 경우:

```
langchain>=0.3.8
```

## 사용법

구성이 완료되면 LangGraph 노드에서 의미 검색을 사용할 수 있습니다. 저장소는 기억을 구성하기 위한 네임스페이스 튜플이 필요합니다:

```python
def search_memory(state: State, *, store: BaseStore):
    # 의미적 유사성을 사용하여 저장소 검색
    # 네임스페이스 튜플은 서로 다른 유형의 기억을 조직하는 데 도움을 줍니다.
    # 예: ("user_facts", "preferences") 또는 ("conversation", "summaries")
    results = store.search(
        namespace=("memory", "facts"),  # 유형별로 기억을 조직
        query="검색 쿼리",
        limit=3  # 반환할 결과 수
    )
    return results
```

## 사용자 지정 임베딩

사용자 지정 임베딩을 사용하고 싶다면 사용자 지정 임베딩 함수에 대한 경로를 전달할 수 있습니다:

```json
{
    ...
    "store": {
        "index": {
            "embed": "path/to/embedding_function.py:embed",
            "dims": 1536,
            "fields": ["$"]
        }
    }
}
```

배포는 지정된 경로에서 함수를 찾습니다. 이 함수는 비동기 함수여야 하며 문자열 목록을 받아야 합니다:

```python
# path/to/embedding_function.py
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def aembed_texts(texts: list[str]) -> list[list[float]]:
    """비동기 사용자 정의 임베딩 함수는 다음과 같은 조건을 만족해야 합니다:
    1. 비동기여야 한다
    2. 문자열 리스트를 받아야 한다
    3. 부동소수점 배열(임베딩)의 리스트를 반환해야 한다
    """
    response = await client.embeddings.create(
        model="text-embedding-3-small",
        input=texts
    )
    return [e.embedding for e in response.data]
```

## API를 통한 조회

LangGraph SDK를 사용하여 저장소를 조회할 수도 있습니다. SDK는 비동기 작업을 사용하므로:

```python
from langgraph_sdk import get_client

async def search_store():
    client = get_client()
    results = await client.store.search_items(
        ("memory", "facts"),
        query="검색 쿼리 입력",
        limit=3  # 반환할 결과 수
    )
    return results

# 비동기 컨텍스트에서 사용
results = await search_store()
```
