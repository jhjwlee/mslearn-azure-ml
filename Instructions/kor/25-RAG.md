## 자체 데이터를 사용하는 생성형 AI 앱 만들기 (Create a generative AI app that uses your own data)

**검색 증강 생성(Retrieval Augmented Generation, RAG)**은 사용자 지정 데이터 원본의 데이터를 생성형 AI 모델용 프롬프트에 통합하는 애플리케이션을 구축하는 데 사용되는 기술입니다. RAG는 언어 모델을 사용하여 입력을 해석하고 적절한 응답을 생성하는 채팅 기반 애플리케이션과 같은 생성형 AI 앱을 개발하는 데 일반적으로 사용되는 패턴입니다.

이 실습에서는 Azure AI Foundry를 사용하여 사용자 지정 데이터를 생성형 AI 솔루션에 통합합니다.

이 실습은 약 **45분**이 소요됩니다.

**참고:** 이 실습은 사전 릴리스 서비스를 기반으로 하며 변경될 수 있습니다.

**학습 목표:**
이 실습을 통해 다음을 수행하는 방법을 배웁니다:
*   `Azure AI Foundry hub resource`를 기반으로 하는 `project`를 생성합니다.
*   RAG에 필요한 두 가지 모델(임베딩 모델, 생성 모델)을 배포합니다.
*   PDF 형식의 사용자 지정 데이터를 `project`에 추가합니다.
*   추가된 데이터를 기반으로 Azure AI Search 인덱스를 생성합니다.
*   `Chat playground`에서 인덱스를 테스트하여 RAG의 효과를 확인합니다.
*   VS Code Online (GitHub Codespaces) 환경에서 Python SDK를 사용하여 RAG 패턴을 구현하는 클라이언트 애플리케이션을 구성하고 실행합니다.

**주요 용어 (영문 유지):**
*   Retrieval Augmented Generation (RAG)
*   Azure AI Foundry portal
*   Azure AI Foundry hub resource
*   Project
*   Embedding model (e.g., `text-embedding-ada-002`)
*   Generation model (e.g., `gpt-4o`)
*   Models + endpoints
*   Deployment name
*   Tokens per Minute Rate Limit (TPM)
*   Content filter
*   Data + indexes
*   Azure AI Search
*   Vector index
*   Hybrid (vector + keyword) search
*   Chat playground
*   OpenAI SDK
*   VS Code Online (GitHub Codespaces)
*   `.env` file

---

### Azure AI Foundry 허브 및 프로젝트 만들기 (Create an Azure AI Foundry hub and project)

이 실습에서 사용할 Azure AI Foundry의 기능에는 `Azure AI Foundry hub resource`를 기반으로 하는 `project`가 필요합니다. (이 부분은 이전 실습과 유사합니다.)

*   **Azure AI Foundry hub resource란?**
    *   엔터프라이즈급 AI 개발을 위한 중앙 집중식 관리 허브입니다. 여러 `project`를 호스팅하고 공유 리소스, 거버넌스 기능을 제공하며, RAG와 같은 고급 기능을 사용하는 데 필요합니다.
*   **핸즈온에서 배울 점:**
    *   RAG 솔루션 구축을 위해 `Azure AI Foundry hub resource` 기반의 `project`를 생성하는 방법을 익힙니다.

1.  웹 브라우저에서 Azure AI Foundry portal (https://ai.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 모든 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 상단의 Azure AI Foundry 로고를 사용하여 홈페이지로 이동합니다.
2.  브라우저에서 https://ai.azure.com/managementCenter/allResources 로 이동하여 **만들기(Create)**를 선택합니다. 그런 다음 새 **AI hub resource**를 만드는 옵션을 선택합니다.
3.  **프로젝트 만들기(Create a project)** 마법사에서 `project`에 유효한 이름을 입력하고, 기존 허브가 제안되면 새 허브를 만드는 옵션을 선택하고 **고급 옵션(Advanced options)**을 확장하여 `project`에 대해 다음 설정을 지정합니다:
    *   **Subscription:** 사용자의 Azure 구독
    *   **Resource group:** `Resource group` 만들기 또는 선택
    *   **Hub name:** 허브에 대한 유효한 이름
    *   **Location:** `East US 2` 또는 `Sweden Central` (실습 후반에 할당량 한도를 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.)
        *   **의미:** 특정 Azure 서비스 및 기능은 지역별로 가용성이 다를 수 있으며, RAG 관련 기능이 지원되는 지역을 선택하는 것이 중요합니다.
    **참고:** 허용되는 리소스 이름을 제한하는 정책이 사용되는 Azure 구독에서 작업하는 경우, **새 프로젝트 만들기(Create a new project)** 대화 상자 하단의 링크를 사용하여 Azure portal을 통해 허브를 만들어야 할 수 있습니다.

4.  `project`가 생성될 때까지 기다린 다음, `project`로 이동합니다.

---

### 모델 배포 (Deploy models)

솔루션을 구현하려면 두 가지 모델이 필요합니다:

*   효율적인 인덱싱 및 처리를 위해 텍스트 데이터를 벡터화하는 **임베딩 모델(Embedding model)**.
*   데이터를 기반으로 질문에 대한 자연어 응답을 생성할 수 있는 **생성 모델(Generation model)**.

*   **RAG를 위한 모델:**
    *   **Embedding model (`text-embedding-ada-002`):** 텍스트(사용자 질문, 문서 내용)를 숫자 벡터(임베딩)로 변환합니다. 이 벡터는 의미론적 유사성을 나타내므로, 유사한 의미를 가진 텍스트는 벡터 공간에서 가깝게 위치합니다. RAG에서는 사용자의 질문과 문서 청크를 임베딩하여 관련성 높은 문서를 찾는 데 사용됩니다.
    *   **Generation model (`gpt-4o`):** 사용자의 질문과 검색된 관련 문서 내용을 바탕으로 최종 답변을 생성합니다.
*   **핸즈온에서 배울 점:**
    *   RAG 파이프라인에 필요한 두 가지 핵심 모델(`text-embedding-ada-002`, `gpt-4o`)을 Azure AI Foundry `project` 내에 배포하는 방법을 익힙니다. 각 모델의 역할을 이해합니다.

1.  Azure AI Foundry portal의 `project` 내 왼쪽 탐색 창에서 **내 자산(My assets)** 아래의 **모델 + 엔드포인트(Models + endpoints)** 페이지를 선택합니다.
2.  **모델 배포(Deploy model)** 마법사에서 **사용자 지정(Customize)**을 선택하여 다음 설정으로 `text-embedding-ada-002` 모델의 새 배포를 만듭니다:
    *   **Deployment name:** 모델 배포에 대한 유효한 이름 (예: `ada-002-embed`)
    *   **Deployment type:** `Global Standard`
    *   **Model version:** 기본 버전 선택
    *   **Connected AI resource:** 이전에 생성된 리소스 선택
    *   **Tokens per Minute Rate Limit (thousands):** `50K` (또는 구독에서 사용 가능한 최대값이 50K 미만인 경우 해당 값)
        *   **의미 (TPM):** 분당 처리할 수 있는 토큰의 양을 제한합니다. 임베딩 모델도 API 호출 시 토큰을 소모하므로 적절한 TPM 설정이 필요합니다.
    *   **Content filter:** `DefaultV2`
        *   **의미:** Azure OpenAI에서 제공하는 콘텐츠 필터링 수준입니다.
    **참고:** 현재 AI 리소스 위치에 배포하려는 모델에 대한 할당량이 없는 경우, 새 AI 리소스가 생성되어 `project`에 연결될 다른 위치를 선택하라는 메시지가 표시됩니다.

3.  **모델 + 엔드포인트(Models + endpoints)** 페이지로 돌아가 이전 단계를 반복하여 `gpt-4o` 모델을 배포합니다. `Global Standard` 배포, 최신 버전, TPM 속도 제한 `50K` (또는 구독에서 사용 가능한 최대값)를 사용합니다. (예: 배포 이름 `gpt-4o-chat`)

    **참고:** 분당 토큰 수(TPM)를 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하는 것을 방지하는 데 도움이 됩니다. 50,000 TPM은 이 실습에서 사용되는 데이터에 충분합니다.

---

### 프로젝트에 데이터 추가 (Add data to your project)

앱의 데이터는 가상의 여행사 Margie’s Travel의 PDF 형식 여행 안내 책자 세트로 구성됩니다. 이를 `project`에 추가해 보겠습니다.

*   **사용자 지정 데이터:**
    *   RAG의 핵심은 LLM이 학습하지 않은 최신 정보나 특정 도메인의 비공개 데이터를 활용하는 것입니다. 이 실습에서는 여행사 안내 책자(PDF 파일)를 사용자 지정 데이터로 사용합니다.
*   **핸즈온에서 배울 점:**
    *   로컬 파일 시스템에 있는 사용자 지정 데이터(이 경우 PDF 파일들이 담긴 폴더)를 Azure AI Foundry `project`의 데이터 저장소에 업로드하는 방법을 배웁니다.

1.  새 브라우저 탭에서 https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip 에서 브로셔의 압축 아카이브를 다운로드하고 로컬 파일 시스템의 `brochures`라는 폴더에 압축을 풉니다.
2.  Azure AI Foundry portal의 `project` 내 왼쪽 탐색 창에서 **내 자산(My assets)** 아래의 **데이터 + 인덱스(Data + indexes)** 페이지를 선택합니다.
3.  **+ 새 데이터(+ New data)**를 선택합니다.
4.  **데이터 추가(Add your data)** 마법사에서 드롭다운 메뉴를 확장하여 **파일/폴더 업로드(Upload files/folders)**를 선택합니다.
5.  **폴더 업로드(Upload folder)**를 선택하고 `brochures` 폴더를 업로드합니다. 폴더의 모든 파일이 나열될 때까지 기다립니다.
6.  **다음(Next)**을 선택하고 데이터 이름을 `brochures`로 설정합니다.
7.  폴더가 업로드될 때까지 기다리고 여러 `.pdf` 파일이 포함되어 있는지 확인합니다.

---

### 데이터에 대한 인덱스 만들기 (Create an index for your data)

`project`에 데이터 원본을 추가했으므로 이제 이를 사용하여 Azure AI Search 리소스에 인덱스를 만들 수 있습니다.

*   **Azure AI Search와 인덱스:**
    *   Azure AI Search는 클라우드 기반 검색 서비스로, 대규모 데이터셋에서 정보를 빠르고 효율적으로 검색할 수 있는 기능을 제공합니다.
    *   **인덱스(Index)**는 검색 가능한 데이터의 구조화된 표현입니다. RAG에서는 업로드된 문서들을 처리(청크 분할, 임베딩 생성)하여 Azure AI Search 인덱스에 저장합니다.
*   **Vector index:**
    *   텍스트의 의미론적 정보를 담고 있는 벡터(임베딩)를 저장하고 검색하는 데 최적화된 인덱스입니다. 사용자의 질문 벡터와 가장 유사한 문서 청크 벡터를 찾아 관련 정보를 검색합니다.
*   **Indexing process (인덱싱 프로세스):**
    *   **Crack, chunk, and embed (분석, 청크 분할, 임베딩):**
        1.  `Crack (분석)`: PDF와 같은 문서에서 텍스트를 추출합니다.
        2.  `Chunk (청크 분할)`: 추출된 텍스트를 임베딩 모델이 처리하기 적합한 작은 단위(청크)로 나눕니다.
        3.  `Embed (임베딩)`: 각 청크를 임베딩 모델(`text-embedding-ada-002`)을 사용하여 숫자 벡터로 변환합니다.
*   **핸즈온에서 배울 점:**
    *   업로드된 데이터를 기반으로 Azure AI Search 리소스에 `vector index`를 생성하는 과정을 이해합니다.
    *   인덱스 생성 시 새로운 Azure AI Search 리소스를 만들거나 기존 리소스를 연결하는 방법을 배웁니다.
    *   인덱싱 프로세스(텍스트 추출, 청크 분할, 임베딩 생성)가 자동으로 수행됨을 확인합니다.
    *   임베딩 생성을 위해 Azure OpenAI 연결 및 배포된 임베딩 모델을 지정하는 방법을 익힙니다.

1.  Azure AI Foundry portal의 `project` 내 왼쪽 탐색 창에서 **내 자산(My assets)** 아래의 **데이터 + 인덱스(Data + indexes)** 페이지를 선택합니다.
2.  **인덱스(Indexes)** 탭에서 다음 설정으로 새 인덱스를 추가합니다:
    *   **소스 위치(Source location):**
        *   **Data source:** `Data in Azure AI Foundry`
        *   `brochures` 데이터 원본 선택
    *   **인덱스 구성(Index configuration):**
        *   **Select Azure AI Search service:** 다음 설정으로 **새 Azure AI Search 리소스 만들기(Create a new Azure AI Search resource)**:
            *   **Subscription:** 사용자의 Azure 구독
            *   **Resource group:** AI 허브와 동일한 `resource group`
            *   **Service name:** AI Search 리소스에 대한 유효한 이름 (예: `margies-rag-search`)
            *   **Location:** AI 허브와 동일한 위치
            *   **Pricing tier:** `Basic`
                *   **의미:** Azure AI Search의 가격 책정 계층입니다. `Basic`은 소규모 테스트 및 개발에 적합하며 벡터 검색 기능을 지원합니다.
        *   AI Search 리소스가 생성될 때까지 기다립니다. 그런 다음 Azure AI Foundry로 돌아가 **다른 Azure AI Search 리소스 연결(Connect other Azure AI Search resource)**을 선택하고 방금 만든 AI Search 리소스에 대한 연결을 추가하여 인덱스 구성을 완료합니다.
    *   **Vector index:** `brochures-index` (인덱스 이름)
    *   **Virtual machine:** `Auto select` (인덱싱 작업에 사용될 VM 자동 선택)
    *   **검색 설정(Search settings):**
        *   **Vector settings:** `Add vector search to this search resource` (이 검색 리소스에 벡터 검색 추가)
        *   **Azure OpenAI connection:** 허브에 대한 기본 Azure OpenAI 리소스 선택
        *   **Embedding model:** `text-embedding-ada-002`
        *   **Embedding model deployment:** `text-embedding-ada-002` 모델의 배포 (예: `ada-002-embed`)
3.  벡터 인덱스를 만들고 구독의 사용 가능한 컴퓨팅 리소스에 따라 시간이 걸릴 수 있는 인덱싱 프로세스가 완료될 때까지 기다립니다.

    인덱스 생성 작업은 다음 작업으로 구성됩니다:
    *   `brochures` 데이터의 텍스트 토큰을 분석, 청크 분할 및 임베딩합니다.
    *   Azure AI Search 인덱스를 만듭니다.
    *   인덱스 자산을 등록합니다.

    **팁:** 인덱스가 생성되기를 기다리는 동안 다운로드한 브로셔를 살펴보고 내용을 익히십시오.

---

### 플레이그라운드에서 인덱스 테스트 (Test the index in the playground)

RAG 기반 `prompt flow`에서 인덱스를 사용하기 전에 생성형 AI 응답에 영향을 미칠 수 있는지 확인해 보겠습니다.

*   **핸즈온에서 배울 점:**
    *   Azure AI Foundry `Chat playground`에서 생성된 인덱스를 직접 연결하여 RAG 기능을 테스트하는 방법을 배웁니다.
    *   인덱스를 사용했을 때와 사용하지 않았을 때 모델의 답변이 어떻게 달라지는지 비교하여 RAG의 효과를 확인합니다.
    *   `Hybrid (vector + keyword) search type`의 의미를 이해합니다.

1.  왼쪽 탐색 창에서 **플레이그라운드(Playgrounds)** 페이지를 선택하고 **채팅(Chat)** 플레이그라운드를 엽니다.
2.  **채팅(Chat)** 플레이그라운드 페이지의 **설정(Setup)** 창에서 `gpt-4o` 모델 배포 (예: `gpt-4o-chat`)가 선택되어 있는지 확인합니다. 그런 다음 기본 채팅 세션 패널에 "뉴욕에서 어디에 머물 수 있나요?(Where can I stay in New York?)"라는 프롬프트를 제출합니다.
3.  인덱스의 데이터 없이 모델에서 생성된 일반적인 답변이어야 하는 응답을 검토합니다.
4.  **설정(Setup)** 창에서 **데이터 추가(Add your data)** 필드를 확장한 다음, `brochures-index` `project` 인덱스를 추가하고 **하이브리드 (벡터 + 키워드)(hybrid (vector + keyword))** 검색 유형을 선택합니다.
    *   **Hybrid (vector + keyword) search:** 벡터 검색(의미 기반 검색)과 키워드 검색(전통적인 텍스트 일치 검색)을 결합하여 검색 결과의 정확성과 관련성을 높이는 방식입니다.
    **팁:** 경우에 따라 새로 만든 인덱스를 즉시 사용하지 못할 수 있습니다. 브라우저를 새로 고치면 일반적으로 도움이 되지만, 인덱스를 찾을 수 없는 문제가 계속 발생하면 인덱스가 인식될 때까지 기다려야 할 수 있습니다.

5.  인덱스가 추가되고 채팅 세션이 다시 시작된 후 "뉴욕에서 어디에 머물 수 있나요?(Where can I stay in New York?)"라는 프롬프트를 다시 제출합니다.
6.  인덱스의 데이터를 기반으로 해야 하는 응답을 검토합니다. (Margie's Travel 브로셔에 뉴욕 관련 정보가 있다면 해당 정보가 답변에 포함될 것입니다.)

---

### RAG 클라이언트 앱 만들기 (Create a RAG client app)

이제 작동하는 인덱스가 있으므로 Azure OpenAI SDK를 사용하여 클라이언트 애플리케이션에서 RAG 패턴을 구현할 수 있습니다. 간단한 예제에서 이를 수행하는 코드를 살펴보겠습니다. (Python 부분만 다룹니다.)

*   **핸즈온에서 배울 점:**
    *   VS Code Online (GitHub Codespaces) 환경을 설정하고, RAG 애플리케이션 개발을 위한 Python 프로젝트를 구성합니다.
    *   `.env` 파일에 Azure OpenAI 서비스 및 Azure AI Search 서비스 연결에 필요한 정보(엔드포인트, API 키, 모델 배포 이름, 인덱스 이름)를 설정하는 방법을 배웁니다.
    *   제공된 Python 코드를 통해 RAG 패턴이 어떻게 구현되는지(사용자 질문 임베딩, 관련 문서 검색, 검색된 문서와 질문을 LLM에 전달하여 답변 생성) 간략하게 이해합니다.
    *   작성된 RAG 애플리케이션을 실행하고, 사용자 지정 데이터를 기반으로 답변하는 것을 확인합니다.

#### 애플리케이션 구성 준비 (Prepare the application configuration)

1.  웹 브라우저에서 GitHub (https://github.com)으로 이동합니다.
2.  리포지토리 `microsoftlearning/mslearn-ai-studio`를 검색하거나 직접 주소 (https://github.com/microsoftlearning/mslearn-ai-studio)로 이동합니다.
3.  리포지토리 페이지에서 **Code** 버튼을 클릭한 다음, **Codespaces** 탭을 선택하고 **Create codespace on main**을 클릭하여 새 Codespace를 만듭니다.
4.  Codespace가 준비되면 터미널을 엽니다. 다음 명령을 실행하여 이 실습에 필요한 파일이 있는 폴더로 이동합니다:

    ```bash
    cd mslearn-ai-studio/labfiles/rag-app/python
    ```

5.  터미널에서 다음 명령을 입력하여 Python 가상 환경을 만들고 필요한 OpenAI SDK 라이브러리를 설치합니다:

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt openai
    ```
    *   `requirements.txt` 파일에는 일반적으로 `python-dotenv`, `openai` 등의 필수 패키지가 포함되어 있습니다.

6.  VS Code Online의 탐색기를 사용하여 현재 폴더 (`mslearn-ai-studio/labfiles/rag-app/python/`)에 `.env`라는 새 파일을 만듭니다 (이미 있다면 해당 파일을 엽니다).
7.  `.env` 파일에 다음 내용을 추가하고, 각 자리 표시자를 실제 값으로 바꿉니다:

    ```env
    AZURE_OPENAI_ENDPOINT="your_openai_endpoint"
    AZURE_OPENAI_API_KEY="your_openai_api_key"
    AZURE_OPENAI_CHAT_DEPLOYMENT="your_chat_model"
    AZURE_OPENAI_EMBEDDING_DEPLOYMENT="your_embedding_model"
    AZURE_SEARCH_ENDPOINT="your_search_endpoint"
    AZURE_SEARCH_KEY="your_search_api_key"
    AZURE_SEARCH_INDEX_NAME="your_index"
    ```
    *   `your_openai_endpoint`: Azure AI Foundry portal의 `project` **개요(Overview)** 페이지에 있는 **Azure OpenAI** 기능 탭에서 찾을 수 있는 OpenAI 엔드포인트입니다. (Azure AI Inference 또는 Azure AI Services 기능 탭이 아님)
    *   `your_openai_api_key`: 위와 동일한 위치에서 찾을 수 있는 OpenAI API 키입니다.
    *   `your_chat_model`: Azure AI Foundry portal의 **모델 + 엔드포인트(Models + endpoints)** 페이지에서 `gpt-4o` 모델 배포에 할당한 이름입니다 (기본 이름은 `gpt-4o` 또는 사용자가 지정한 이름, 예: `gpt-4o-chat`).
    *   `your_embedding_model`: **모델 + 엔드포인트(Models + endpoints)** 페이지에서 `text-embedding-ada-002` 모델 배포에 할당한 이름입니다 (기본 이름은 `text-embedding-ada-002` 또는 사용자가 지정한 이름, 예: `ada-002-embed`).
    *   `your_search_endpoint`: Azure AI Search 리소스의 URL입니다. Azure AI Foundry portal의 **관리 센터(Management center)**에서 찾거나, Azure portal에서 해당 AI Search 리소스를 직접 찾아 확인할 수 있습니다.
    *   `your_search_api_key`: Azure AI Search 리소스의 API 키입니다. 위와 동일한 위치에서 찾을 수 있습니다. (일반적으로 관리 키 중 하나를 사용합니다.)
    *   `your_index`: Azure AI Foundry portal의 `project`에 대한 **데이터 + 인덱스(Data + indexes)** 페이지의 인덱스 이름입니다 (이 실습에서는 `brochures-index` 여야 합니다).

    파일을 저장합니다 (Ctrl+S 또는 Cmd+S).

#### RAG 패턴 구현 코드 탐색 (Explore code to implement the RAG pattern)

1.  VS Code Online의 탐색기에서 `rag-app.py` 파일을 찾아 엽니다.
2.  파일의 코드를 검토하고 다음 사항에 유의하십시오:
    *   엔드포인트, 키 및 채팅 모델을 사용하여 Azure OpenAI 클라이언트를 만듭니다.
    *   여행 관련 채팅 솔루션에 적합한 시스템 메시지를 만듭니다.
    *   다음 정보를 추가하여 Azure OpenAI 클라이언트에 프롬프트(사용자 입력을 기반으로 한 시스템 및 사용자 메시지 포함)를 제출합니다:
        *   쿼리할 Azure AI Search 인덱스에 대한 연결 세부 정보.
        *   쿼리를 벡터화하는 데 사용할 임베딩 모델 세부 정보\*.
    *   보강된(grounded) 프롬프트에서 응답을 표시합니다.
    *   응답을 채팅 기록에 추가합니다.

    \* 검색 인덱스에 대한 쿼리는 프롬프트를 기반으로 하며, 인덱싱된 문서에서 관련 텍스트를 찾는 데 사용됩니다. 쿼리를 텍스트로 제출하는 키워드 기반 검색을 사용할 수 있지만, 벡터 기반 검색을 사용하면 더 효율적일 수 있으므로 쿼리 텍스트를 제출하기 전에 임베딩 모델을 사용하여 벡터화합니다.

    *   **핸즈온에서 배울 점:**
        *   코드가 `.env` 파일에서 구성 값을 로드하는 방법.
        *   OpenAI Python SDK를 사용하여 Azure OpenAI 서비스 및 Azure AI Search 서비스와 상호 작용하는 방법.
        *   RAG의 핵심 로직:
            1.  사용자 질문을 받습니다.
            2.  질문을 임베딩 모델을 사용해 벡터로 변환합니다. (코드 내에서 SDK가 처리)
            3.  이 벡터를 사용해 Azure AI Search에서 유사한 문서 청크를 검색합니다. (SDK의 `extensions` 기능 활용)
            4.  검색된 문서 청크와 원래 질문, 시스템 메시지를 결합하여 LLM(`gpt-4o`)에 프롬프트를 구성합니다.
            5.  LLM으로부터 최종 답변을 받습니다.

3.  변경 내용을 저장하지 않고 코드 편집기를 닫으려면 CTRL+Q를 사용합니다 (VS Code Online에서는 탭을 닫으면 됩니다). 터미널은 계속 열어둡니다.

#### 채팅 애플리케이션 실행 (Run the chat application)

1.  VS Code Online 터미널에서 다음 명령을 입력하여 앱을 실행합니다:

    ```bash
    python rag-app.py
    ```

2.  메시지가 표시되면 "건축물을 보러 휴가를 어디로 가야 할까요?(Where should I go on vacation to see architecture?)"와 같은 질문을 입력하고 생성형 AI 모델의 응답을 검토합니다.

    응답에는 답변이 발견된 인덱싱된 데이터를 나타내는 소스 참조가 포함되어 있습니다.

3.  예를 들어 "거기서 어디에 머물 수 있나요?(Where can I stay there?)"와 같은 후속 질문을 시도해 보십시오.

4.  완료되면 "quit"을 입력하여 프로그램을 종료합니다.

---
