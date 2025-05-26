MLflow는 엔드투엔드 기계 학습 수명 주기를 관리하기 위한 오픈 소스 플랫폼입니다. MLflow Tracking은 학습 작업의 메트릭, 파라미터 및 모델 아티팩트를 로깅하고 추적하는 구성 요소입니다.

이 연습에서는 MLflow를 사용하여 명령 작업으로 실행되는 모델 학습을 추적합니다.

---

### Notebook에서 MLflow 작업 제출하기

이제 필요한 모든 리소스가 있으므로, Notebook을 실행하여 모델 파라미터, 메트릭 및 아티팩트를 추적하기 위해 MLflow를 사용하는 작업을 제출할 수 있습니다.

1.  **`Labs/08/Use MLflow to track jobs.ipynb`** Notebook을 엽니다.
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> `08` 폴더로 이동하여 `Use MLflow to track jobs.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.

3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다. (Notebook 상단 오른쪽에서 커널을 확인할 수 있으며, 필요시 변경합니다.)

4.  **모든 셀을 실행합니다**.

---

### Notebook 내용 해설 (`Use MLflow to track jobs.ipynb`)

이 Notebook은 MLflow를 사용하여 Azure Machine Learning 명령 작업을 통해 모델 학습을 추적하는 방법을 심층적으로 보여줍니다. 이전 실습에서 터미널 내에서 MLflow를 사용했던 것과 달리, 이번에는 Azure ML의 `command` 객체를 통해 MLflow 로깅이 내장된 스크립트를 원격으로 실행하는 방법을 다룹니다.

**주요 단계 및 코드 설명:**

1.  **시작 전 준비 (셀 1-5)**:
    *   **`pip show azure-ai-ml`**: Azure Machine Learning Python SDK v2가 설치되었는지 확인합니다.
    *   **작업 영역 연결**: `DefaultAzureCredential` 또는 `InteractiveBrowserCredential`을 사용하여 Azure에 인증하고, `MLClient.from_config(credential=credential)`를 통해 현재 작업 영역에 대한 클라이언트 핸들(`ml_client`)을 가져옵니다.

2.  **MLflow를 사용한 사용자 지정 추적 (Custom Tracking) (셀 6-9)**:
    *   **`src` 폴더 생성**: `os.makedirs(script_folder, exist_ok=True)`를 사용하여 스크립트 파일을 저장할 `src` 폴더를 생성합니다.
    *   **`train-model-mlflow.py` 스크립트 생성**:
        *   `%%writefile $script_folder/train-model-mlflow.py` Jupyter 매직 명령을 사용하여 바로 뒤에 오는 Python 코드를 `./src/train-model-mlflow.py` 파일로 저장합니다.
        *   **스크립트 내용 분석**:
            *   `import mlflow`: MLflow 라이브러리를 임포트합니다.
            *   `main(args)` 함수는 데이터 읽기, 분할, 모델 학습, 모델 평가의 주요 단계를 포함합니다.
            *   **`train_model` 함수**: `mlflow.log_param("Regularization rate", reg_rate)`를 사용하여 모델의 규제율(파라미터)을 명시적으로 로깅합니다.
            *   **`eval_model` 함수**:
                *   `mlflow.log_metric("Accuracy", acc)`: 계산된 정확도(메트릭)를 명시적으로 로깅합니다.
                *   `mlflow.log_metric("AUC", auc)`: 계산된 AUC(메트릭)를 명시적으로 로깅합니다.
                *   ROC 곡선을 그리고 `plt.savefig("ROC-Curve.png")`로 이미지 파일로 저장한 후, `mlflow.log_artifact("ROC-Curve.png")`를 사용하여 이 이미지를 아티팩트로 로깅합니다.
            *   `parse_args()`: `argparse`를 사용하여 `--training_data`와 `--reg_rate` 두 가지 명령줄 인자를 정의합니다.
    *   **명령 작업 제출**:
        *   `from azure.ai.ml import command`: Azure ML SDK에서 `command` 함수를 임포트합니다.
        *   `job = command(...)`: 명령 작업을 정의하는 핵심 부분입니다.
            *   `code="./src"`: 학습 스크립트가 포함된 로컬 디렉터리(`src` 폴더)를 지정합니다. 이 내용은 작업 실행 시 원격 컴퓨팅 대상으로 업로드됩니다.
            *   `command="python train-model-mlflow.py --training_data diabetes.csv"`: 원격 컴퓨팅에서 실행할 실제 셸 명령입니다. `--training_data` 인자로 `diabetes.csv` 파일을 전달합니다.
            *   `environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest"`: 작업이 실행될 conda 환경을 지정합니다. Azure ML에서 제공하는 관리형 환경을 사용합니다.
            *   `compute="aml-cluster"`: 이전에 `setup.sh` 스크립트로 생성한 컴퓨팅 클러스터(`aml-cluster`)에서 작업을 실행하도록 지정합니다.
            *   `display_name="diabetes-train-mlflow"`, `experiment_name="diabetes-training"`, `tags={"model_type": "LogisticRegression"}`: Azure ML 스튜디오에서 작업을 식별하고 그룹화, 필터링하는 데 사용되는 메타데이터입니다.
        *   `returned_job = ml_client.create_or_update(job)`: 정의된 명령 작업을 Azure ML에 제출합니다.
        *   `aml_url = returned_job.studio_url`: 작업 실행을 Azure ML 스튜디오에서 직접 모니터링할 수 있는 URL을 출력합니다.
    *   **스튜디오에서 결과 확인 지침**: `diabetes-train-mlflow` 작업에서 **개요(Overview)** 탭의 **파라미터(Params)**, **메트릭(Metrics)** 탭, **이미지(Images)** 탭 (ROC 곡선), **출력 + 로그(Outputs + logs)** 탭 (모든 파일)에서 MLflow가 로깅한 정보들을 확인할 수 있습니다.

3.  **MLflow를 사용한 자동 로깅 (Autologging) (셀 10-12)**:
    *   **`train-model-autolog.py` 스크립트 생성**:
        *   `%%writefile $script_folder/train-model-autolog.py`를 사용하여 두 번째 스크립트를 저장합니다.
        *   **스크립트 내용 분석 (주요 차이점)**:
            *   `mlflow.autolog()`: `main` 함수 시작 부분에서 이 한 줄만 추가하면, MLflow가 지원되는 라이브러리(여기서는 Scikit-learn)에서 학습된 모델의 파라미터, 메트릭, 모델 아티팩트 등을 자동으로 로깅합니다. 이전 `train-model-mlflow.py` 스크립트에서 명시적으로 호출했던 `mlflow.log_param`과 `mlflow.log_metric` 호출이 이 스크립트에서는 제거됩니다.
            *   **주의**: ROC 곡선과 같은 사용자 지정 아티팩트는 여전히 `mlflow.log_artifact`를 사용하여 명시적으로 로깅해야 합니다. `mlflow.autolog()`는 모든 것을 자동으로 로깅하지는 않습니다.
    *   **명령 작업 제출**: `command` 객체를 사용하여 `train-model-autolog.py` 스크립트를 실행하는 새로운 명령 작업을 제출합니다. `display_name`은 `diabetes-train-autolog`로 변경됩니다.
    *   **스튜디오에서 결과 확인 지침**: 마찬가지로 **개요(Overview)**, **메트릭(Metrics)**, **이미지(Images)**, **출력 + 로그(Outputs + logs)** 탭에서 자동 로깅된 결과들을 확인할 수 있습니다.

4.  **MLflow를 사용하여 실험(Experiment) 조회 및 검색 (셀 13-18)**:
    *   Azure Machine Learning 스튜디오 UI 외에도 MLflow Python 클라이언트 API를 사용하여 프로그래밍 방식으로 작업 및 실험 기록을 조회하고 관리할 수 있습니다.
    *   `mlflow.search_experiments()`: 작업 영역에 있는 모든 실험 목록을 반환합니다.
    *   `mlflow.get_experiment_by_name(experiment_name)`: 특정 이름의 실험 객체를 가져옵니다.
    *   `mlflow.search_runs(exp.experiment_id)`: 특정 실험 ID에 속하는 모든 실행(작업) 목록을 반환합니다. 결과는 `pandas.DataFrame` 형태로 반환되어 분석하기 용이합니다.
    *   **정렬 및 제한**: `order_by=["start_time DESC"]`, `max_results=2`와 같은 인자를 사용하여 결과를 정렬하고 반환되는 최대 실행 수를 제한할 수 있습니다.
    *   **필터링**: `filter_string` 인자를 사용하여 SQL `WHERE` 절과 유사한 구문으로 실행을 필터링할 수 있습니다.
        *   **숫자 비교 연산자**: `=`, `!=`, `>`, `>=`, `<`, `<=` (메트릭에 사용)
        *   **문자열 비교 연산자**: `=`, `!=` (파라미터, 태그, 속성에 사용)
        *   **예시 쿼리**: `query = "metrics.AUC > 0.8 and tags.model_type = 'LogisticRegression'"`는 AUC가 0.8보다 크고 `model_type` 태그가 'LogisticRegression'인 실행을 필터링합니다.

---

**실습의 중요성 및 의미:**

*   **재현성(Reproducibility)**: MLflow를 사용하면 모델 학습에 사용된 모든 파라미터, 메트릭, 코드를 추적할 수 있어 실험의 재현성을 크게 높입니다.
*   **협업(Collaboration)**: 팀원 간에 누가 어떤 실험을 어떤 결과로 실행했는지 쉽게 공유하고 확인할 수 있습니다.
*   **모델 버전 관리(Model Versioning)**: 학습된 모델 아티팩트를 자동으로 기록하고, 나중에 특정 실행에서 학습된 모델을 불러와 재사용하거나 배포할 수 있습니다.
*   **실험 관리(Experiment Management)**: 여러 실험을 체계적으로 그룹화하고, 다양한 모델 및 하이퍼파라미터 조합의 성능을 효과적으로 비교 분석할 수 있습니다.
*   **프로덕션 환경으로의 전환**: Notebook에서 개발한 코드를 스크립트화하고, 이를 Azure ML의 명령 작업으로 제출하여 프로덕션 환경에 맞는 자동화된 워크플로우를 구축하는 데 필수적인 단계입니다.

이 실습을 통해 MLflow의 강력한 추적 기능을 Azure Machine Learning의 명령 작업과 통합하여 머신러닝 개발 프로세스를 더욱 효율적이고 관리하기 쉽게 만드는 방법을 깊이 이해할 수 있습니다.
