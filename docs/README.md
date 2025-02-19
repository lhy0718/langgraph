_한국어로 기계번역됨_

# 설정

문서 작성을 위한 요구사항을 설정하려면 다음 명령어를 실행할 수 있습니다:

```bash
poetry install --with test
```

## 로컬에서 문서 제공

문서 서버를 로컬에서 실행하려면 다음 명령어를 실행할 수 있습니다:

```bash
make serve-docs
```

## 노트북 실행

모든 노트북을 자동으로 실행하고 싶다면, "노트북 실행" GHA를 모방하기 위해 다음 명령어를 실행할 수 있습니다:

```bash
python _scripts/prepare_notebooks_for_ci.py
./_scripts/execute_notebooks.sh
```

**참고**: `%pip install` 셀 없이 노트북을 실행하고 싶다면 다음 명령어를 실행할 수 있습니다:

```bash
python _scripts/prepare_notebooks_for_ci.py --comment-install-cells
./_scripts/execute_notebooks.sh
```

`prepare_notebooks_for_ci.py` 스크립트는 각 노트북 셀에 대해 VCR 카세트 컨텍스트 관리자를 추가하여:
* 노트북이 처음 실행될 때, 네트워크 요청이 있는 셀은 VCR 카세트 파일에 기록됩니다.
* 이후에 노트북이 실행될 때, 네트워크 요청이 있는 셀은 카세트에서 재생됩니다.

## 새로운 노트북 추가

API 요청이 포함된 노트북을 추가할 경우, 후에 재생할 수 있도록 네트워크 요청을 기록하는 것이 **권장**됩니다. 이를 수행하지 않으면, 노트북이 실행될 때마다 API 요청이 발생하게 되어 비용이 많이 들고 느려질 수 있습니다.

네트워크 요청을 기록하려면, 먼저 `prepare_notebooks_for_ci.py` 스크립트를 실행해야 합니다.

그런 다음, 다음을 실행하세요:

```bash
jupyter execute <path_to_notebook>
```

노트북이 실행되면, `cassettes` 디렉토리에 새로운 VCR 카세트가 기록되며 업데이트된 노트북은 무시할 수 있습니다.

## 기존 노트북 업데이트

기존 노트북을 업데이트하는 경우, `cassettes` 디렉토리에서 해당 노트북의 기존 카세트를 모두 제거해야 합니다 (각 카세트는 노트북 이름으로 접두사가 붙습니다). 그런 다음 위의 "새로운 노트북 추가" 섹션의 단계를 실행하세요.

노트북에 대한 카세트를 삭제하려면 다음을 실행할 수 있습니다:

```bash
rm cassettes/<notebook_name>*
```