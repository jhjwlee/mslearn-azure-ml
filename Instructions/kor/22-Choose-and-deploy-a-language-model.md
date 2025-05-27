## 언어 모델 선택 및 배포 (Choose and deploy a language model)

Azure AI Foundry `model catalog`는 다양한 모델을 탐색하고 사용하여 생성형 AI 시나리오를 만들 수 있도록 지원하는 중앙 리포지토리 역할을 합니다.

이 실습에서는 Azure AI Foundry portal에서 `model catalog`를 탐색하고, 문제 해결을 지원하는 생성형 AI 애플리케이션을 위한 잠재적 모델을 비교합니다.

이 실습은 약 **25분**이 소요됩니다.

**참고:** 이 실습에 사용된 일부 기술은 미리 보기(preview) 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

**학습 목표:**
이 실습을 통해 다음을 수행하는 방법을 배웁니다:
*   Azure AI Foundry `model catalog`를 탐색하여 사용 가능한 모델을 검토합니다.
*   `gpt-4o` 및 `Phi-3.5-mini-instruct`와 같은 특정 모델의 세부 정보(`Details`) 및 성능 벤치마크(`Benchmarks`)를 확인합니다.
*   `Compare models` 기능을 사용하여 여러 모델을 시각적으로 비교합니다.
*   품질 지수(`Quality Index`), 비용(`Cost`), 정확도(`Accuracy`), 일관성(`Coherence`), 유창성(`Fluency`), 관련성(`Relevance`)과 같은 다양한 메트릭을 기반으로 모델을 평가합니다.
*   선택한 모델(`gpt-4o`)을 사용하여 Azure AI Foundry `project`를 생성합니다.
*   `Chat playground`에서 배포된 모델과 상호 작용하여 테스트합니다.
*   기존 `project`에 추가 모델(`Phi-3.5-mini-instruct`)을 배포합니다.
*   여러 배포된 모델을 동일한 프롬프트로 테스트하고 응답을 비교합니다.

**주요 용어 (영문 유지):**
*   Azure AI Foundry portal
*   Model catalog
*   gpt-4o
*   Phi-3.5-mini-instruct
*   Details tab
*   Benchmarks tab
*   Compare models
*   Quality Index
*   Cost
*   Accuracy, Coherence, Fluency, Relevance
*   Overview tab
*   Use this model
*   Advanced options
*   Azure AI Foundry resource
*   Subscription
*   Resource group
*   Region
*   Chat playground
*   Setup pane
*   Give the model instructions and context
*   System prompt
*   My assets
*   Models + endpoints
*   Model deployments tab
*   Deploy base model
*   Deployment name
*   Deployment type (Global Standard)

---

### 모델 탐색 (Explore models)

먼저 Azure AI Foundry portal에 로그인하고 사용 가능한 일부 모델을 탐색해 보겠습니다.

1.  웹 브라우저에서 Azure AI Foundry portal (https://ai.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 모든 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 상단의 Azure AI Foundry 로고를 사용하여 홈페이지로 이동합니다. 홈페이지는 다음 이미지와 유사하게 보일 것입니다 (도움말 창이 열려 있으면 닫으십시오):

    ![Azure AI Foundry portal 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/ai-studio-home.png)

    *   **핸즈온에서 배울 점:**
        *   Azure AI Foundry portal에 성공적으로 접속하고 초기 화면 구성을 확인합니다.

2.  홈페이지의 정보를 검토합니다.
3.  홈페이지의 **모델 및 기능 탐색(Explore models and capabilities)** 섹션에서 `project`에서 사용할 `gpt-4o` 모델을 검색합니다.
    *   **Model catalog란?**
        *   Azure AI Foundry에서 제공하는 다양한 사전 훈련된 AI 모델들의 컬렉션입니다. 사용자는 여기서 모델을 검색, 필터링, 비교하고 세부 정보를 확인할 수 있습니다. Azure OpenAI 모델, 오픈 소스 모델 등 다양한 모델이 포함될 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `Model catalog`에서 원하는 모델(`gpt-4o`)을 검색하는 방법을 익힙니다.

4.  검색 결과에서 `gpt-4o` 모델을 선택하여 세부 정보를 확인합니다.
5.  설명을 읽고 **세부 정보(Details)** 탭에서 사용 가능한 다른 정보를 검토합니다.

    ![gpt-4o 모델 세부 정보 페이지 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/choose-deploy-language-model/media/gpt-4o-details.png)

    *   **Details tab이란?**
        *   선택한 모델에 대한 일반적인 설명, 사용 사례, 제약 조건, 입력/출력 유형, 버전 정보 등과 같은 핵심 정보를 제공합니다.
    *   **핸즈온에서 배울 점:**
        *   모델의 `Details` 탭에서 제공하는 정보(설명, 사용 사례 등)를 확인하고 이해하는 방법을 배웁니다.

6.  `gpt-4o` 페이지에서 **벤치마크(Benchmarks)** 탭을 확인하여 유사한 시나리오에서 사용되는 다른 모델과 비교하여 일부 표준 성능 벤치마크에서 모델이 어떻게 비교되는지 확인합니다.

    ![gpt-4o 모델 벤치마크 페이지 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/choose-deploy-language-model/media/gpt-4o-benchmarks.png)

    *   **Benchmarks tab이란?**
        *   선택한 모델이 표준화된 데이터셋 및 평가 메트릭(예: 정확도, 유창성, MMLU, HellaSwag 등)에서 어떤 성능을 보이는지 보여줍니다. 종종 다른 유사 모델과의 성능 비교도 제공하여 모델 선택에 도움을 줍니다.
    *   **핸즈온에서 배울 점:**
        *   모델의 `Benchmarks` 탭을 통해 객관적인 성능 지표를 확인하고, 다른 모델과의 성능을 비교하는 방법을 익힙니다.

7.  `gpt-4o` 페이지 제목 옆의 뒤로 화살표(←)를 사용하여 `model catalog`로 돌아갑니다.
8.  `Phi-3.5-mini-instruct`를 검색하고 `Phi-3.5-mini-instruct` 모델의 세부 정보 및 벤치마크를 확인합니다.
    *   **Phi-3.5-mini-instruct 모델이란?**
        *   Microsoft에서 개발한 작지만 강력한 언어 모델(SLM)입니다. `gpt-4o`보다 작은 규모의 모델로, 특정 작업에서는 경쟁력 있는 성능을 보이면서도 비용 효율적이고 빠른 응답 속도를 가질 수 있습니다. "instruct"는 지침을 따르도록 미세 조정되었음을 의미합니다.
    *   **핸즈온에서 배울 점:**
        *   다른 유형의 모델(`Phi-3.5-mini-instruct`)을 탐색하고, 그 세부 정보와 벤치마크를 확인하여 모델의 다양성을 인지합니다.

---

### 모델 비교 (Compare models)

두 가지 다른 모델을 검토했으며, 두 모델 모두 생성형 AI 채팅 애플리케이션을 구현하는 데 사용할 수 있습니다. 이제 이 두 모델의 메트릭을 시각적으로 비교해 보겠습니다.

1.  뒤로 화살표(←)를 사용하여 `model catalog`로 돌아갑니다.
2.  **모델 비교(Compare models)**를 선택합니다. 일반적인 모델 선택 항목과 함께 모델 비교를 위한 시각적 차트가 표시됩니다.

    ![모델 비교 페이지 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/choose-deploy-language-model/media/model-comparison.png)

    *   **Compare models 기능이란?**
        *   `Model catalog` 내에서 여러 모델을 선택하여 다양한 성능 메트릭과 비용 등을 기준으로 시각적으로 비교할 수 있는 도구입니다. 사용자가 특정 요구 사항에 가장 적합한 모델을 선택하는 데 도움을 줍니다.
    *   **핸즈온에서 배울 점:**
        *   `Compare models` 기능을 사용하여 여러 모델을 동시에 비교하는 방법을 익힙니다.

3.  **비교할 모델(Models to compare)** 창에서 질문 답변과 같은 인기 있는 작업을 선택하여 특정 작업에 일반적으로 사용되는 모델을 자동으로 선택할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   특정 작업(예: 질의응답)에 적합한 모델들을 빠르게 필터링하고 비교 목록에 추가할 수 있는 방법을 이해합니다.

4.  **모든 모델 지우기(🗑)(Clear all models)** 아이콘을 사용하여 미리 선택된 모든 모델을 제거합니다.
5.  **+ 비교할 모델(+ Model to compare)** 버튼을 사용하여 `gpt-4o` 모델을 목록에 추가합니다. 그런 다음 동일한 버튼을 사용하여 `Phi-3.5-mini-instruct` 모델을 목록에 추가합니다.
    *   **핸즈온에서 배울 점:**
        *   비교 차트에 원하는 특정 모델들을 수동으로 추가하고 제거하는 방법을 익힙니다.

6.  품질 지수(`Quality Index` - 모델 품질을 나타내는 표준화된 점수)와 비용(`Cost`)을 기준으로 모델을 비교하는 차트를 검토합니다. 차트에서 해당 모델을 나타내는 지점 위에 마우스를 올려 특정 값을 볼 수 있습니다.

    ![gpt-4o 및 Phi-3.5-mini-instruct 모델 비교 차트 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/choose-deploy-language-model/media/model-comparison-chart.png)

    *   **Quality Index란?**
        *   다양한 벤치마크 결과를 종합하여 모델의 전반적인 품질을 나타내는 표준화된 점수입니다. 모델 간 상대적인 성능을 쉽게 비교할 수 있도록 도와줍니다.
    *   **Cost란?**
        *   모델 사용에 따른 상대적인 비용을 나타냅니다. 일반적으로 토큰 사용량이나 API 호출 당 비용으로 계산될 수 있으며, 여기서는 상대적인 비교 지표로 표시됩니다.
    *   **핸즈온에서 배울 점:**
        *   `Quality Index`와 `Cost`를 기준으로 모델을 시각적으로 비교하고, 마우스 오버를 통해 구체적인 수치를 확인하는 방법을 배웁니다.

7.  **X축(X-axis)** 드롭다운 메뉴의 **품질(Quality)** 아래에서 다음 메트릭을 선택하고 다음으로 전환하기 전에 각 결과 차트를 관찰합니다:
    *   **정확도(Accuracy)**
        *   **의미:** 모델이 생성하는 정보가 사실에 얼마나 부합하고 정확한지를 나타내는 지표입니다.
    *   **일관성(Coherence)**
        *   **의미:** 모델이 생성하는 텍스트가 논리적으로 연결되고, 내부적으로 모순이 없으며, 전체적으로 의미가 통하는 정도를 나타냅니다.
    *   **유창성(Fluency)**
        *   **의미:** 모델이 생성하는 텍스트가 문법적으로 정확하고, 자연스러우며, 읽기 쉬운 정도를 나타냅니다.
    *   **관련성(Relevance)**
        *   **의미:** 모델의 응답이 사용자의 질문이나 입력된 프롬프트의 의도에 얼마나 부합하고 관련성이 있는지를 나타냅니다.
    *   **핸즈온에서 배울 점:**
        *   다양한 품질 메트릭(`Accuracy`, `Coherence`, `Fluency`, `Relevance`)을 X축으로 변경해가며 모델들을 비교함으로써, 특정 품질 측면에서 어떤 모델이 우수한지 분석하는 방법을 익힙니다.

8.  벤치마크를 기준으로 볼 때, `gpt-4o` 모델이 전반적으로 최고의 성능을 제공하는 것으로 보이지만 비용이 더 높습니다.

9.  비교할 모델 목록에서 `gpt-4o` 모델을 선택하여 벤치마크 페이지를 다시 엽니다.
10. `gpt-4o` 모델 페이지에서 **개요(Overview)** 탭을 선택하여 모델 세부 정보를 확인합니다.
    *   **Overview tab이란?** (이전 실습에서 `Details` 탭으로 언급되었을 수 있음. 맥락상 모델의 주요 정보를 요약하는 탭)
        *   모델의 핵심 정보, 설명, 사용 사례, 제공자 등을 요약하여 보여줍니다.
    *   **핸즈온에서 배울 점:**
        *   비교 후 관심 있는 모델의 상세 정보 페이지로 돌아가 추가 정보를 확인하는 과정을 반복합니다.

---

### Azure AI Foundry 프로젝트 생성 (Create an Azure AI Foundry project)

모델을 사용하려면 Azure AI Foundry `project`를 만들어야 합니다. (이 부분은 이전 실습의 반복입니다.)

1.  `gpt-4o` 모델 개요 페이지 상단에서 **이 모델 사용(Use this model)**을 선택합니다.
2.  `project`를 만들라는 메시지가 표시되면 `project`에 유효한 이름을 입력하고 **고급 옵션(Advanced options)**을 확장합니다.
3.  **고급 옵션(Advanced options)** 섹션에서 허브에 대해 다음 설정을 지정합니다:
    *   **Azure AI Foundry resource:** `Azure AI Foundry resource`에 대한 유효한 이름
    *   **Subscription:** 사용자의 Azure 구독
    *   **Resource group:** `Resource group` 만들기 또는 선택
    *   **Region:** AI 서비스가 지원되는 위치 선택\*
    \* 일부 Azure AI 리소스는 지역별 모델 할당량에 의해 제한됩니다. 실습 후반에 할당량 한도를 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.

4.  **만들기(Create)**를 선택하고 선택한 `gpt-4o` 모델 배포를 포함하여 `project`가 생성될 때까지 기다립니다.
5.  `project`가 생성되면 `chat playground`가 자동으로 열리므로 모델을 테스트할 수 있습니다:

    ![Azure AI Foundry 프로젝트 채팅 플레이그라운드 스크린샷](https://learn.microsoft.com/ko-kr/training/wwl-data-ai/prepare-ai-development-project/media/chat-playground.png)

    *   **핸즈온에서 배울 점:**
        *   선택한 모델(`gpt-4o`)을 기반으로 새 `project`를 생성하고, 이 과정에서 모델이 자동으로 배포되어 `chat playground`에서 즉시 테스트 가능 상태가 됨을 확인합니다.

---

### gpt-4o 모델과 채팅 (Chat with the gpt-4o model)

이제 모델 배포가 완료되었으므로 `playground`를 사용하여 테스트할 수 있습니다.

1.  `chat playground`의 **설정(Setup)** 창에서 `gpt-4o` 모델이 선택되어 있는지 확인하고 **모델에게 지침 및 컨텍스트 제공(Give the model instructions and context)** 필드에 시스템 프롬프트를 `You are an AI assistant that helps solve problems.`로 설정합니다.
    *   **System prompt (시스템 프롬프트)란?**
        *   모델에게 대화의 전반적인 맥락, 역할, 지침을 제공하는 초기 메시지입니다. 모델이 특정 페르소나를 갖거나 특정 방식으로 응답하도록 유도하는 데 사용됩니다.
    *   **핸즈온에서 배울 점:**
        *   `Chat playground`의 `Setup` 창에서 `system prompt`를 설정하여 모델의 행동(여기서는 "문제 해결을 돕는 AI 어시스턴트")을 정의하는 방법을 익힙니다.

2.  **변경 내용 적용(Apply changes)**을 선택하여 시스템 프롬프트를 업데이트합니다.

3.  채팅 창에 다음 쿼리를 입력합니다.

    ```
    I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
    ```
    응답을 확인합니다. 그런 다음 다음 후속 쿼리를 입력합니다:

    ```
    Explain your reasoning.
    ```
    *   **핸즈온에서 배울 점:**
        *   고전적인 논리 퍼즐(여우, 닭, 곡식 문제)을 프롬프트로 사용하여 `gpt-4o` 모델의 문제 해결 능력과 추론 능력을 테스트합니다. 모델의 초기 응답과 후속 질문("Explain your reasoning")에 대한 응답을 통해 모델의 이해도와 설명 능력을 평가합니다.

---

### 다른 모델 배포 (Deploy another model)

`project`를 만들 때 선택한 `gpt-4o` 모델이 자동으로 배포되었습니다. 이제 고려했던 `Phi-3.5-mini-instruct` 모델도 배포해 보겠습니다.

1.  왼쪽 탐색 모음의 **내 자산(My assets)** 섹션에서 **모델 + 엔드포인트(Models + endpoints)**를 선택합니다.
    *   **Models + endpoints 란?**
        *   `project` 내에서 사용 가능한 모델들과 배포된 엔드포인트들을 관리하는 곳입니다. 여기서 새로운 모델을 배포하거나 기존 배포를 관리(업데이트, 삭제 등)할 수 있습니다.
    *   **핸즈온에서 배울 점:**
        *   `project` 내에서 모델 배포를 관리하는 `Models + endpoints` 메뉴로 이동하는 방법을 익힙니다.

2.  **모델 배포(Model deployments)** 탭의 **+ 모델 배포(+ Deploy model)** 드롭다운 목록에서 **기본 모델 배포(Deploy base model)**를 선택합니다. 그런 다음 `Phi-3.5-mini-instruct`를 검색하고 선택을 확인합니다.
    *   **Deploy base model이란?**
        *   `Model catalog`에 있는 사전 훈련된 기본 모델 중 하나를 선택하여 현재 `project`에 배포하는 옵션입니다.
    *   **핸즈온에서 배울 점:**
        *   기존 `project`에 `model catalog`에서 새로운 기본 모델(`Phi-3.5-mini-instruct`)을 찾아 배포하는 과정을 시작하는 방법을 배웁니다.

3.  모델 라이선스에 동의합니다.
4.  다음 설정으로 `Phi-3.5-mini-instruct` 모델을 배포합니다:
    *   **배포 이름(Deployment name):** 모델 배포에 대한 유효한 이름
        *   **의미:** 사용자가 배포된 모델 인스턴스를 식별하기 위해 지정하는 고유한 이름입니다.
    *   **배포 유형(Deployment type):** `Global Standard`
        *   **의미:** Azure AI Foundry에서 제공하는 표준 배포 옵션 중 하나를 나타냅니다. 지역별 또는 특정 성능/비용 특성을 가질 수 있습니다.
    *   **배포 세부 정보(Deployment details):** 기본 설정 사용
    *   **핸즈온에서 배울 점:**
        *   새로운 모델을 배포할 때 필요한 설정(`Deployment name`, `Deployment type` 등)을 구성하고 배포를 시작하는 방법을 익힙니다.

5.  배포가 완료될 때까지 기다립니다.

---

### Phi-3.5 모델과 채팅 (Chat with the Phi-3.5 model)

이제 `playground`에서 새 모델과 채팅해 보겠습니다.

1.  탐색 모음에서 **플레이그라운드(Playgrounds)**를 선택합니다. 그런 다음 **채팅(Chat)** 플레이그라운드를 선택합니다.
2.  `chat playground`의 **설정(Setup)** 창에서 `Phi-3.5-mini-instruct` 모델이 선택되어 있는지 확인하고 **모델에게 지침 및 컨텍스트 제공(Give the model instructions and context)** 필드에 시스템 프롬프트를 `You are an AI assistant that helps solve problems.`로 설정합니다. (`gpt-4o` 모델을 테스트하는 데 사용한 것과 동일한 시스템 프롬프트입니다.)
    *   **핸즈온에서 배울 점:**
        *   `Chat playground`에서 방금 배포한 `Phi-3.5-mini-instruct` 모델로 전환하고, 이전 모델과 동일한 `system prompt`를 설정하여 공정한 비교를 준비합니다.

3.  **변경 내용 적용(Apply changes)**을 선택하여 시스템 프롬프트를 업데이트합니다.
4.  이전에 `gpt-4o` 모델을 테스트하는 데 사용한 것과 동일한 프롬프트를 반복하기 전에 새 채팅 세션이 시작되었는지 확인합니다.
5.  채팅 창에 다음 쿼리를 입력합니다.

    ```
    I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
    ```
    응답을 확인합니다. 그런 다음 다음 후속 쿼리를 입력합니다:

    ```
    Explain your reasoning.
    ```
    *   **핸즈온에서 배울 점:**
        *   `Phi-3.5-mini-instruct` 모델을 `gpt-4o`와 동일한 문제(여우, 닭, 곡식)와 후속 질문으로 테스트하여 두 모델의 응답 내용, 정확성, 추론 능력 등을 직접 비교합니다.

---

### 추가 비교 수행 (Perform a further comparison)

**설정(Setup)** 창의 드롭다운 목록을 사용하여 모델 간에 전환하면서 다음 퍼즐로 두 모델을 모두 테스트합니다 (정답은 40개입니다!):

```
I have 53 socks in my drawer: 21 identical blue, 15 identical black and 17 identical red. The lights are out, and it is completely dark. How many socks must I take out to make 100 percent certain I have at least one pair of black socks?
```

*   **핸즈온에서 배울 점:**
    *   다른 유형의 논리 퍼즐(양말 문제)을 사용하여 두 모델(`gpt-4o`, `Phi-3.5-mini-instruct`)의 성능을 추가로 비교합니다. 이를 통해 다양한 문제 해결 시나리오에서 각 모델의 강점과 약점을 파악하는 데 도움이 됩니다.

---

### 모델에 대한 고찰 (Reflect on the models)

적절한 응답을 생성하는 능력과 비용 측면에서 다를 수 있는 두 가지 모델을 비교했습니다. 모든 생성형 시나리오에서는 수행해야 하는 작업에 대한 적합성과 예상되는 요청 수를 처리하는 데 드는 모델 사용 비용 간의 올바른 균형을 갖춘 모델을 찾아야 합니다.

`model catalog`에서 제공되는 세부 정보 및 벤치마크와 모델을 시각적으로 비교할 수 있는 기능은 생성형 AI 솔루션의 후보 모델을 식별할 때 유용한 출발점을 제공합니다. 그런 다음 `chat playground`에서 다양한 시스템 및 사용자 프롬프트로 후보 모델을 테스트할 수 있습니다.

**핵심 고찰 (Key Reflections):**
*   **성능 vs. 비용:** `gpt-4o`는 일반적으로 더 높은 성능을 보이지만 비용도 더 높을 수 있습니다. `Phi-3.5-mini-instruct`는 특정 작업에 대해 충분한 성능을 제공하면서 비용 효율적일 수 있습니다.
*   **작업 적합성:** 모델의 성능은 해결하려는 특정 문제나 작업 유형에 따라 달라질 수 있습니다. 한 문제에서 뛰어난 모델이 다른 문제에서는 그렇지 않을 수 있습니다.
*   **실험의 중요성:** `Model catalog`의 벤치마크와 `Compare models` 기능은 초기 필터링에 유용하지만, 실제 사용 사례에 가장 적합한 모델을 찾기 위해서는 `chat playground`에서의 직접적인 테스트와 다양한 프롬프트를 통한 실험이 필수적입니다.
*   **균형 찾기:** 최종 모델 선택은 성능, 비용, 특정 애플리케이션의 요구 사항 등 여러 요소를 고려한 균형 잡힌 결정이어야 합니다.
