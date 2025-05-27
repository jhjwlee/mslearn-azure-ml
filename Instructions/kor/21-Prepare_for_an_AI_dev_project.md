## AI 개발 프로젝트 준비 (Prepare for an AI development project)

이 실습에서는 Azure AI Foundry portal을 사용하여 AI 솔루션을 구축할 준비가 된 프로젝트를 생성합니다.

이 실습은 약 **30분**이 소요됩니다.

**참고:** 이 실습에 사용된 일부 기술은 미리 보기(preview) 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

**학습 목표:**
이 실습을 통해 다음을 수행하는 방법을 배웁니다:
*   Azure AI Foundry portal에 로그인합니다.
*   원하는 AI 모델을 선택하고 이를 사용하기 위한 `project`를 생성합니다.
*   `Azure AI Foundry resource`와 `project`의 관계를 이해합니다.
*   `Management center`를 통해 리소스 및 프로젝트 설정을 검토합니다.
*   Azure Portal에서 생성된 실제 리소스를 확인합니다.
*   `project`의 `Endpoints and keys`를 검토합니다.
*   `Chat playground`를 사용하여 배포된 생성형 AI 모델을 테스트합니다.

**주요 용어 (영문 유지):**
*   Azure AI Foundry portal
*   Project
*   Azure AI Foundry resource
*   gpt-4o
*   Subscription
*   Resource group
*   Region
*   Chat playground
*   Management center
*   Endpoints and keys
*   System message

---

### Azure AI Foundry portal 열기 (Open Azure AI Foundry portal)

먼저 Azure AI Foundry portal에 로그인하여 시작하겠습니다.

1.  웹 브라우저에서 Azure AI Foundry portal (https://ai.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다.
2.  처음 로그인할 때 열리는 모든 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 상단의 Azure AI Foundry 로고를 사용하여 홈페이지로 이동합니다. 홈페이지는 다음 이미지와 유사하게 보일 것입니다 (도움말 창이 열려 있으면 닫으십시오):

    ![Azure AI Foundry portal 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/ai-studio-home.png)

    *   **Azure AI Foundry portal이란?**
        *   Azure AI 개발을 위한 통합 인터페이스입니다. 모델 탐색, `project` 생성 및 관리, 배포, 실험, 그리고 AI 서비스와의 연동 등을 한 곳에서 수행할 수 있는 중앙 허브 역할을 합니다.
    *   **핸즈온에서 배울 점:**
        *   Azure AI Foundry portal에 성공적으로 접속하고 초기 화면 구성을 확인합니다.

3.  홈페이지의 정보를 검토합니다.

---

### 프로젝트 생성 (Create a project)

Azure AI `project`는 AI 개발을 위한 협업 작업 공간을 제공합니다. 작업할 모델을 선택하고 이를 사용할 `project`를 생성하는 것으로 시작하겠습니다.

*   **Project란?**
    *   Azure AI에서 AI 개발 작업을 구성하고 관리하는 기본 단위입니다. 특정 목표(예: 챗봇 개발, 이미지 분석 모델 훈련)를 위한 리소스, 모델, 데이터, 코드, 실험 등을 논리적으로 그룹화합니다.
    *   이를 통해 팀원들과 협업하고, 진행 상황을 추적하며, 재현 가능한 AI 솔루션을 구축할 수 있습니다.

**참고:** AI Foundry `project`는 AI 모델(Azure OpenAI 포함), Azure AI 서비스 및 기타 AI 에이전트 및 채팅 솔루션 개발용 리소스에 대한 액세스를 제공하는 `Azure AI Foundry resource`를 기반으로 할 수 있습니다. 또는 `project`는 보안 스토리지, 컴퓨팅 및 특수 도구를 위한 Azure 리소스에 대한 연결을 포함하는 AI hub resource를 기반으로 할 수 있습니다. `Azure AI Foundry resource` 기반 `project`는 AI 에이전트 또는 채팅 앱 개발을 위한 리소스를 관리하려는 개발자에게 적합합니다. AI hub resource 기반 `project`는 복잡한 AI 솔루션을 작업하는 엔터프라이즈 개발 팀에 더 적합합니다.
*   **본 실습에서는 `Azure AI Foundry resource` 기반 `project`를 사용합니다.**

1.  홈페이지의 **모델 및 기능 탐색(Explore models and capabilities)** 섹션에서 `project`에서 사용할 `gpt-4o` 모델을 검색합니다.
    *   **gpt-4o 모델이란?**
        *   OpenAI에서 개발한 최신의 강력한 멀티모달 모델입니다. 텍스트, 이미지, 오디오 입력을 처리하고 생성할 수 있으며, 이전 모델들보다 더 빠르고 비용 효율적입니다. 이 실습에서는 주로 텍스트 기반 상호작용에 사용됩니다.
    *   **핸즈온에서 배울 점:**
        *   Model catalog에서 사용 가능한 다양한 모델을 검색하고, 특정 모델(`gpt-4o`)을 찾아 선택하는 방법을 익힙니다.

2.  검색 결과에서 `gpt-4o` 모델을 선택하여 세부 정보를 확인한 다음, 모델 페이지 상단에서 **이 모델 사용(Use this model)**을 선택합니다.
    *   **핸즈온에서 배울 점:**
        *   선택한 모델의 상세 정보를 확인하고, 해당 모델을 기반으로 새 `project` 생성을 시작하는 방법을 배웁니다.

3.  `project`를 만들라는 메시지가 표시되면 `project`에 유효한 이름을 입력하고 **고급 옵션(Advanced options)**을 확장합니다.
    *   **핸즈온에서 배울 점:**
        *   `project` 이름을 지정하고, 고급 옵션을 통해 세부적인 리소스 구성을 사용자 정의할 수 있음을 인지합니다.

4.  **사용자 지정(Customize)**을 선택하고 허브에 대해 다음 설정을 지정합니다:
    *   **Azure AI Foundry resource:** `Azure AI Foundry resource`에 대한 유효한 이름
        *   **의미:** 이 `project`를 지원하고 관련된 Azure 서비스(예: Azure OpenAI 모델 배포, Azure AI Services)에 대한 액세스를 제공하는 핵심 리소스입니다. 새 이름을 입력하면 새로 생성됩니다.
    *   **Subscription:** 사용자의 Azure 구독
        *   **의미:** Azure 서비스 사용량 및 청구가 이루어지는 계정 단위입니다. 올바른 구독을 선택해야 합니다.
    *   **Resource group:** `Resource group` 만들기 또는 선택
        *   **의미:** Azure 리소스들을 논리적으로 그룹화하는 컨테이너입니다. 여기서 생성될 `Azure AI Foundry resource` 및 관련 서비스들이 이 `resource group`에 속하게 되어 관리가 용이해집니다.
    *   **Region:** AI 서비스가 지원되는 위치 선택\*
        *   **의미:** AI 모델 및 서비스가 배포되고 실행될 Azure 데이터센터의 지리적 위치입니다. 사용자와의 지연 시간, 데이터 상주 요구 사항, 서비스 가용성 등을 고려하여 선택합니다.
    \* 일부 Azure AI 리소스는 지역별 모델 할당량에 의해 제한됩니다. 실습 후반에 할당량 한도를 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.

    *   **핸즈온에서 배울 점:**
        *   `project` 생성 시 필요한 Azure 인프라(`Azure AI Foundry resource`, `Subscription`, `Resource group`, `Region`) 설정 방법을 이해하고, 각 설정 항목의 의미를 파악합니다.

5.  **만들기(Create)**를 선택하고 선택한 `gpt-4o` 모델 배포를 포함하여 `project`가 생성될 때까지 기다립니다.
    *   **핸즈온에서 배울 점:**
        *   `project` 및 선택한 모델(`gpt-4o`) 배포가 백그라운드에서 자동으로 생성되고 프로비저닝되는 과정을 확인합니다.

6.  `project`가 생성되면 `chat playground`가 자동으로 열리므로 모델을 테스트할 수 있습니다:

    ![Azure AI Foundry 프로젝트 채팅 플레이그라운드 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/chat-playground.png)

    *   **Chat playground란?**
        *   배포된 생성형 AI 모델과 직접 대화형으로 상호작용하며 테스트할 수 있는 웹 기반 인터페이스입니다. 사용자는 시스템 메시지를 설정하고, 사용자 입력을 보내 모델의 응답을 즉시 확인할 수 있어 프롬프트 엔지니어링 및 모델 성능 평가에 매우 유용합니다.
    *   **핸즈온에서 배울 점:**
        *   `project` 및 모델 배포가 완료되면 `chat playground`로 자동 이동되어 즉시 모델 테스트가 가능한 환경이 제공됨을 확인합니다.

7.  왼쪽 탐색 창에서 **개요(Overview)**를 선택하여 `project`의 기본 페이지를 확인합니다. 이 페이지는 다음과 같습니다:

    **참고:** "권한 부족(Insufficient permissions)" 오류가 표시되면 **수정(Fix me)** 버튼을 사용하여 해결하십시오.

    ![Azure AI Foundry 프로젝트 개요 페이지 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/project-overview.png)

    *   **Project Overview 페이지란?**
        *   생성된 `project`의 요약 정보를 보여주는 대시보드입니다. `project`의 주요 리소스, `Endpoints and keys`, 최근 활동, 사용된 모델 등의 정보를 한눈에 파악할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `project`의 핵심 정보가 요약된 `Overview` 페이지의 구성을 살펴보고, 여기서 어떤 정보를 얻을 수 있는지 이해합니다.

8.  왼쪽 탐색 창 하단에서 **관리 센터(Management center)**를 선택합니다. 관리 센터는 리소스 수준과 `project` 수준 모두에서 설정을 구성할 수 있는 곳이며, 두 가지 모두 탐색 창에 표시됩니다.

    ![Azure AI Foundry portal의 관리 센터 페이지 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/management-center.png)

    *   **Management center란?**
        *   `Azure AI Foundry resource` 수준과 개별 `project` 수준의 설정을 구성하고 관리할 수 있는 중앙 허브입니다. 연결(Connections), 컴퓨팅(Compute), 배포(Deployments), 데이터, 접근 제어(Access control) 등 `project` 운영에 필요한 다양한 요소들을 관리합니다.
    *   **Resource level (리소스 수준):** `project`를 지원하기 위해 만들어진 `Azure AI Foundry resource`와 관련됩니다. 이 리소스는 Azure AI Services 및 Azure AI Foundry 모델에 대한 연결을 포함하며 AI 개발 `project`에 대한 사용자 액세스를 관리하는 중앙 집중식 장소를 제공합니다.
    *   **Project level (프로젝트 수준):** 개별 `project`와 관련되며, `project`별 리소스를 추가하고 관리할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `Management center`의 존재와 역할을 이해하고, `Azure AI Foundry resource` 수준과 `project` 수준에서 각각 어떤 설정들을 관리할 수 있는지 구분합니다.

9.  탐색 창에서 사용자의 `Azure AI Foundry resource` 섹션에 있는 **개요(Overview)** 페이지를 선택하여 세부 정보를 확인합니다.
    *   **핸즈온에서 배울 점:**
        *   `Management center` 내에서 `Azure AI Foundry resource`의 세부 정보(예: 구독 ID, 리소스 그룹, 위치 등)를 확인하는 방법을 익힙니다.

10. 리소스와 연결된 **리소스 그룹(Resource group)** 링크를 선택하여 새 브라우저 탭을 열고 Azure Portal로 이동합니다. 메시지가 표시되면 Azure 자격 증명으로 로그인합니다.
    *   **Resource group이란?**
        *   Azure에서 관련된 리소스들을 담는 논리적인 컨테이너입니다. 이 실습에서는 `Azure AI Foundry resource`와 이 리소스에 종속된 다른 서비스들(예: Azure OpenAI 배포, 스토리지 계정 등)이 이 `resource group` 내에 함께 생성되어 배포, 관리, 삭제가 용이해집니다.
    *   **핸즈온에서 배울 점:**
        *   Azure AI Foundry portal에서 Azure Portal로 직접 이동하여 `project`를 지원하는 실제 Azure 리소스들을 확인하는 방법을 배웁니다.

11. Azure Portal에서 `resource group`을 확인하여 `Azure AI Foundry resource` 및 `project`를 지원하기 위해 생성된 Azure 리소스를 확인합니다.

    ![Azure Portal의 Azure AI Foundry 리소스 및 프로젝트 리소스 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/azure-portal-resources.png)

    `project`를 만들 때 선택한 지역에 리소스가 생성되었음을 확인합니다.

    *   **핸즈온에서 배울 점:**
        *   Azure AI Foundry `project` 및 `Azure AI Foundry resource`를 생성하면 백엔드에서 실제로 어떤 Azure 리소스들(예: Azure AI Services 계정, Azure OpenAI 서비스, 스토리지 등)이 생성되는지 Azure Portal에서 직접 확인합니다. 또한, 이 리소스들이 사용자가 지정한 `Region`에 배포되었음을 인지합니다.

12. Azure Portal 탭을 닫고 Azure AI Foundry portal로 돌아갑니다.

---

### 프로젝트 연결 검토 (Review project connections)

Azure AI Foundry `project`와 이 `project`가 속한 `Azure AI Foundry resource`에는 AI 애플리케이션에서 사용할 수 있는 리소스에 대한 연결이 포함됩니다.

*   **Project connections이란?**
    *   `project`가 AI 애플리케이션 개발 및 실행에 필요한 다양한 Azure 서비스(예: Azure OpenAI 모델, Azure AI Services, 스토리지 등)와의 연결 정보를 의미합니다. 이러한 연결은 `project` 생성 시 또는 이후에 구성되어, 개발자가 애플리케이션 코드에서 해당 서비스에 쉽게 접근하고 활용할 수 있도록 합니다.

1.  `Management center` 페이지의 탐색 창에서 `project` 아래의 **프로젝트로 이동(Go to project)**을 선택합니다.
2.  `project` **개요(Overview)** 페이지에서 **엔드포인트 및 키(Endpoints and keys)** 섹션을 확인합니다. 이 섹션에는 애플리케이션 코드에서 다음에 액세스하는 데 사용할 수 있는 엔드포인트 및 권한 부여 키가 포함되어 있습니다:
    *   Azure AI Foundry `project` 및 배포된 모든 모델.
    *   Azure AI Foundry 모델 내의 Azure OpenAI.
    *   Azure AI services.

    *   **Endpoints and keys란?**
        *   **Endpoints (엔드포인트):** 애플리케이션 코드에서 특정 AI 서비스(예: 배포된 `gpt-4o` 모델)에 접근하기 위한 고유한 URL 주소입니다. API 요청을 이 주소로 보내게 됩니다.
        *   **Keys (키):** 해당 엔드포인트에 접근하기 위한 인증 수단(보통 API 키 또는 비밀 키 형태)입니다. 코드에서 이 키를 사용하여 API 요청을 인증하고, 승인된 사용자/애플리케이션만 서비스에 접근할 수 있도록 합니다.
    *   **핸즈온에서 배울 점:**
        *   `project` `Overview` 페이지에서 애플리케이션 개발 시 AI 모델 및 서비스에 프로그래밍 방식으로 접근하는 데 필요한 `Endpoints and keys`를 찾는 방법을 배웁니다. 이 정보는 외부 애플리케이션에서 AI 기능을 통합할 때 필수적입니다.

---

### 생성형 AI 모델 테스트 (Test a generative AI model)

이제 Azure AI Foundry `project`의 구성에 대해 알았으므로, `chat playground`로 돌아가 배포한 모델을 탐색할 수 있습니다.

*   **핸즈온에서 배울 점:**
    *   `Chat playground`를 사용하여 배포된 `gpt-4o` 모델의 기능을 직접 테스트하고, `system message`를 통해 모델의 행동(페르소나, 응답 스타일 등)을 제어하는 기본적인 프롬프트 엔지니어링 방법을 익힙니다.

1.  `project`의 왼쪽 탐색 창에서 **플레이그라운드(Playgrounds)**를 선택합니다.
2.  **채팅(Chat)** 플레이그라운드를 열고 **배포(Deployment)** 섹션에서 `gpt-4o` 모델 배포가 선택되어 있는지 확인합니다.
    *   **핸즈온에서 배울 점:**
        *   `Playgrounds` 메뉴를 통해 `Chat playground`에 접근하고, 테스트할 특정 모델 배포(`gpt-4o`)가 올바르게 선택되었는지 확인하는 방법을 익힙니다. 하나의 `project` 내에 여러 모델 배포가 있을 수 있습니다.

3.  **설정(Setup)** 창의 **모델에게 지침 및 컨텍스트 제공(Give the model instructions and context)** 상자에 다음 지침을 입력합니다:

    ```
    You are a history teacher who can answer questions about past events all around the world.
    ```

    *   **System message (시스템 메시지)란?**
        *   `Chat playground`의 `Setup` 창(또는 API 호출 시 시스템 역할 프롬프트)에서 모델에게 주어지는 초기 지침입니다. 모델의 역할, 페르소나, 응답 스타일, 준수해야 할 규칙 등을 정의하여 대화의 전반적인 맥락과 방향을 설정합니다. 효과적인 `system message`는 모델이 사용자의 의도에 더 부합하는 응답을 생성하도록 유도하는 데 중요합니다.
    *   **핸즈온에서 배울 점:**
        *   `system message`를 입력하여 모델에게 특정 역할("역사 선생님")을 부여하고, 해당 역할에 맞는 응답을 하도록 유도하는 방법을 실습합니다.

4.  **변경 내용 적용(Apply the changes)**을 눌러 시스템 메시지를 업데이트합니다.
5.  채팅 창에 "스코틀랜드 역사의 주요 사건은 무엇인가요?(What are the key events in the history of Scotland?)"와 같은 질문을 입력하고 응답을 확인합니다:

    ![Azure AI Foundry portal의 플레이그라운드 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/playground-test.png)

    *   **핸즈온에서 배울 점:**
        *   `system message`가 적용된 모델에게 관련된 질문을 하고, 모델이 설정된 페르소나에 따라 응답하는지 관찰합니다. 이를 통해 `system message`의 효과를 직접 확인하고, 모델과의 상호작용을 경험합니다.

---

### 요약 (Summary)

이 실습에서는 Azure AI Foundry를 탐색하고, `project` 및 관련 리소스를 만들고 관리하는 방법을 살펴보았습니다.

**핵심 요약 (Key Takeaways):**
*   Azure AI Foundry portal은 AI 개발을 위한 중앙 집중식 작업 공간입니다.
*   `Project`는 AI 모델, 데이터, 코드 등을 포함하는 AI 개발의 기본 단위입니다.
*   `Azure AI Foundry resource`는 `project`를 지원하며 Azure OpenAI 모델 및 Azure AI 서비스에 대한 액세스를 제공합니다.
*   `gpt-4o`와 같은 모델을 선택하여 `project` 내에 배포할 수 있습니다.
*   `Management center`를 통해 `Azure AI Foundry resource` 및 `project` 수준의 설정을 관리합니다.
*   Azure Portal에서 `project`와 관련된 실제 Azure 리소스(예: `Resource group` 내의 서비스)를 확인할 수 있습니다.
*   `Project Overview` 페이지에서 `Endpoints and keys`를 찾아 애플리케이션에서 AI 기능을 통합할 수 있습니다.
*   `Chat playground`는 배포된 모델을 쉽게 테스트하고, `system message`를 통해 모델의 동작을 제어할 수 있는 유용한 도구입니다.
