프롬프트 흐름을 사용하여 챗 앱에서 대화 관리 (Use a prompt flow to manage conversation in a chat app)

이 실습에서는 Azure AI Foundry portal의 prompt flow를 사용하여 사용자 프롬프트와 채팅 기록을 입력으로 사용하고 Azure OpenAI의 GPT 모델을 사용하여 출력을 생성하는 사용자 지정 챗 앱을 만듭니다.

이 실습은 약 30분이 소요됩니다.

참고: 이 실습에 사용된 일부 기술은 미리 보기(preview) 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

학습 목표:
이 실습을 통해 다음을 수행하는 방법을 배웁니다:

Azure AI Foundry hub resource를 기반으로 하는 project를 생성합니다.

prompt flow 자산에 대한 스토리지 접근 권한을 구성합니다.

project 내에 생성형 AI 모델(gpt-4o)을 배포합니다.

prompt flow를 사용하여 채팅 흐름을 생성하고 구성합니다.

입력(사용자 질문, 채팅 기록)과 출력(모델 답변)을 정의합니다.

Chat LLM 도구를 설정하고 시스템 프롬프트를 사용자 정의합니다.

생성된 prompt flow를 테스트합니다.

prompt flow를 엔드포인트로 배포하여 애플리케이션에서 사용할 수 있도록 합니다.

주요 용어 (영문 유지):

Azure AI Foundry portal

Azure AI Foundry hub resource

Project

Prompt flow

System assigned identity

Storage account

Access Control (IAM)

Storage Blob Data Reader

Models + endpoints

Deploy base model

gpt-4o

Deployment name

Tokens per Minute Rate Limit (TPM)

Content filter

Chat flow template

Compute session

Inputs, Outputs, Tools (in prompt flow)

Chat LLM tool

Connection (in Chat LLM tool)

Api (chat)

response_format

Prompt (system message, user message, chat history)

Endpoint

Virtual machine (Standard_DS3_v2)

Instance count

Consume page

Azure AI Foundry 허브 및 프로젝트 만들기 (Create an Azure AI Foundry hub and project)

이 실습에서 사용할 Azure AI Foundry의 기능에는 Azure AI Foundry hub resource를 기반으로 하는 project가 필요합니다.

Azure AI Foundry hub resource란?

이전 실습에서 다룬 Azure AI Foundry resource보다 더 포괄적인 개념으로, 엔터프라이즈급 AI 개발을 위한 중앙 집중식 관리 허브입니다. 여러 project를 호스팅하고, 공유 가능한 리소스(연결, 컴퓨팅, 모델 등)와 거버넌스 기능을 제공합니다. 특히 prompt flow와 같은 고급 기능을 사용하기 위해 필요합니다.

핸즈온에서 배울 점:

prompt flow 사용을 위해 Azure AI Foundry hub resource 기반의 project를 생성하는 방법을 익힙니다. 일반적인 Azure AI Foundry resource 기반 project와 차이가 있음을 인지합니다.

웹 브라우저에서 Azure AI Foundry portal (https://ai.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 모든 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 상단의 Azure AI Foundry 로고를 사용하여 홈페이지로 이동합니다. 홈페이지는 다음 이미지와 유사하게 보일 것입니다 (도움말 창이 열려 있으면 닫으십시오):

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/ai-studio-home.png)

브라우저에서 https://ai.azure.com/managementCenter/allResources 로 이동하여 **만들기(Create)**를 선택합니다. 그런 다음 새 AI hub resource를 만드는 옵션을 선택합니다.

핸즈온에서 배울 점:

Management Center를 통해 새로운 AI hub resource 생성 프로세스를 시작하는 방법을 배웁니다.

프로젝트 만들기(Create a project) 마법사에서 project에 유효한 이름을 입력하고, 기존 허브가 제안되면 새 허브를 만드는 옵션을 선택하고 **고급 옵션(Advanced options)**을 확장하여 project에 대해 다음 설정을 지정합니다:

Subscription: 사용자의 Azure 구독

Resource group: Resource group 만들기 또는 선택

Hub name: 허브에 대한 유효한 이름

의미: 생성될 AI hub resource의 고유한 이름입니다.

Location: East US 2 또는 Sweden Central*

의미: AI hub resource 및 관련 서비스가 배포될 Azure 데이터센터의 지리적 위치입니다. prompt flow와 같은 특정 기능은 특정 지역에서만 지원될 수 있으므로 주의해야 합니다.
* 일부 Azure AI 리소스는 지역별 모델 할당량에 의해 제한됩니다. 실습 후반에 할당량 한도를 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.

핸즈온에서 배울 점:

AI hub resource와 동시에 project를 생성하는 과정을 이해하고, 필요한 설정 항목(구독, 리소스 그룹, 허브 이름, 위치)을 구성합니다.

project가 생성될 때까지 기다립니다.

리소스 권한 부여 구성 (Configure resource authorization)

Azure AI Foundry의 prompt flow 도구는 prompt flow를 정의하는 파일 기반 자산을 Blob 스토리지의 폴더에 만듭니다. prompt flow를 탐색하기 전에 Azure AI Foundry 리소스가 Blob 저장소를 읽을 수 있도록 필요한 액세스 권한을 갖도록 하겠습니다.

Prompt flow 자산과 Blob 스토리지:

Prompt flow는 다이어그램, 코드 파일, 구성 파일 등 여러 파일로 구성됩니다. 이러한 파일들은 Azure Blob 스토리지에 저장되어 관리됩니다.

System assigned identity와 역할 기반 접근 제어(RBAC):

Azure 리소스(여기서는 Azure AI Foundry resource)가 다른 Azure 리소스(여기서는 Blob 스토리지)에 안전하게 접근하기 위해 관리 ID(Managed Identity)를 사용합니다. 시스템 할당 관리 ID는 Azure가 자동으로 생성하고 관리하는 ID입니다. 이 ID에 적절한 역할(예: Storage Blob Data Reader)을 할당하여 필요한 권한만 부여합니다.

핸즈온에서 배울 점:

Azure AI Foundry resource의 시스템 할당 관리 ID를 활성화하고, 이 ID에 Storage Blob Data Reader 역할을 할당하여 prompt flow 자산이 저장된 Blob 스토리지에 대한 읽기 권한을 부여하는 방법을 배웁니다. 이는 prompt flow가 정상적으로 작동하기 위한 필수 구성입니다.

새 브라우저 탭에서 Azure portal (https://portal.azure.com)을 열고, 메시지가 표시되면 Azure 자격 증명으로 로그인합니다. 그런 다음 Azure AI 허브 리소스가 포함된 resource group을 확인합니다.

허브에 대한 Azure AI Foundry resource를 선택하여 엽니다. 그런 다음 리소스 관리(Resource Management) 섹션을 확장하고 ID(Identity) 페이지를 선택합니다:

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/use-prompt-flow-manage-conversation-chat-app/media/system-identity.png)

시스템 할당 ID의 상태가 **꺼짐(Off)**이면 **켜짐(On)**으로 전환하고 변경 내용을 저장합니다. 그런 다음 상태 변경이 확인될 때까지 기다립니다.

System assigned identity란?

Azure 리소스(이 경우 Azure AI Foundry resource)에 자동으로 할당되는 Azure Active Directory의 ID입니다. 이 ID를 사용하여 다른 Azure 서비스에 인증하고 권한을 부여받을 수 있으므로, 코드에 자격 증명을 하드코딩할 필요가 없습니다.

resource group 페이지로 돌아가서 허브에 대한 Storage account 리소스를 선택하고 해당 액세스 제어 (IAM)(Access Control (IAM)) 페이지를 확인합니다:

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/use-prompt-flow-manage-conversation-chat-app/media/storage-access-control.png)

Azure AI Foundry 리소스에서 사용하는 관리 ID에 대해 Storage Blob 데이터 판독기(Storage Blob Data Reader) 역할에 역할 할당을 추가합니다:

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/use-prompt-flow-manage-conversation-chat-app/media/storage-role-assignment.png)

Storage Blob Data Reader 역할이란?

이 역할을 할당받은 ID는 스토리지 계정 내의 Blob 데이터(파일 내용)를 읽을 수 있는 권한을 갖게 됩니다. Prompt flow는 저장된 흐름 정의 파일을 읽어야 하므로 이 권한이 필요합니다.

Azure AI Foundry 관리 ID가 스토리지 계정의 Blob을 읽을 수 있도록 역할 액세스를 검토하고 할당한 후 Azure portal 탭을 닫고 Azure AI Foundry portal로 돌아갑니다.

생성형 AI 모델 배포 (Deploy a generative AI model)

이제 prompt flow 애플리케이션을 지원할 생성형 AI 언어 모델을 배포할 준비가 되었습니다.

핸즈온에서 배울 점:

prompt flow에서 사용할 언어 모델(gpt-4o)을 project 내에 배포하고, 배포 시 주요 설정(TPM, 콘텐츠 필터 등)을 구성하는 방법을 익힙니다.

project의 왼쪽 창에 있는 내 자산(My assets) 섹션에서 모델 + 엔드포인트(Models + endpoints) 페이지를 선택합니다.

모델 + 엔드포인트(Models + endpoints) 페이지의 모델 배포(Model deployments) 탭에 있는 + 모델 배포(+ Deploy model) 메뉴에서 **기본 모델 배포(Deploy base model)**를 선택합니다.

목록에서 gpt-4o 모델을 검색한 다음 선택하고 확인합니다.

배포 세부 정보에서 **사용자 지정(Customize)**을 선택하여 다음 설정으로 모델을 배포합니다:

Deployment name: 모델 배포에 대한 유효한 이름

Deployment type: Global Standard

Automatic version update: Enabled (활성화됨)

Model version: 사용 가능한 최신 버전 선택

Connected AI resource: Azure OpenAI 리소스 연결 선택

의미: 이 모델 배포가 사용할 실제 Azure OpenAI 서비스 인스턴스와의 연결을 지정합니다. 일반적으로 허브 생성 시 함께 프로비저닝됩니다.

Tokens per Minute Rate Limit (thousands): 50K (또는 구독에서 사용 가능한 최대값이 50K 미만인 경우 해당 값)

의미 (TPM): 분당 처리할 수 있는 토큰의 양을 제한합니다. 구독의 할당량 내에서 설정하여 과도한 사용 및 비용 발생을 방지합니다. 이 실습에서는 50,000 TPM이 권장됩니다.

Content filter: DefaultV2

의미: Azure OpenAI 서비스에서 제공하는 콘텐츠 필터링 수준을 설정합니다. 유해 콘텐츠 생성을 방지하는 데 도움이 됩니다. DefaultV2는 표준 필터링 설정을 의미합니다.

참고: TPM을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하는 것을 방지하는 데 도움이 됩니다. 50,000 TPM은 이 실습에서 사용되는 데이터에 충분해야 합니다. 사용 가능한 할당량이 이보다 낮으면 실습을 완료할 수 있지만 속도 제한을 초과하면 오류가 발생할 수 있습니다.

배포가 완료될 때까지 기다립니다.

프롬프트 흐름 만들기 (Create a prompt flow)

Prompt flow는 생성형 AI 모델과의 상호 작용을 정의하기 위해 프롬프트 및 기타 활동을 오케스트레이션하는 방법을 제공합니다. 이 실습에서는 템플릿을 사용하여 여행사의 AI 도우미를 위한 기본 채팅 흐름을 만듭니다.

Prompt flow란?

LLM(대규모 언어 모델) 기반 AI 애플리케이션 개발을 위한 시각적 개발 도구 및 실행 환경입니다. 프롬프트 엔지니어링, 여러 LLM 호출, 외부 도구 연동, 조건부 논리 등을 워크플로우 형태로 구성하여 복잡한 대화형 AI를 쉽게 만들고 테스트하며 배포할 수 있도록 지원합니다.

핸즈온에서 배울 점:

Chat flow 템플릿을 사용하여 새로운 prompt flow를 생성하는 방법을 배웁니다.

prompt flow 편집기의 기본 구성 요소(입력, 출력, 도구)를 이해합니다.

Chat LLM 도구를 구성하여 배포된 모델에 연결하고, 시스템 프롬프트를 수정하여 모델의 행동을 정의합니다.

Azure AI Foundry portal 탐색 모음의 빌드 및 사용자 지정(Build and customize) 섹션에서 **프롬프트 흐름(Prompt flow)**을 선택합니다.

채팅 흐름(Chat flow) 템플릿을 기반으로 새 흐름을 만들고 폴더 이름으로 Travel-Chat을 지정합니다.

간단한 채팅 흐름이 생성됩니다.

팁: 권한 오류가 발생하면 몇 분 정도 기다렸다가 필요한 경우 다른 흐름 이름을 지정하여 다시 시도하십시오.

흐름을 테스트하려면 컴퓨팅이 필요하며 시작하는 데 시간이 걸릴 수 있으므로, 기본 흐름을 탐색하고 수정하는 동안 **컴퓨팅 세션 시작(Start compute session)**을 선택하여 시작하십시오.

Compute session이란?

Prompt flow를 실행하고 테스트하기 위한 백엔드 컴퓨팅 환경입니다. prompt flow 편집기에서 흐름을 실행하거나 디버깅할 때 이 세션이 활성화되어 있어야 합니다.

일련의 입력, 출력 및 도구로 구성된 prompt flow를 확인합니다. 왼쪽 편집 창에서 이러한 개체의 속성을 확장하고 편집할 수 있으며, 오른쪽 그래프에서 전체 흐름을 볼 수 있습니다:

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/use-prompt-flow-manage-conversation-chat-app/media/prompt-flow-editor.png)

입력(Inputs) 창을 보고 두 가지 입력(채팅 기록 및 사용자 질문)이 있는지 확인합니다.

출력(Outputs) 창을 보고 모델의 답변을 반영하는 출력이 있는지 확인합니다.

모델에 프롬프트를 제출하는 데 필요한 정보가 포함된 Chat LLM 도구 창을 확인합니다.

Chat LLM 도구란?

Prompt flow 내에서 LLM(여기서는 배포된 gpt-4o)과 상호작용하는 핵심 구성 요소입니다. 프롬프트를 LLM에 전달하고 응답을 받는 역할을 합니다.

Chat LLM 도구 창에서 **연결(Connection)**에 대해 AI 허브의 Azure OpenAI 서비스 리소스에 대한 연결을 선택합니다. 그런 다음 다음 연결 속성을 구성합니다:

Api: chat

의미: LLM과의 상호작용 유형을 지정합니다. chat은 채팅 완료(Chat Completions) API를 사용함을 의미합니다.

deployment_name: 배포한 gpt-4o 모델

의미: 이 Chat LLM 도구가 사용할 특정 모델 배포의 이름입니다.

response_format: {"type":"text"}

의미: 모델로부터 받을 응답의 형식을 지정합니다. 여기서는 일반 텍스트 응답을 기대합니다.

프롬프트(Prompt) 필드를 다음과 같이 수정합니다:

# system:
**Objective**: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

**Capabilities**:
- Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
- Offer personalized travel suggestions based on user preferences, budget, and travel dates.
- Share tips on packing, safety, and navigating travel disruptions.
- Help with itinerary planning, including optimal routes and must-see landmarks.
- Answer common travel questions and provide solutions to potential travel issues.

**Instructions**:
1. Engage with the user in a friendly and professional manner, as a travel agent would.
2. Use available resources to provide accurate and relevant travel information.
3. Tailor responses to the user's specific travel needs and interests.
4. Ensure recommendations are practical and consider the user's safety and comfort.
5. Encourage the user to ask follow-up questions for further assistance.

   

# user:
{{question}}

# chat_history:
{% for item in chat_history %}
# user:
{{item.inputs.question}}
# assistant:
{{item.outputs.answer}}
{% endfor %}


추가한 프롬프트를 읽고 익숙해지십시오. 이는 시스템 메시지(목표, 기능 정의 및 일부 지침 포함)와 사용자 질문 입력 및 이전 어시스턴트 답변 출력을 보여주도록 정렬된 채팅 기록으로 구성됩니다.

프롬프트 구조 설명:

# system:: 이 부분은 LLM에게 역할, 목표, 능력, 따라야 할 지침 등을 제공하는 시스템 메시지입니다. 모델의 전반적인 행동과 응답 스타일을 정의합니다.

**Objective**: 이 AI의 주요 목표 (여행 관련 문의 지원)

**Capabilities**: 이 AI가 수행할 수 있는 작업 목록

**Instructions**: AI가 사용자와 상호작용할 때 따라야 할 구체적인 지침

# user:\n{{question}}: 사용자의 현재 질문을 나타내는 부분입니다. {{question}}은 prompt flow의 입력 변수인 inputs.question 값으로 대체됩니다.

# chat_history:\n{% for item in chat_history %}...{% endfor %}: 이전 대화 기록을 나타냅니다. chat_history는 prompt flow의 입력 변수인 inputs.chat_history 값(이전 대화 턴들의 리스트)으로 대체되며, 루프를 돌면서 각 이전 사용자 질문과 어시스턴트 답변을 프롬프트에 포함시킵니다. 이는 모델이 대화의 맥락을 이해하는 데 중요합니다. 이 부분은 Jinja2 템플릿 구문을 사용합니다.

핸즈온에서 배울 점:

구체적인 시스템 메시지를 작성하여 모델의 페르소나("지식이 풍부한 여행사 직원")와 행동 양식을 정의하는 방법을 배웁니다.

prompt flow에서 입력 변수({{question}}, {{chat_history}})를 사용하여 동적으로 프롬프트를 구성하는 방법을 이해합니다.

채팅 기록을 프롬프트에 포함시켜 맥락을 유지하는 기법을 확인합니다.

Chat LLM 도구의 입력(Inputs) 섹션(프롬프트 아래)에서 다음 변수가 설정되어 있는지 확인합니다:

question (string): ${inputs.question}

chat_history (list): ${inputs.chat_history} (참고: 원본은 string으로 되어있으나, 위 프롬프트의 Jinja 루프를 고려하면 list of objects가 더 적합할 수 있습니다. 템플릿에서 처리 방식에 따라 달라집니다. 만약 string으로 전달된다면, 프롬프트 내에서 파싱하거나 다른 방식으로 처리해야 합니다. 이 실습에서는 지정된 대로 string으로 가정하고 진행합니다. 실제로는 chat_history는 일반적으로 객체들의 리스트 형태입니다.)

의미: Chat LLM 도구의 내부 변수 question과 chat_history가 prompt flow의 전역 입력인 inputs.question과 inputs.chat_history로부터 값을 받아오도록 매핑합니다. ${...} 구문은 prompt flow에서 값을 참조하는 방식입니다.

흐름에 대한 변경 내용을 저장합니다.

참고: 이 실습에서는 간단한 채팅 흐름을 고수하지만, prompt flow 편집기에는 흐름에 추가할 수 있는 다른 많은 도구가 포함되어 있어 대화를 조정하기 위한 복잡한 논리를 만들 수 있습니다.

흐름 테스트 (Test the flow)

흐름을 개발했으므로 이제 채팅 창을 사용하여 테스트할 수 있습니다.

핸즈온에서 배울 점:

prompt flow 편집기 내의 채팅 인터페이스를 사용하여 생성한 흐름을 실시간으로 테스트하고, 모델이 정의된 프롬프트에 따라 응답하는지 확인합니다.

컴퓨팅 세션이 실행 중인지 확인합니다. 그렇지 않은 경우 시작될 때까지 기다립니다.

도구 모음에서 **채팅(Chat)**을 선택하여 채팅 창을 열고 채팅이 초기화될 때까지 기다립니다.

"런던에서 하루를 보낼 수 있는데, 무엇을 해야 할까요?(I have one day in London, what should I do?)"라는 질문을 입력하고 출력을 검토합니다. 채팅 창은 다음과 유사하게 보일 것입니다:

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/use-prompt-flow-manage-conversation-chat-app/media/prompt-flow-chat.png)

흐름 배포 (Deploy the flow)

만든 흐름의 동작에 만족하면 흐름을 배포할 수 있습니다.

Prompt flow 배포란?

개발하고 테스트한 prompt flow를 실시간으로 호출 가능한 엔드포인트로 만드는 과정입니다. 배포된 엔드포인트를 통해 외부 애플리케이션에서 prompt flow의 기능을 사용할 수 있게 됩니다.

핸즈온에서 배울 점:

완성된 prompt flow를 온라인 엔드포인트로 배포하는 과정을 익힙니다.

배포 시 필요한 설정(엔드포인트 이름, VM 크기, 인스턴스 수)을 이해합니다.

배포된 엔드포인트를 테스트하고, 애플리케이션에서 호출하기 위한 정보를 확인합니다.

참고: 배포는 시간이 오래 걸릴 수 있으며 구독 또는 테넌트의 용량 제약으로 인해 영향을 받을 수 있습니다.

도구 모음에서 **배포(Deploy)**를 선택하고 다음 설정으로 흐름을 배포합니다:

기본 설정(Basic settings):

Endpoint: New (새로 만들기)

Endpoint name: 고유한 이름 입력

Deployment name: 고유한 이름 입력 (엔드포인트 내의 특정 배포 버전 이름)

Virtual machine: Standard_DS3_v2

의미: 엔드포인트가 실행될 가상 머신의 크기를 지정합니다. 트래픽 양과 응답 속도 요구 사항에 따라 선택합니다.

Instance count: 1

의미: 배포에 사용할 가상 머신 인스턴스의 수입니다. 더 많은 인스턴스는 더 높은 처리량과 가용성을 제공합니다.

Inferencing data collection: Disabled (비활성화됨)

고급 설정(Advanced settings):

기본 설정 사용

Azure AI Foundry portal의 탐색 창에 있는 내 자산(My assets) 섹션에서 모델 + 엔드포인트(Models + endpoints) 페이지를 선택합니다.

gpt-4o 모델에 대해 페이지가 열리면 뒤로 버튼을 사용하여 모든 모델과 엔드포인트를 확인합니다.

처음에는 페이지에 모델 배포만 표시될 수 있습니다. 배포가 목록에 표시되기까지 시간이 걸릴 수 있으며 성공적으로 생성되기까지는 훨씬 더 오래 걸릴 수 있습니다.

배포가 성공하면 선택합니다. 그런 다음 테스트(Test) 페이지를 확인합니다.

팁: 테스트 페이지에 엔드포인트가 비정상(unhealthy)으로 설명되어 있으면 모델 및 엔드포인트로 돌아가서 1분 정도 기다렸다가 보기를 새로 고치고 엔드포인트를 다시 선택하십시오.

"샌프란시스코에서 할 일이 무엇인가요?(What is there to do in San Francisco?)"라는 프롬프트를 입력하고 응답을 검토합니다.

"도시의 역사에 대해 알려주세요.(Tell me something about the history of the city.)"라는 프롬프트를 입력하고 응답을 검토합니다.

테스트 창은 다음과 유사하게 보일 것입니다:

![alt text](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/use-prompt-flow-manage-conversation-chat-app/media/endpoint-test.png)

엔드포인트의 사용(Consume) 페이지를 확인하고, 엔드포인트에 대한 클라이언트 애플리케이션을 빌드하는 데 사용할 수 있는 연결 정보 및 샘플 코드가 포함되어 있어 prompt flow 솔루션을 생성형 AI 애플리케이션으로 애플리케이션에 통합할 수 있습니다.

Consume 페이지란?

배포된 엔드포인트를 애플리케이션에서 호출하는 데 필요한 정보(REST API 엔드포인트 URL, 인증 키 등)와 다양한 프로그래밍 언어(Python, C# 등)로 작성된 샘플 코드를 제공합니다. 개발자가 쉽게 엔드포인트를 통합할 수 있도록 돕습니다.

요약 (Summary)
이 실습에서는 Azure AI Foundry portal의 prompt flow를 사용하여 AI hub resource 기반의 project를 만들고, gpt-4o 모델을 배포했습니다. 그런 다음, Chat flow 템플릿을 사용하여 여행사 AI 도우미를 위한 prompt flow를 생성하고, 시스템 프롬프트를 사용자 정의하여 모델의 행동을 정의했습니다. 생성된 흐름을 테스트하고, 실시간으로 사용할 수 있도록 온라인 엔드포인트로 배포하는 과정을 거쳤습니다. 마지막으로, 배포된 엔드포인트를 테스트하고 Consume 페이지를 통해 애플리케이션 통합 정보를 확인했습니다.
