주어진 노트북 내용은 Azure Machine Learning에서 **MLflow 모델을 관리형 온라인 엔드포인트(Managed Online Endpoint)에 배포**하고 실시간으로 예측을 수행하는 과정을 보여줍니다. MLflow 모델은 환경 및 스코어링 스크립트를 별도로 정의할 필요가 없어 배포가 간편하다는 장점이 있습니다.

각 셀의 목적과 주요 개념을 설명해 드리겠습니다.

---

### **전반적인 목표:**

*   애플리케이션에서 모델을 사용하여 실시간 예측을 얻기 위해 MLflow 모델을 관리형 온라인 엔드포인트에 배포합니다.
*   배포된 엔드포인트를 샘플 데이터로 테스트합니다.

---

### **단계별 설명:**

#### **1. 필수 패키지 확인 및 설치 (`pip show azure-ai-ml`)**

```python
pip show azure-ai-ml
```

*   **목적:** Azure Machine Learning SDK v2인 `azure-ai-ml` 패키지가 현재 환경에 설치되어 있는지 확인합니다. 이 SDK는 Azure ML 리소스와 상호작용하는 데 사용됩니다.
*   **설명:** 만약 설치되어 있지 않다면 `pip install azure-ai-ml` 명령을 통해 설치해야 합니다.

#### **2. 작업 영역(Workspace) 연결**

```python
from azure.identity import DefaultAzureCredential, InteractiveBrowserCredential
from azure.ai.ml import MLClient

try:
    credential = DefaultAzureCredential()
    credential.get_token("https://management.azure.com/.default")
except Exception as ex:
    credential = InteractiveBrowserCredential()

ml_client = MLClient.from_config(credential=credential)
```

*   **목적:** Python SDK를 사용하여 Azure Machine Learning 작업 영역에 연결합니다. 작업 영역은 모든 Azure ML 리소스의 최상위 단위입니다.
*   **설명:**
    *   `DefaultAzureCredential`: 다양한 인증 메커니즘(환경 변수, 관리 ID, Azure CLI, VS Code 등)을 자동으로 시도하여 인증을 시도합니다.
    *   `InteractiveBrowserCredential`: `DefaultAzureCredential`이 실패할 경우, 브라우저 팝업을 통해 사용자에게 대화형으로 로그인하도록 요청합니다.
    *   `MLClient.from_config()`: Azure ML 작업 영역의 구성 정보(구독 ID, 리소스 그룹, 작업 영역 이름)를 읽어와 `MLClient` 객체를 초기화합니다. 이 객체를 통해 작업 영역 내의 다양한 리소스(엔드포인트, 모델, 컴퓨팅 등)를 관리할 수 있습니다.

#### **3. 엔드포인트 정의 및 생성**

```python
from azure.ai.ml.entities import ManagedOnlineEndpoint
import datetime

online_endpoint_name = "endpoint-" + datetime.datetime.now().strftime("%m%d%H%M%f")

endpoint = ManagedOnlineEndpoint(
    name=online_endpoint_name,
    description="Online endpoint for MLflow diabetes model",
    auth_mode="key",
)

ml_client.begin_create_or_update(endpoint).result()
```

*   **목적:** 모델을 호스팅할 관리형 온라인 엔드포인트를 정의하고 Azure에 생성합니다.
*   **설명:**
    *   **관리형 온라인 엔드포인트(Managed Online Endpoint):** Azure Machine Learning에서 제공하는 완전 관리형 서비스로, 사용자가 컴퓨팅 인프라를 직접 관리할 필요 없이 모델을 배포하고 실시간으로 추론할 수 있는 HTTPS 엔드포인트입니다. 확장성, 고가용성, 보안 기능을 내장하고 있습니다.
    *   `online_endpoint_name`: 엔드포인트의 고유한 이름을 생성합니다. `datetime`을 사용하여 실행할 때마다 고유한 이름이 되도록 합니다.
    *   `description`: 엔드포인트에 대한 설명을 추가합니다.
    *   `auth_mode="key"`: 엔드포인트 인증 방식을 지정합니다. 여기서는 API 키를 사용합니다.
    *   `ml_client.begin_create_or_update(endpoint).result()`: 엔드포인트를 생성하는 비동기 작업입니다. `.result()`는 작업이 완료될 때까지 기다리도록 합니다. 이 과정은 몇 분이 소요될 수 있습니다.

#### **4. 배포(Deployment) 구성**

```python
from azure.ai.ml.entities import Model, ManagedOnlineDeployment
from azure.ai.ml.constants import AssetTypes

model = Model(
    path="./model",
    type=AssetTypes.MLFLOW_MODEL,
    description="my sample mlflow model",
)

blue_deployment = ManagedOnlineDeployment(
    name="blue",
    endpoint_name=online_endpoint_name,
    model=model,
    instance_type="Standard_D2as_v4",
    instance_count=1,
)
```

*   **목적:** 특정 엔드포인트에 배포할 모델과 해당 배포의 인프라 설정을 정의합니다. 하나의 엔드포인트는 여러 배포(예: "blue"와 "green")를 가질 수 있어 A/B 테스트나 점진적 롤아웃이 가능합니다.
*   **설명:**
    *   `Model`: 배포할 모델을 정의합니다.
        *   `path="./model"`: 모델 파일이 있는 로컬 경로를 지정합니다. 이 노트북이 실행되는 환경에 `model`이라는 폴더가 존재해야 합니다.
        *   `type=AssetTypes.MLFLOW_MODEL`: 모델 유형을 MLflow 모델로 지정합니다. 이 부분이 MLflow 모델 배포의 핵심적인 장점입니다. MLflow 모델은 환경 종속성 및 스코어링 로직이 모델 자체에 패키징되어 있어, 별도의 환경 정의나 스코어링 스크립트(`score.py`)가 필요 없습니다.
    *   `ManagedOnlineDeployment`: 특정 엔드포인트에 모델을 배포하기 위한 구성을 정의합니다.
        *   `name="blue"`: 배포의 이름을 "blue"로 지정합니다. 이는 블루/그린 배포 전략에서 유용합니다.
        *   `endpoint_name`: 이 배포가 연결될 엔드포인트의 이름을 지정합니다.
        *   `model`: 위에서 정의한 `Model` 객체를 참조하여 어떤 모델을 배포할지 지정합니다.
        *   `instance_type="Standard_D2as_v4"`: 모델이 실행될 가상 머신의 크기(컴퓨팅 사양)를 지정합니다.
        *   `instance_count=1`: 모델을 호스팅할 인스턴스(가상 머신)의 개수를 지정합니다.

#### **5. 배포 생성**

```python
ml_client.online_deployments.begin_create_or_update(blue_deployment).result()
```

*   **목적:** 정의된 배포를 실제로 엔드포인트에 생성하고 모델을 배포합니다.
*   **설명:** 이 과정은 모델을 지정된 인스턴스에 로드하고 예측을 수행할 준비를 하는 과정이므로 10~15분 정도 소요될 수 있습니다.

#### **6. 트래픽 설정**

```python
endpoint.traffic = {"blue": 100}
ml_client.begin_create_or_update(endpoint).result()
```

*   **목적:** 엔드포인트로 들어오는 모든 트래픽(요청)을 "blue" 배포로 라우팅하도록 설정합니다.
*   **설명:** 엔드포인트에 여러 배포가 있을 경우(`blue`, `green` 등), 각 배포에 할당할 트래픽 비율을 여기서 설정할 수 있습니다 (예: `{"blue": 80, "green": 20}`). 이를 통해 점진적 롤아웃이나 A/B 테스트를 구현할 수 있습니다. 여기서는 단일 배포이므로 100%를 할당합니다.

#### **7. 배포 테스트**

```python
response = ml_client.online_endpoints.invoke(
    endpoint_name=online_endpoint_name,
    deployment_name="blue",
    request_file="sample-data.json",
)

if response[1]=='1':
    print("Diabetic")
else:
    print ("Not diabetic")
```

*   **목적:** 배포된 모델이 올바르게 작동하는지 확인하기 위해 엔드포인트를 호출(invoke)하여 예측을 수행합니다.
*   **설명:**
    *   `ml_client.online_endpoints.invoke()`: 지정된 엔드포인트와 배포에 예측 요청을 보냅니다.
    *   `request_file="sample-data.json"`: 모델 예측에 사용될 입력 데이터를 포함하는 JSON 파일의 경로입니다. 이 파일에는 당뇨병 예측 모델이 필요로 하는 환자 데이터(나이, BMI, 임신 횟수 등)가 들어 있습니다.
    *   모델의 예측 결과에 따라 환자가 당뇨병 환자인지 아닌지를 출력합니다.

#### **8. 엔드포인트 목록 조회 및 상세 정보 확인**

```python
endpoints = ml_client.online_endpoints.list()
for endp in endpoints:
    print(endp.name)

endpoint = ml_client.online_endpoints.get(name=online_endpoint_name)
print(endpoint.traffic)
print(endpoint.scoring_uri)
```

*   **목적:** Azure ML 작업 영역에 배포된 모든 온라인 엔드포인트의 목록을 확인하고, 특정 엔드포인트의 상세 정보(트래픽 설정, 스코어링 URI)를 조회합니다.
*   **설명:**
    *   `ml_client.online_endpoints.list()`: 작업 영역의 모든 온라인 엔드포인트를 반환합니다.
    *   `ml_client.online_endpoints.get(name=...)`: 특정 이름의 엔드포인트 객체를 가져옵니다.
    *   `endpoint.traffic`: 현재 엔드포인트의 트래픽 라우팅 설정을 보여줍니다.
    *   `endpoint.scoring_uri`: 외부 애플리케이션이 모델에 예측 요청을 보낼 때 사용할 수 있는 실제 HTTPS URL입니다.

#### **9. 엔드포인트 및 배포 삭제**

```python
ml_client.online_endpoints.begin_delete(name=online_endpoint_name)
```

*   **목적:** 불필요한 비용이 발생하지 않도록 배포된 엔드포인트를 삭제합니다.
*   **설명:** 관리형 온라인 엔드포인트는 항상 가동 상태이므로, 사용하지 않을 때는 삭제하여 컴퓨팅 리소스 비용을 절약해야 합니다. `begin_delete()`는 비동기적으로 삭제를 시작합니다.

---

### **핵심 개념 요약:**

*   **관리형 온라인 엔드포인트 (Managed Online Endpoint):** Azure가 컴퓨팅 인프라 관리를 담당하는 모델 배포 환경입니다. 안정적인 REST API 엔드포인트를 제공하며, 실시간 예측에 사용됩니다.
*   **배포 (Deployment):** 엔드포인트 아래에 실제로 모델과 컴퓨팅 리소스(VM 인스턴스)가 연결된 단위입니다. 하나의 엔드포인트는 여러 배포를 가질 수 있으며, 트래픽을 분산할 수 있습니다.
*   **MLflow 모델:** MLflow 프로젝트에서 생성된 모델은 표준화된 형식으로 패키징되어 있습니다. 이 덕분에 Azure ML에서 환경 및 스코어링 스크립트를 별도로 지정할 필요 없이 쉽게 배포할 수 있습니다. MLflow는 모델 재현성 및 배포의 간소화를 돕습니다.
*   **스코어링 스크립트 (Scoring Script):** 일반적으로 모델 배포 시 모델을 로드하고 입력 데이터를 처리하여 예측을 반환하는 Python 스크립트(`score.py`)가 필요하지만, MLflow 모델은 이 스크립트가 내장되어 있어 명시적으로 작성할 필요가 없습니다.

이 노트북은 Azure Machine Learning SDK v2를 사용하여 모델 배포의 전체 워크플로우를 보여주며, 특히 MLflow 모델의 배포 편의성을 강조합니다.
