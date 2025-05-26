## Responsible AI 대시보드 생성 및 탐색

모델을 학습한 후, 예상대로 작동하는지 탐색하기 위해 모델을 평가하고 싶을 것입니다. 성능 메트릭 외에도 고려해야 할 다른 요소들이 있습니다. Azure Machine Learning의 Responsible AI 대시보드를 사용하면 데이터와 모델의 예측을 분석하여 편향(bias)이나 불공정성(unfairness)을 식별할 수 있습니다.

이 연습에서는 데이터를 준비하고 Azure Machine Learning에서 Responsible AI 대시보드를 생성합니다.

**실습 목표:**

*   Responsible AI 대시보드 생성을 위한 데이터 준비 및 자산 등록 (Parquet 및 MLtable).
*   사전 학습된 모델을 모델 레지스트리에 등록하기.
*   Azure ML에서 기본 제공 Responsible AI 구성 요소를 가져오기.
*   가져온 구성 요소를 사용하여 Responsible AI 대시보드 생성 파이프라인 구축하기.
*   파이프라인 작업을 제출하고 Azure ML 스튜디오에서 대시보드 결과 탐색하기.

---

### Notebook에서 모델을 평가하고 제출하기 위한 파이프라인 생성

필요한 모든 리소스가 준비되었으므로, Notebook을 실행하여 기본 제공 Responsible AI 구성 요소를 가져오고, 파이프라인을 생성하며, 파이프라인을 제출하여 Responsible AI 대시보드를 생성할 수 있습니다.

1.  **`Labs/10/Create Responsible AI dashboard.ipynb`** Notebook을 엽니다.
    *   **파일(Files)** 탐색기에서 `azure-ml-labs` -> `Labs` -> `10` 폴더로 이동하여 `Create Responsible AI dashboard.ipynb` 파일을 클릭합니다.

2.  인증하라는 알림이 나타나면 **인증(Authenticate)** 을 선택하고 필요한 단계를 따릅니다.

3.  Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다.

4.  **모든 셀을 실행합니다**.

---

### Notebook 내용 해설 (`Create Responsible AI dashboard.ipynb`)

이 Notebook은 Azure Machine Learning의 **Responsible AI(RAI) 대시보드**를 사용하여 모델의 성능, 공정성, 해석 가능성, 오류 분석 등을 종합적으로 평가하는 방법을 보여줍니다. RAI 대시보드는 MLops의 중요한 부분으로, AI 시스템의 책임감 있는 개발을 지원합니다.

**주요 단계 및 코드 설명:**

1.  **데이터 준비 (셀 2-4)**:
    *   **데이터 읽기**: `train-data/diabetes.csv` 및 `test-data/diabetes-test.csv` 파일을 Pandas DataFrame으로 읽어옵니다.
    *   **Parquet으로 변환**: Responsible AI 대시보드를 생성하려면 학습 및 테스트 데이터셋이 **Parquet 파일** 형식으로 저장되어야 합니다. `pyarrow` 라이브러리를 사용하여 CSV 파일을 Parquet 파일(`diabetes-training.parquet`, `diabetes-test.parquet`)로 변환하고 로컬 폴더에 저장합니다.
        *   **Parquet의 의미**: 컬럼 기반 저장 형식으로, 대규모 데이터셋 처리 시 효율적이고 압축률이 높아 클라우드 환경에서 선호됩니다.

2.  **작업 영역 연결 (셀 5-8)**:
    *   `pip show azure-ai-ml`: Azure Machine Learning Python SDK v2가 설치되었는지 확인합니다.
    *   `MLClient.from_config(credential=credential)`: Azure Machine Learning 작업 영역에 대한 클라이언트 핸들(`ml_client`)을 가져옵니다.

3.  **데이터 자산 생성 (셀 9-11)**:
    *   Responsible AI 대시보드를 생성하려면 학습 및 테스트 데이터셋이 **MLtable** 데이터 자산으로 등록되어야 합니다. MLtable은 Azure ML에서 표 형식 데이터에 최적화된 데이터 자산 유형입니다.
    *   `from azure.ai.ml.entities import Data`
    *   `from azure.ai.ml.constants import AssetTypes`
    *   `ml_client.data.get(...)`과 `try-except` 블록을 사용하여 이미 등록된 데이터가 있는지 확인하고, 없으면 `Data` 객체를 생성하여 `AssetTypes.MLTABLE` 유형으로 학습 및 테스트 데이터를 등록합니다.
    *   `ml_client.data.create_or_update(train_data)` / `test_data`: Parquet 파일을 참조하는 MLtable 데이터 자산을 작업 영역에 등록합니다.

4.  **모델 등록 (셀 12-14)**:
    *   RAI 대시보드는 모델을 분석하기 위해 등록된 모델이 필요합니다. Notebook에서 제공된 `model` 폴더에는 미리 학습된 모델이 포함되어 있습니다.
    *   `registry_name = "azureml"`: Azure ML에서 기본 제공 구성 요소(built-in components)에 액세스하기 위해 `azureml` 레지스트리에 연결합니다.
    *   `ml_client_registry = MLClient(...)`: `azureml` 레지스트리에 대한 별도의 `MLClient` 객체를 생성합니다.
    *   `file_model = Model(path="model", type=AssetTypes.MLFLOW_MODEL, name="local-mlflow-diabetes", ...)`: 로컬 `model` 폴더에 있는 MLflow 모델을 나타내는 `Model` 객체를 생성합니다.
    *   `model = ml_client.models.create_or_update(file_model)`: 이 모델을 작업 영역의 모델 레지스트리에 등록합니다.

5.  **Responsible AI 대시보드 구축 파이프라인 정의 (셀 15-18)**:
    *   RAI 대시보드는 여러 기본 제공 구성 요소로 구성된 파이프라인을 통해 생성됩니다.
    *   **기본 제공 구성 요소 가져오기 (셀 16)**:
        *   `rai_constructor_component = ml_client_registry.components.get(name="rai_tabular_insight_constructor", label=label)`: RAI 대시보드 생성을 시작하는 구성 요소입니다.
        *   `rai_erroranalysis_component = ml_client_registry.components.get(name="rai_tabular_erroranalysis", version=version)`: 모델의 오류 패턴을 분석하는 구성 요소입니다.
        *   `rai_explanation_component = ml_client_registry.components.get(name="rai_tabular_explanation", version=version)`: 모델의 해석 가능성(특성 중요도 등)을 생성하는 구성 요소입니다.
        *   `rai_gather_component = ml_client_registry.components.get(name="rai_tabular_insight_gather", version=version)`: 앞에서 생성된 모든 인사이트를 수집하고 최종 대시보드를 생성하는 구성 요소입니다.
    *   **파이프라인 함수 정의 (`@dsl.pipeline`) (셀 18)**:
        *   `@dsl.pipeline(...)` 데코레이터를 사용하여 `rai_decision_pipeline` 함수를 파이프라인으로 정의합니다. 이 파이프라인은 `target_column_name`, `train_data`, `test_data`를 입력으로 받습니다.
        *   **`create_rai_job` (Constructor)**:
            *   `rai_constructor_component(...)`: 파이프라인의 첫 번째 단계로, 대시보드의 기본 구조를 만듭니다.
            *   `model_info=expected_model_id`: 이전에 등록한 모델의 ID를 전달하여 어떤 모델을 분석할지 지정합니다.
            *   `train_dataset`, `test_dataset`, `target_column_name`: 분석할 학습/테스트 데이터와 대상 컬럼을 전달합니다.
        *   **`error_job` (Error Analysis)**:
            *   `rai_erroranalysis_component(...)`: `create_rai_job.outputs.rai_insights_dashboard`를 입력으로 받아 오류 분석을 수행합니다. 즉, Constructor의 출력이 Error Analysis의 입력이 됩니다.
        *   **`explanation_job` (Explanation)**:
            *   `rai_explanation_component(...)`: `create_rai_job.outputs.rai_insights_dashboard`를 입력으로 받아 모델 설명을 생성합니다.
        *   **`rai_gather_job` (Gather)**:
            *   `rai_gather_component(...)`: Constructor (`create_rai_job.outputs.rai_insights_dashboard`), Error Analysis (`error_job.outputs.error_analysis`), Explanation (`explanation_job.outputs.explanation`)의 출력을 모두 입력으로 받아 최종 RAI 대시보드를 조립합니다.
        *   `set_limits(timeout=300)`: 각 단계의 최대 실행 시간을 설정하여 무한 실행을 방지합니다.
        *   `rai_gather_job.outputs.dashboard.mode = "upload"`: 최종 대시보드 결과가 Azure Storage에 업로드되도록 합니다.

6.  **파이프라인 입력 정의 및 인스턴스화 (셀 19-20)**:
    *   `diabetes_train_pq = Input(...)`, `diabetes_test_pq = Input(...)`: 이전에 등록한 MLtable 데이터 자산을 `Input` 객체로 정의하여 파이프라인에 전달할 준비를 합니다. `mode="download"`는 컴퓨팅 대상으로 데이터를 다운로드하여 처리함을 의미합니다.
    *   `insights_pipeline_job = rai_decision_pipeline(...)`: `rai_decision_pipeline` 함수를 호출하여 실제 데이터 입력과 대상 컬럼을 전달함으로써 실행 가능한 파이프라인 작업 객체를 생성합니다.
    *   `uuid.uuid4()`: 고유한 출력 경로를 생성하여 충돌을 피하기 위한 일반적인 workaround입니다.

7.  **파이프라인 실행 (셀 21-22)**:
    *   `submit_and_wait(ml_client, pipeline_job)`: 헬퍼 함수를 사용하여 파이프라인 작업을 Azure ML에 제출하고, 작업이 완료되거나 실패할 때까지 상태를 폴링하여 출력합니다.
    *   `ml_client.jobs.create_or_update(pipeline_job)`: 실제 파이프라인 작업을 제출합니다. 작업이 시작되면 Azure ML 스튜디오의 URL이 출력되어 진행 상황을 모니터링할 수 있습니다.

**Azure ML 스튜디오에서 결과 확인:**

*   파이프라인 작업이 완료되면 Azure ML 스튜디오의 **작업(Jobs)** 페이지로 이동합니다.
*   `RAI_insights_local-mlflow-diabetes` (또는 `RAI_insights_` 뒤에 모델 이름이 붙은) 실험을 찾아서 선택합니다.
*   실행된 파이프라인 작업을 클릭합니다. 파이프라인의 시각적 그래프와 각 단계의 상태를 볼 수 있습니다.
*   파이프라인이 성공적으로 완료되면, 작업 페이지 상단에 **Responsible AI 대시보드 링크**가 표시됩니다. 이 링크를 클릭하여 대시보드를 탐색할 수 있습니다.
*   대시보드에서는 다음과 같은 정보를 확인할 수 있습니다:
    *   **데이터 탐색(Data Explorer)**: 데이터셋의 분포 및 통계.
    *   **모델 성능(Model Performance)**: 전체 모델의 성능 지표와 특정 데이터 코호트(cohort)별 성능 비교.
    *   **오류 분석(Error Analysis)**: 모델이 잘못 예측하는 데이터 하위 그룹 식별.
    *   **특성 중요도(Feature Importance / Interpretability)**: 모델의 예측에 어떤 특성이 가장 큰 영향을 미쳤는지 (전역 및 지역 수준).
    *   **공정성(Fairness)**: 민감 특성(예: 성별, 나이)에 따른 모델 성능의 불균형 여부.

---

**실습의 중요성 및 의미:**

*   **책임감 있는 AI(Responsible AI)**: AI 모델의 투명성, 공정성, 책임감을 확보하는 데 필수적인 도구입니다. 단순히 성능이 좋은 모델을 넘어, 모델이 어떻게 작동하고, 잠재적인 편향이 있는지 등을 이해하는 데 도움을 줍니다.
*   **종합적인 모델 평가**: 전통적인 성능 메트릭 외에 모델의 내부 동작과 사회적 영향까지 고려한 심층적인 평가를 가능하게 합니다.
*   **파이프라인 기반 자동화**: Responsible AI 분석을 위한 대시보드 생성을 자동화된 파이프라인으로 구현하여 재현성과 효율성을 높입니다.
*   **내장 구성 요소 활용**: Azure ML이 제공하는 미리 정의된 구성 요소를 활용하여 복잡한 RAI 분석을 쉽게 수행할 수 있습니다.
*   **MLops와의 통합**: 학습된 모델에 대한 RAI 분석을 MLops 워크플로우에 통합하여 지속적으로 모델의 책임감 있는 측면을 평가하고 개선할 수 있습니다.

이 실습을 통해 Azure Machine Learning의 Responsible AI 대시보드를 사용하여 모델의 성능을 다각적으로 평가하고, 잠재적인 편향과 불공정성을 식별하여 보다 신뢰할 수 있는 AI 시스템을 구축하는 방법을 배울 수 있습니다.
