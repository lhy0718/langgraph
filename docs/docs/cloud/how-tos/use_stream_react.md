_한국어로 기계번역됨_

# LangGraph를 React 애플리케이션에 통합하는 방법

!!! 정보 "전제 조건" - [LangGraph 플랫폼](../../concepts/langgraph_platform.md) - [LangGraph 서버](../../concepts/langgraph_server.md)

`useStream()` React 훅은 LangGraph를 React 애플리케이션에 통합하는 매끄러운 방법을 제공합니다. 이 훅은 스트리밍, 상태 관리 및 분기 논리에 대한 모든 복잡성을 처리하여 훌륭한 채팅 경험을 구축하는 데 집중할 수 있도록 해줍니다.

주요 기능:

- 메시지 스트리밍: 메시지 조각의 스트림을 처리하여 전체 메시지를 형성
- 메시지, 로딩 상태 및 오류에 대한 자동 상태 관리
- 대화 분기: 채팅 기록의 어느 지점에서든 대화의 대체 경로 생성
- UI 비의존적인 디자인 - 자신만의 구성 요소 및 스타일 사용 가능

이제 React 애플리케이션에서 `useStream()`을 사용하는 방법을 살펴보겠습니다.

`useStream()`은 맞춤형 채팅 경험을 만들기 위한 튼튼한 기초를 제공합니다. 사전 구축된 채팅 구성 요소 및 인터페이스에 대해서는 [CopilotKit](https://docs.copilotkit.ai/coagents/quickstart/langgraph) 및 [assistant-ui](https://www.assistant-ui.com/docs/runtimes/langgraph)를 확인하는 것이 좋습니다.

## 설치

```bash
npm install @langchain/langgraph-sdk @langchain/langchain-core react
```

## 예제

```tsx
"use client"

import { useStream } from "@langchain/langgraph-sdk/react"
import type { Message } from "@langchain/langgraph-sdk"

export default function App() {
  const thread = useStream<{ messages: Message[] }>({
    apiUrl: "http://localhost:2024",
    assistantId: "agent",
    messagesKey: "messages",
  })

  return (
    <div>
      <div>
        {thread.messages.map((message) => (
          <div key={message.id}>{message.content as string}</div>
        ))}
      </div>

      <form
        onSubmit={(e) => {
          e.preventDefault()

          const form = e.target as HTMLFormElement
          const message = new FormData(form).get("message") as string

          form.reset()
          thread.submit({ messages: [{ type: "human", content: message }] })
        }}
      >
        <input type="text" name="message" />

        {thread.isLoading ? (
          <button key="stop" type="button" onClick={() => thread.stop()}>
            중지
          </button>
        ) : (
          <button key="submit" type="submit">
            전송
          </button>
        )}
      </form>
    </div>
  )
}
```

## UI 사용자 정의

`useStream()` 훅은 복잡한 상태 관리를 비하인드에서 처리하며, UI를 구축하기 위한 간단한 인터페이스를 제공합니다. 기본적으로 제공되는 내용은 다음과 같습니다:

- 스레드 상태 관리
- 로딩 및 오류 상태
- 메시지 처리 및 업데이트
- 분기 지원

이 기능을 효과적으로 사용하는 방법에 대한 몇 가지 예는 다음과 같습니다:

### 로딩 상태

`isLoading` 속성은 스트림이 활성 상태일 때를 알려줍니다. 이를 통해 다음과 같은 작업을 수행할 수 있습니다:

- 로딩 표시기 표시
- 처리 중 입력 필드 비활성화
- 취소 버튼 표시

```tsx
export default function App() {
  const { isLoading, stop } = useStream<{ messages: Message[] }>({
    apiUrl: "http://localhost:2024",
    assistantId: "agent",
    messagesKey: "messages",
  })

  return (
    <form>
      {isLoading && (
        <button key="stop" type="button" onClick={() => stop()}>
          중지
        </button>
      )}
    </form>
  )
}
```

### 스레드 관리

내장된 스레드 관리 기능으로 대화를 추적하세요. 현재 스레드 ID에 접근하고 새 스레드가 생성될 때 알림을 받을 수 있습니다:

```tsx
const [threadId, setThreadId] = useState<string | null>(null)

const thread = useStream<{ messages: Message[] }>({
  apiUrl: "http://localhost:2024",
  assistantId: "agent",

  threadId: threadId,
  onThreadId: setThreadId,
})
```

사용자가 페이지 새로 고침 후 대화를 다시 시작할 수 있도록 하려면 `threadId`를 URL의 쿼리 매개변수에 저장하는 것을 권장합니다.

### 메시지 처리

메시지 처리를 활성화하려면 `useStream()` 훅에 `messagesKey` 옵션을 전달해야 합니다.

활성화된 상태에서 `useStream()` 훅은 서버에서 수신한 메시지 청크를 추적하고 이를 연결하여 전체 메시지를 형성합니다. 완성된 메시지 청크는 `messages` 속성을 통해 검색할 수 있습니다.

```tsx
import type { Message } from "@langchain/langgraph-sdk"
import { useStream } from "@langchain/langgraph-sdk/react"

export default function HomePage() {
  const thread = useStream<{ messages: Message[] }>({
    apiUrl: "http://localhost:2024",
    assistantId: "agent",
    messagesKey: "messages",
  })

  return (
    <div>
      {thread.messages.map((message) => (
        <div key={message.id}>{message.content as string}</div>
      ))}
    </div>
  )
}
```

### 브랜칭 지원

브랜칭을 활성화하려면 메시지 처리를 활성화해야 합니다. `useStream()` 훅에 `messagesKey` 옵션을 전달하세요. 각 메시지에 대해 `getMessagesMetadata()`를 사용해 메시지가 처음으로 표시된 첫 번째 체크포인트를 가져올 수 있습니다. 그런 다음 첫 번째로 표시된 체크포인트 이전의 체크포인트에서 새 실행을 생성하여 스레드 내에 새로운 브랜치를 만들 수 있습니다.

브랜치는 다음과 같은 방법으로 생성할 수 있습니다:

1. 이전 사용자 메시지를 편집합니다.
2. 이전 어시스턴트 메시지의 재생성을 요청합니다.

```tsx
/* eslint-disable @typescript-eslint/no-floating-promises */
"use client"

import type { Message } from "@langchain/langgraph-sdk"
import { useStream } from "@langchain/langgraph-sdk/react"
import {
  Annotation,
  MessagesAnnotation,
  type StateType,
  type UpdateType,
} from "@langchain/langgraph/web"
import { useState } from "react"

const AgentState = Annotation.Root({
  ...MessagesAnnotation.spec,
})

function BranchSwitcher({
  branch,
  branchOptions,
  onSelect,
}: {
  branch: string | undefined
  branchOptions: string[] | undefined
  onSelect: (branch: string) => void
}) {
  if (!branchOptions || !branch) return null
  const index = branchOptions.indexOf(branch)

  return (
    <div className="flex items-center gap-2">
      <button
        type="button"
        onClick={() => {
          const prevBranch = branchOptions[index - 1]
          if (!prevBranch) return
          onSelect(prevBranch)
        }}
      >
        이전
      </button>
      <span>
        {index + 1} / {branchOptions.length}
      </span>
      <button
        type="button"
        onClick={() => {
          const nextBranch = branchOptions[index + 1]
          if (!nextBranch) return
          onSelect(nextBranch)
        }}
      >
        다음
      </button>
    </div>
  )
}

function EditMessage({
  message,
  onEdit,
}: {
  message: Message
  onEdit: (message: Message) => void
}) {
  const [editing, setEditing] = useState(false)

  if (!editing) {
    return (
      <button type="button" onClick={() => setEditing(true)}>
        수정
      </button>
    )
  }

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        const form = e.target as HTMLFormElement
        const content = new FormData(form).get("content") as string

        form.reset()
        onEdit({ type: "human", content })
        setEditing(false)
      }}
    >
      <input name="content" defaultValue={message.content as string} />
      <button type="submit">저장</button>
    </form>
  )
}

export default function App() {
  const thread = useStream<
    StateType<typeof AgentState.spec>,
    UpdateType<typeof AgentState.spec>
  >({
    apiUrl: "http://localhost:2024",
    assistantId: "agent",
    messagesKey: "messages",
  })

  return (
    <div>
      <div>
        {thread.messages.map((message) => {
          const meta = thread.getMessagesMetadata(message)
          const parentCheckpoint = meta?.firstSeenState?.parent_checkpoint

          return (
            <div key={message.id}>
              <div>{message.content as string}</div>

              {message.type === "human" && (
                <EditMessage
                  message={message}
                  onEdit={(message) =>
                    thread.submit(
                      { messages: [message] },
                      { checkpoint: parentCheckpoint }
                    )
                  }
                />
              )}

              {message.type === "ai" && (
                <button
                  type="button"
                  onClick={() =>
                    thread.submit(undefined, { checkpoint: parentCheckpoint })
                  }
                >
                  <span>다시 생성</span>
                </button>
              )}

              <BranchSwitcher
                branch={meta?.branch}
                branchOptions={meta?.branchOptions}
                onSelect={(branch) => thread.setBranch(branch)}
              />
            </div>
          )
        })}
      </div>

      <form
        onSubmit={(e) => {
          e.preventDefault()

          const form = e.target as HTMLFormElement
          const message = new FormData(form).get("message") as string

          form.reset()
          thread.submit({ messages: [message] })
        }}
      >
        <input type="text" name="message" />

        {thread.isLoading ? (
          <button key="stop" type="button" onClick={() => thread.stop()}>
            중지
          </button>
        ) : (
          <button key="submit" type="submit">
            전송
          </button>
        )}
      </form>
    </div>
  )
}
```

### TypeScript

`useStream()` 훅은 오류를 조기에 잡고 더 나은 IDE 지원을 제공하기 위해 완전히 타입이 지정되어 있습니다. 다음과 같은 타입을 지정할 수 있습니다:

- 상태 형태
- 업데이트 형식
- 사용자 정의 이벤트

```tsx
// 타입 정의
type State = {
  messages: Message[]
  context?: Record<string, unknown>
}

type Update = {
  messages: Message[] | Message
  context?: Record<string, unknown>
}

type CustomEvent = {
  type: "progress" | "debug"
  payload: unknown
}

// 훅과 함께 사용
const thread = useStream<State, Update, CustomEvent>({
  apiUrl: "http://localhost:2024",
  assistantId: "agent",
  messagesKey: "messages",
})
```

LangGraph.js를 사용하는 경우 그래프의 주석 타입을 재사용할 수 있습니다:

```tsx
import {
  Annotation,
  MessagesAnnotation,
  type StateType,
  type UpdateType,
} from "@langchain/langgraph/web"

const AgentState = Annotation.Root({
  ...MessagesAnnotation.spec,
  context: Annotation<string>(),
})

const thread = useStream<
  StateType<typeof AgentState.spec>,
  UpdateType<typeof AgentState.spec>
>({
  apiUrl: "http://localhost:2024",
  assistantId: "agent",
  messagesKey: "messages",
})
```

## 이벤트 처리

`useStream()` 훅은 다양한 이벤트에 반응할 수 있도록 여러 콜백 옵션을 제공합니다:

- `onError`: 오류가 발생할 때 호출됩니다.
- `onFinish`: 스트림이 종료될 때 호출됩니다.
- `onUpdateEvent`: 업데이트 이벤트를 받을 때 호출됩니다.
- `onCustomEvent`: 사용자 정의 이벤트를 받을 때 호출됩니다. 사용자 정의 이벤트를 스트리밍하는 방법에 대해서는 [사용자 정의 이벤트](../../concepts/streaming.md#custom)를 참조하세요.
- `onMetadataEvent`: 메타데이터 이벤트를 받을 때 호출됩니다.

## 자세히 알아보기

- [JS/TS SDK 참고](../reference/sdk/js_ts_sdk_ref.md)
