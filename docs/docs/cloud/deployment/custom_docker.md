_한국어로 기계번역됨_

# Dockerfile 사용자 정의 방법

사용자는 부모 LangGraph 이미지에서 가져온 후 Dockerfile에 추가할 여러 줄을 추가할 수 있습니다. 이를 수행하려면 `langgraph.json` 파일을 수정하여 `dockerfile_lines` 키에 실행할 명령어를 전달하면 됩니다. 예를 들어, 그래프에서 `Pillow`를 사용하고 싶다면 다음과 같은 종속성을 추가해야 합니다:

```
{
    "dependencies": ["."],
    "graphs": {
        "openai_agent": "./openai_agent.py:agent",
    },
    "env": "./.env",
    "dockerfile_lines": [
        "RUN apt-get update && apt-get install -y libjpeg-dev zlib1g-dev libpng-dev",
        "RUN pip install Pillow"
    ]
}
```

이렇게 하면 `jpeg` 또는 `png` 이미지 형식을 사용할 때 Pillow에 필요한 시스템 패키지가 설치됩니다. 