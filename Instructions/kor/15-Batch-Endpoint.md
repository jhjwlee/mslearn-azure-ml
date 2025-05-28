제공된 노트북 내용은 Azure Machine Learning에서 **MLflow 모델을 배치 엔드포인트(Batch Endpoint)에 배포**하고 대규모 데이터셋에 대한 추론을 수행하는 과정을 보여줍니다. 실시간 예측을 위한 온라인 엔드포인트와 달리, 배치 엔드포인트는 한 번에 많은 양의 데이터를 비동기적으로 처리하는 데 특화되어 있습니다.

각 셀의 목적과 주요 개념을 설명해 드리겠습니다.

---

### **전반적인 목표:**

*   대규모의 데이터를 처리하여 예측 결과를 얻기 위해 MLflow 모델을 **관리형 배치 엔드포인트**에 배포합니다.
*   배치 엔드포인트에 잡(Job)을 제출하여 모델을 테스트하고 예측 결과를 확인합니다.

---

### **온라인 엔드포인트와의 주요 차이점 (배치 엔드포인트 특징):**

1.  **목적:**
    *   **온라인 엔드포인트:** 실시간 예측, 즉각적인 응답이 필요한 소량의 데이터 처리 (예: 웹 애플리케이션의 추천 시스템).
    *   **배치 엔드포인트:** 비동기적으로 대규모 데이터 처리, 즉각적인 응답이 필요 없는 시나리오 (예: 야간 배치 처리, 월별 보고서 생성).
2.  **호출 방식:**
    *   **온라인 엔드포인트:** 직접 HTTP(S) 요청을 보내어 실시간으로 응답을 받습니다.
    *   **배치 엔드포인트:** 엔드포인트를 호출하면 Azure ML 파이프라인 잡(Job)이 생성되어 실행됩니다. 예측 결과는 잡이 완료된 후 지정된 스토리지에 저장됩니다.
3.  **컴퓨팅 리소스:**
    *   **온라인 엔드포인트:** `instance_type` (VM 크기) 및 `instance_count`로 실시간 요청을 처리할 웹 서버 인스턴스를 지정합니다.
    *   **배치 엔드포인트:** `compute` (컴퓨팅 클러스터)를 지정하여 대규모 병렬 처리를 수행합니다. `instance_count`, `max_concurrency_per_instance`, `mini_batch_size` 등 배치 처리에 특화된 파라미터가 있습니다.
4.  **모델 등록:**
    *   **온라인 엔드포인트:** 로컬 경로에 있는 모델을 직접 배포할 수 있습니다 (이 경우 Azure ML이 자동으로 모델을 등록).
    *   **배치 엔드포인트:** 모델을 배포하기 전에 반드시 **Azure ML 작업 영역에 등록**되어 있어야 합니다.

---

### **단계별 설명:**

#### **1. 필수 패키지 확인 및 설치 (`pip show azure-ai-ml`)**

```python
pip show azure-ai-ml
```

*   **목적:** Azure Machine Learning SDK v2인 `azure-ai-ml` 패키지가 현재 환경에 설치되어 있는지 확인합니다.

#### **2. 작업 영역(Workspace) 연결**

```python
from azure.identity import DefaultAzureCredential, InteractiveBrowserCredential
from azure.ai.ml import MLClient

# ... (인증 로직) ...

ml_client = MLClient.from_config(credential=credential)
```

*   **목적:** Python SDK를 사용하여 Azure Machine Learning 작업 영역에 연결합니다. 온라인 엔드포인트와 동일하게, `MLClient` 객체를 통해 Azure ML 리소스를 관리합니다.

#### **3. 모델 등록 (Register the model)**

```python
from azure.ai.ml.entities import Model
from azure.ai.ml.constants import AssetTypes

model_name = 'diabetes-mlflow'
model = ml_client.models.create_or_update(
    Model(name=model_name, path='./model', type=AssetTypes.MLFLOW_MODEL)
)
```

*   **목적:** 배치 배포는 **작업 영역에 등록된 모델만 배포**할 수 있습니다. 따라서 로컬 `model` 폴더에 있는 MLflow 모델을 Azure ML 작업 영역에 자산(Asset)으로 등록합니다.
*   **설명:**
    *   `Model`: Azure ML에서 모델 자산을 나타내는 클래스입니다.
    *   `name`: 모델의 이름을 지정합니다.
    *   `path='./model'`: 모델 파일이 저장된 로컬 경로를 지정합니다.
    *   `type=AssetTypes.MLFLOW_MODEL`: 이 모델이 MLflow 형식임을 나타냅니다. MLflow 모델의 가장 큰 장점은 **스코어링 스크립트(`score.py`)나 환경 정의가 필요 없다는 점**입니다. Azure ML이 `MLmodel` 파일의 메타데이터를 사용하여 필요한 환경과 스코어링 로직을 자동으로 생성합니다.
    *   `ml_client.models.create_or_update()`: 모델을 Azure ML 작업 영역에 등록하거나, 이미 같은 이름의 모델이 있다면 업데이트합니다.

#### **4. 배치 엔드포인트 생성 (Create a batch endpoint)**

```python
import datetime
from azure.ai.ml.entities import BatchEndpoint

endpoint_name = "batch-" + datetime.datetime.now().strftime("%m%d%H%M%f")

endpoint = BatchEndpoint(
    name=endpoint_name,
    description="A batch endpoint for classifying diabetes in patients",
)

ml_client.batch_endpoints.begin_create_or_update(endpoint)
```

*   **목적:** 대규모 데이터에 대한 추론을 시작할 수 있는 HTTPS 배치 엔드포인트를 정의하고 생성합니다.
*   **설명:**
    *   `BatchEndpoint`: 배치 엔드포인트를 나타내는 클래스입니다.
    *   `name`: 엔드포인트의 고유한 이름을 생성합니다. (Azure 지역 내에서 고유해야 합니다.)
    *   `ml_client.batch_endpoints.begin_create_or_update()`: 배치 엔드포인트를 생성하는 비동기 작업입니다. 온라인 엔드포인트와 유사하게, 실제 모델이 배포되기 전의 상위 추상화 계층입니다.

#### **5. 배포 생성 (Create the deployment)**

```python
from azure.ai.ml.entities import BatchDeployment, BatchRetrySettings
from azure.ai.ml.constants import BatchDeploymentOutputAction

deployment = BatchDeployment(
    name="classifier-diabetes-mlflow",
    description="A diabetes classifier",
    endpoint_name=endpoint.name,
    model=model, # 위에서 등록한 모델 객체 참조
    compute="aml-cluster", # 예측에 사용될 컴퓨팅 클러스터 지정
    instance_count=2, # 컴퓨팅 노드 수
    max_concurrency_per_instance=2, # 각 노드에서 동시에 실행될 스코어링 스크립트 수
    mini_batch_size=2, # 각 스코어링 스크립트 실행에 전달될 파일 수
    output_action=BatchDeploymentOutputAction.APPEND_ROW, # 예측 결과를 출력 파일에 행으로 추가
    output_file_name="predictions.csv", # 예측 결과가 저장될 파일 이름
    retry_settings=BatchRetrySettings(max_retries=3, timeout=300), # 미니배치 실패 시 재시도 설정
    logging_level="info",
)
ml_client.batch_deployments.begin_create_or_update(deployment)
```

*   **목적:** 특정 배치 엔드포인트에 모델을 배포하기 위한 구성을 정의하고 실제로 배포합니다.
*   **설명:**
    *   `BatchDeployment`: 배치 배포를 나타내는 클래스입니다.
    *   `model`: 위에서 등록한 모델 객체를 참조합니다.
    *   `compute`: 배치 추론을 실행할 컴퓨팅 클러스터(`aml-cluster`와 같은 이름)를 지정합니다. 온라인 엔드포인트의 `instance_type`과 달리, 배치 엔드포인트는 분산 처리 및 확장성을 위해 컴퓨팅 클러스터를 사용합니다.
    *   `instance_count`: 배치 작업을 실행할 컴퓨팅 클러스터의 노드 수입니다.
    *   `max_concurrency_per_instance`: 각 컴퓨팅 노드에서 동시에 실행할 수 있는 스코어링 프로세스(또는 스레드)의 최대 수입니다. 병렬 처리를 제어하여 효율성을 높입니다.
    *   `mini_batch_size`: 한 번의 스코어링 스크립트 실행에 전달되는 입력 파일(또는 데이터 레코드)의 수입니다. 이 크기에 따라 데이터가 분할되어 병렬 처리됩니다.
    *   `output_action`: 예측 결과를 처리하는 방법을 지정합니다. `APPEND_ROW`는 각 예측을 출력 파일에 새 행으로 추가합니다.
    *   `output_file_name`: 최종 예측 결과가 저장될 파일의 이름입니다.
    *   `retry_settings`: 임시 오류 발생 시 재시도 로직을 정의하여 배치 작업의 견고성을 높입니다.
    *   온라인 엔드포인트와 마찬가지로, MLflow 모델을 사용하므로 **환경 및 스코어링 스크립트를 별도로 정의할 필요가 없습니다.**

#### **6. 기본 배포 설정 (Set default deployment)**

```python
endpoint.defaults = {}
endpoint.defaults["deployment_name"] = deployment.name
ml_client.batch_endpoints.begin_create_or_update(endpoint)
```

*   **목적:** 여러 배포가 있는 경우, 엔드포인트 호출 시 기본적으로 사용될 배포를 지정합니다.
*   **설명:** 배치 엔드포인트도 여러 배포를 가질 수 있습니다. `defaults["deployment_name"]`을 설정하면, 엔드포인트 호출 시 특정 배포를 명시하지 않아도 이 기본 배포가 사용됩니다.

#### **7. 배치 예측을 위한 데이터 준비 (Prepare the data for batch predictions)**

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

data_path = "./data"
dataset_name = "patient-data-unlabeled"

patient_dataset_unlabeled = Data(
    path=data_path,
    type=AssetTypes.URI_FOLDER,
    description="An unlabeled dataset for diabetes classification",
    name=dataset_name,
)
ml_client.data.create_or_update(patient_dataset_unlabeled)

patient_dataset_unlabeled = ml_client.data.get(
    name="patient-data-unlabeled", label="latest"
)
```

*   **목적:** 배치 추론에 사용할 입력 데이터를 Azure ML 작업 영역에 데이터 자산(Data Asset)으로 등록합니다.
*   **설명:**
    *   `Data`: Azure ML에서 데이터 자산을 나타내는 클래스입니다.
    *   `path="./data"`: 입력 데이터 파일이 저장된 로컬 폴더를 지정합니다. 배치 엔드포인트는 일반적으로 여러 개의 입력 파일(예: 여러 환자의 데이터가 담긴 CSV 파일)을 처리합니다.
    *   `type=AssetTypes.URI_FOLDER`: 데이터가 폴더 형태임을 나타냅니다.
    *   `ml_client.data.create_or_update()`: 로컬 데이터를 Azure ML의 기본 데이터스토어에 업로드하고 데이터 자산으로 등록합니다.
    *   `ml_client.data.get()`: 등록된 데이터 자산의 정보를 가져옵니다.

#### **8. 잡(Job) 제출 (Submit the job)**

```python
from azure.ai.ml import Input
from azure.ai.ml.constants import AssetTypes

input = Input(type=AssetTypes.URI_FOLDER, path=patient_dataset_unlabeled.id)

job = ml_client.batch_endpoints.invoke(
    endpoint_name=endpoint.name, 
    input=input
)

ml_client.jobs.get(job.name)
```

*   **목적:** 등록된 입력 데이터를 사용하여 배치 엔드포인트를 호출하고, 이에 따라 배치 추론 잡(Job)을 제출합니다.
*   **설명:**
    *   `Input`: 배치 잡의 입력을 정의합니다. 위에서 등록한 `patient_dataset_unlabeled` 데이터 자산을 참조합니다.
    *   `ml_client.batch_endpoints.invoke()`: 배치 엔드포인트를 호출하여 실제 추론 작업을 시작합니다. 이 호출은 `PipelineJob`을 생성하고 실행합니다.
    *   `ml_client.jobs.get(job.name)`: 제출된 잡의 상태 및 정보를 모니터링합니다. 이 잡은 모델의 스코어링 스크립트 실행을 나타내는 하위 잡을 포함합니다.

#### **9. 결과 확인 (Get the results)**

```python
ml_client.jobs.download(name=job.name, download_path=".", output_name="score")

import pandas as pd
score = pd.read_csv("predictions.csv", index_col=0, names=["patient", "prediction", "file"])
score
```

*   **목적:** 배치 잡이 완료된 후, 생성된 예측 결과를 다운로드하고 확인합니다.
*   **설명:**
    *   `ml_client.jobs.download()`: 완료된 잡의 출력 데이터를 다운로드합니다. `output_name="score"`는 배치 배포 시 설정된 `output_file_name`과 관련된 기본 출력 폴더를 다운로드합니다.
    *   다운로드된 `predictions.csv` 파일을 Pandas DataFrame으로 읽어와 예측 결과를 시각화합니다. 이 파일에는 각 환자의 예측 결과가 포함됩니다.

---

**결론적으로,** 이 노트북은 Azure Machine Learning에서 대규모 데이터셋에 대한 오프라인 추론을 자동화하는 데 사용되는 **배치 엔드포인트**의 전체 수명 주기를 보여줍니다. MLflow 모델의 사용은 환경 및 스코어링 스크립트 정의의 복잡성을 줄여주어 배포 프로세스를 간소화합니다. 배치 엔드포인트는 ETL 파이프라인의 일부로 예측을 통합하거나, 정기적으로 대규모 데이터에 대한 통찰력을 얻어야 하는 시나리오에 매우 유용합니다.
