# 실습 환경 셋팅 가이드

## 환경 정보
- OS: Ubuntu 24.04
- Shell: bash
- 패키지 관리: uv
- LLM API: OpenAI API (직접 호출) + MonoRouter(OpenAI 호환 엔드포인트, `common_config.py`)

---

## 1. uv 설치

터미널을 열고 아래 명령어를 실행합니다.

```bash
# curl이 없다면 먼저 설치
sudo apt update
sudo apt install -y curl

# uv 설치
curl -LsSf https://astral.sh/uv/install.sh | sh

# 현재 터미널에 PATH 반영 (또는 터미널 재시작)
source $HOME/.local/bin/env
```

설치 확인:

```bash
uv --version
```

---

## 2. 프로젝트 폴더 이동

```bash
mkdir rag_agent_ex
cd rag_agent_ex
```

---

## 3. 가상환경 생성 및 활성화

```bash
# uv 초기화
uv init --bare --python 3.12 

# python 버전 고정
uv python pin 3.12

# 확인
cat .python-version

# 가상환경 활성화(uv add, uv run 명령시 생략 가능)
source .venv/bin/activate
```

- 가상환경 삭제[참고]
```bash
rm -rf .venv
# 파이썬 고정 파일 삭제
rm -f .python-version
# 셸 세션에 남아 있을 수 있는 환경변수 제거
unset VIRTUAL_ENV UV_PROJECT_ENVIRONMENT
```
---

## 4. 패키지 설치
- 프로젝트 의존성 추가하려면 uv add로 설치하기
- pyproject.toml 파일에 기록을함.

```bash
uv add langchain langchain-openai langchain-community langchain-classic
uv add langchain-ollama      # MonoRouter 등 OpenAI 호환 엔드포인트 연동
uv add langgraph
uv add faiss-cpu             # 벡터 스토어 (2.naive_rag)
uv add langchain-text-splitters
uv add jupyterlab ipykernel
uv add python-dotenv
uv add tiktoken
uv add openai
uv add requests
```

- 한 번에 설치:

```bash
uv add langchain langchain-openai langchain-community langchain-classic langchain-ollama langgraph faiss-cpu langchain-text-splitters jupyterlab ipykernel python-dotenv tiktoken openai requests
```

- 설치한 패키지 버전확인
```bash
uv pip show langchain
```

---

## 5. Jupyter 커널 등록 및 삭제
- 등록
```bash
python -m ipykernel install --user --name agent_rag

# 확인
jupyter kernelspec list
```
- 삭제[참고]
```
jupyter kernelspec uninstall .venv
```
---

## 6. API 키 설정

프로젝트 루트의 `.env.example`을 복사해 `.env` 파일을 생성하고, 본인 키로 값을 채웁니다.

```bash
cp .env.example .env
```

```
# 01_openai_test.ipynb 등 OpenAI API를 직접 호출하는 실습에서 사용
OPENAI_API_KEY=sk-...여기에_본인_키_입력...

# common_config.py의 llm_connect()가 사용하는 MonoRouter(OpenAI 호환) 엔드포인트
LLM_BASE_URL=https://monogpt.kr/api/monorouter/v1
LLM_API_KEY=your_mono_apikey
```

> `.env` 파일은 절대 GitHub 등 외부에 공유하지 마세요. (`.gitignore`에 이미 등록되어 있음)

대부분의 실습 노트북은 공통 모듈([common_config.py](common_config.py))의 `llm_connect()`를 통해 LLM을 호출합니다.

```python
import sys
sys.path.append("..")
from common_config import llm_connect

llm = llm_connect(model="gpt-5.4")
```

`llm_connect()`는 내부적으로 다음과 같이 `.env`를 읽어 `ChatOpenAI`를 생성합니다 (`base_url`을 쓰므로 `use_responses_api=False` 설정이 필요함):

```python
from dotenv import load_dotenv
load_dotenv(override=True)
```

---

## 7. Jupyter Notebook 실행

```bash
jupyter lab
```

브라우저에서 자동으로 열리며 (열리지 않으면 터미널에 출력된 `http://localhost:8888/...` 주소를 브라우저에 붙여넣기), 커널은 `rag_agent` 를 선택합니다.

---

## 8. 폴더 구조
## 8. 폴더 구조

```
rag_agent_ex/
├── .env.local            # API 키 (.env 수정해서 쓰기)
├── .venv/                # 가상환경
├── 1.langchain_basic/
├── 2.naive_rag/
├── 3.advanced_rag_trend/
├── 4.langgraph_basic/
├── 5.langgraph_naver_shopping
└── 6.modular_rag_manual/
```

---

## 설치 패키지 요약

| 패키지 | 용도 |
|---|---|
| langchain | LangChain 핵심 |
| langchain-openai | OpenAI 연동 (ChatOpenAI, OpenAIEmbeddings) |
| langchain-community | 커뮤니티 컴포넌트 (문서 로더, FAISS 등) |
| langchain-classic | 레거시 리트리버 (예: MultiQueryRetriever) |
| langchain-text-splitters | 텍스트 분할 (CharacterTextSplitter 등) |
| langchain-ollama | MonoRouter 등 OpenAI 호환 엔드포인트 연동 |
| langgraph | LangGraph (추후 Agent/Modular RAG 실습) |
| faiss-cpu | 벡터 스토어 (2.naive_rag) |
| openai | OpenAI 공식 SDK |
| jupyterlab | 실습 환경 |
| ipykernel | Jupyter 커널 |
| python-dotenv | .env 파일 로드 |
| tiktoken | 토큰 계산 |
| requests | HTTP 요청 |

