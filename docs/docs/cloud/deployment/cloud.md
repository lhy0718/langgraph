_한국어로 기계번역됨_

# LangGraph Cloud에 배포하는 방법

LangGraph Cloud는 <a href="https://www.langchain.com/langsmith" target="_blank">LangSmith</a> 내에서 사용할 수 있습니다. LangGraph Cloud API를 배포하려면 <a href="https://smith.langchain.com/" target="_blank">LangSmith UI</a>로 이동합니다.

## 전제 조건

1. LangGraph Cloud 애플리케이션은 GitHub 리포지토리에서 배포됩니다. LangGraph Cloud 애플리케이션을 GitHub 리포지토리에 설정하고 업로드해야 LangGraph Cloud에 배포할 수 있습니다.
2. [LangGraph API가 로컬에서 실행되는지 확인](test_locally.md)합니다. API가 성공적으로 실행되지 않으면(`langgraph dev`가 실패하면), LangGraph Cloud에 배포도 실패합니다.

## 새 배포 생성

<a href="https://smith.langchain.com/" target="_blank">LangSmith UI</a>에서 시작합니다...

1. 왼쪽 탐색 패널에서 `LangGraph Platform`을 선택합니다. `LangGraph Platform` 뷰에는 기존의 LangGraph Cloud 배포 목록이 포함되어 있습니다.
2. 오른쪽 상단 모서리에서 `+ New Deployment`를 선택하여 새 배포를 생성합니다.
3. `Create New Deployment` 패널에서 필수 필드를 채웁니다.
    1. `배포 세부정보`
        1. `Import from GitHub`를 선택하고 GitHub OAuth 워크플로를 따라 LangChain의 `hosted-langserve` GitHub 앱을 선택한 리포지토리에 접근할 수 있도록 설치하고 권한을 부여합니다. 설치가 완료되면 `Create New Deployment` 패널로 돌아가 드롭다운 메뉴에서 배포할 GitHub 리포지토리를 선택합니다. **참고**: LangChain의 `hosted-langserve` GitHub 앱을 설치하는 GitHub 사용자 계정은 조직/계정의 [소유자](https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/roles-in-an-organization#organization-owners)여야 합니다.
        2. 배포의 이름을 지정합니다.
        3. 원하는 `Git Branch`를 지정합니다. 배포는 분기와 연결됩니다. 새 수정이 생성되면 연결된 분기의 코드가 배포됩니다. 분기는 [배포 설정](#deployment-settings)에서 나중에 업데이트할 수 있습니다.
        4. 파일 이름을 포함하여 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)의 전체 경로를 지정합니다. 예를 들어, 파일 `langgraph.json`이 리포지토리의 루트에 있는 경우, 단순히 `langgraph.json`을 지정합니다.
        5. `Automatically update deployment on push to branch` 체크박스를 체크/체크 해제합니다. 체크하면, 지정된 `Git Branch`에 변경 사항이 푸시될 때 배포가 자동으로 업데이트됩니다. 이 설정은 나중에 [배포 설정](#deployment-settings)에서 활성화/비활성화할 수 있습니다.
    2. 원하는 `배포 유형`을 선택합니다.
        1. `Development` 배포는 비생산 용도에 사용되며 최소한의 리소스로 프로비저닝됩니다.
        2. `Production` 배포는 초당 최대 500개의 요청을 처리할 수 있으며 자동 백업이 있는 고가용성 스토리지로 프로비저닝됩니다.
    3. 배포가 `LangGraph Studio를 통해 공유 가능`해야 하는지 결정합니다.
        1. 체크 해제 시, 해당 배포는 워크스페이스에 대한 유효한 LangSmith API 키로만 접근할 수 있습니다.
        2. 체크 시, 해당 배포는 모든 LangSmith 사용자가 LangGraph Studio를 통해 접근할 수 있습니다. 다른 LangSmith 사용자와 공유할 수 있도록 해당 배포에 대한 LangGraph Studio의 직접 URL이 제공됩니다.
    4. `환경 변수`와 비밀을 지정합니다. [환경 변수 참조](../reference/env_var.md)를 참조하여 배포를 위한 추가 변수를 구성합니다.
        1. API 키와 같은 민감한 값 (예: `OPENAI_API_KEY`)은 비밀로 지정해야 합니다.
        2. 추가 비밀이 아닌 환경 변수도 지정할 수 있습니다.
    5. 새 LangSmith `Tracing Project`가 배포와 동일한 이름으로 자동으로 생성됩니다.
4. 오른쪽 상단에서 `Submit`을 선택합니다. 몇 초 후에 `Deployment` 뷰가 나타나고 새 배포가 프로비저닝 대기열에 추가됩니다.

## 새 수정 생성

[새 배포 생성](#create-new-deployment) 시 기본적으로 새 수정이 생성됩니다. 이후에 새 코드 변경 사항을 배포하기 위해 수정할 수 있습니다.

<a href="https://smith.langchain.com/" target="_blank">LangSmith UI</a>에서 시작합니다...

1. 왼쪽 탐색 패널에서 `LangGraph Platform`을 선택합니다. `LangGraph Platform` 뷰에는 기존의 LangGraph Cloud 배포 목록이 포함되어 있습니다.
2. 수정할 기존 배포를 선택합니다.
3. `Deployment` 뷰에서 오른쪽 상단 모서리에서 `+ New Revision`을 선택합니다.
4. `New Revision` 모달에서 필수 필드를 채웁니다.
    1. 파일 이름을 포함하여 [LangGraph API 구성 파일](../reference/cli.md#configuration-file)의 전체 경로를 지정합니다. 예를 들어, 파일 `langgraph.json`이 리포지토리의 루트에 있는 경우, 단순히 `langgraph.json`을 지정합니다.
    2. 배포가 `LangGraph Studio를 통해 공유 가능`해야 하는지 결정합니다.
        1. 체크 해제 시, 배포는 워크스페이스에 대한 유효한 LangSmith API 키로만 접근할 수 있습니다.
        2. 체크 시, 배포는 모든 LangSmith 사용자가 LangGraph Studio를 통해 접근할 수 있습니다. 다른 LangSmith 사용자와 공유할 수 있도록 해당 배포에 대한 LangGraph Studio의 직접 URL이 제공됩니다.
    3. `환경 변수`와 비밀을 지정합니다. 기존 비밀과 환경 변수는 미리 채워져 있습니다. [환경 변수 참조](../reference/env_var.md)를 참조하여 수정에 대한 추가 변수를 구성합니다.
        1. 새 비밀이나 환경 변수를 추가합니다.
        2. 기존 비밀이나 환경 변수를 제거합니다.
        3. 기존 비밀이나 환경 변수의 값을 업데이트합니다.
5. `Submit`을 선택합니다. 몇 초 후에 `New Revision` 모달이 닫히고 새 수정이 배포 대기열에 추가됩니다.

## 빌드 및 서버 로그 보기

각 수정에 대한 빌드 및 서버 로그가 제공됩니다.

`LangGraph Platform` 뷰에서 시작합니다...

1. `Revisions` 테이블에서 원하는 수정을 선택합니다. 오른쪽에서 패널이 열리며 기본적으로 `Build` 탭이 선택되어 수정의 빌드 로그가 표시됩니다.
2. 패널에서 `Server` 탭을 선택하여 수정의 서버 로그를 봅니다. 서버 로그는 수정이 배포된 후에만 제공됩니다.
3. `Server` 탭 내에서 날짜/시간 범위 선택기를 필요에 따라 조정합니다. 기본적으로 날짜/시간 범위 선택기는 `지난 7일`로 설정됩니다.

## 수정 중단

수정을 중단하면 해당 수정의 배포가 중지됩니다.

!!! 경고 "정의되지 않은 동작"
    중단된 수정은 정의되지 않은 동작을 가집니다. 이는 수정된 새 배포를 해야 하고 이미 중단된 수정이 진행 중일 때만 유용합니다. 미래에는 이 기능이 제거될 수 있습니다.

`LangGraph Platform` 뷰에서 시작합니다...

1. `Revisions` 테이블에서 원하는 수정의 오른쪽 행에 있는 메뉴 아이콘(점 3개)을 선택합니다.
2. 메뉴에서 `Interrupt`를 선택합니다.
3. 모달이 나타납니다. 확인 메시지를 검토합니다. `Interrupt revision`을 선택합니다.

## 배포 삭제

<a href="https://smith.langchain.com/" target="_blank">LangSmith UI</a>에서 시작합니다...

1. 왼쪽 탐색 패널에서 `LangGraph Platform`을 선택합니다. `LangGraph Platform` 뷰에는 기존의 LangGraph Cloud 배포 목록이 포함되어 있습니다.
2. 원하는 배포의 오른쪽 행에 있는 메뉴 아이콘(점 3개)을 선택하고 `Delete`를 선택합니다.
3. `Confirmation` 모달이 나타납니다. `Delete`를 선택합니다.

## 배포 설정

`LangGraph Platform` 뷰에서 시작합니다...

1. 오른쪽 상단 모서리에서 기어 아이콘(`Deployment Settings`)을 선택합니다.
2. 원하는 분기로 `Git Branch`를 업데이트합니다.
3. `Automatically update deployment on push to branch` 체크박스를 체크/체크 해제합니다.
    1. 분기 생성/삭제 및 태그 생성/삭제 이벤트는 업데이트를 트리거하지 않습니다. 기존 분기로의 푸시만 업데이트를 트리거합니다.
    2. 분기에 대한 연속적인 푸시는 후속 업데이트를 트리거하지 않습니다. 향후 이 기능은 변경되거나 개선될 수 있습니다.
