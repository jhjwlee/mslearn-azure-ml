## Azure Machine Learning에서 환경(Environments) 사용하기

Notebook과 스크립트를 실행하려면 필요한 패키지가 설치되어 있는지 확인해야 합니다. 환경(Environments)을 사용하면 코드를 실행하기 위해 컴퓨팅에서 사용해야 하는 런타임 및 Python 패키지를 지정할 수 있습니다.

이 실습에서는 환경에 대해 배우고 Azure Machine Learning 컴퓨팅으로 기계 학습 모델을 학습할 때 환경을 사용하는 방법을 알아봅니다.

**실습 목표:**

*   Azure CLI를 사용하여 Azure Machine Learning 작업 영역 및 컴퓨팅 리소스 프로비저닝하기
*   Python SDK를 사용하여 컴퓨팅 인스턴스 터미널에서 Git 리포지토리 복제하기
*   큐레이트된 환경(Curated Environment)을 사용하여 Azure ML 작업(Job) 실행하기
*   작업 영역의 환경 목록 확인하고 특정 환경 정보 검토하기
*   Docker 이미지만으로 사용자 지정 환경(Custom Environment) 생성하고 작업에 사용해보기 (오류 발생 예상)
*   Docker 이미지와 Conda 명세 파일을 결합하여 필요한 패키지가 포함된 사용자 지정 환경 생성하고 작업 실행하기
*   환경 빌드 로그 확인 방법 이해하기

**준비물:**

*   관리자 수준 액세스 권한이 있는 Azure 구독

---

### Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스와 자산을 관리하기 위한 중앙 집중식 공간을 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

이 실습에서는 Azure CLI를 사용하여 작업 영역과 필요한 컴퓨팅을 프로비저닝하고, Python SDK를 사용하여 환경을 관리하고 작업을 실행합니다.

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
    cd azure-ml-labs/Labs/04
    ./setup.sh
    ```

    **참고**: 확장이 설치되지 않았다는 (오류) 메시지는 무시하십시오.

7.  스크립트가 완료될 때까지 기다립니다. 일반적으로 약 5-10분 정도 소요됩니다.

    **문제 해결 팁: 작업 영역 생성 오류**
    만약 `setup.sh` 스크립트 실행 중 작업 영역 생성에 실패하는 경우, 이는 주로 선택한 지역의 특정 VM 크기에 대한 구독 할당량이 부족하기 때문일 수 있습니다.
    *   **해결 방법**: `setup.sh` 스크립트 내용을 확인하여 (`cat azure-ml-labs/Labs/04/setup.sh`) 컴퓨팅 인스턴스나 클러스터 생성 시 사용되는 VM 크기를 구독에서 사용 가능한 다른 크기로 변경한 후 다시 실행해 보십시오.

---

### 실습 자료 복제 (Azure ML 작업 영역 내)

작업 영역과 필요한 컴퓨팅 리소스를 만들었으면 Azure Machine Learning 스튜디오를 열고 작업 영역에 실습 자료를 복제할 수 있습니다.

1.  Azure Portal에서 `mlw-dp100-…` (스크립트가 생성한 실제 작업 영역 이름)라는 Azure Machine Learning 작업 영역으로 이동합니다.
2.  Azure Machine Learning 작업 영역을 선택하고 **개요(Overview)** 페이지에서 **스튜디오 시작(Launch studio)** 을 선택합니다. 브라우저에서 다른 탭이 열리면서 Azure Machine Learning 스튜디오가 열립니다.
3.  스튜디오에 나타나는 모든 팝업을 닫습니다.
4.  Azure Machine Learning 스튜디오 내에서 **컴퓨팅(Compute)** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스는 실행 중이어야 하고, 클러스터는 유휴 상태이며 실행 중인 노드가 0개여야 합니다.
5.  **컴퓨팅 인스턴스(Compute instances)** 탭에서 컴퓨팅 인스턴스를 찾아 **터미널(Terminal)** 애플리케이션을 선택합니다.
6.  터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```bash
    pip uninstall azure-ai-ml -y
    pip install azure-ai-ml
    ```

    **참고**: 패키지를 찾을 수 없거나 제거할 수 없다는 (오류) 메시지는 무시하십시오.

7.  다음 명령을 실행하여 Notebook, 데이터 및 기타 파일을 포함하는 Git 리포지토리를 작업 영역에 복제합니다.

    ```bash
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

8.  명령이 완료되면 **파일(Files)** 창에서 **↻ (새로고침)** 을 클릭하여 보기를 새로 고치고 새 `Users/your-user-name/azure-ml-labs` 폴더가 생성되었는지 확인합니다.

---

### 환경(Environments) 작업

Python SDK를 사용하여 환경을 만들고 관리하는 코드가 Notebook에 제공됩니다.

1.  **`Labs/04/Work with environments.ipynb`** Notebook을 엽니다.
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> `04` 폴더로 이동하여 `Work with environments.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.
3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다. (Notebook 상단 오른쪽에서 커널을 확인할 수 있으며, 필요시 변경합니다.)
4.  Notebook의 모든 셀을 실행합니다.

---

## `Work with environments.ipynb` Notebook 상세 분석

이 Notebook은 Azure Machine Learning (Azure ML)에서 스크립트 실행 환경을 정의하고 관리하는 **환경(Environments)** 의 개념과 사용법을 Python SDK (v2)를 통해 보여줍니다.

### 1. 소개 및 환경 설정

*   스크립트를 Azure ML 작업(Job)으로 실행할 때 필요한 실행 컨텍스트, 특히 컴퓨팅 대상(Compute Target)과 실행 환경(Environment)의 중요성을 설명합니다.
*   `azure-ai-ml` 패키지 설치 여부를 확인합니다.

### 2. 작업 영역(Workspace)에 연결

*   이전 실습과 동일하게 `DefaultAzureCredential` 또는 `InteractiveBrowserCredential`을 사용하여 `MLClient`를 통해 Azure ML 작업 영역에 연결합니다.

### 3. 스크립트를 작업(Job)으로 실행

#### 3.1 학습 스크립트 생성 (`diabetes-training.py`)

```python
%%writefile src/diabetes-training.py
# import libraries
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import roc_auc_score
# ... (rest of the script) ...
```

*   **목적**: 당뇨병 데이터셋(`diabetes.csv`)을 사용하여 로지스틱 회귀 모델을 학습하고 평가하는 간단한 Python 스크립트를 생성합니다.
*   **중요 사항**: 스크립트 상단에 `pandas`, `numpy`, `sklearn`과 같은 라이브러리를 `import`합니다. 이 스크립트를 실행하는 모든 컴퓨팅 환경에는 이러한 라이브러리가 설치되어 있어야 합니다.
*   **데이터 파일**: 이 스크립트는 같은 폴더(`src/`)에 `diabetes.csv` 파일이 있다고 가정하고 데이터를 로드합니다. (실제 `setup.sh` 스크립트가 이 파일을 해당 위치에 복사해 두었을 것입니다.)

#### 3.2 큐레이트된 환경(Curated Environment)을 사용하여 작업 제출

```python
from azure.ai.ml import command

# configure job
job = command(
    code="./src",  # diabetes-training.py와 diabetes.csv가 있는 폴더
    command="python diabetes-training.py",
    environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest", # 큐레이트된 환경
    compute="aml-cluster", # setup.sh에서 생성된 컴퓨팅 클러스터
    display_name="diabetes-train-curated-env",
    experiment_name="diabetes-training"
)

# submit job
returned_job = ml_client.create_or_update(job)
aml_url = returned_job.studio_url
print("Monitor your job at", aml_url)
```

*   **목적**: `diabetes-training.py` 스크립트를 Azure ML 작업으로 실행합니다.
*   **`environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest"`**:
    *   **큐레이트된 환경(Curated Environment)**: Azure ML에서 미리 빌드하여 제공하는 환경입니다. 일반적인 기계 학습 작업에 필요한 패키지들(예: Scikit-learn, Pandas, NumPy 등)이 이미 설치되어 있어 사용자가 직접 환경을 구성하는 수고를 덜어줍니다.
    *   이름 형식: `AzureML-<framework>-<version>-<os>-<python_version>-<cpu/gpu>`
    *   `@latest`: 해당 환경의 최신 버전을 사용하겠다는 의미입니다.
*   **`compute="aml-cluster"`**: `setup.sh` 스크립트를 통해 미리 프로비저닝된 컴퓨팅 클러스터에서 이 작업을 실행하도록 지정합니다.
*   **결과**: 이 작업은 큐레이트된 환경에 필요한 라이브러리가 모두 포함되어 있으므로 성공적으로 실행될 것입니다.

### 4. 환경(Environments) 목록 확인 및 검토

#### 4.1 작업 영역의 모든 환경 나열

```python
envs = ml_client.environments.list()
for env in envs:
    print(env.name)
```

*   **목적**: 현재 Azure ML 작업 영역에 등록된 모든 환경(큐레이트된 환경 및 사용자가 만든 사용자 지정 환경)의 이름을 출력합니다.
*   **큐레이트된 환경 이름 규칙**: `AzureML-`로 시작합니다.

#### 4.2 특정 환경 정보 가져오기

```python
env = ml_client.environments.get("AzureML-sklearn-0.24-ubuntu18.04-py37-cpu", version=44) # 버전은 예시, 실제 버전은 다를 수 있음
print(env.description, env.tags)
```

*   **목적**: 특정 이름과 버전의 환경 객체를 가져와서 설명(description)이나 태그(tags)와 같은 메타데이터를 확인할 수 있습니다. (실제 버전 번호는 환경 목록을 확인하거나 `@latest`로 가져온 후 확인해야 합니다.)

### 5. 사용자 지정 환경(Custom Environment) 생성 및 사용

큐레이트된 환경에 필요한 모든 패키지가 없다면, 사용자 정의 환경을 만들 수 있습니다.

#### 5.1 Docker 이미지만으로 환경 생성

```python
from azure.ai.ml.entities import Environment

env_docker_image = Environment(
    image="mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04", # 기본 Docker 이미지
    name="docker-image-example",
    description="Environment created from a Docker image.",
)
ml_client.environments.create_or_update(env_docker_image)
```

*   **목적**: 공개적으로 사용 가능한 Docker 이미지를 기반으로 사용자 지정 환경을 만듭니다.
*   **`image="mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04"`**: Microsoft Container Registry(MCR)에서 제공하는 기본 Docker 이미지입니다. 이 이미지는 OS(Ubuntu 18.04)와 MPI(OpenMPI) 정도만 포함하고 있을 가능성이 높으며, Python이나 일반적인 ML 패키지는 포함하지 않을 수 있습니다.

#### 5.2 Docker 이미지만으로 생성된 환경을 사용하여 작업 제출 (실패 예상)

```python
# ... (job configuration using environment="docker-image-example:1") ...
job = command(
    # ...
    environment="docker-image-example:1", # 방금 만든 사용자 지정 환경
    # ...
)
# ... (submit job) ...
```

```markdown
<p style="color:red;font-size:120%;background-color:yellow;font-weight:bold"> The job will quickly fail! Review the error message. </p>
...
The error message will tell you that there is no module named pandas.
...
```

*   **목적**: 이전 단계에서 Docker 이미지만으로 만든 환경을 사용하여 `diabetes-training.py` 스크립트를 실행합니다.
*   **예상 결과: 실패 (ModuleNotFoundError: No module named 'pandas')**
    *   `mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04` 이미지는 `pandas`, `scikit-learn` 등의 Python 패키지를 포함하고 있지 않습니다.
    *   `diabetes-training.py` 스크립트는 이러한 패키지를 필요로 하므로, 스크립트 실행 시 "module not found" 오류가 발생하며 작업이 실패합니다.
*   **교훈**: 환경을 정의할 때는 스크립트 실행에 필요한 **모든 종속성**이 포함되도록 해야 합니다.

#### 5.3 Conda 명세 파일 생성 (`conda-env.yml`)

```python
%%writefile src/conda-env.yml
name: basic-env-cpu
channels:
  - conda-forge
dependencies:
  - python=3.11
  - scikit-learn
  - pandas
  - numpy
  - matplotlib
```

*   **목적**: `diabetes-training.py` 스크립트 실행에 필요한 Python 버전과 패키지들을 정의하는 Conda 환경 명세 파일을 만듭니다.
*   **내용**: Python 3.11 버전과 `scikit-learn`, `pandas`, `numpy`, `matplotlib` 패키지를 `conda-forge` 채널에서 설치하도록 지정합니다.

#### 5.4 Docker 이미지와 Conda 명세 파일을 결합하여 환경 생성

```python
from azure.ai.ml.entities import Environment

env_docker_conda = Environment(
    image="mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04", # 기본 Docker 이미지
    conda_file="./src/conda-env.yml", # Conda 명세 파일 경로
    name="docker-image-plus-conda-example",
    description="Environment created from a Docker image plus Conda environment.",
)
ml_client.environments.create_or_update(env_docker_conda)
```

*   **목적**: 기본 Docker 이미지 위에 Conda 명세 파일에 정의된 패키지들을 설치하여 완전한 실행 환경을 구축합니다.
*   **`conda_file="./src/conda-env.yml"`**: 이 매개변수를 통해 Azure ML은 지정된 Docker 이미지 내부에 Conda 환경을 구성하고 `conda-env.yml` 파일에 명시된 패키지들을 설치합니다.
*   **환경 빌드**: 이 환경이 작업 영역에 처음 등록될 때, Azure ML은 백그라운드에서 실제 Docker 이미지를 빌드합니다 (기본 이미지 + Conda 패키지 설치). 이 과정은 몇 분에서 몇십 분까지 소요될 수 있습니다.

#### 5.5 Docker 이미지 + Conda 명세로 생성된 환경을 사용하여 작업 제출

```python
# ... (job configuration using environment="docker-image-plus-conda-example:1") ...
job = command(
    # ...
    environment="docker-image-plus-conda-example:1", # Conda 패키지가 포함된 사용자 지정 환경
    # ...
)
# ... (submit job) ...
```

*   **목적**: 필요한 모든 패키지가 포함된 사용자 지정 환경을 사용하여 `diabetes-training.py` 스크립트를 실행합니다.
*   **예상 결과: 성공** (환경 빌드가 완료된 후)
    *   이 환경에는 `pandas`, `scikit-learn` 등이 설치되어 있으므로 스크립트가 정상적으로 실행됩니다.
*   **환경 빌드 시간**: 이 환경을 처음 사용하는 작업은 환경 빌드 시간 때문에 평소보다 오래 걸릴 수 있습니다. 한 번 빌드된 환경은 캐시되어 다음 사용 시에는 빠르게 로드됩니다.
*   **빌드 로그**: Markdown 설명에 언급된 것처럼, 환경 빌드 과정은 작업의 "Outputs + logs" 탭에 있는 `azureml-logs/20_image_build_log.txt` 파일에서 확인할 수 있습니다. 이 로그는 환경 구성 중 문제가 발생했을 때 디버깅에 매우 유용합니다.

---

이 Notebook은 Azure ML에서 재현 가능하고 이식 가능한 스크립트 실행을 위해 환경을 어떻게 정의하고 사용하는지 잘 보여줍니다. 큐레이트된 환경의 편리함과 사용자 지정 환경의 유연성을 모두 다루고 있으며, 특히 Docker 이미지와 Conda 명세 파일을 결합하는 방법은 실무에서 매우 유용하게 사용될 수 있는 방식입니다. 환경 빌드 과정과 로그 확인의 중요성도 강조하고 있습니다.
