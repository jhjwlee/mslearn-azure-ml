## 스크립트를 파이프라인 작업으로 실행하기

파이프라인을 사용하면 여러 단계를 하나의 워크플로우로 그룹화할 수 있습니다. 구성 요소(Components)를 사용하여 파이프라인을 구축할 수 있습니다. 각 구성 요소는 실행할 Python 스크립트를 반영합니다. 구성 요소는 스크립트와 실행 방법을 지정하는 YAML 파일에 정의됩니다.

**실습 목표:**

*   Azure CLI를 사용하여 Azure Machine Learning 작업 영역 및 컴퓨팅 리소스 프로비저닝하기.
*   데이터 준비 및 모델 학습을 위한 두 가지 Python 스크립트 생성하기.
*   각 스크립트를 실행하기 위한 재사용 가능한 구성 요소(Component) 정의하기.
*   Python SDK를 사용하여 두 구성 요소를 연결하여 Azure ML 파이프라인 구축하기.
*   파이프라인 작업을 제출하고 Azure ML 스튜디오에서 실행 흐름 검토하기.

**준비물:**

*   관리자 수준 액세스 권한이 있는 Azure 구독.

---

### Azure Machine Learning 작업 영역 프로비저닝

(이전 실습과 동일한 초기 설정 단계입니다. 이미 작업 영역이 있다면 이 단계를 건너뛸 수 있습니다.)

1.  브라우저에서 Microsoft 계정으로 로그인하여 [https://portal.azure.com/](https://portal.azure.com/)에서 Azure Portal을 엽니다.
2.  페이지 상단의 검색창 오른쪽에 있는 **[\>\_] (Cloud Shell)** 버튼을 선택합니다.
3.  메시지가 나타나면 **Bash**를 선택합니다.
4.  올바른 구독이 지정되어 있고 **저장소 계정 필요 없음(No storage account required)** 이 선택되어 있는지 확인합니다. **적용(Apply)** 을 선택합니다.
5.  터미널에 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```bash
    rm -r azure-ml-labs -f
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

    **팁**: 복사한 코드를 Cloud Shell에 붙여넣으려면 `SHIFT + INSERT`를 사용하십시오.

6.  리포지토리가 복제된 후 다음 명령을 입력하여 이 실습용 폴더로 변경하고 포함된 `setup.sh` 스크립트를 실행합니다.

    ```bash
    cd azure-ml-labs/Labs/09 # 또는 이전 실습에서 사용한 Lab 폴더 (예: Labs/08)
    ./setup.sh
    ```

    **참고**: 확장이 설치되지 않았다는 (오류) 메시지는 무시하십시오.

7.  스크립트가 완료될 때까지 기다립니다. 일반적으로 약 5-10분 정도 소요됩니다.

---

### 실습 자료 복제하기

(이전 실습과 동일한 단계입니다.)

1.  Azure Portal에서 `mlw-dp100-…` (스크립트가 생성한 실제 이름)라는 Azure Machine Learning 작업 영역으로 이동합니다.
2.  Azure Machine Learning 작업 영역을 선택하고 **개요(Overview)** 페이지에서 **스튜디오 시작(Launch studio)** 을 선택합니다.
3.  스튜디오에 나타나는 모든 팝업을 닫습니다.
4.  Azure Machine Learning 스튜디오 내에서 **컴퓨팅(Compute)** 페이지로 이동하여 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다.
5.  **컴퓨팅 인스턴스(Compute instances)** 탭에서 컴퓨팅 인스턴스를 찾아 **터미널(Terminal)** 애플리케이션을 선택합니다.
6.  터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```bash
    pip uninstall azure-ai-ml -y
    pip install azure-ai-ml
    ```

7.  다음 명령을 실행하여 Git 리포지토리를 작업 영역에 복제합니다.

    ```bash
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

8.  명령이 완료되면 **파일(Files)** 창에서 **↻ (새로고침)** 을 클릭하여 새 `Users/your-user-name/azure-ml-labs` 폴더가 생성되었는지 확인합니다.

---

### 파이프라인 작업으로 스크립트 실행하기

이제 필요한 모든 리소스가 있으므로, Notebook을 실행하여 파이프라인 작업을 제출할 수 있습니다.

1.  **`Labs/09/Run scripts as a pipeline job.ipynb`** Notebook을 엽니다.
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> `09` 폴더로 이동하여 `Run scripts as a pipeline job.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.

3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다.

4.  **모든 셀을 실행합니다**.

---

### Notebook 내용 해설 (`Run scripts as a pipeline job.ipynb`)

이 Notebook은 Azure Machine Learning에서 **파이프라인(Pipeline)** 을 사용하여 여러 머신러닝 단계를 하나의 자동화된 워크플로우로 묶는 방법을 보여줍니다. 파이프라인은 MLops(Machine Learning Operations)의 핵심 개념 중 하나로, 모델 개발 및 배포 과정을 표준화하고 자동화하는 데 필수적입니다.

**주요 단계 및 코드 설명:**

1.  **시작 전 준비 (셀 1-5)**:
    *   `pip show azure-ai-ml`: Azure Machine Learning Python SDK v2가 설치되었는지 확인합니다.
    *   **작업 영역 연결**: `DefaultAzureCredential` 또는 `InteractiveBrowserCredential`을 사용하여 Azure에 인증하고, `MLClient.from_config(credential=credential)`를 통해 작업 영역에 연결합니다.

2.  **스크립트 생성 (셀 6-9)**:
    *   **`src` 폴더 생성**: 스크립트 파일을 저장할 `src` 폴더를 생성합니다.
    *   **`%%writefile $script_folder/prep-data.py`**: 첫 번째 파이프라인 단계에 해당하는 `prep-data.py` 스크립트를 생성합니다.
        *   **목적**: 누락된 데이터를 처리하고(dropna), 특정 숫자형 컬럼(`MinMaxScaler` 사용)을 정규화하여 데이터를 준비합니다.
        *   **입력**: `--input_data` (원본 데이터 파일 경로).
        *   **출력**: `--output_data` (처리된 데이터를 저장할 폴더 경로). `to_csv((Path(args.output_data) / "diabetes.csv"), index = False)`를 통해 `output_data` 경로의 폴더 안에 `diabetes.csv` 파일로 저장합니다.
    *   **`%%writefile $script_folder/train-model.py`**: 두 번째 파이프라인 단계에 해당하는 `train-model.py` 스크립트를 생성합니다.
        *   **목적**: 준비된 데이터를 사용하여 로지스틱 회귀 모델을 학습하고 평가합니다.
        *   **입력**: `--training_data` (학습 데이터 폴더 경로, `glob.glob`으로 폴더 내 CSV 파일을 읽음), `--reg_rate` (규제율 하이퍼파라미터).
        *   **출력**: `--model_output` (학습된 모델을 저장할 폴더 경로).
        *   **MLflow 로깅**: `mlflow.autolog()`를 사용하여 자동 로깅을 활성화하고, 규제율, 정확도, AUC를 로깅합니다. ROC 곡선을 이미지 아티팩트로 저장하고, `mlflow.sklearn.save_model(model, args.model_output)`을 사용하여 학습된 모델을 MLflow 형식으로 아티팩트로 저장합니다.

3.  **구성 요소(Component) 정의 (셀 10-12)**:
    *   파이프라인의 각 단계는 **구성 요소(Component)** 로 정의됩니다. 구성 요소는 재사용 가능하며 버전 관리되는 실행 단위입니다. YAML 파일로 정의하는 것이 일반적입니다.
    *   **`%%writefile prep-data.yml`**: `prep-data.py` 스크립트에 대한 구성 요소를 정의합니다.
        *   `name`, `display_name`, `version`, `type`: 구성 요소의 메타데이터.
        *   `inputs`:
            *   `input_data: type: uri_file`: 단일 파일(`diabetes.csv` 원본 파일)을 입력으로 받음을 명시합니다.
        *   `outputs`:
            *   `output_data: type: uri_folder`: 처리된 데이터를 폴더(`uri_folder`) 형태로 출력함을 명시합니다.
        *   `code`: 스크립트가 있는 로컬 폴더 경로 (`./src`).
        *   `environment`: 스크립트 실행에 필요한 환경.
        *   `command`: 실행할 명령과 입력/출력 매핑 (`${{inputs.input_data}}`, `${{outputs.output_data}}`).
    *   **`%%writefile train-model.yml`**: `train-model.py` 스크립트에 대한 구성 요소를 정의합니다.
        *   `inputs`:
            *   `training_data: type: uri_folder`: 학습 데이터를 폴더(`uri_folder`) 형태로 입력받음을 명시합니다. (`prep-data.py`의 출력이 이 형태로 전달될 것입니다.)
            *   `reg_rate: type: number, default: 0.01`: 규제율 하이퍼파라미터.
        *   `outputs`:
            *   `model_output: type: mlflow_model`: MLflow 형식의 모델을 출력함을 명시합니다.
        *   `command`: 실행할 명령과 입력/출력 매핑.

4.  **구성 요소 로드 (셀 13-14)**:
    *   `from azure.ai.ml import load_component`: YAML 파일로 정의된 구성 요소를 Python 객체로 로드하기 위한 함수입니다.
    *   `prep_data = load_component(source=parent_dir + "./prep-data.yml")`: `prep-data.yml`을 로드하여 `prep_data` 구성 요소 객체를 만듭니다.
    *   `train_logistic_regression = load_component(source=parent_dir + "./train-model.yml")`: `train-model.yml`을 로드하여 `train_logistic_regression` 구성 요소 객체를 만듭니다.

5.  **파이프라인 구축 (셀 15-16)**:
    *   `from azure.ai.ml.dsl import pipeline`: 파이프라인을 정의하는 데 사용되는 데코레이터입니다.
    *   `@pipeline()`: 이 데코레이터를 사용하여 `diabetes_classification` 함수를 파이프라인으로 정의합니다.
    *   `def diabetes_classification(pipeline_job_input):`: 파이프라인 함수는 전체 파이프라인의 입력(여기서는 원본 데이터)을 받습니다.
    *   **단계 정의 및 연결**:
        *   `clean_data = prep_data(input_data=pipeline_job_input)`: `prep_data` 구성 요소를 첫 번째 단계로 호출합니다. 입력은 파이프라인의 전체 입력(`pipeline_job_input`)입니다. `clean_data`는 이 단계의 출력을 참조하는 객체가 됩니다.
        *   `train_model = train_logistic_regression(training_data=clean_data.outputs.output_data)`: `train_logistic_regression` 구성 요소를 두 번째 단계로 호출합니다. **가장 중요한 부분은 첫 번째 단계(`clean_data`)의 출력(`clean_data.outputs.output_data`)을 두 번째 단계(`train_model`)의 입력(`training_data`)으로 전달하는 것입니다.** 이는 두 단계 간의 데이터 흐름(종속성)을 명확하게 정의합니다.
    *   `return { ... }`: 파이프라인 전체의 최종 출력을 정의합니다. 여기서는 변환된 데이터와 학습된 모델을 출력으로 설정합니다.
    *   `pipeline_job = diabetes_classification(Input(type=AssetTypes.URI_FILE, path="azureml:diabetes-data:1"))`: `diabetes_classification` 파이프라인을 특정 입력(`azureml:diabetes-data:1`이라는 등록된 데이터 자산의 버전 1)으로 인스턴스화하여 실행 가능한 `pipeline_job` 객체를 생성합니다.
    *   `print(pipeline_job)`: 생성된 `pipeline_job` 객체의 YAML 구성을 출력하여 정의된 파이프라인 구조를 검토할 수 있습니다.
    *   **파이프라인 설정 변경 (셀 17-18)**:
        *   `pipeline_job.outputs.pipeline_job_transformed_data.mode = "upload"`: 파이프라인의 최종 출력 모드를 `upload`로 설정하여 결과를 기본 데이터스토어에 업로드하도록 합니다.
        *   `pipeline_job.settings.default_compute = "aml-cluster"`: 파이프라인 내의 모든 단계가 기본적으로 `aml-cluster` 컴퓨팅 대상을 사용하도록 설정합니다.
        *   `pipeline_job.settings.default_datastore = "workspaceblobstore"`: 파이프라인 내의 모든 단계가 기본적으로 `workspaceblobstore` 데이터스토어를 사용하도록 설정합니다.

6.  **파이프라인 작업 제출 (셀 19-20)**:
    *   `pipeline_job = ml_client.jobs.create_or_update(pipeline_job, experiment_name="pipeline_diabetes")`: 구성된 `pipeline_job`을 Azure ML에 제출합니다. `pipeline_diabetes`라는 실험 아래에 작업이 생성됩니다.
    *   `pipeline_job`: 제출된 파이프라인 작업의 객체를 반환하며, 이는 Azure ML 스튜디오에서 작업을 모니터링할 수 있는 URL을 포함합니다.

**Azure ML 스튜디오에서 결과 확인:**

*   **작업(Jobs)** 페이지로 이동하여 `pipeline_diabetes` 실험을 선택합니다.
*   해당 파이프라인 작업을 클릭하면, 각 단계(예: `prep_data`, `train_model`)가 별도의 자식 작업으로 시각화되어 파이프라인의 흐름을 한눈에 볼 수 있습니다.
*   각 자식 작업을 클릭하여 해당 단계의 입력, 출력, 로그, 메트릭 등을 개별적으로 검토할 수 있습니다. 예를 들어, `prep_data` 단계의 출력 폴더에는 정규화된 `diabetes.csv` 파일이 있을 것이고, `train_model` 단계의 출력 폴더에는 MLflow 모델 아티팩트와 ROC 곡선 이미지가 저장되어 있을 것입니다.

---

**실습의 중요성 및 의미:**

*   **모듈화 및 재사용성**: 스크립트를 재사용 가능한 구성 요소로 분리하여 복잡한 머신러닝 워크플로우를 더 작은 관리 가능한 조각으로 나눌 수 있습니다.
*   **워크플로우 자동화**: 데이터 준비부터 모델 학습까지 여러 단계를 자동으로 실행하고, 각 단계의 출력은 다음 단계의 입력이 되어 수동 개입을 줄입니다.
*   **버전 관리**: 구성 요소는 버전 관리될 수 있으므로, 특정 버전의 데이터 준비 로직이나 학습 로직을 사용한 파이프라인을 정확히 재현할 수 있습니다.
*   **가시성 및 모니터링**: Azure ML 스튜디오는 파이프라인의 각 단계의 진행 상황, 입력, 출력, 로그 등을 시각적으로 보여주어 워크플로우를 쉽게 모니터링하고 디버깅할 수 있습니다.
*   **확장성**: 파이프라인의 각 단계는 독립적으로 실행될 수 있으므로, 다른 컴퓨팅 대상에서 병렬로 실행되거나, 필요에 따라 스케일 업/다운될 수 있습니다.
*   **MLOps 기반**: 파이프라인은 CI/CD (지속적 통합/지속적 배포) 파이프라인의 핵심 구성 요소가 되어 머신러닝 모델의 지속적인 학습, 평가 및 배포를 가능하게 합니다.

이 실습을 통해 Azure Machine Learning 파이프라인을 사용하여 복잡한 머신러닝 프로젝트를 구조화하고 자동화하는 데 필요한 기초 지식과 실질적인 경험을 얻을 수 있습니다.
