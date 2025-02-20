_한국어로 기계번역됨_

# LangGraph Studio에서의 프롬프트 엔지니어링

LangGraph Studio에서는 LangSmith Playground를 활용하여 그래프 내에서 사용되는 프롬프트를 반복적으로 수정할 수 있습니다. 그렇게 하려면:

1. 기존 스레드를 열거나 새로 만듭니다.
2. 스레드 로그 내에서 LLM 호출을 수행한 모든 노드에는 "View LLM Runs" 버튼이 있습니다. 이를 클릭하면 해당 노드의 LLM 실행 결과가 나타나는 팝오버가 열립니다.
3. 수정하려는 LLM 실행 결과를 선택합니다. 그러면 선택한 LLM 실행 결과와 함께 LangSmith Playground가 열립니다.

![스튜디오의 플레이그라운드](../img/studio_playground.png){width=1200}

여기에서 프롬프트를 수정하고, 다양한 모델 구성으로 테스트하며, 전체 그래프를 다시 실행하지 않고도 이 LLM 호출만 다시 실행할 수 있습니다. 변경 사항에 만족하면 업데이트된 프롬프트를 그래프에 다시 복사할 수 있습니다.

LangSmith Playground 사용 방법에 대한 자세한 내용은 [LangSmith Playground 문서](https://docs.smith.langchain.com/prompt_engineering/how_to_guides#playground)를 참조하세요.
