_한국어로 기계번역됨_

# LangGraph CLI

LangGraph의 공식 커맨드 라인 인터페이스로, LangGraph 애플리케이션을 생성, 개발 및 배포하는 도구를 제공합니다.

## 설치

pip를 통해 설치:
```bash
pip install langgraph-cli
```

핫 리로딩을 위한 개발 모드:
```bash
pip install "langgraph-cli[inmem]"
```

## 명령어

### `langgraph new` 🌱
템플릿에서 새로운 LangGraph 프로젝트 생성
```bash
langgraph new [PATH] --template TEMPLATE_NAME
```

### `langgraph dev` 🏃‍♀️
핫 리로딩이 가능한 개발 모드에서 LangGraph API 서버 실행
```bash
langgraph dev [OPTIONS]
  --host TEXT                 연결할 호스트 (기본값: 127.0.0.1)
  --port INTEGER             연결할 포트 (기본값: 2024)
  --no-reload               자동 리로딩 비활성화
  --debug-port INTEGER      원격 디버깅 활성화
  --no-browser             브라우저 창 열기 건너뛰기
  -c, --config FILE        구성 파일 경로 (기본값: langgraph.json)
```

### `langgraph up` 🚀
Docker에서 LangGraph API 서버 실행
```bash
langgraph up [OPTIONS]
  -p, --port INTEGER        노출할 포트 (기본값: 8123)
  --wait                   서비스 시작 대기
  --watch                  파일 변경 시 재시작
  --verbose               상세 로그 표시
  -c, --config FILE       구성 파일 경로
  -d, --docker-compose    추가 서비스 파일
```

### `langgraph build`
LangGraph 애플리케이션을 위한 Docker 이미지 생성
```bash
langgraph build -t IMAGE_TAG [OPTIONS]
  --platform TEXT          대상 플랫폼 (예: linux/amd64,linux/arm64)
  --pull / --no-pull      최신/로컬 기본 이미지 사용
  -c, --config FILE       구성 파일 경로
```

### `langgraph dockerfile`
사용자 정의 배포를 위한 Dockerfile 생성
```bash
langgraph dockerfile SAVE_PATH [OPTIONS]
  -c, --config FILE       구성 파일 경로
```

## 구성

CLI는 다음과 같은 주요 설정을 포함한 `langgraph.json` 구성 파일을 사용합니다:

```json
{
  "dependencies": ["langchain_openai", "./your_package"],  // 필수: 패키지 의존성
  "graphs": {
    "my_graph": "./your_package/file.py:graph"            // 필수: 그래프 정의
  },
  "env": "./.env",                                        // 선택 사항: 환경 변수
  "python_version": "3.11",                               // 선택 사항: 파이썬 버전 (3.11/3.12)
  "pip_config_file": "./pip.conf",                        // 선택 사항: pip 구성
  "dockerfile_lines": []                                  // 선택 사항: 추가 Dockerfile 명령
}
```

자세한 구성 옵션은 [전체 문서](https://langchain-ai.github.io/langgraph/docs/cloud/reference/cli.html)를 참조하세요.

## 개발

CLI 자체를 개발하려면:

1. 리포지토리를 클론합니다.
2. CLI 디렉토리로 이동: `cd libs/cli`
3. 개발 종속성 설치: `poetry install`
4. CLI 코드에 변경 사항을 적용합니다.
5. 변경 사항을 테스트합니다:
   ```bash
   # CLI 명령어 직접 실행
   poetry run langgraph --help
   
   # 또는 예제 사용
   cd examples
   poetry install
   poetry run langgraph dev  # 또는 다른 명령어
   ```

## 라이선스

이 프로젝트는 리포지토리의 LICENSE 파일에 명시된 조건에 따라 라이선스가 부여됩니다.
