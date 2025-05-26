MLflow는 엔드투엔드 기계 학습 수명 주기를 관리하기 위한 오픈 소스 플랫폼입니다. MLflow로 모델을 로깅하면 모델을 플랫폼과 워크로드 간에 쉽게 이동할 수 있습니다.

이 연습에서는 MLflow를 사용하여 기계 학습 모델을 로깅합니다.

---

### Notebook에서 MLflow 작업 제출하기

이제 필요한 모든 리소스가 있으므로, Notebook을 실행하여 MLflow를 사용하여 모델 파라미터, 메트릭, 아티팩트를 추적하는 작업을 제출할 수 있습니다.

1.  **`Labs/08/Use MLflow to track jobs.ipynb`** Notebook을 엽니다. (참고: 파일 경로가 Labs/08로 되어 있지만, 이전 실습(`Labs/08/Use MLflow to track jobs.ipynb`)의 내용과 거의 동일합니다. 이 실습에서는 모델 로깅에 더 초점을 맞춥니다. 실제로는 `Labs/10/Log and register models with MLflow.ipynb` 파일을 열어야 합니다. 교재의 오탈자일 가능성이 있습니다.)
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> **`10`** 폴더로 이동하여 `Log and register models with MLflow.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.

3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다.

4.  **모든 셀을 실행합니다**.

---

### Notebook 내용 해설 (`Log and register models with MLflow.ipynb`)

이 Notebook은 MLflow를 사용하여 Azure Machine Learning에서 모델을 로깅하고 등록하는 다양한 방법을 보여줍니다. MLflow로 모델을 로깅하면 단순히 파일로 저장하는 것을 넘어, 모델의 메타데이터(예: 유형, 스키마, 파라미터)를 함께 기록하여 재사용 및 배포를 용이하게 합니다.

**주요 단계 및 코드 설명:**

1.  **시작 전 준비 (셀 1-5)**:
    *   `pip show azure-ai-ml`: Azure Machine Learning Python SDK v2가 설치되었는지 확인합니다.
    *   **작업 영역 연결**: `DefaultAzureCredential` 또는 `InteractiveBrowserCredential`을 사용하여 Azure에 인증하고, `MLClient.from_config(credential=credential)`를 통해 작업 영역에 연결합니다.

2.  **MLflow 자동 로깅(Autologging)으로 모델 로깅 (셀 6-9)**:
    *   **`src` 폴더 생성**: 스크립트 파일을 저장할 `src` 폴더를 생성합니다.
    *   **`%%writefile $script_folder/train-model-autolog.py`**: `mlflow.autolog()`를 사용하는 학습 스크립트를 생성합니다.
        *   **`mlflow.autolog()`**: 이 한 줄의 코드만으로 Scikit-learn 모델의 학습 과정에서 사용된 하이퍼파라미터, 성능 메트릭, 그리고 학습된 **모델 자체(MLmodel 형식)**가 자동으로 MLflow에 로깅됩니다.
        *   `MLmodel` 파일은 모델의 "설명서" 역할을 하며, 모델의 종류(flavor), 입력/출력 스키마, 의존성 등을 포함합니다.
    *   **명령 작업 제출**: 이 스크립트를 `diabetes-train-autolog`라는 `display_name`을 가진 명령 작업으로 Azure ML에 제출합니다.
    *   **스튜디오에서 결과 확인**:
        *   제출된 작업의 **출력 + 로그(Outputs + logs)** 탭으로 이동합니다.
        *   `model` 폴더를 찾아 그 안에 있는 `MLmodel` 파일의 내용을 탐색합니다. 이 파일은 MLflow가 자동으로 추론한 모델의 메타데이터를 보여줍니다.

3.  **자동 로깅 시 모델 Flavor 지정 (셀 10-13)**:
    *   **`%%writefile $script_folder/train-model-sklearn.py`**: `mlflow.sklearn.autolog()`를 사용하는 스크립트를 생성합니다.
        *   **`mlflow.sklearn.autolog()`**: 일반 `mlflow.autolog()`보다 더 구체적으로 Scikit-learn 모델에 대한 자동 로깅을 활성화합니다. MLflow는 기본적으로 모델 종류(flavor)를 잘 추론하지만, 특정 라이브러리(예: `sklearn`, `tensorflow`, `pytorch` 등)에 대한 `autolog()`를 사용하면 더 정확한 flavor를 보장할 수 있습니다.
    *   **명령 작업 제출**: 이 스크립트를 `diabetes-train-sklearn`이라는 `display_name`을 가진 명령 작업으로 제출합니다.
    *   **스튜디오에서 결과 확인**: `model` 폴더 내 `MLmodel` 파일을 검토합니다. 이전 `autolog()` 실행과 동일한 내용(flavor 추론)을 확인할 수 있습니다. 이는 MLflow의 자동 추론 기능이 강력함을 의미합니다.

4.  **추론된 서명(Inferred Signature)으로 모델 사용자 지정 로깅 (셀 14-17)**:
    *   **`%%writefile $script_folder/train-model-infer.py`**: 수동으로 모델을 로깅하고, **자동으로 서명을 추론**하여 지정하는 스크립트를 생성합니다.
        *   `mlflow.autolog()` 호출이 없습니다.
        *   `from mlflow.models.signature import infer_signature`: MLflow 모델 서명을 추론하는 함수입니다.
        *   `signature = infer_signature(X_train, y_hat)`: 학습 데이터의 입력(`X_train`)과 모델의 예측 결과(`y_hat`)를 기반으로 모델 서명(입력/출력 스키마)을 자동으로 추론합니다.
        *   `mlflow.sklearn.log_model(model, "model", signature=signature)`: 학습된 모델(`model`)을 `model`이라는 이름으로 로깅하고, 추론된 `signature`를 명시적으로 연결합니다.
    *   **명령 작업 제출**: 이 스크립트를 `diabetes-train-infer`라는 `display_name`을 가진 명령 작업으로 제출합니다.
    *   **스튜디오에서 결과 확인**: `model` 폴더 내 `MLmodel` 파일을 검토합니다. 이전 `autolog()` 실행과 비교하여 `signature` 섹션이 어떻게 추론되었는지 확인합니다. (이전 자동 로깅은 주로 텐서 기반 서명을 사용했을 수 있고, 이 실행은 컬럼 기반 서명을 더 명확하게 보여줄 수 있습니다.)

5.  **정의된 서명(Defined Signature)으로 모델 사용자 지정 로깅 (셀 18-21)**:
    *   **`%%writefile $script_folder/train-model-signature.py`**: 수동으로 모델을 로깅하고, **서명을 수동으로 정의**하여 지정하는 스크립트를 생성합니다.
        *   `from mlflow.models.signature import ModelSignature`
        *   `from mlflow.types.schema import Schema, ColSpec`: 입력 및 출력 데이터의 스키마(컬럼 이름, 데이터 타입)를 정의하는 클래스입니다.
        *   `input_schema = Schema([...])`, `output_schema = Schema([ColSpec("boolean")])`: 모델의 입력 특성(컬럼 이름과 데이터 타입) 및 출력(여기서는 불리언 타입의 예측)을 직접 정의합니다.
        *   `signature = ModelSignature(inputs=input_schema, outputs=output_schema)`: 정의된 입력 및 출력 스키마를 사용하여 `ModelSignature` 객체를 생성합니다.
        *   `mlflow.sklearn.log_model(model, "model", signature=signature)`: 학습된 모델과 수동으로 정의된 `signature`를 로깅합니다.
    *   **명령 작업 제출**: 이 스크립트를 `diabetes-train-signature`라는 `display_name`을 가진 명령 작업으로 제출합니다.
    *   **스튜디오에서 결과 확인**: `model` 폴더 내 `MLmodel` 파일을 검토합니다. 이전 실행들과 비교하여 서명 부분이 어떻게 명확하게 정의되었는지 확인합니다. 특히 컬럼 기반 서명이 명확히 명시된 것을 볼 수 있습니다.

6.  **모델 등록(Register the model) (셀 22-24)**:
    *   모델 학습이 완료되고 MLflow로 로깅된 후, 프로덕션 환경에서 모델을 배포하고 관리하기 위해 **모델 레지스트리(Model Registry)** 에 등록할 수 있습니다.
    *   `from azure.ai.ml.entities import Model`
    *   `from azure.ai.ml.constants import AssetTypes`
    *   `job_name = returned_job.name`: 가장 최근에 제출된 작업의 이름을 가져옵니다.
    *   `run_model = Model(...)`: `Model` 객체를 생성하여 등록할 모델의 정보를 정의합니다.
        *   `path=f"azureml://jobs/{job_name}/outputs/artifacts/paths/model/"`: 이 경로가 중요합니다. 이는 특정 **작업(`job_name`)의 출력 아티팩트(`outputs/artifacts`) 중 `model` 폴더**에 MLflow가 로깅한 모델이 위치함을 지정합니다.
        *   `name="mlflow-diabetes"`: 모델 레지스트리에 등록될 모델의 이름.
        *   `type=AssetTypes.MLFLOW_MODEL`: 모델이 MLflow 모델임을 명시하여 Azure ML이 MLflow 통합 기능을 활용할 수 있도록 합니다.
    *   `ml_client.models.create_or_update(run_model)`: 정의된 모델 객체를 Azure ML 모델 레지스트리에 등록합니다. 동일한 이름의 모델이 이미 있으면 새 버전이 생성됩니다.
    *   **스튜디오에서 결과 확인**:
        *   Azure ML 스튜디오의 왼쪽 메뉴에서 **모델(Models)** 페이지로 이동합니다.
        *   `mlflow-diabetes` 모델을 찾아 선택합니다.
        *   **세부 정보(Details)** 탭에서 모델의 유형(`MLFLOW`)과 모델을 학습시킨 작업 정보를 확인할 수 있습니다.
        *   **아티팩트(Artifacts)** 탭에서 `MLmodel` 파일과 모델 관련 기타 아티팩트(예: `conda.yaml`, `requirements.txt`)를 볼 수 있습니다.

---

**실습의 중요성 및 의미:**

*   **모델 이식성(Portability)**: MLflow `MLmodel` 형식은 모델의 종류(flavor), 입력/출력 스키마, 코드 의존성 등을 표준화하여 모델을 다른 환경이나 플랫폼으로 쉽게 이동하고 배포할 수 있도록 합니다.
*   **모델 재사용성(Reusability)**: 모델 레지스트리에 등록된 모델은 중앙 집중식으로 관리되어, 다른 팀원이나 애플리케이션에서 쉽게 검색하고 재사용할 수 있습니다.
*   **버전 관리(Versioning)**: 모델 레지스트리는 모델의 버전을 자동으로 관리하여, 특정 버전의 모델을 선택하거나 이전 버전으로 롤백하는 것이 가능합니다.
*   **데이터 계약(Data Contract)**: 모델 서명(Signature)은 모델의 입력 및 출력 데이터 스키마를 명확하게 정의하여, 모델 배포 시 데이터 불일치로 인한 오류를 방지하고 예측 서비스의 안정성을 높입니다.
*   **MLops 워크플로우의 핵심**: 모델 로깅과 등록은 모델 학습-배포-모니터링으로 이어지는 MLOps 파이프라인의 필수적인 단계입니다.

이 실습을 통해 MLflow의 모델 로깅 및 등록 기능을 깊이 이해하고, 이를 통해 모델 관리 및 배포 프로세스를 표준화하고 효율화하는 방법을 배울 수 있습니다.
