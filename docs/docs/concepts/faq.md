_한국어로 기계번역됨_

# 자주 묻는 질문

일반적인 질문과 그에 대한 답변입니다!

## LangGraph를 사용하기 위해 LangChain을 사용할 필요가 있습니까? 차이점은 무엇인가요?

아니요. LangGraph는 복잡한 에이전트 시스템을 위한 오케스트레이션 프레임워크로, LangChain 에이전트보다 더 낮은 수준에서 제어할 수 있습니다. LangChain은 모델 및 다른 구성 요소와 상호 작용하기 위한 표준 인터페이스를 제공하여 직관적인 체인 및 검색 흐름에서 유용합니다.

## LangGraph는 다른 에이전트 프레임워크와 어떻게 다른가요?

다른 에이전트 프레임워크는 간단하고 일반적인 작업에 대해 작동할 수 있지만, 회사의 요구에 맞춘 복잡한 작업에서는 부족함이 있습니다. LangGraph는 사용자에게 단일 블랙박스 인지 아키텍처로 제한하지 않고 회사의 고유한 작업을 처리할 수 있는 더 표현적인 프레임워크를 제공합니다.

## LangGraph가 제 앱의 성능에 영향을 미치나요?

LangGraph는 귀하의 코드에 추가적인 오버헤드를 추가하지 않으며, 스트리밍 워크플로우를 염두에 두고 특별히 설계되었습니다.

## LangGraph는 오픈 소스인가요? 무료인가요?

네. LangGraph는 MIT 라이센스의 오픈 소스 라이브러리이며, 무료로 사용할 수 있습니다.

## LangGraph와 LangGraph 플랫폼의 차이점은 무엇인가요?

LangGraph는 에이전트 워크플로우에 추가적인 제어를 제공하는 상태 저장 오케스트레이션 프레임워크입니다. LangGraph 플랫폼은 LangGraph 애플리케이션을 배포하고 확장하기 위한 서비스로, 에이전트 사용자 경험을 구축하기 위한 의견이 반영된 API와 통합된 개발자 스튜디오를 제공합니다.

| 기능        | LangGraph (오픈 소스)                                            | LangGraph 플랫폼                                                                          |
| ----------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| 설명        | 에이전트 애플리케이션을 위한 상태 저장 오케스트레이션 프레임워크 | LangGraph 애플리케이션을 배포하기 위한 확장 가능한 인프라                                 |
| SDKs        | 파이썬 및 자바스크립트                                           | 파이썬 및 자바스크립트                                                                    |
| HTTP API    | 없음                                                             | 있음 - 상태 또는 장기 기억을 검색 및 업데이트하거나 구성 가능한 도우미를 만드는 데 유용함 |
| 스트리밍    | 기본                                                             | 토큰 단위 메시지를 위한 전용 모드                                                         |
| 체크포인터  | 커뮤니티 기여                                                    | 기본 제공                                                                                 |
| 지속성 계층 | 자가 관리                                                        | 효율적인 스토리지와 함께 관리되는 Postgres                                                |
| 배포        | 자가 관리                                                        | • 클라우드 SaaS <br> • 무료 자가 호스팅 <br> • 엔터프라이즈(BYOC 또는 유료 자가 호스팅)   |
| 확장성      | 자가 관리                                                        | 작업 큐와 서버의 자동 확장                                                                |
| 결함 허용   | 자가 관리                                                        | 자동 재시도                                                                               |
| 동시성 제어 | 간단한 스레딩                                                    | 이중 텍스트 지원                                                                          |
| 스케줄링    | 없음                                                             | 크론 스케줄링                                                                             |
| 모니터링    | 없음                                                             | 관찰 가능성을 위한 LangSmith와 통합                                                       |
| IDE 통합    | 데스크탑용 LangGraph 스튜디오                                    | 데스크탑 및 클라우드용 LangGraph 스튜디오                                                 |

## LangGraph 플랫폼의 배포 옵션은 무엇인가요?

현재 LangGraph 애플리케이션을 위한 다음과 같은 배포 옵션이 있습니다:

- [‍자가 호스팅 라이트](./deployment_options.md#self-hosted-lite): 로컬 또는 자가 호스팅 방식으로 실행할 수 있는 무료 버전(1M 노드까지 실행 가능)입니다. 이 버전은 LangSmith API 키가 필요하며 모든 사용을 LangSmith에 기록합니다. 유료 플랜보다 사용할 수 있는 기능이 적습니다.
- [클라우드 SaaS](./deployment_options.md#cloud-saas): LangSmith의 일환으로 완전히 관리 및 호스팅되며, 자동 업데이트와 유지 관리가 필요 없습니다.
- [‍자신의 클라우드 가져오기(BYOC)](./deployment_options.md#bring-your-own-cloud): 귀하의 VPC 내에서 LangGraph 플랫폼을 배포하며, 서비스로 제공됩니다. 서비스 관리 아웃소싱하면서 데이터는 귀하의 환경에 보관합니다.
- [자가 호스팅 엔터프라이즈](./deployment_options.md#self-hosted-enterprise): 귀하의 인프라에서 LangGraph를 전적으로 배포합니다.

## LangGraph 플랫폼은 오픈 소스인가요?

아니요. LangGraph 플랫폼은 독점 소프트웨어입니다.

기본 기능에 접근할 수 있는 무료 자가 호스팅 버전이 있습니다. 클라우드 SaaS 배포 옵션은 베타 기간 동안 무료이지만, 결국 유료 서비스가 될 것입니다. 우리는 서비스에 대한 유료 청구 전에 충분한 통지를 제공하고 초기 사용자를 선호 가격으로 보상할 것입니다. Bring Your Own Cloud(BYOC) 및 Self-Hosted Enterprise 옵션은 유료 서비스입니다. [영업 팀에 문의](https://www.langchain.com/contact-sales)하여 자세히 알아보세요.

자세한 내용은 [LangGraph 플랫폼 가격 페이지](https://www.langchain.com/pricing-langgraph-platform)를 참조하십시오.

## LangGraph는 도구 호출을 지원하지 않는 LLM과 함께 작동하나요?

네! LangGraph는 모든 LLM과 함께 사용할 수 있습니다. 도구 호출을 지원하는 LLM을 사용하는 주요 이유는 LLM이 수행할 작업에 대한 결정을 내리는 가장 편리한 방법이기 때문입니다. LLM이 도구 호출을 지원하지 않는 경우에도 사용할 수 있으며, 원시 LLM 문자열 응답을 작업에 대한 결정으로 변환하는 논리를 작성하면 됩니다.

## LangGraph는 OSS LLM과 함께 작동하나요?

네! LangGraph는 내부에서 사용되는 LLM에 대해 완전히 중립적입니다. 대부분의 튜토리얼에서 폐쇄형 LLM을 사용하는 주요 이유는 그들이 도구 호출을 원활하게 지원하기 때문이며, OSS LLM은 종종 그렇지 않습니다. 그러나 도구 호출이 필요하지 않기 때문에(참조: [이 섹션](#does-langgraph-work-with-llms-that-dont-support-tool-calling)) LangGraph를 OSS LLM과 함께 사용할 수 있습니다.
