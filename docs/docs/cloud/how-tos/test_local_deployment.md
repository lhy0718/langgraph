_한국어로 기계번역됨_

# 로컬 배포가 포함된 LangGraph 스튜디오

!!! 경고 "브라우저 호환성"
    로컬 LangGraph 배포의 스튜디오 페이지는 Safari에서 작동하지 않습니다. 대신 Chrome을 사용하세요.

## 설정

올바르게 앱을 설정했는지 확인하세요. 컴파일된 그래프, 환경 변수를 포함하는 `.env` 파일, 환경 파일과 컴파일된 그래프를 가리키는 `langgraph.json` 구성 파일을 생성해야 합니다. 보다 자세한 지침은 [여기](https://langchain-ai.github.io/langgraph/cloud/deployment/setup/)를 참조하세요.

앱 설정이 완료되면 `langgraph.json` 파일이 있는 디렉토리로 이동하여 `langgraph dev`를 호출하여 코드 변경 시 재시작되는 워치 모드에서 API 서버를 시작하세요. 이렇게 하면 로컬 테스트에 이상적입니다. API 서버가 올바르게 시작되면 다음과 유사한 로그가 표시됩니다:

>    준비 완료!
> 
>    - API: [http://localhost:2024](http://localhost:2024/)
>     
>    - 문서: http://localhost:2024/docs
>     
>    - LangGraph 스튜디오 웹 UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024

API 서버를 시작하기 위한 모든 옵션에 대한 내용을 보려면 이 [참조](https://langchain-ai.github.io/langgraph/cloud/reference/cli/#up)를 읽으세요.

## 스튜디오 접근

API 서버를 성공적으로 시작하면 다음 URL로 이동하여 스튜디오에 접근할 수 있습니다: `https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024` (Safari를 사용하는 경우 위의 경고를 참조하세요).

모든 것이 제대로 작동하면 왼쪽에 그래프 다이어그램이 있는 스튜디오 화면이 다음과 유사하게 표시됩니다:

![LangGraph 스튜디오](./img/studio_screenshot.png)

## 테스트를 위한 스튜디오 사용

테스트를 위해 스튜디오를 사용하는 방법에 대해 알아보려면 [LangGraph 스튜디오 사용법](https://langchain-ai.github.io/langgraph/cloud/how-tos/#langgraph-studio)을 읽으세요.