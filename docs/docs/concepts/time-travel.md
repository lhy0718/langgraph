_한국어로 기계번역됨_

# 시간 여행 ⏱️

!!! 참고 "전제 조건"

    이 가이드는 LangGraph의 체크포인트와 상태에 익숙하다고 가정합니다. 그렇지 않으시다면, 먼저 [지속성](./persistence.md) 개념을 검토해 주시기 바랍니다.


모델 기반 결정을 내리는 비결정론적 시스템(예: LLM으로 구동되는 에이전트)과 작업할 때, 그들의 의사결정 과정의 세부 사항을 살펴보는 것이 유용할 수 있습니다:

1. 🤔 **추론 이해**: 성공적인 결과로 이어진 단계를 분석합니다.
2. 🐞 **오류 디버깅**: 오류가 발생한 위치와 이유를 식별합니다.
3. 🔍 **대안 탐색**: 더 나은 솔루션을 찾기 위해 다양한 경로를 테스트합니다.

이러한 디버깅 기법을 **시간 여행**이라고 부르며, 두 가지 주요 작업인 [**재생**](#replaying) 🔁과 [**포크**](#forking) 🔀로 구성됩니다.

## 재생

![](./img/human_in_the_loop/replay.png)

재생은 특정 단계(체크포인트)까지 에이전트의 이전 행동을 다시 방문하고 재현할 수 있게 해줍니다.

특정 체크포인트 이전의 행동을 재생하려면, 먼저 스레드에 대한 모든 체크포인트를 검색합니다:

```python
all_checkpoints = []
for state in graph.get_state_history(thread):
    all_checkpoints.append(state)
```

각 체크포인트에는 고유한 ID가 있습니다. 원하는 체크포인트, 예를 들어 `xyz`를 확인한 후, 해당 ID를 구성에 포함시킵니다:

```python
config = {'configurable': {'thread_id': '1', 'checkpoint_id': 'xyz'}}
for event in graph.stream(None, config, stream_mode="values"):
    print(event)
```

그래프는 제공된 `checkpoint_id` _이전_에 실행된 이전 단계를 재생하고, `checkpoint_id` _이후_의 단계를 실행합니다(즉, 새로운 포크를 생성합니다). 이전에 실행된 경우에도 마찬가지입니다.

## 포크

![](./img/human_in_the_loop/forking.png)

포크는 에이전트의 이전 행동을 다시 방문하고 그래프 내에서 대안 경로를 탐색할 수 있게 해줍니다.

특정 체크포인트, 예를 들어 `xyz`를 수정하려면, 그래프의 상태를 업데이트할 때 그 `checkpoint_id`를 제공해야 합니다:

```python
config = {"configurable": {"thread_id": "1", "checkpoint_id": "xyz"}}
graph.update_state(config, {"state": "updated state"})
```

이렇게 하면 xyz-fork라는 새로운 포크된 체크포인트가 생성되며, 여기서 그래프를 계속 실행할 수 있습니다:

```python
config = {'configurable': {'thread_id': '1', 'checkpoint_id': 'xyz-fork'}}
for event in graph.stream(None, config, stream_mode="values"):
    print(event)
```

## 추가 자료 📚

- [**개념적 가이드: 지속성**](https://langchain-ai.github.io/langgraph/concepts/persistence/#replay): 재생에 대한 더 많은 내용을 알고 싶다면 지속성 가이드를 읽어보세요.
- [**과거 그래프 상태 보기 및 업데이트하기**](../how-tos/human_in_the_loop/time-travel.ipynb): **재생** 및 **포크** 작업을 시연하여 그래프 상태 작업에 대한 단계별 지침입니다.
