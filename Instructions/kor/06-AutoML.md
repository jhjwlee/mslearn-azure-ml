## 자동화된 Machine Learning으로 최상의 분류 모델 찾기

모델 학습을 위한 올바른 알고리즘과 전처리 변환을 결정하는 것은 많은 추측과 실험을 필요로 할 수 있습니다.

이 연습에서는 자동화된 기계 학습을 사용하여 여러 학습 실행을 병렬로 수행함으로써 모델에 대한 최적의 알고리즘과 전처리 단계를 결정합니다.

**실습 목표:**

*   Azure CLI를 사용하여 Azure Machine Learning 작업 영역 및 컴퓨팅 리소스 프로비저닝하기
*   Python SDK를 사용하여 컴퓨팅 인스턴스 터미널에서 Git 리포지토리 복제하기
*   Python SDK와 자동화된 ML을 사용하여 분류 모델 학습시키기
*   자동화된 ML 작업 결과 검토하고 최상의 모델 설명 이해하기
*   생성된 Azure 리소스 정리하기

**준비물:**

*   관리자 수준 액세스 권한이 있는 Azure 구독

---

### Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스와 자산을 관리하기 위한 중앙 집중식 공간을 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

이 실습에서는 Azure CLI를 사용하여 작업 영역과 필요한 컴퓨팅을 프로비저닝하고, Python SDK를 사용하여 자동화된 Machine Learning으로 분류 모델을 학습합니다.

**작업 영역 및 컴퓨팅 리소스 만들기**

Azure Machine Learning 작업 영역, 컴퓨팅 인스턴스 및 컴퓨팅 클러스터를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 사용자가 실행할 수 있도록 셸 스크립트에 그룹화되어 있습니다.

1.  브라우저에서 Microsoft 계정으로 로그인하여 [https://portal.azure.com/](https://portal.azure.com/)에서 Azure Portal을 엽니다.
2.  페이지 상단의 검색창 오른쪽에 있는 **[\>\_] (Cloud Shell)** 버튼을 선택합니다. 그러면 포털 하단에 Cloud Shell 창이 열립니다.
3.  메시지가 나타나면 **Bash**를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다.
4.  올바른 구독이 지정되어 있고 **저장소 계정 필요 없음(No storage account required)** 이 선택되어 있는지 확인합니다. **적용(Apply)** 을 선택합니다.
5.  터미널에 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```bash
    rm -r azure-ml-labs -f
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

    **팁**: 복사한 코드를 Cloud Shell에 붙여넣으려면 `SHIFT + INSERT`를 사용하십시오.

6.  리포지토리가 복제된 후 다음 명령을 입력하여 이 실습용 폴더로 변경하고 포함된 `setup.sh` 스크립트를 실행합니다.

    ```bash
    cd azure-ml-labs/Labs/06
    ./setup.sh
    ```

    **참고**: 확장이 설치되지 않았다는 (오류) 메시지는 무시하십시오.

7.  스크립트가 완료될 때까지 기다립니다. 일반적으로 약 5-10분 정도 소요됩니다.

    **문제 해결 팁: 작업 영역 생성 오류**
    만약 `setup.sh` 스크립트 실행 중 작업 영역 생성에 실패하는 경우, 이는 주로 선택한 지역의 특정 VM 크기(이 스크립트에서는 컴퓨팅 인스턴스 및 클러스터에 대해 특정 VM 크기를 사용하려고 할 수 있음)에 대한 구독 할당량이 부족하기 때문일 수 있습니다.
    *   **해결 방법 1 (권장)**: `setup.sh` 스크립트를 열어보면 (`cat setup.sh` 또는 Cloud Shell 편집기 사용) 작업 영역, 컴퓨팅 인스턴스, 컴퓨팅 클러스터를 생성하는 `az ml ...` 명령들이 있습니다. 여기서 VM 크기를 지정하는 부분을 찾아 구독에서 사용 가능한 다른 크기로 변경해 보십시오. (예: `Standard_DS2_v2` 대신 `Standard_DS1_v2` 등).
    *   **해결 방법 2**: Azure Portal에서 수동으로 작업 영역을 먼저 만들고, 그 다음 스크립트에서 컴퓨팅 생성 부분만 실행하거나, Azure Machine Learning 스튜디오에서 직접 컴퓨팅을 생성할 수도 있습니다. 하지만 이 실습의 흐름을 따르려면 스크립트를 수정하는 것이 좋습니다.

---

### 실습 자료 복제하기

작업 영역과 필요한 컴퓨팅 리소스를 만들었으면 Azure Machine Learning 스튜디오를 열고 작업 영역에 실습 자료를 복제할 수 있습니다.

1.  Azure Portal에서 `mlw-dp100-…` (스크립트가 생성한 실제 이름)라는 Azure Machine Learning 작업 영역으로 이동합니다.
2.  Azure Machine Learning 작업 영역을 선택하고 **개요(Overview)** 페이지에서 **스튜디오 시작(Launch studio)** 을 선택합니다. 브라우저에서 다른 탭이 열리면서 Azure Machine Learning 스튜디오가 열립니다.
3.  스튜디오에 나타나는 모든 팝업을 닫습니다.
4.  Azure Machine Learning 스튜디오 내에서 **컴퓨팅(Compute)** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스는 실행 중이어야 하고, 클러스터는 유휴 상태이며 실행 중인 노드가 0개여야 합니다.
5.  **컴퓨팅 인스턴스(Compute instances)** 탭에서 컴퓨팅 인스턴스를 찾아 **터미널(Terminal)** 애플리케이션을 선택합니다.
6.  터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```bash
    pip uninstall azure-ai-ml -y
    pip install azure-ai-ml
    ```

    **참고**: 패키지를 찾을 수 없거나 제거할 수 없다는 (오류) 메시지는 무시하십시오. 이는 해당 패키지가 이미 없거나 다른 버전으로 설치되어 있을 때 발생할 수 있으며, `pip install`이 최신 버전을 설치하므로 문제되지 않습니다.

7.  다음 명령을 실행하여 Notebook, 데이터 및 기타 파일을 포함하는 Git 리포지토리를 작업 영역에 복제합니다.

    ```bash
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

8.  명령이 완료되면 **파일(Files)** 창에서 **↻ (새로고침)** 을 클릭하여 보기를 새로 고치고 새 `Users/your-user-name/azure-ml-labs` 폴더가 생성되었는지 확인합니다. (`your-user-name`은 실제 사용자 이름으로 표시됩니다.)

---

### 자동화된 기계 학습으로 분류 모델 학습

이제 필요한 모든 리소스가 있으므로 Notebook을 실행하여 자동화된 Machine Learning 작업을 구성하고 제출할 수 있습니다.

1.  **`Labs/06/Classification with Automated Machine Learning.ipynb`** Notebook을 엽니다.
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> `06` 폴더로 이동하여 `Classification with Automated Machine Learning.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.
    *   보통 브라우저에서 새 탭이 열리고 코드를 입력하라는 메시지가 표시됩니다. Notebook에 표시된 코드를 복사하여 해당 탭에 붙여넣고 로그인합니다.

3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다. (Notebook 상단 오른쪽에서 커널을 확인할 수 있으며, 필요시 변경합니다.)
4.  Notebook의 모든 셀을 실행합니다.
    *   메뉴에서 **모두 실행(Run all)** 을 선택하거나, 각 셀을 순서대로 `Shift + Enter`를 눌러 실행합니다.

Azure Machine Learning 작업 영역에 새 작업이 생성됩니다. 이 작업은 작업 구성에 정의된 입력, 사용된 데이터 자산, 모델 평가를 위한 메트릭과 같은 출력을 추적합니다.

자동화된 Machine Learning 작업에는 자식 작업이 포함되며, 이는 학습된 개별 모델 및 실행에 필요한 기타 작업을 나타냅니다.

1.  Azure Machine Learning 스튜디오의 왼쪽 메뉴에서 **작업(Jobs)** 으로 이동하여 **`auto-ml-class-dev`** 실험을 선택합니다. (Notebook 코드에서 실험 이름을 이렇게 설정했을 것입니다.)
2.  **표시 이름(Display name)** 열 아래에서 작업을 선택합니다. (보통은 `auto-ml-class-dev` 뒤에 임의의 문자열이 붙은 형태입니다.)
3.  상태가 **완료됨(Completed)** 으로 변경될 때까지 기다립니다. (AutoML 작업은 데이터 크기와 설정에 따라 시간이 걸릴 수 있습니다. Notebook에서는 `max_trials`를 제한하여 너무 오래 걸리지 않도록 설정했을 것입니다.)

자동화된 Machine Learning 작업 상태가 **완료됨(Completed)** 으로 변경되면 스튜디오에서 작업 세부 정보를 탐색합니다.

1.  **데이터 보호(Data guardrails)** 탭에는 학습 데이터에 문제가 있었는지 여부가 표시됩니다. 예를 들어, 클래스 불균형, 누락된 값 처리 등에 대한 정보를 제공합니다.
2.  **모델 + 자식 작업(Models + child jobs)** 탭에는 학습된 모든 모델이 표시됩니다. 최상의 모델에 대해 **모델 설명(Explain model)** 을 선택하고 **`aml-cluster`**(Notebook 코드에서 지정한 컴퓨팅 클러스터 이름)를 사용하여 설명 작업 실행을 만듭니다.
    *   **최상의 모델 선택**: 목록에서 기본 메트릭(예: AUC\_weighted 또는 Accuracy)이 가장 높은 모델을 선택합니다.
    *   **모델 설명(Explain model)**: 이 버튼을 클릭하면 모델의 예측을 해석하기 위한 새 작업이 시작됩니다. 이 작업은 이전에 생성한 **`aml-cluster`** 를 사용하여 실행됩니다.
3.  **알고리즘 이름(Algorithm name)** 열 옆에 새 열 **설명됨(Explained)** 이 나타날 때까지 기다린 다음 **설명 보기(View explanation)** 를 선택합니다. 이 옵션이 나타나려면 알고리즘 목록을 새로 고쳐야 할 수 있습니다.
4.  생성된 대시보드를 검토하여 어떤 특징(feature)이 대상 값에 가장 큰 영향을 미쳤는지 이해합니다.
    *   **전역 중요도(Global importance)**: 모델 전체적으로 어떤 특징이 예측에 중요한지 보여줍니다.
    *   **개별 예측 설명(Individual feature importance for specific predictions)**: 특정 데이터 포인트에 대한 예측이 어떻게 이루어졌는지 살펴볼 수도 있습니다.

---

### Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 만든 리소스를 삭제해야 합니다.

1.  Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
2.  Azure Portal의 **홈(Home)** 페이지에서 **리소스 그룹(Resource groups)** 을 선택합니다.
3.  `rg-dp100-…` (스크립트가 생성한 리소스 그룹 이름) 리소스 그룹을 선택합니다.
4.  리소스 그룹의 **개요(Overview)** 페이지 상단에서 **리소스 그룹 삭제(Delete resource group)** 를 선택합니다.
5.  삭제를 확인하기 위해 리소스 그룹 이름을 입력하고 **삭제(Delete)** 를 선택합니다.

이것으로 "자동화된 Machine Learning으로 최상의 분류 모델 찾기" 실습이 완료됩니다. 이 실습을 통해 Azure CLI와 Python SDK를 함께 사용하여 Azure ML 환경을 설정하고, AutoML을 통해 효율적으로 최적의 분류 모델을 찾아내고, 그 결과를 해석하는 과정을 경험했습니다.
