## Azure Machine Learning에서 데이터 사용 가능하게 만들기

로컬 파일 시스템에서 데이터를 사용하는 것이 일반적이지만, 엔터프라이즈 환경에서는 여러 데이터 과학자 및 기계 학습 엔지니어가 액세스할 수 있는 중앙 위치에 데이터를 저장하는 것이 더 효과적일 수 있습니다.

이 연습에서는 Azure Machine Learning에서 데이터 액세스를 추상화하는 데 사용되는 주요 개체인 데이터 저장소(Datastores)와 데이터 자산(Data Assets)을 살펴봅니다.

**실습 목표:**

*   Azure CLI를 사용하여 Azure Machine Learning 작업 영역 및 컴퓨팅 리소스 프로비저닝하기
*   Azure Portal에서 기본 데이터 저장소(Datastores)의 구성 요소인 Storage Account 탐색하기
*   Storage Account 액세스 키를 사용하여 사용자 지정 데이터 저장소(Datastore)를 만들 준비하기
*   Python SDK를 사용하여 Azure ML 작업 영역에 실습 자료 복제하기
*   Python SDK를 사용하여 데이터 저장소(Datastore) 및 데이터 자산(Data Asset) 만들기
*   생성된 데이터 자산(Data Asset) 탐색하기
*   생성된 Azure 리소스 정리하기

**준비물:**

*   관리자 수준 액세스 권한이 있는 Azure 구독

---

### Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스와 자산을 관리하기 위한 중앙 집중식 공간을 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

이 실습에서는 Azure CLI를 사용하는 셸 스크립트를 사용하여 작업 영역과 필요한 리소스를 프로비저닝합니다. 다음으로, Azure Machine Learning 스튜디오의 디자이너를 사용하여 모델을 학습하고 비교합니다. (이 부분은 다음 실습 내용으로 보이며, 이번 실습에서는 주로 데이터 관련 작업에 집중합니다.)

**작업 영역 및 컴퓨팅 리소스 만들기**

Azure Machine Learning 작업 영역과 컴퓨팅 리소스를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 사용자가 실행할 수 있도록 셸 스크립트에 그룹화되어 있습니다.

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
    cd azure-ml-labs/Labs/03
    ./setup.sh
    ```

    **참고**: 확장이 설치되지 않았다는 (오류) 메시지는 무시하십시오.

7.  스크립트가 완료될 때까지 기다립니다. 일반적으로 약 5-10분 정도 소요됩니다.

    **문제 해결 팁: 작업 영역 생성 오류**
    만약 `setup.sh` 스크립트 실행 중 작업 영역 생성에 실패하는 경우, 이는 주로 선택한 지역의 특정 VM 크기에 대한 구독 할당량이 부족하기 때문일 수 있습니다.
    *   **해결 방법**: `setup.sh` 스크립트 내용을 확인하여 (`cat azure-ml-labs/Labs/03/setup.sh`) 컴퓨팅 인스턴스나 클러스터 생성 시 사용되는 VM 크기를 구독에서 사용 가능한 다른 크기로 변경한 후 다시 실행해 보십시오. (예: `Standard_DS3_v2`를 `Standard_DS2_v2` 등으로 변경)

---

### 기본 데이터 저장소(Datastores) 탐색

Azure Machine Learning 작업 영역을 만들면 Storage Account가 자동으로 생성되어 작업 영역에 연결됩니다. Storage Account가 어떻게 연결되어 있는지 살펴보겠습니다.

1.  Azure Portal에서 `rg-dp100-…` (스크립트가 생성한 실제 리소스 그룹 이름)라는 새 리소스 그룹으로 이동합니다.
2.  리소스 그룹에서 **Storage Account**를 선택합니다. 이름은 종종 작업 영역에 대해 제공한 이름(하이픈 제외)으로 시작합니다. (예: `mlwdp100labsstorage...`)
3.  Storage Account의 **개요(Overview)** 페이지를 검토합니다. 개요 창과 왼쪽 메뉴에 표시된 것처럼 Storage Account에는 **데이터 저장소(Data storage)** 에 대한 몇 가지 옵션이 있습니다.
    *   **주요 데이터 저장소 옵션의 의미:**
        *   **컨테이너(Containers) (Blob Storage)**: 비정형 데이터(이미지, 비디오, 로그 파일 등) 및 대량의 데이터를 저장하는 데 사용됩니다. Azure ML에서는 학습 데이터, 모델 파일, 스크립트 등을 저장하는 데 주로 활용됩니다.
        *   **파일 공유(File shares) (Azure Files)**: SMB 프로토콜을 통해 액세스할 수 있는 완전 관리형 파일 공유입니다. 여러 VM에서 동시에 탑재하여 사용할 수 있습니다. Azure ML에서는 Notebook 파일, 스크립트 등을 저장하고 컴퓨팅 인스턴스에서 쉽게 접근할 수 있도록 하는 데 사용됩니다.
        *   **테이블(Tables) (Azure Table Storage)**: NoSQL 키-값 저장소로, 구조화된 비관계형 데이터를 저장하는 데 적합합니다.
        *   **큐(Queues) (Azure Queue Storage)**: 비동기 작업 처리를 위한 메시지 큐 서비스입니다.
4.  Storage Account의 Blob 저장소 부분을 탐색하려면 **컨테이너(Containers)** 를 선택합니다.
5.  `azureml-blobstore-…` 컨테이너를 확인합니다. 데이터 자산(data assets)을 위한 기본 데이터 저장소(datastore)는 이 컨테이너를 사용하여 데이터를 저장합니다.
6.  화면 상단의 **+ 컨테이너(+ Container)** 버튼을 사용하여 새 컨테이너를 만들고 이름을 `training-data`로 지정합니다.
    *   **`training-data` 컨테이너의 의미:** 이 컨테이너는 사용자가 직접 관리하고자 하는 학습 데이터를 별도로 구성하기 위해 생성하는 예시입니다. 예를 들어, 여러 프로젝트나 팀에서 공통으로 사용하는 데이터를 저장하거나, 특정 접근 권한을 설정하고자 할 때 유용합니다. 이 실습의 Notebook에서는 이 컨테이너를 직접 사용하지 않을 수 있지만, 데이터 관리의 한 가지 방법을 보여줍니다.
7.  왼쪽 메뉴에서 **파일 공유(File shares)** 를 선택하여 Storage Account의 파일 공유 부분을 탐색합니다.
8.  `code-…` 파일 공유를 확인합니다. 작업 영역의 모든 Notebook은 여기에 저장됩니다. 실습 자료를 복제한 후, `code-.../Users/your-user-name/azure-ml-labs` 폴더에서 해당 파일 공유의 파일을 찾을 수 있습니다.

---

### 액세스 키 복사

Azure Machine Learning 작업 영역에서 데이터 저장소(datastore)를 만들려면 몇 가지 자격 증명을 제공해야 합니다. 작업 영역에 Blob 저장소에 대한 액세스 권한을 부여하는 쉬운 방법은 계정 키를 사용하는 것입니다.

1.  Storage Account에서 왼쪽 메뉴의 **액세스 키(Access keys)** 탭을 선택합니다.
2.  두 개의 키(`key1` 및 `key2`)가 제공되는 것을 확인합니다. 각 키는 동일한 기능을 수행합니다. (하나는 기본, 다른 하나는 키 순환 시 서비스 중단을 방지하기 위한 백업용입니다.)
3.  `key1` 아래의 **키(Key)** 필드에 대해 **표시(Show)** 를 선택합니다.
4.  **키(Key)** 필드의 값을 메모장에 복사합니다. 나중에 Notebook에 이 값을 붙여넣어야 합니다.
5.  페이지 상단에서 **Storage Account 이름**을 복사합니다. 이름은 `mlwdp100storage…` (실제 생성된 이름)로 시작해야 합니다. 이 값도 나중에 Notebook에 붙여넣어야 합니다.

**참고**: (Word에서 발생하는) 자동 대문자 변환을 피하려면 계정 키와 이름을 메모장에 복사하십시오. 키는 대소문자를 구분합니다.

---

### 실습 자료 복제 (Azure ML 작업 영역 내)

Python SDK를 사용하여 데이터 저장소(datastore)와 데이터 자산(data asset)을 만들려면 작업 영역에 실습 자료를 복제해야 합니다.

1.  Azure Portal에서 `mlw-dp100-labs` (스크립트가 생성한 실제 작업 영역 이름)라는 Azure Machine Learning 작업 영역으로 이동합니다.
2.  Azure Machine Learning 작업 영역을 선택하고 **개요(Overview)** 페이지에서 **스튜디오 시작(Launch studio)** 을 선택합니다. 브라우저에서 다른 탭이 열리면서 Azure Machine Learning 스튜디오가 열립니다.
3.  스튜디오에 나타나는 모든 팝업을 닫습니다.
4.  Azure Machine Learning 스튜디오 내에서 **컴퓨팅(Compute)** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스는 실행 중이어야 하고, 클러스터는 유휴 상태이며 실행 중인 노드가 0개여야 합니다.
5.  **컴퓨팅 인스턴스(Compute instances)** 탭에서 컴퓨팅 인스턴스를 찾아 **터미널(Terminal)** 애플리케이션을 선택합니다.
6.  터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```bash
    pip uninstall azure-ai-ml -y
    pip install azure-ai-ml
    pip install mltable
    ```

    **참고**: 패키지가 설치되지 않았다는 (오류) 메시지는 무시하십시오. `mltable`은 Azure ML에서 테이블 형식 데이터를 정의하고 사용하는 데 사용되는 형식입니다.

7.  다음 명령을 실행하여 Notebook, 데이터 및 기타 파일을 포함하는 Git 리포지토리를 작업 영역에 복제합니다.

    ```bash
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

8.  명령이 완료되면 **파일(Files)** 창에서 **↻ (새로고침)** 을 클릭하여 보기를 새로 고치고 새 `Users/your-user-name/azure-ml-labs` 폴더가 생성되었는지 확인합니다.
9.  선택적으로, 다른 브라우저 탭에서 Azure Portal로 다시 이동합니다. Storage Account의 `code-...` 파일 공유를 다시 탐색하여 새로 생성된 `azure-ml-labs` 폴더에서 복제된 실습 자료를 찾습니다. 이는 컴퓨팅 인스턴스의 파일 시스템이 실제로 작업 영역과 연결된 Storage Account의 파일 공유에 매핑되어 있음을 보여줍니다.

---

### 데이터 저장소(Datastore) 및 데이터 자산(Data Asset) 만들기

Python SDK를 사용하여 데이터 저장소(datastore)와 데이터 자산(data asset)을 만드는 코드가 Notebook에 제공됩니다.

1.  **`Labs/03/Work with data.ipynb`** Notebook을 엽니다.
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> `03` 폴더로 이동하여 `Work with data.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.
3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다.
4.  Notebook의 모든 셀을 실행합니다.
    *   이 Notebook에서는 이전에 복사한 **Storage Account 이름**과 **액세스 키**를 사용하여 Azure Blob Storage에 대한 사용자 지정 **데이터 저장소(Datastore)** 를 만듭니다.
    *   또한 로컬 파일(또는 URL)에서 데이터를 업로드하여 **데이터 자산(Data Asset)** 을 생성하고, 앞에서 만든 데이터 저장소(Datastore)를 통해 특정 경로의 데이터에 대한 참조로서 데이터 자산(Data Asset)을 생성하는 방법을 보여줍니다.

---

### 선택 사항: 데이터 자산(Data Asset) 탐색

선택적으로, 데이터 자산(data asset)이 연결된 Storage Account에 어떻게 저장되는지 탐색할 수 있습니다.

1.  Azure Machine Learning 스튜디오의 **데이터(Data)** 탭으로 이동하여 데이터 자산(data asset)을 탐색합니다.
2.  `diabetes-local` 데이터 자산(data asset) 이름을 선택하여 세부 정보를 탐색합니다. (Notebook에서 이 이름으로 데이터 자산을 생성했을 것입니다.)
3.  `diabetes-local` 데이터 자산(data asset)의 **데이터 원본(Data sources)** 아래에서 파일이 업로드된 위치를 찾을 수 있습니다. `LocalUpload/...`로 시작하는 경로는 Storage Account 컨테이너 `azureml-blobstore-...` (기본 데이터 저장소(datastore) 컨테이너) 내의 경로를 보여줍니다. Azure Portal에서 해당 경로로 이동하여 파일이 실제로 존재하는지 확인할 수 있습니다.
    *   **데이터 자산의 의미:**
        *   **`diabetes-local`**: 이 데이터 자산은 Notebook을 실행하는 컴퓨팅 인스턴스의 로컬 환경(또는 Notebook 코드 내에서 접근 가능한 위치)에서 `diabetes.csv` 파일을 Azure ML 작업 영역과 연결된 기본 Blob Storage (`workspaceblobstore`)의 특정 경로 (`LocalUpload/...`)로 업로드하여 생성됩니다. 이 방식은 로컬 데이터를 Azure ML에서 사용 가능하게 만드는 가장 직접적인 방법 중 하나입니다.
        *   Notebook에서 다른 유형의 데이터 자산(예: 특정 URI를 참조하는 데이터 자산)도 생성했을 수 있습니다. 이러한 데이터 자산은 데이터를 복제하는 대신 기존 저장소 위치에 있는 데이터에 대한 포인터 역할을 합니다. 이를 통해 데이터 중복을 피하고 최신 데이터에 항상 접근할 수 있습니다.

---

### Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 만든 리소스를 삭제해야 합니다.

1.  Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
2.  Azure Portal의 **홈(Home)** 페이지에서 **리소스 그룹(Resource groups)** 을 선택합니다.
3.  `rg-dp100-…` (스크립트가 생성한 리소스 그룹 이름) 리소스 그룹을 선택합니다.
4.  리소스 그룹의 **개요(Overview)** 페이지 상단에서 **리소스 그룹 삭제(Delete resource group)** 를 선택합니다.
5.  삭제를 확인하기 위해 리소스 그룹 이름을 입력하고 **삭제(Delete)** 를 선택합니다.

이것으로 "Azure Machine Learning에서 데이터 사용 가능하게 만들기" 실습이 완료됩니다. 이 실습을 통해 Azure ML에서 데이터를 관리하고 접근하는 핵심 구성 요소인 데이터 저장소(Datastores)와 데이터 자산(Data Assets)의 개념 및 생성 방법을 이해했습니다.
