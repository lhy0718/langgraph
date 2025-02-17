_한국어로 기계번역됨_

# API 참조

LangGraph Cloud API 참조는 각 배포에 대해 `/docs` URL 경로에서 사용할 수 있습니다 (예: `http://localhost:8124/docs`).

<a href="/langgraph/cloud/reference/api/api_ref.html" target="_blank">여기</a>를 클릭하여 API 참조를 확인하세요.

## 인증

LangGraph Cloud에 배포할 경우 인증이 필요합니다. LangGraph Cloud API에 대한 각 요청에 `X-Api-Key` 헤더를 전달하세요. 헤더의 값은 API가 배포된 조직의 유효한 LangSmith API 키로 설정해야 합니다.

예제 `curl` 명령어:
```shell
curl --request POST \
  --url http://localhost:8124/assistants/search \
  --header 'Content-Type: application/json' \
  --header 'X-Api-Key: LANGSMITH_API_KEY' \
  --data '{
  "metadata": {},
  "limit": 10,
  "offset": 0
}'  
```
