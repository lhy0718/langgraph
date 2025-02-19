_한국어로 기계번역됨_

# 보안 정책

## OSS 취약점 신고

LangChain은 우리의 오픈 소스 프로젝트를 위한 보상 프로그램을 제공하기 위해 [Protect AI의 huntr](https://huntr.com/)와 협력하고 있습니다.

LangChain 오픈 소스 프로젝트와 관련된 보안 취약점을 신고하려면 다음 링크를 방문하십시오:

[https://huntr.com/bounties/disclose/](https://huntr.com/bounties/disclose/?target=https%3A%2F%2Fgithub.com%2Flangchain-ai%2Flangchain&validSearch=true)

취약점을 신고하기 전에 다음을 검토하십시오:

1) 아래의 범위 내 대상 및 범위 외 대상.
2) [langchain-ai/langchain](https://python.langchain.com/docs/contributing/repo_structure) 모노레포 구조.
3) LangChain의 [보안 가이드라인](https://python.langchain.com/docs/security)을 검토하여
   우리가 보안 취약점으로 간주하는 것과 개발자의 책임을 이해합니다.

### 범위 내 대상

다음 패키지 및 저장소는 버그 보상 대상입니다:

- langchain-core
- langchain (예외 참조)
- langchain-community (예외 참조)
- langgraph
- langserve

### 범위 외 대상

huntr에 의해 정의된 모든 범위 외 대상 및:

- **langchain-experimental**: 이 저장소는 실험적 코드용이며
  버그 보상 대상이 아닙니다. 이곳에 대한 버그 보고서는 흥미롭거나 시간 낭비로 표시되며 보상 없이 게시됩니다.
- **tools**: langchain 또는 langchain-community의 도구는 버그 보상 대상이 아닙니다. 다음 디렉토리가 포함됩니다:
  - langchain/tools
  - langchain-community/tools
  - 자세한 내용은 우리의 [보안 가이드라인](https://python.langchain.com/docs/security)을 검토하십시오. 일반적으로 도구는 현실 세계와 상호작용합니다. 개발자는 자신의 코드의 보안 함의에 대한 이해가 필요하며 자신의 도구의 보안에 대해 책임을 져야 합니다.
- 보안 공지가 있는 코드. 이는 사례별로 결정되지만, 개발자들이 애플리케이션을 안전하게 만드는 데 따라야 할 지침이 이미 문서화되어 있기 때문에 보상 대상이 되지 않을 가능성이 높습니다.
- 아래에 언급된 LangSmith 관련 저장소 또는 API.

## LangSmith 취약점 신고

LangSmith와 관련된 보안 취약점을 신고하려면 `security@langchain.dev`로 이메일을 보내십시오.

- LangSmith 사이트: https://smith.langchain.com
- SDK 클라이언트: https://github.com/langchain-ai/langsmith-sdk

### 기타 보안 우려 사항

기타 보안 우려 사항은 `security@langchain.dev`로 문의해 주십시오.
