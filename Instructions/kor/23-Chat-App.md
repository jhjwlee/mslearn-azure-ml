## 생성형 AI 챗 앱 만들기 (Create a generative AI chat app)

이 실습에서는 Azure AI Foundry SDK를 사용하여 `project`에 연결하고 언어 모델과 채팅하는 간단한 챗 앱을 만듭니다.

이 실습은 약 **40분**이 소요됩니다.

**참고:** 이 실습은 사전 릴리스 SDK를 기반으로 하며 변경될 수 있습니다. 필요한 경우 특정 버전의 패키지를 사용했으며, 이는 사용 가능한 최신 버전과 다를 수 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

**학습 목표:**
이 실습을 통해 다음을 수행하는 방법을 배웁니다:
*   Azure AI Foundry `project`에 `gpt-4o` 모델을 배포합니다.
*   VS Code Online (GitHub Codespaces) 환경을 설정하여 Python 애플리케이션을 개발합니다.
*   Azure AI Foundry SDK를 사용하여 `project` 엔드포인트에 연결합니다.
*   Azure AI Model Inference SDK를 사용하여 배포된 모델과 상호 작용하는 채팅 애플리케이션 코드를 작성합니다.
*   시스템 메시지, 사용자 메시지, 어시스턴트 메시지를 활용하여 대화 흐름을 관리합니다.
*   작성한 애플리케이션을 실행하여 생성형 AI 모델과 채팅합니다.

**주요 용어 (영문 유지):**
*   Azure AI Foundry portal
*   Azure AI Foundry SDK
*   Azure AI Model Inference SDK
*   Project
*   gpt-4o
*   Model deployment
*   Project endpoint
*   VS Code Online (GitHub Codespaces)
*   `DefaultAzureCredential`
*   `AIProjectClient`
*   `get_chat_completions_client`
*   `SystemMessage`, `UserMessage`, `AssistantMessage`
*   `.env` file
*   `az login`

---

### Azure AI Foundry 프로젝트에 모델 배포 (Deploy a model in an Azure AI Foundry project)

먼저 Azure AI Foundry `project`에 모델을 배포하는 것으로 시작하겠습니다. (이 부분은 이전 실습과 유사합니다.)

1.  웹 브라우저에서 Azure AI Foundry portal (https://ai.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 모든 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 상단의 Azure AI Foundry 로고를 사용하여 홈페이지로 이동합니다. 홈페이지는 다음 이미지와 유사하게 보일 것입니다 (도움말 창이 열려 있으면 닫으십시오):

    ![Azure AI Foundry portal 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/ai-studio-home.png)

2.  홈페이지의 **모델 및 기능 탐색(Explore models and capabilities)** 섹션에서 `project`에서 사용할 `gpt-4o` 모델을 검색합니다.
3.  검색 결과에서 `gpt-4o` 모델을 선택하여 세부 정보를 확인한 다음, 모델 페이지 상단에서 **이 모델 사용(Use this model)**을 선택합니다.
4.  `project`를 만들라는 메시지가 표시되면 `project`에 유효한 이름을 입력하고 **고급 옵션(Advanced options)**을 확장합니다.
5.  **사용자 지정(Customize)**을 선택하고 허브에 대해 다음 설정을 지정합니다:
    *   **Azure AI Foundry resource:** `Azure AI Foundry resource`에 대한 유효한 이름
    *   **Subscription:** 사용자의 Azure 구독
    *   **Resource group:** `Resource group` 만들기 또는 선택
    *   **Region:** AI 서비스가 지원되는 위치 선택\*
    \* 일부 Azure AI 리소스는 지역별 모델 할당량에 의해 제한됩니다. 실습 후반에 할당량 한도를 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.

6.  **만들기(Create)**를 선택하고 선택한 `gpt-4o` 모델 배포를 포함하여 `project`가 생성될 때까지 기다립니다.
7.  `project`가 생성되면 `chat playground`가 자동으로 열립니다.
    *   **핸즈온에서 배울 점:**
        *   `project` 생성 시 선택한 모델(`gpt-4o`)이 자동으로 `project` 내에 배포되는 과정을 이해합니다.

8.  **설정(Setup)** 창에서 모델 배포 이름(기본적으로 `gpt-4o` 여야 함)을 확인합니다. 왼쪽 탐색 창의 **모델 및 엔드포인트(Models and endpoints)** 페이지에서 배포를 확인하여 이를 확인할 수 있습니다.
    *   **Model deployment 이름이란?**
        *   `project` 내에 배포된 특정 모델 인스턴스를 식별하는 고유한 이름입니다. 코드에서 이 이름을 사용하여 특정 모델 배포와 상호작용합니다.
    *   **핸즈온에서 배울 점:**
        *   배포된 모델의 이름을 확인하는 방법을 배우고, 이 이름이 나중에 코드에서 사용될 것임을 인지합니다.

9.  왼쪽 탐색 창에서 **개요(Overview)**를 선택하여 `project`의 기본 페이지를 확인합니다. 이 페이지는 다음과 같습니다:

    **참고:** "**권한 부족(Insufficient permissions)**" 오류가 표시되면 **수정(Fix me)** 버튼을 사용하여 해결하십시오.

    ![Azure AI Foundry 프로젝트 개요 페이지 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/project-overview.png)

    *   **핸즈온에서 배울 점:**
        *   `project` `Overview` 페이지에서 애플리케이션 개발에 필요한 주요 정보(특히, `Project endpoint`)를 찾을 수 있음을 확인합니다.

---

### 모델과 채팅할 클라이언트 애플리케이션 만들기 (Create a client application to chat with the model)

이제 모델을 배포했으므로 Azure AI Foundry 및 Azure AI Model Inference SDK를 사용하여 모델과 채팅하는 애플리케이션을 개발할 수 있습니다.

**팁:** Python 또는 Microsoft C#을 사용하여 솔루션을 개발할 수 있습니다. 선택한 언어에 대한 해당 섹션의 지침을 따르십시오. (이 문서에서는 Python만 다룹니다.)

#### 애플리케이션 구성 준비 (Prepare the application configuration)

1.  Azure AI Foundry portal에서 `project`의 **개요(Overview)** 페이지를 확인합니다.
2.  **프로젝트 세부 정보(Project details)** 영역에서 **Azure AI Foundry project endpoint**를 확인하고 복사해 둡니다. 클라이언트 애플리케이션에서 `project`에 연결하는 데 이 엔드포인트를 사용합니다.
    *   **Project endpoint란?**
        *   외부 애플리케이션이 Azure AI Foundry `project`의 리소스 및 배포된 모델에 접근하기 위한 고유한 URL 주소입니다. SDK는 이 엔드포인트를 사용하여 `project`와의 연결을 설정합니다.
    *   **핸즈온에서 배울 점:**
        *   코드에서 `project`에 연결하는 데 필요한 `project endpoint` 값을 찾는 방법을 배웁니다.

3.  웹 브라우저에서 GitHub (https://github.com)으로 이동합니다.
4.  리포지토리 `microsoftlearning/mslearn-ai-studio`를 검색하거나 직접 주소 (https://github.com/microsoftlearning/mslearn-ai-studio)로 이동합니다.
5.  리포지토리 페이지에서 **Code** 버튼을 클릭한 다음, **Codespaces** 탭을 선택하고 **Create codespace on main** (또는 유사한 옵션)을 클릭하여 새 Codespace를 만듭니다.
    *   **GitHub Codespaces (VS Code Online)란?**
        *   클라우드에서 호스팅되는 개발 환경입니다. 웹 브라우저를 통해 VS Code와 유사한 편집기 및 터미널 환경을 제공하여, 로컬에 개발 환경을 구성할 필요 없이 코드를 작성하고 실행할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   GitHub Codespaces를 사용하여 클라우드 기반 개발 환경을 설정하고, 실습용 코드가 포함된 리포지토리를 여는 방법을 배웁니다.

6.  Codespace가 준비될 때까지 기다립니다. VS Code와 유사한 웹 기반 편집기가 열립니다. Codespace가 열리면, 터미널이 자동으로 열리거나, 메뉴 (Terminal > New Terminal)를 통해 열 수 있습니다.

7.  터미널에서 다음 명령을 실행하여 이 실습에 필요한 파일이 있는 폴더로 이동합니다 (Python 선택):

    ```bash
    cd mslearn-ai-studio/labfiles/chat-app/python
    ```
    *   **핸즈온에서 배울 점:**
        *   VS Code Online 터미널을 사용하여 Codespace 내의 특정 폴더로 이동하는 방법을 익힙니다.

8.  터미널에서 다음 명령을 입력하여 Python 가상 환경을 만들고 필요한 라이브러리를 설치합니다:

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```
    *   **가상 환경 (.venv)이란?**
        *   프로젝트별로 격리된 Python 실행 환경을 만들어 패키지 충돌을 방지하고, 프로젝트 의존성을 관리하기 용이하게 합니다.
    *   **필수 라이브러리:**
        *   `python-dotenv`: `.env` 파일에서 환경 변수(예: API 키, 엔드포인트)를 로드합니다.
        *   `azure-identity`: Azure 서비스에 대한 인증을 처리합니다. `DefaultAzureCredential`을 제공하여 다양한 인증 방법을 자동으로 시도합니다.
        *   `azure-ai-projects`: Azure AI Foundry `project`와 상호작용하기 위한 SDK입니다. `AIProjectClient`를 제공합니다.
        *   `azure-ai-inference`: Azure AI 모델(특히 생성형 AI 모델)과의 추론(inference) 작업을 위한 SDK입니다. 채팅 완료(chat completions) 등의 기능을 제공합니다.
    *   **핸즈온에서 배울 점:**
        *   Python 프로젝트를 위한 가상 환경을 설정하고, Azure AI SDK 관련 패키지를 설치하는 방법을 배웁니다. 각 패키지의 역할을 이해합니다.

9.  VS Code Online의 탐색기(Explorer)를 사용하여 현재 폴더 (`mslearn-ai-studio/labfiles/chat-app/python/`)에 `.env`라는 새 파일을 만듭니다.
10. 생성된 `.env` 파일에 다음 내용을 추가하고, 자리 표시자를 실제 값으로 바꿉니다:

    ```env
    PROJECT_ENDPOINT="your_project_endpoint"
    MODEL_DEPLOYMENT_NAME="your_model_deployment"
    ```
    *   `your_project_endpoint`: 2단계에서 Azure AI Foundry portal의 `project` `Overview` 페이지에서 복사한 **Azure AI Foundry project endpoint** 값으로 바꿉니다.
    *   `your_model_deployment`: Azure AI Foundry portal의 `chat playground` **설정(Setup)** 창 또는 **모델 및 엔드포인트(Models and endpoints)** 페이지에서 확인한 `gpt-4o` 모델 배포 이름으로 바꿉니다 (일반적으로 `gpt-4o` 입니다).
    *   파일을 저장합니다 (Ctrl+S 또는 Cmd+S).
    *   **.env 파일이란?**
        *   애플리케이션 구성 정보(특히 민감한 정보나 환경에 따라 달라지는 값)를 코드와 분리하여 저장하는 데 사용되는 텍스트 파일입니다. `python-dotenv` 라이브러리를 통해 이 파일의 값들을 환경 변수로 로드할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `.env` 파일을 생성하고, `project endpoint`와 `model deployment name` 같은 중요한 구성 정보를 안전하게 저장하는 방법을 배웁니다.

#### 프로젝트에 연결하고 모델과 채팅하는 코드 작성 (Write code to connect to your project and chat with your model)

**팁:** 코드를 추가할 때 올바른 들여쓰기를 유지하십시오.

1.  VS Code Online의 탐색기에서 `chat-app.py` 파일을 찾아 엽니다.
2.  파일 코드에서 파일 상단에 필요한 SDK 네임스페이스를 가져오기 위해 추가된 기존 문을 확인합니다. 그런 다음 `Add references` 주석을 찾아 이전에 설치한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다:

    ```python
    # Add references
    import os # 운영 체제와 상호 작용하기 위한 모듈
    from dotenv import load_dotenv # .env 파일에서 환경 변수를 로드하기 위한 함수
    from azure.identity import DefaultAzureCredential # Azure 서비스에 대한 기본 자격 증명 공급자
    from azure.ai.projects import AIProjectClient # Azure AI Project와 상호 작용하기 위한 클라이언트
    # Azure AI Inference SDK에서 채팅 메시지 유형들을 가져옴
    from azure.ai.inference.models import SystemMessage, UserMessage, AssistantMessage
    ```
    *   **핸즈온에서 배울 점:**
        *   Python 코드에서 필요한 라이브러리(모듈 및 특정 클래스/함수)를 `import` 하는 방법을 이해합니다.

3.  `main` 함수에서 `Get configuration settings` 주석 아래에서 코드가 구성 파일에 정의한 `project` 연결 문자열과 모델 배포 이름 값을 로드하는 것을 확인합니다.

    ```python
    # Get configuration settings
    load_dotenv() # .env 파일의 내용을 환경 변수로 로드
    project_connection = os.getenv("PROJECT_ENDPOINT") # 환경 변수에서 프로젝트 엔드포인트 값 가져오기
    model_deployment = os.getenv("MODEL_DEPLOYMENT_NAME") # 환경 변수에서 모델 배포 이름 가져오기
    ```
    *   **핸즈온에서 배울 점:**
        *   `dotenv` 라이브러리를 사용하여 `.env` 파일에 저장된 구성 값을 Python 코드로 안전하게 로드하는 방법을 배웁니다.

4.  `Initialize the project client` 주석을 찾아 현재 로그인한 Azure 자격 증명을 사용하여 Azure AI Foundry `project`에 연결하는 다음 코드를 추가합니다:

    **팁:** 코드에 대해 올바른 들여쓰기 수준을 유지하도록 주의하십시오.

    ```python
    # Initialize the project client
    # DefaultAzureCredential을 사용하여 Azure에 인증합니다.
    # 특정 자격 증명 유형(환경 변수, 관리 ID)은 제외할 수 있습니다.
    credential = DefaultAzureCredential(
        exclude_environment_credential=True, # 환경 변수 기반 자격 증명 사용 안 함
        exclude_managed_identity_credential=True # 관리 ID 기반 자격 증명 사용 안 함
    )

    # AIProjectClient 인스턴스를 생성합니다.
    # project_connection은 .env 파일에서 로드한 프로젝트 엔드포인트입니다.
    projectClient = AIProjectClient(
        endpoint=project_connection, # 연결할 프로젝트의 엔드포인트 URI
        credential=credential, # 인증에 사용할 자격 증명
    )
    ```
    *   **`DefaultAzureCredential`이란?**
        *   Azure SDK에서 다양한 인증 메커니즘(예: Azure CLI 로그인, VS Code 로그인, 환경 변수, 관리 ID)을 순서대로 시도하여 자동으로 자격 증명을 가져오는 편리한 방법입니다. 여기서는 `az login`을 통해 로그인한 자격 증명을 사용하게 됩니다.
    *   **`AIProjectClient`란?**
        *   Azure AI Foundry `project`와 프로그래밍 방식으로 상호 작용하기 위한 기본 클라이언트 객체입니다. 이 객체를 통해 `project` 내의 모델 추론 서비스 등에 접근할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `DefaultAzureCredential`을 사용하여 Azure 인증을 처리하고, `AIProjectClient`를 초기화하여 Azure AI Foundry `project`에 연결하는 방법을 배웁니다.

5.  `Get a chat client` 주석을 찾아 모델과 채팅하기 위한 클라이언트 객체를 만드는 다음 코드를 추가합니다:

    ```python
    # Get a chat client
    # projectClient의 inference 속성을 통해 채팅 완료 클라이언트를 가져옵니다.
    # 이 클라이언트는 배포된 모델과 채팅 상호 작용을 수행하는 데 사용됩니다.
    chat_client = projectClient.get_chat_completions_client(deployment_name=model_deployment)
    ```
    *   **`get_chat_completions_client()`란?**
        *   `AIProjectClient`의 메서드로, 특정 모델 배포(`deployment_name`으로 지정)와 채팅 형식의 상호작용(질문-응답)을 할 수 있는 클라이언트를 반환합니다. 이 클라이언트를 통해 모델에 프롬프트를 보내고 응답을 받을 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `AIProjectClient`를 사용하여 특정 모델 배포와 채팅하기 위한 `chat_client`를 얻는 방법을 배웁니다.

    **참고:** 이 코드는 Azure AI Foundry `project` 클라이언트를 사용하여 `project`와 연결된 기본 Azure AI Model Inference 서비스 엔드포인트에 대한 보안 연결을 만듭니다. Azure AI Foundry portal 또는 Azure portal의 해당 Azure AI Services 리소스 페이지에 표시된 서비스 연결에 대한 엔드포인트 URI를 지정하고 인증 키 또는 Entra 자격 증명 토큰을 사용하여 Azure AI Model Inference SDK를 사용하여 엔드포인트에 직접 연결할 수도 있습니다. Azure AI Model Inferencing 서비스 연결에 대한 자세한 내용은 [Azure AI Model Inference API](https://aka.ms/azure-ai-inference-api)를 참조하십시오.

6.  `Initialize prompt with system message` 주석을 찾아 시스템 프롬프트로 메시지 컬렉션을 초기화하는 다음 코드를 추가합니다.

    ```python
    # Initialize prompt with system message
    # 대화 기록을 저장할 리스트를 초기화하고, 시스템 메시지를 추가합니다.
    # SystemMessage는 모델에게 역할이나 행동 지침을 제공합니다.
    messages = [
        SystemMessage("You are a helpful AI assistant that answers questions.")
    ]
    ```
    *   **`SystemMessage`란?**
        *   대화형 AI 모델에게 전반적인 지침, 역할, 또는 페르소나를 설정하는 메시지입니다. 대화의 시작 부분에 포함되어 모델의 응답 스타일과 내용에 영향을 줍니다.
    *   **핸즈온에서 배울 점:**
        *   `SystemMessage`를 사용하여 모델의 초기 행동을 설정하고, 대화 기록을 관리하기 위한 메시지 리스트를 초기화하는 방법을 배웁니다.

7.  코드는 사용자가 "quit"을 입력할 때까지 프롬프트를 입력할 수 있도록 루프를 포함합니다. 그런 다음 루프 섹션에서 `Get a chat completion` 주석을 찾아 사용자 입력을 프롬프트에 추가하고, 모델에서 완료를 검색하고, 완료를 프롬프트에 추가하는 다음 코드를 추가합니다 (향후 반복을 위해 채팅 기록을 유지하도록).

    ```python
    # 루프 내:
    # Get a chat completion
    # 사용자의 입력을 UserMessage 형태로 메시지 리스트에 추가합니다.
    messages.append(UserMessage(user_input))

    # chat_client.complete를 호출하여 모델로부터 응답을 받습니다.
    # messages에는 시스템 메시지, 이전 사용자 입력 및 어시스턴트 응답이 포함됩니다.
    response = chat_client.complete(messages=messages)
    # 모델의 응답은 response.choices[0].message.content에 있습니다.
    completion = response.choices[0].message.content
    print(f"Assistant: {completion}") # 모델의 응답을 출력

    # 모델의 응답(completion)을 AssistantMessage 형태로 메시지 리스트에 추가합니다.
    # 이는 다음 대화 턴에서 컨텍스트로 사용됩니다.
    messages.append(AssistantMessage(content=completion))
    ```
    *   **`UserMessage`란?**
        *   사용자가 AI 모델에게 입력하는 메시지(질문, 요청 등)를 나타냅니다.
    *   **`AssistantMessage`란?**
        *   AI 모델이 생성한 응답을 나타냅니다.
    *   **`chat_client.complete(messages=messages)`:**
        *   이 함수는 현재까지의 대화 기록(`messages` 리스트)을 모델에 전달하고, 모델의 다음 응답을 요청합니다.
    *   **대화 기록 유지:**
        *   사용자 입력(`UserMessage`)과 모델 응답(`AssistantMessage`)을 `messages` 리스트에 계속 추가함으로써, 모델이 이전 대화의 맥락을 이해하고 더 일관성 있는 응답을 생성하도록 합니다.
    *   **핸즈온에서 배울 점:**
        *   사용자 입력을 `UserMessage`로, 모델 응답을 `AssistantMessage`로 처리하는 방법을 배웁니다.
        *   `chat_client.complete()` 메서드를 사용하여 모델로부터 응답을 받고, 대화 기록을 유지하여 연속적인 채팅을 구현하는 방법을 이해합니다.

8.  VS Code Online 편집기에서 CTRL+S (또는 Cmd+S)를 사용하여 코드 파일의 변경 내용을 저장합니다.

#### Azure에 로그인하고 앱 실행 (Sign into Azure and run the app)

1.  VS Code Online 터미널에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```bash
    az login --use-device-code
    ```
    *   **`az login --use-device-code`란?**
        *   Azure CLI를 통해 Azure에 로그인하는 명령입니다. `--use-device-code` 옵션은 브라우저 기반 인증 흐름을 사용하며, 특히 Codespaces와 같은 환경에서 유용합니다.
    *   **핸즈온에서 배울 점:**
        *   VS Code Online 터미널에서 Azure CLI를 사용하여 Azure 계정에 로그인하는 방법을 배웁니다.

2.  메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드를 입력한 다음 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고, 메시지가 표시되면 Azure AI Foundry 허브가 포함된 구독을 선택합니다. (Azure에 이미 로그인한 경우에도 이 단계를 수행해야 할 수 있습니다.)

    **참고:** 대부분의 경우 `az login`만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 `–tenant` 매개변수를 사용하여 테넌트를 지정해야 할 수 있습니다. 자세한 내용은 [Azure CLI를 사용하여 대화형으로 Azure에 로그인](https://learn.microsoft.com/cli/azure/authenticate-azure-cli#sign-in-interactively)을 참조하십시오.

3.  로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다:

    ```bash
    python chat-app.py
    ```
    *   **핸즈온에서 배울 점:**
        *   작성한 Python 채팅 애플리케이션을 터미널에서 실행하는 방법을 배웁니다.

4.  메시지가 표시되면 "지구상에서 가장 빠른 동물은 무엇인가요?(What is the fastest animal on Earth?)"와 같은 질문을 입력하고 생성형 AI 모델의 응답을 검토합니다.
5.  "어디서 볼 수 있나요?(Where can I see one?)" 또는 "멸종 위기인가요?(Are they endangered?)"와 같은 후속 질문을 시도해 보십시오. 대화는 각 반복에 대한 컨텍스트로 채팅 기록을 사용하여 계속되어야 합니다.
6.  완료되면 "quit"을 입력하여 프로그램을 종료합니다.

    *   **핸즈온에서 배울 점:**
        *   실행 중인 애플리케이션과 상호 작용하여 모델의 응답을 확인하고, 대화 기록이 유지되어 연속적인 대화가 가능한지 테스트합니다.

**팁:** 비율 제한이 초과되어 앱이 실패하면 몇 초 기다렸다가 다시 시도하십시오. 구독에 사용 가능한 할당량이 부족하면 모델이 응답하지 못할 수 있습니다.

---

### 요약 (Summary)

이 실습에서는 Azure AI Foundry SDK를 사용하여 Azure AI Foundry `project`에 배포한 생성형 AI 모델용 클라이언트 애플리케이션을 만들었습니다. VS Code Online 환경에서 Python을 사용하여 `project`에 연결하고, 모델과 채팅하며, 대화 기록을 관리하는 방법을 배웠습니다.
