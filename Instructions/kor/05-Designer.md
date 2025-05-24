## Azure Machine Learning 디자이너로 모델 학습하기

Azure Machine Learning 디자이너(Designer)는 워크플로를 정의할 수 있는 드래그 앤 드롭 인터페이스를 제공합니다. 워크플로를 만들어 모델을 학습하고 여러 알고리즘을 손쉽게 테스트하고 비교할 수 있습니다.

이 실습에서는 디자이너를 사용하여 두 가지 분류 알고리즘을 신속하게 학습하고 비교합니다.

**실습 목표:**

*   Azure CLI를 사용하여 Azure Machine Learning 작업 영역 및 컴퓨팅 클러스터 프로비저닝하기
*   Azure ML 디자이너를 사용하여 데이터 준비 및 단일 모델 학습 파이프라인 구성 및 실행하기
*   기존 파이프라인에 다른 모델 학습 구성 요소를 추가하여 두 모델을 비교하는 파이프라인 구성 및 실행하기
*   두 모델의 학습 결과를 비교하고 성능 평가하기
*   생성된 Azure 리소스 정리하기

**준비물:**

*   관리자 수준 액세스 권한이 있는 Azure 구독

---

### Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스와 자산을 관리하기 위한 중앙 집중식 공간을 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

이 실습에서는 Azure CLI를 사용하는 셸 스크립트를 사용하여 작업 영역과 필요한 리소스를 프로비저닝합니다. 다음으로, Azure Machine Learning 스튜디오의 디자이너를 사용하여 모델을 학습하고 비교합니다.

**작업 영역 및 컴퓨팅 클러스터 만들기**

Azure Machine Learning 작업 영역과 컴퓨팅 클러스터를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 사용자가 실행할 수 있도록 셸 스크립트에 그룹화되어 있습니다.

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
    cd azure-ml-labs/Labs/05
    ./setup.sh
    ```

    **참고**: 확장이 설치되지 않았다는 (오류) 메시지는 무시하십시오.

7.  스크립트가 완료될 때까지 기다립니다. 일반적으로 약 5-10분 정도 소요됩니다. `setup.sh` 스크립트는 작업 영역 외에도 이 실습에서 사용할 **컴퓨팅 클러스터(Compute Cluster)** (예: `aml-cluster`)와 디자이너에서 사용할 **사용자 지정 구성 요소(Custom Components)** 및 **데이터 자산(Data Asset)** (예: `diabetes-folder`)을 미리 생성해 줍니다.

    **문제 해결 팁: 작업 영역 생성 오류**
    만약 `setup.sh` 스크립트 실행 중 작업 영역 또는 컴퓨팅 클러스터 생성에 실패하는 경우, 이는 주로 선택한 지역의 특정 VM 크기에 대한 구독 할당량이 부족하기 때문일 수 있습니다.
    *   **해결 방법**: `setup.sh` 스크립트 내용을 확인하여 (`cat azure-ml-labs/Labs/05/setup.sh`) 컴퓨팅 클러스터 생성 시 사용되는 VM 크기를 구독에서 사용 가능한 다른 크기로 변경한 후 다시 실행해 보십시오.

---

### 새 파이프라인 구성

작업 영역과 필요한 컴퓨팅 클러스터를 만들었으면 Azure Machine Learning 스튜디오를 열고 디자이너로 학습 파이프라인을 만들 수 있습니다.

1.  Azure Portal에서 `mlw-dp100-…` (스크립트가 생성한 실제 작업 영역 이름)라는 Azure Machine Learning 작업 영역으로 이동합니다.
2.  Azure Machine Learning 작업 영역을 선택하고 **개요(Overview)** 페이지에서 **스튜디오 시작(Launch studio)** 을 선택합니다. 브라우저에서 다른 탭이 열리면서 Azure Machine Learning 스튜디오가 열립니다.
3.  스튜디오에 나타나는 모든 팝업을 닫습니다.
4.  Azure Machine Learning 스튜디오 내에서 **컴퓨팅(Compute)** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 클러스터가 있는지 확인합니다. 클러스터는 유휴 상태이며 실행 중인 노드가 0개여야 합니다. (이름은 보통 `aml-cluster`입니다.)
5.  **디자이너(Designer)** 페이지로 이동합니다. (스튜디오 왼쪽 메뉴의 **작성(Authoring)** 섹션 아래에 있습니다.)
6.  페이지 상단에서 **사용자 지정(Custom)** 탭을 선택합니다. (Azure ML 디자이너는 클래식 미리 빌드된 구성 요소와 새로운 사용자 지정 구성 요소를 사용하는 두 가지 모드를 제공할 수 있습니다. 이 실습은 사용자 지정 구성 요소를 사용합니다.)
7.  **사용자 지정 구성 요소를 사용하여 새 빈 파이프라인 만들기(Create a new empty pipeline using custom components)** 를 선택하거나, 비슷한 옵션을 클릭하여 새 파이프라인을 시작합니다.
8.  기본 파이프라인 이름(예: `Pipeline-Created-on-date`) 오른쪽에 있는 연필 아이콘을 선택하여 **`Train-Diabetes-Classifier`** 로 변경합니다.

---

### 새 파이프라인 만들기

모델을 학습하려면 데이터가 필요합니다. 데이터 저장소에 저장된 모든 데이터를 사용하거나 공개적으로 액세스 가능한 URL을 사용할 수 있습니다.

1.  왼쪽 메뉴(자산 라이브러리)에서 **데이터(Data)** 탭을 선택합니다.
2.  **`diabetes-folder`** 구성 요소를 캔버스로 끌어다 놓습니다.
    *   **`diabetes-folder` 데이터 자산의 의미**: `setup.sh` 스크립트는 당뇨병 관련 데이터 (일반적으로 CSV 파일)를 포함하는 폴더를 가리키는 `diabetes-folder`라는 이름의 데이터 자산을 미리 생성했을 것입니다. 이 구성 요소는 해당 데이터 자산을 파이프라인의 입력으로 가져옵니다. 이 데이터는 일반적으로 여러 특성(예: 임신 횟수, 혈당 수치 등)과 당뇨병 발병 여부를 나타내는 대상 변수를 포함합니다.

이제 데이터가 있으므로 작업 영역 내에 이미 존재하는 (설정 중에 생성된) 사용자 지정 구성 요소를 사용하여 파이프라인을 계속 만들 수 있습니다.

1.  왼쪽 메뉴(자산 라이브러리)에서 **구성 요소(Components)** 탭을 선택합니다.
2.  **`Remove Empty Rows`** 구성 요소를 캔버스로 끌어다 `diabetes-folder` 아래에 놓습니다.
3.  데이터(`diabetes-folder`)의 출력 포트를 새 구성 요소(`Remove Empty Rows`)의 입력 포트로 연결합니다. (데이터 구성 요소의 아래쪽 작은 원을 클릭하여 `Remove Empty Rows` 구성 요소의 위쪽 작은 원으로 드래그합니다.)
    *   **`Remove Empty Rows` 구성 요소의 의미**: 데이터셋에서 모든 값이 비어 있거나 누락된 행을 제거하는 전처리 단계입니다. 데이터 품질을 향상시키는 데 도움이 됩니다.
4.  **`Normalize Numerical Columns`** 구성 요소를 캔버스로 끌어다 `Remove Empty Rows` 아래에 놓습니다.
5.  이전 구성 요소(`Remove Empty Rows`)의 출력을 새 구성 요소(`Normalize Numerical Columns`)의 입력으로 연결합니다.
    *   **`Normalize Numerical Columns` 구성 요소의 의미**: 데이터셋의 숫자형 특성들의 값 범위를 비슷한 스케일로 조정합니다. 이는 많은 기계 학습 알고리즘(특히 거리를 기반으로 하거나 경사 하강법을 사용하는 알고리즘)의 성능을 향상시키는 데 도움이 됩니다. 예를 들어, Min-Max 정규화 (값을 0과 1 사이로 조정) 또는 Z-점수 정규화 (평균 0, 표준 편차 1로 조정) 등을 수행할 수 있습니다. (구성 요소의 오른쪽 패널에서 어떤 정규화 방법을 사용할지, 어떤 열에 적용할지 등을 설정할 수 있습니다.)
6.  **`Train a Decision Tree Classifier Model`** 구성 요소를 캔버스로 끌어다 `Normalize Numerical Columns` 아래에 놓습니다.
7.  이전 구성 요소(`Normalize Numerical Columns`)의 출력을 새 구성 요소(`Train a Decision Tree Classifier Model`)의 입력으로 연결합니다. (보통 "Dataset" 또는 "Training data" 입력 포트에 연결합니다.)
    *   **`Train a Decision Tree Classifier Model` 구성 요소의 의미**: 전처리된 데이터를 사용하여 의사 결정 트리(Decision Tree) 분류 모델을 학습시키는 구성 요소입니다. 이 구성 요소는 내부적으로 데이터를 학습용과 테스트용으로 분할하고, 지정된 하이퍼파라미터를 사용하여 모델을 학습시킨 후, 모델과 평가 결과를 출력합니다. (구성 요소의 오른쪽 패널에서 대상 열, 하이퍼파라미터 등을 설정할 수 있습니다.)
8.  오른쪽 상단의 **구성 및 제출(Configure & Submit)** 버튼을 선택합니다.
9.  **파이프라인 작업 설정(Set up pipeline job)** 페이지에서 새 실험(experiment)을 만들고 이름을 **`diabetes-designer-pipeline`** 으로 지정한 다음 **다음(Next)** 을 선택합니다.
10. **입력 및 출력(Inputs & Outputs)** 페이지에서는 변경 없이 **다음(Next)** 을 선택합니다.
11. **런타임 설정(Runtime settings)** 페이지에서 **컴퓨팅 유형(Compute type)** 으로 **컴퓨팅 클러스터(Compute Cluster)** 를 선택하고, **Azure ML 컴퓨팅 클러스터 선택(Select Azure ML compute cluster)** 아래에서 `aml-cluster`를 선택합니다.
12. **검토 + 제출(Review + Submit)** 을 선택한 다음 **제출(Submit)** 을 선택하여 파이프라인 실행을 시작합니다.
13. **파이프라인(Pipelines)** 페이지로 이동하여 **`Train-Diabetes-Classifier`** 파이프라인을 선택하여 실행 상태를 확인할 수 있습니다.
14. 모든 구성 요소가 성공적으로 완료될 때까지 기다립니다.

작업을 제출하면 컴퓨팅 클러스터가 초기화됩니다. 지금까지 컴퓨팅 클러스터가 유휴 상태였으므로 클러스터 크기가 0개 노드 이상으로 조정되는 데 시간이 걸릴 수 있습니다. 클러스터 크기가 조정되면 자동으로 파이프라인 실행을 시작합니다.

각 구성 요소의 실행을 추적할 수 있습니다. 파이프라인이 실패하면 어떤 구성 요소가 실패했고 그 이유를 탐색할 수 있습니다. 오류 메시지는 작업 개요의 **출력 + 로그(Outputs + logs)** 탭에 표시됩니다.

---

### 비교를 위해 두 번째 모델 학습

알고리즘 간에 비교하고 어떤 것이 더 나은 성능을 보이는지 평가하기 위해 하나의 파이프라인 내에서 두 개의 모델을 학습하고 비교할 수 있습니다.

1.  **디자이너(Designer)** 로 돌아가서 **`Train-Diabetes-Classifier`** 파이프라인 초안(draft)을 선택합니다. (이전에 실행한 작업이 아니라, 편집 가능한 파이프라인 디자인을 엽니다.)
2.  **`Train a Logistic Regression Classifier Model`** 구성 요소를 캔버스의 다른 학습 구성 요소 옆에 추가합니다.
3.  **`Normalize Numerical Columns`** 구성 요소의 출력을 새 학습 구성 요소(`Train a Logistic Regression Classifier Model`)의 입력으로 연결합니다. (이렇게 하면 두 모델이 동일하게 전처리된 데이터를 사용하여 학습합니다.)
    *   **`Train a Logistic Regression Classifier Model` 구성 요소의 의미**: 로지스틱 회귀(Logistic Regression) 분류 모델을 학습시키는 구성 요소입니다. 의사 결정 트리와는 다른 원리로 작동하는 선형 모델입니다.
4.  상단에서 **구성 및 제출(Configure & Submit)** 을 선택합니다.
5.  **기본(Basics)** 페이지에서 **`designer-compare-classification`** 이라는 새 실험을 만들고 실행합니다. (실험 이름을 변경하면 이전 실행과 구분하여 결과를 추적할 수 있습니다.)
6.  **검토 + 제출(Review + Submit)** 을 선택한 다음 **제출(Submit)** 을 선택하여 파이프라인 실행을 시작합니다.
7.  **파이프라인(Pipelines)** 페이지로 이동하여 `designer-compare-classification` 실험을 사용하는 **`Train-Diabetes-Classifier`** 파이프라인을 선택하여 실행 상태를 확인할 수 있습니다.
8.  모든 구성 요소가 성공적으로 완료될 때까지 기다립니다.
9.  **작업 개요(Job overview)** 를 선택한 다음, **메트릭(Metrics)** 탭을 선택하여 두 학습 구성 요소의 결과를 검토합니다.
    *   각 학습 구성 요소(예: `Train a Decision Tree Classifier Model`, `Train a Logistic Regression Classifier Model`)를 클릭하면 해당 모델의 성능 메트릭(예: 정확도(Accuracy), AUC, 정밀도(Precision), 재현율(Recall) 등)을 시각화된 차트나 테이블 형태로 볼 수 있습니다.
10. 어떤 모델이 더 나은 성능을 보였는지 확인해 보십시오. (예를 들어, AUC 값이 더 높거나, 특정 비즈니스 요구에 맞는 메트릭 값이 더 좋은 모델을 선택할 수 있습니다.)

---

### Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 만든 리소스를 삭제해야 합니다.

1.  Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
2.  Azure Portal의 **홈(Home)** 페이지에서 **리소스 그룹(Resource groups)** 을 선택합니다.
3.  `rg-dp100-…` (스크립트가 생성한 리소스 그룹 이름) 리소스 그룹을 선택합니다.
4.  리소스 그룹의 **개요(Overview)** 페이지 상단에서 **리소스 그룹 삭제(Delete resource group)** 를 선택합니다.
5.  삭제를 확인하기 위해 리소스 그룹 이름을 입력하고 **삭제(Delete)** 를 선택합니다.

이것으로 "Azure Machine Learning 디자이너로 모델 학습하기" 실습이 완료됩니다. 이 실습을 통해 코드를 직접 작성하지 않고도 시각적인 인터페이스를 사용하여 데이터 전처리, 모델 학습, 모델 비교까지 수행하는 전체적인 기계 학습 파이프라인을 구축하고 실행하는 방법을 경험했습니다. 디자이너는 특히 프로토타이핑이나 복잡한 코딩 없이 빠르게 모델을 실험해보고자 할 때 유용합니다.
