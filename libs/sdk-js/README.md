_한국어로 기계번역됨_

# LangGraph JS/TS SDK

이 리포지토리는 LangGraph REST API와 상호 작용하기 위한 JS/TS SDK를 포함하고 있습니다.

## 빠른 시작

JS/TS SDK를 시작하려면 [패키지를 설치](https://www.npmjs.com/package/@langchain/langgraph-sdk)하세요.

```bash
yarn add @langchain/langgraph-sdk
```

동작 중인 LangGraph API 서버가 필요합니다. `langgraph-cli`를 사용하여 로컬에서 서버를 실행 중이라면, SDK는 자동으로 `http://localhost:8123`를 가리킵니다. 그렇지 않으면 클라이언트를 생성할 때 서버 URL을 지정해야 합니다.

```js
import { Client } from "@langchain/langgraph-sdk";

const client = new Client();

// 모든 어시스턴트 목록 검색
const assistants = await client.assistants.search({
  metadata: null,
  offset: 0,
  limit: 10,
});

// 구성에서 등록된 각 그래프에 대해 어시스턴트를 자동 생성합니다.
const agent = assistants[0];

// 새로운 스레드 시작
const thread = await client.threads.create();

// 스트리밍 실행 시작
const messages = [{ role: "human", content: "라의 날씨는 어때요?" }];

const streamResponse = client.runs.stream(
  thread["thread_id"],
  agent["assistant_id"],
  {
    input: { messages },
  }
);

for await (const chunk of streamResponse) {
  console.log(chunk);
}
```

## 문서화

문서를 생성하려면 다음 명령어를 실행하세요:

1. 문서 생성.

        yarn typedoc

2. 문서 파일을 하나의 마크다운 파일로 통합.

        npx concat-md --decrease-title-levels --ignore=js_ts_sdk_ref.md --start-title-level-at 2 docs > docs/js_ts_sdk_ref.md

3. `js_ts_sdk_ref.md`를 MkDocs 디렉토리로 복사.

        cp docs/js_ts_sdk_ref.md ../../docs/docs/cloud/reference/sdk/js_ts_sdk_ref.md
