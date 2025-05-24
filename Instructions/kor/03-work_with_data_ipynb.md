## `Work with data.ipynb` Notebook 상세 분석

이 Notebook은 Azure Machine Learning (Azure ML) 환경에서 데이터 작업을 위한 핵심 개념인 **데이터 저장소(Datastores)** 와 **데이터 자산(Data Assets)** 을 다루는 방법을 Python SDK (v2)를 사용하여 보여줍니다.

각 셀의 주요 내용과 의미를 자세히 살펴보겠습니다.

---

### 1. 소개 및 환경 설정 (Markdown & Code)

```markdown
# Work with Data
...
## Before you start
...
> **Note**:
> If the **azure-ai-ml** package is not installed, run `pip install azure-ai-ml` to install it.
```

```python
pip show azure-ai-ml
```

*   **목적**: Notebook의 주제를 소개하고, 필요한 Python SDK 패키지(`azure-ai-ml`)가 설치되어 있는지 확인합니다.
*   **의미**: Azure ML v2 SDK는 Azure ML 리소스와 상호 작용하기 위한 최신 라이브러리입니다. 이 셀은 사용자가 올바른 환경에서 Notebook을 실행하고 있는지 확인하는 초기 단계입니다.

---

### 2. 작업 영역(Workspace)에 연결 (Markdown & Code)

```markdown
## Connect to your workspace
...
```

```python
from azure.identity import DefaultAzureCredential, InteractiveBrowserCredential
from azure.ai.ml import MLClient

try:
    credential = DefaultAzureCredential()
    # Check if given credential can get token successfully.
    credential.get_token("https://management.azure.com/.default")
except Exception as ex:
    # Fall back to InteractiveBrowserCredential in case DefaultAzureCredential not work
    credential = InteractiveBrowserCredential()
```

```python
# Get a handle to workspace
ml_client = MLClient.from_config(credential=credential)
```

*   **목적**: Azure ML 작업 영역에 프로그래밍 방식으로 연결합니다.
*   **`DefaultAzureCredential` vs `InteractiveBrowserCredential`**:
    *   `DefaultAzureCredential`: 다양한 인증 메커니즘(환경 변수, 관리 ID, Azure CLI 로그인 등)을 순서대로 시도하여 자동으로 인증을 처리합니다. 컴퓨팅 인스턴스와 같이 관리 ID가 구성된 환경에서 편리합니다.
    *   `InteractiveBrowserCredential`: `DefaultAzureCredential`이 실패할 경우, 웹 브라우저를 통해 사용자가 직접 로그인하여 인증하는 방식입니다. 로컬 환경에서 주로 사용됩니다.
*   **`MLClient.from_config(credential=credential)`**:
    *   `MLClient`: Azure ML 서비스와 상호 작용하기 위한 기본 클라이언트 객체입니다.
    *   `.from_config()`: 현재 환경(예: 컴퓨팅 인스턴스)의 구성 정보(일반적으로 작업 영역 루트에 있는 `config.json` 파일 또는 환경 변수)를 읽어와서 특정 작업 영역에 대한 핸들(연결 객체)을 가져옵니다. 이 핸들을 통해 작업 영역 내의 다양한 리소스(데이터 저장소, 데이터 자산, 작업, 모델 등)를 관리할 수 있습니다.

---

### 3. 데이터 저장소(Datastores) 나열 (Markdown & Code)

```markdown
## List the datastores
...
```

```python
stores = ml_client.datastores.list()
for ds_name in stores:
    print(ds_name.name)
```

```markdown
Note the `workspaceblobstore` which connects to the **azureml-blobstore-...** container you explored earlier. The `workspacefilestore` connects to the **code-...** file share.
```

*   **목적**: 현재 Azure ML 작업 영역에 등록된 모든 데이터 저장소를 나열합니다.
*   **데이터 저장소(Datastore)란?**: Azure 저장소 서비스(예: Azure Blob Storage, Azure Files, Azure Data Lake Storage)에 대한 **연결 정보**를 Azure ML 작업 영역 내에 추상화하여 저장하는 객체입니다. 실제 데이터는 원래 저장소 서비스에 그대로 있고, 데이터 저장소는 해당 데이터에 접근하는 방법을 정의합니다.
*   **`workspaceblobstore`**: 작업 영역 생성 시 자동으로 만들어지는 기본 데이터 저장소입니다. Azure Blob Storage 컨테이너(`azureml-blobstore-...`)에 연결됩니다. 주로 학습 데이터, 모델 아티팩트, 로그 등을 저장하는 데 사용됩니다.
*   **`workspacefilestore`**: 작업 영역 생성 시 자동으로 만들어지는 기본 데이터 저장소입니다. Azure Files 공유(`code-...`)에 연결됩니다. 주로 Notebook 파일, 스크립트, 작업 환경 구성 파일 등을 저장하며, 컴퓨팅 인스턴스에 마운트되어 쉽게 접근할 수 있습니다.

---

### 4. 데이터 저장소(Datastore) 생성 (Markdown & Code)

```markdown
## Create a datastore
...
**Important**:
- Replace the **YOUR-STORAGE-ACCOUNT-NAME** with the name of the Storage Account that was automatically created for you.
- Replace the **XXXX-XXXX** for `account_key` with the account key of your Azure Storage Account.
...
```

```python
from azure.ai.ml.entities import AzureBlobDatastore
from azure.ai.ml.entities import AccountKeyConfiguration

store = AzureBlobDatastore(
    name="blob_training_data",
    description="Blob Storage for training data",
    account_name="YOUR-STORAGE-ACCOUNT-NAME", # 사용자가 실제 스토리지 계정 이름으로 변경해야 함
    container_name="training-data",          # 실습에서 미리 생성한 컨테이너 이름
    credentials=AccountKeyConfiguration(
        account_key="XXXX-XXXX"               # 사용자가 실제 스토리지 계정 키로 변경해야 함
    ),
)

ml_client.create_or_update(store)
```

*   **목적**: 기존 Azure Storage Account의 특정 컨테이너에 연결되는 새로운 데이터 저장소를 Azure ML 작업 영역에 등록합니다.
*   **주요 매개변수**:
    *   `name`: 작업 영역 내에서 이 데이터 저장소를 식별할 이름 (예: `blob_training_data`).
    *   `account_name`: 연결할 Azure Storage Account의 이름.
    *   `container_name`: 해당 Storage Account 내에서 연결할 Blob 컨테이너의 이름 (실습에서는 `training-data`라는 이름으로 미리 생성했음).
    *   `credentials`: Storage Account에 접근하기 위한 인증 정보. 여기서는 `AccountKeyConfiguration`을 사용하여 Storage Account의 액세스 키를 직접 제공합니다. (실무에서는 서비스 주체나 관리 ID를 사용하는 것이 더 안전하고 권장됩니다.)
*   **`ml_client.create_or_update(store)`**: 정의된 데이터 저장소 객체를 작업 영역에 생성하거나, 이미 같은 이름의 데이터 저장소가 있다면 업데이트합니다.

---

### 5. 생성된 데이터 저장소(Datastore) 확인 (Markdown & Code)

```python
stores = ml_client.datastores.list()
for ds_name in stores:
    print(ds_name.name)
```

*   **목적**: 이전 단계에서 생성한 `blob_training_data` 데이터 저장소가 정상적으로 등록되었는지 확인합니다.

---

### 6. 데이터 자산(Data Assets) 생성 (Markdown & Code)

```markdown
## Create data assets
...
- `URI_FILE` points to a specific file.
- `URI_FOLDER` points to a specific folder.
- `MLTABLE` points to a MLTable file which specifies how to read one or more files within a folder.
...
```

*   **데이터 자산(Data Asset)이란?**: 데이터 저장소 내의 특정 데이터(파일, 폴더, 또는 테이블 형식 데이터 정의)를 가리키는 **포인터 또는 참조**입니다. 데이터 자산을 사용하면 데이터의 버전 관리, 추적, 공유가 용이해집니다.

#### 6.1 `URI_FILE` 데이터 자산 생성 (로컬 경로에서)

```markdown
To create a `URI_FILE` data asset, you have to specify a path that points to a specific file. The path can be a local path or cloud path.
...
In this case, the `diabetes.csv` file will be uploaded to **LocalUpload** folder in the **workspaceblobstore** datastore.
...
```

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

my_path = './data/diabetes.csv' # 컴퓨팅 인스턴스 내 로컬 경로

my_data = Data(
    path=my_path,
    type=AssetTypes.URI_FILE,
    description="Data asset pointing to a local file, automatically uploaded to the default datastore",
    name="diabetes-local"
)

ml_client.data.create_or_update(my_data)
```

*   **목적**: 컴퓨팅 인스턴스의 로컬 파일 시스템에 있는 `diabetes.csv` 파일을 참조하는 `URI_FILE` 타입의 데이터 자산을 생성합니다.
*   **중요 동작**: `path`가 로컬 경로로 지정되면, `ml_client.data.create_or_update()` 실행 시 SDK는 해당 로컬 파일을 작업 영역의 **기본 데이터 저장소(`workspaceblobstore`)** 로 **자동으로 업로드(복제)** 합니다. 그리고 업로드된 Blob Storage 상의 URI를 가리키는 데이터 자산(`diabetes-local`)을 등록합니다.
*   **`AssetTypes.URI_FILE`**: 데이터 자산이 단일 파일을 가리킴을 나타냅니다.

#### 6.2 `URI_FOLDER` 데이터 자산 생성 (클라우드 경로에서)

```markdown
To create a `URI_FOLDER` data asset, you have to specify a path that points to a specific folder. The path can be a local path or cloud path.
...
The path doesn't have to exist yet. The folder will be created when data is uploaded to the path.
```

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

datastore_path = 'azureml://datastores/blob_training_data/paths/data-asset-path/' # 데이터 저장소 내 클라우드 경로

my_data = Data(
    path=datastore_path,
    type=AssetTypes.URI_FOLDER,
    description="Data asset pointing to data-asset-path folder in datastore",
    name="diabetes-datastore-path"
)

ml_client.data.create_or_update(my_data)
```

*   **목적**: 이전에 생성한 `blob_training_data` 데이터 저장소 내의 특정 폴더 경로(`data-asset-path/`)를 참조하는 `URI_FOLDER` 타입의 데이터 자산을 생성합니다.
*   **`azureml://` 스키마**: 데이터 저장소 내의 경로를 지정하는 Azure ML 고유의 URI 스키마입니다.
    *   `azureml://datastores/<datastore_name>/paths/<path_in_datastore>` 형식입니다.
*   **`AssetTypes.URI_FOLDER`**: 데이터 자산이 폴더를 가리킴을 나타냅니다.
*   **경로 존재 여부**: 이 시점에서는 `data-asset-path/` 폴더가 실제로 `blob_training_data` 데이터 저장소에 존재하지 않아도 됩니다. 데이터 자산은 단순히 해당 위치를 가리키는 참조를 만드는 것입니다. 나중에 작업(job)의 출력 등으로 해당 경로에 데이터가 저장될 수 있습니다.

#### 6.3 `MLTABLE` 데이터 자산 생성 (로컬 경로에서)

```markdown
To create a `MLTable` data asset, you have to specify a path that points to a folder which contains a MLTable file. The path can be a local path or cloud path. 
...
> **Note**:
> Do **not** rename the `MLTable` file to `MLTable.yaml` or `MLTable.yml`. Azure machine learning expects an `MLTable` file.
...
```

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

local_path = 'data/' # MLTable 파일과 CSV 파일이 포함된 로컬 폴더

my_data = Data(
    path=local_path,
    type=AssetTypes.MLTABLE,
    description="MLTable pointing to diabetes.csv in data folder",
    name="diabetes-table"
)

ml_client.data.create_or_update(my_data)
```

*   **목적**: 컴퓨팅 인스턴스의 로컬 `data/` 폴더를 참조하는 `MLTABLE` 타입의 데이터 자산을 생성합니다. 이 폴더에는 `MLTable` 파일과 해당 파일에서 참조하는 데이터 파일(예: `diabetes.csv`)이 함께 있어야 합니다.
*   **`MLTable`이란?**: 하나 이상의 파일에서 테이블 형식 데이터를 읽고 구문 분석하는 방법을 정의하는 YAML 파일입니다. 데이터의 스키마, 파일 경로 패턴, 구분자 등을 지정할 수 있습니다. 복잡한 데이터 구조나 여러 파일에 분산된 데이터를 단일 테이블로 표현하는 데 유용합니다.
*   **`AssetTypes.MLTABLE`**: 데이터 자산이 `MLTable` 정의를 사용함을 나타냅니다.
*   **동작**: `URI_FILE`과 유사하게, `path`가 로컬 경로로 지정되면, `MLTable` 파일과 그 파일이 참조하는 데이터 파일들이 기본 데이터 저장소로 업로드됩니다.

---

### 7. 생성된 데이터 자산(Data Asset) 확인 (Markdown & Code)

```python
datasets = ml_client.data.list()
for ds_name in datasets:
    print(ds_name.name)
```

*   **목적**: 이전 단계들에서 생성한 데이터 자산(`diabetes-local`, `diabetes-datastore-path`, `diabetes-table`)이 작업 영역에 정상적으로 등록되었는지 확인합니다.

---

### 8. Notebook에서 데이터 읽기 (Markdown & Code)

```markdown
## Read data in notebook
...
A `MLTable` type data asset is already *read* by the **MLTable** file, which specifies the schema and how to interpret the data. Since the data is already *read*, you can easily convert a MLTable data asset to a pandas dataframe.
...
```

```python
import mltable

registered_data_asset = ml_client.data.get(name='diabetes-table', version=1) # 등록된 MLTable 데이터 자산 가져오기
tbl = mltable.load(f"azureml:/{registered_data_asset.id}") # MLTable 로드
df = tbl.to_pandas_dataframe() # Pandas DataFrame으로 변환
df.head(5)
```

*   **목적**: 등록된 `MLTable` 타입의 데이터 자산을 Notebook에서 직접 읽어 Pandas DataFrame으로 변환하고 내용을 확인합니다.
*   **`mltable.load(f"azureml:/{registered_data_asset.id}")`**:
    *   `ml_client.data.get()`: 이름과 버전으로 등록된 데이터 자산 객체를 가져옵니다.
    *   `registered_data_asset.id`: 데이터 자산의 고유 ID (예: `azureml:diabetes-table:1`).
    *   `mltable.load()`: `azureml:/` 스키마를 사용하여 Azure ML 작업 영역에 등록된 `MLTable` 데이터 자산을 로드합니다. 이 과정에서 `MLTable` 파일의 정의에 따라 데이터가 읽히고 처리됩니다.
*   **`tbl.to_pandas_dataframe()`**: 로드된 `MLTable` 객체를 Pandas DataFrame으로 쉽게 변환할 수 있게 해줍니다.
*   **`URI_FILE` / `URI_FOLDER` 읽기**: Markdown 설명에서 언급된 것처럼, `URI_FILE`이나 `URI_FOLDER` 타입의 데이터 자산이 CSV 파일을 가리킨다면, 해당 파일의 URI를 가져와 `pd.read_csv()`와 같은 일반적인 Pandas 함수로 직접 읽을 수 있습니다. (이 Notebook에서는 `MLTable` 예제만 코드로 보여줍니다.)

---

### 9. 작업(Job)에서 데이터 사용 (Markdown & Code)

```markdown
## Use data in a job
...
You can use either **data assets** or **datastore paths** as inputs or outputs of a job.
...
```

#### 9.1 스크립트 파일 생성 (`move-data.py`)

```python
import os

# create a folder for the script files
script_folder = 'src'
os.makedirs(script_folder, exist_ok=True)
print(script_folder, 'folder created')
```

```python
%%writefile $script_folder/move-data.py
# import libraries
import argparse
import pandas as pd
import numpy as np
from pathlib import Path

def main(args):
    # read data
    df = get_data(args.input_data)

    output_df = df.to_csv((Path(args.output_datastore) / "diabetes.csv"), index = False)

# function that reads the data
def get_data(path):
    df = pd.read_csv(path)

    # Count the rows and print the result
    row_count = (len(df))
    print('Analyzing {} rows of data'.format(row_count))

    return df

def parse_args():
    # setup arg parser
    parser = argparse.ArgumentParser()

    # add arguments
    parser.add_argument("--input_data", dest='input_data',
                        type=str)
    parser.add_argument("--output_datastore", dest='output_datastore',
                        type=str)

    # parse args
    args = parser.parse_args()

    # return args
    return args

# run script
if __name__ == "__main__":
    # add space in logs
    print("\n\n")
    print("*" * 60)

    # parse args
    args = parse_args()

    # run main function
    main(args)

    # add space in logs
    print("*" * 60)
    print("\n\n")
```

*   **목적**: Azure ML 작업(Job)으로 실행될 Python 스크립트(`move-data.py`)를 생성합니다.
*   **`move-data.py` 스크립트의 기능**:
    *   `argparse`를 사용하여 명령줄 인자(`--input_data`, `--output_datastore`)를 받습니다.
    *   입력 데이터 경로(`args.input_data`)에서 CSV 파일을 읽어 Pandas DataFrame으로 로드합니다.
    *   로드된 DataFrame을 출력 데이터 저장소 경로(`args.output_datastore`)에 `diabetes.csv`라는 이름으로 다시 저장합니다.
*   **`%%writefile`**: Jupyter 매직 명령어. 셀의 내용을 지정된 파일로 저장합니다. 여기서는 `src/move-data.py` 파일을 생성합니다.

#### 9.2 작업(Job) 구성 및 제출

```markdown
To submit a job that runs the **move-data.py** script, run the cell below.

The job is configured to use the data asset `diabetes-local`, pointing to the local **diabetes.csv** file as input. The output is a path pointing to a folder in the new datastore `blob_training_data`.
```

```python
from azure.ai.ml import Input, Output
from azure.ai.ml.constants import AssetTypes
from azure.ai.ml import command

# configure input and output
my_job_inputs = {
    "local_data": Input(type=AssetTypes.URI_FILE, path="azureml:diabetes-local:1") # 입력: diabetes-local 데이터 자산
}

my_job_outputs = {
    "datastore_data": Output(type=AssetTypes.URI_FOLDER, path="azureml://datastores/blob_training_data/paths/datastore-path") # 출력: 데이터 저장소 내 폴더 경로
}

# configure job
job = command(
    code="./src",  # 스크립트가 있는 폴더
    command="python move-data.py --input_data ${{inputs.local_data}} --output_datastore ${{outputs.datastore_data}}", # 실행할 명령
    inputs=my_job_inputs,
    outputs=my_job_outputs,
    environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest", # 실행 환경 (큐레이트된 환경)
    compute="aml-cluster", # 실행할 컴퓨팅 타겟 (setup.sh에서 생성된 클러스터)
    display_name="move-diabetes-data",
    experiment_name="move-diabetes-data"
)

# submit job
returned_job = ml_client.create_or_update(job)
aml_url = returned_job.studio_url
print("Monitor your job at", aml_url)
```

*   **목적**: `move-data.py` 스크립트를 실행하는 Azure ML 명령 작업(Command Job)을 구성하고 제출합니다.
*   **`Input` / `Output`**:
    *   `Input(type=AssetTypes.URI_FILE, path="azureml:diabetes-local:1")`: 작업의 입력으로 `diabetes-local` 데이터 자산 (버전 1)을 사용하도록 지정합니다. Azure ML은 작업 실행 시 이 데이터 자산이 가리키는 실제 데이터 파일(Blob Storage에 복제된 `diabetes.csv`)을 컴퓨팅 환경으로 다운로드하여 스크립트가 접근할 수 있도록 합니다.
    *   `Output(type=AssetTypes.URI_FOLDER, path="azureml://datastores/blob_training_data/paths/datastore-path")`: 작업의 출력이 저장될 위치를 지정합니다. 여기서는 `blob_training_data` 데이터 저장소 내의 `datastore-path/` 폴더입니다. 스크립트가 이 경로에 파일을 쓰면, 작업 완료 후 해당 파일들이 이 데이터 저장소 위치에 업로드(저장)됩니다.
*   **`command(...)`**: 명령 작업을 정의합니다.
    *   `code="./src"`: 스크립트(`move-data.py`)가 포함된 로컬 폴더를 지정합니다. 이 폴더의 내용이 작업 실행을 위해 컴퓨팅 타겟으로 업로드됩니다.
    *   `command="python move-data.py --input_data ${{inputs.local_data}} --output_datastore ${{outputs.datastore_data}}"`: 컴퓨팅 타겟에서 실행될 실제 명령입니다.
        *   `${{inputs.local_data}}`: 작업 실행 시 `my_job_inputs`에 정의된 `local_data` 입력(즉, `diabetes-local` 데이터 자산이 가리키는 파일의 경로)으로 대체됩니다.
        *   `${{outputs.datastore_data}}`: 작업 실행 시 `my_job_outputs`에 정의된 `datastore_data` 출력(즉, 출력이 저장될 폴더 경로)으로 대체됩니다.
    *   `environment`: 작업 실행에 필요한 Python 환경, 라이브러리 등을 정의합니다. 여기서는 Azure ML에서 제공하는 큐레이트된 환경 중 하나를 사용합니다.
    *   `compute`: 작업을 실행할 컴퓨팅 리소스를 지정합니다. `setup.sh` 스크립트에서 `aml-cluster`라는 이름의 컴퓨팅 클러스터를 생성했을 것이므로, 해당 클러스터를 사용합니다.
*   **`ml_client.create_or_update(job)`**: 정의된 작업을 Azure ML 서비스에 제출하여 실행합니다.
*   **`returned_job.studio_url`**: 제출된 작업의 진행 상황을 Azure Machine Learning 스튜디오에서 모니터링할 수 있는 URL을 제공합니다.

---

이 Notebook은 Azure ML에서 데이터를 체계적으로 관리하고, 다양한 컴퓨팅 환경에서 데이터에 접근하며, 자동화된 작업 파이프라인에서 데이터를 입력 및 출력으로 사용하는 기본적인 방법을 포괄적으로 보여주는 훌륭한 예제입니다. 로컬 데이터의 자동 업로드, 데이터 저장소 및 데이터 자산의 개념, 그리고 이를 작업과 연동하는 방법을 이해하는 데 큰 도움이 됩니다.
