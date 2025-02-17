_한국어로 기계번역됨_

# OpenAPI에서 API 인증 문서화 방법

이 가이드는 LangGraph 플랫폼 API 문서의 OpenAPI 보안 스키마를 사용자화하는 방법을 보여줍니다. 잘 문서화된 보안 스키마는 API 소비자가 API와 인증하는 방법을 이해하는 데 도움이 되며, 자동 클라이언트 생성을 가능하게 합니다. LangGraph의 인증 시스템에 대한 자세한 내용은 [인증 및 접근 제어 개념 가이드](../../concepts/auth.md)를 참조하세요.

!!! 노트 "구현 vs 문서화"
    이 가이드는 OpenAPI에서 보안 요구 사항을 문서화하는 방법만 다룹니다. 실제 인증 로직을 구현하려면 [사용자 정의 인증 추가 방법](./custom_auth.md)을 참조하세요.

이 가이드는 모든 LangGraph 플랫폼 배포(클라우드, BYOC 및 자가 호스팅)에 적용됩니다. LangGraph 플랫폼을 사용하지 않는 경우 LangGraph 오픈 소스 라이브러리에는 적용되지 않습니다.

## 기본 스키마

기본 보안 스키마는 배포 유형에 따라 다릅니다:

=== "LangGraph 클라우드"

기본적으로 LangGraph 클라우드는 `x-api-key` 헤더에 LangSmith API 키를 요구합니다:

```yaml
components:
  securitySchemes:
    apiKeyAuth:
      type: apiKey
      in: header
      name: x-api-key
security:
  - apiKeyAuth: []
```

LangGraph SDK 중 하나를 사용할 때는 환경 변수에서 이를 유추할 수 있습니다.

=== "자가 호스팅"

기본적으로 자가 호스팅 배포에는 보안 스키마가 없습니다. 이는 보안 네트워크에서만 배포되거나 인증과 함께 배포되어야 함을 의미합니다. 사용자 정의 인증을 추가하려면 [사용자 정의 인증 추가 방법](./custom_auth.md)을 참조하세요.

## 사용자 정의 보안 스키마

OpenAPI 문서에서 보안 스키마를 사용자화하려면 `langgraph.json`의 `auth` 구성에 `openapi` 필드를 추가합니다. 이 방법은 API 문서만 업데이트되며, [사용자 정의 인증 추가 방법](./custom_auth.md)에 설명된 대로 해당 인증 로직도 구현해야 함을 기억하세요.

LangGraph 플랫폼은 인증 엔드포인트를 제공하지 않으므로, 사용자 인증은 클라이언트 애플리케이션에서 처리하고 결과로 얻은 자격 증명을 LangGraph API에 전달해야 합니다.

=== "Bearer Token을 사용하는 OAuth2"

    ```json
    {
      "auth": {
        "path": "./auth.py:my_auth",  // 여기서 인증 로직 구현
        "openapi": {
          "securitySchemes": {
            "OAuth2": {
              "type": "oauth2",
              "flows": {
                "implicit": {
                  "authorizationUrl": "https://your-auth-server.com/oauth/authorize",
                  "scopes": {
                    "me": "현재 사용자에 대한 정보 읽기",
                    "threads": "스레드 생성 및 관리 접근"
                  }
                }
              }
            }
          },
          "security": [
            {"OAuth2": ["me", "threads"]}
          ]
        }
      }
    }
    ```

=== "API 키"

    ```json
    {
      "auth": {
        "path": "./auth.py:my_auth",  // 여기서 인증 로직 구현
        "openapi": {
          "securitySchemes": {
            "apiKeyAuth": {
              "type": "apiKey",
              "in": "header",
              "name": "X-API-Key"
            }
          },
          "security": [
            {"apiKeyAuth": []}
          ]
        }
      }
    }
    ```

## 테스트

구성을 업데이트한 후:

1. 애플리케이션을 배포하세요.
2. `/docs`를 방문하여 업데이트된 OpenAPI 문서를 확인하세요.
3. 인증 서버에서 자격 증명을 사용하여 엔드포인트를 시험해보세요(먼저 인증 로직을 구현했는지 확인하세요).
