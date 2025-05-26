Azure Machine Learning에서 명령 작업으로 학습 스크립트 실행하기
Notebook은 실험과 개발에 이상적입니다. 기계 학습 모델을 개발하고 프로덕션 준비가 되면 스크립트로 학습시키고 싶을 것입니다. 스크립트를 명령 작업으로 실행할 수 있습니다.
이 연습에서는 스크립트를 테스트한 다음 명령 작업으로 실행합니다.
실습 목표:
Azure CLI를 사용하여 Azure Machine Learning 작업 영역 및 컴퓨팅 리소스 프로비저닝하기.
Notebook을 Python 스크립트로 변환하기.
터미널에서 인자를 사용하여 Python 스크립트 테스트하기.
Python SDK를 사용하여 스크립트를 Azure ML 명령 작업으로 실행하기.
Azure ML 스튜디오에서 명령 작업 결과(코드, 출력, 로그) 검토하기.
생성된 Azure 리소스 정리하기.
준비물:
관리자 수준 액세스 권한이 있는 Azure 구독.
Azure Machine Learning 작업 영역 프로비저닝
Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스와 자산을 관리하기 위한 중앙 집중식 공간을 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.
이 실습에서는 Azure CLI를 사용하여 작업 영역과 필요한 컴퓨팅을 프로비저닝하고, Python SDK를 사용하여 명령 작업을 실행합니다.
작업 영역 및 컴퓨팅 리소스 만들기
Azure Machine Learning 작업 영역, 컴퓨팅 인스턴스 및 컴퓨팅 클러스터를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 사용자가 실행할 수 있도록 셸 스크립트에 그룹화되어 있습니다.
브라우저에서 Microsoft 계정으로 로그인하여 https://portal.azure.com/에서 Azure Portal을 엽니다.
페이지 상단의 검색창 오른쪽에 있는 [>_] (Cloud Shell) 버튼을 선택합니다. 그러면 포털 하단에 Cloud Shell 창이 열립니다.
메시지가 나타나면 Bash를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다.
올바른 구독이 지정되어 있고 저장소 계정 필요 없음(No storage account required) 이 선택되어 있는지 확인합니다. 적용(Apply) 을 선택합니다.
터미널에 다음 명령을 입력하여 이 리포지토리를 복제합니다.
rm -r azure-ml-labs -f
git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
Use code with caution.
Bash
팁: 복사한 코드를 Cloud Shell에 붙여넣으려면 SHIFT + INSERT를 사용하십시오.
리포지토리가 복제된 후 다음 명령을 입력하여 이 실습용 폴더로 변경하고 포함된 setup.sh 스크립트를 실행합니다.
cd azure-ml-labs/Labs/08
./setup.sh
Use code with caution.
Bash
참고: 확장이 설치되지 않았다는 (오류) 메시지는 무시하십시오.
스크립트가 완료될 때까지 기다립니다. 일반적으로 약 5-10분 정도 소요됩니다.
문제 해결 팁: 작업 영역 생성 오류
(이전 실습에서 언급된 내용과 동일하게, setup.sh 스크립트 내 VM 크기 관련 문제 발생 시 스크립트를 수정하거나 수동으로 리소스를 생성하는 방법을 고려할 수 있습니다.)
실습 자료 복제하기
작업 영역과 필요한 컴퓨팅 리소스를 만들었으면 Azure Machine Learning 스튜디오를 열고 작업 영역에 실습 자료를 복제할 수 있습니다.
Azure Portal에서 mlw-dp100-… (스크립트가 생성한 실제 이름)라는 Azure Machine Learning 작업 영역으로 이동합니다.
Azure Machine Learning 작업 영역을 선택하고 개요(Overview) 페이지에서 스튜디오 시작(Launch studio) 을 선택합니다. 브라우저에서 다른 탭이 열리면서 Azure Machine Learning 스튜디오가 열립니다.
스튜디오에 나타나는 모든 팝업을 닫습니다.
Azure Machine Learning 스튜디오 내에서 컴퓨팅(Compute) 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스는 실행 중이어야 하고, 클러스터는 유휴 상태이며 실행 중인 노드가 0개여야 합니다.
컴퓨팅 인스턴스(Compute instances) 탭에서 컴퓨팅 인스턴스를 찾아 터미널(Terminal) 애플리케이션을 선택합니다.
터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.
pip uninstall azure-ai-ml -y
pip install azure-ai-ml
Use code with caution.
Bash
참고: 패키지를 찾을 수 없거나 제거할 수 없다는 (오류) 메시지는 무시하십시오.
다음 명령을 실행하여 Notebook, 데이터 및 기타 파일을 포함하는 Git 리포지토리를 작업 영역에 복제합니다.
git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
Use code with caution.
Bash
명령이 완료되면 파일(Files) 창에서 ↻ (새로고침) 을 클릭하여 보기를 새로 고치고 새 Users/your-user-name/azure-ml-labs 폴더가 생성되었는지 확인합니다.
Notebook을 스크립트로 변환하기
컴퓨팅 인스턴스에 연결된 Notebook을 사용하는 것은 작성한 코드를 즉시 실행하고 출력을 검토할 수 있으므로 실험 및 개발에 이상적입니다. 개발에서 프로덕션으로 전환하려면 스크립트를 사용하고 싶을 것입니다. 첫 번째 단계로 Azure Machine Learning 스튜디오를 사용하여 Notebook을 스크립트로 변환할 수 있습니다.
Labs/08/src/Train classification model.ipynb Notebook을 엽니다.
파일(Files) 탐색기에서 azure-ml-labs -> Labs -> 08 -> src 폴더로 이동하여 Train classification model.ipynb 파일을 클릭합니다.
인증하라는 알림이 나타나면 인증(Authenticate) 을 선택하고 필요한 단계를 따릅니다.
Notebook이 Python 3.10 - AzureML 커널을 사용하는지 확인합니다.
모든 셀을 실행하여 코드를 탐색하고 모델을 학습합니다.
Notebook 상단의 ☰ 아이콘을 선택하여 Notebook 메뉴를 봅니다.
다음으로 내보내기(Export as) 를 확장하고 Python (.py) 를 선택하여 Notebook을 Python 스크립트로 변환합니다.
새 파일 이름을 train-classification-model.py 로 지정합니다.
새 파일이 생성되면 스크립트가 자동으로 열립니다. 파일을 탐색하고 Notebook과 동일한 코드가 포함되어 있는지 확인합니다.
Notebook(또는 스크립트 편집기) 상단의 ▷▷ (모두 실행 또는 스크립트 실행) 아이콘을 선택하여 터미널에서 스크립트를 저장하고 실행합니다.
스크립트는 python train-classification-model.py 명령으로 시작되며 출력은 명령 아래에 표시되어야 합니다.
참고: 스크립트가 libstdc++6에 대한 ImportError를 반환하는 경우 스크립트를 다시 실행하기 전에 터미널에서 다음 명령을 실행합니다.
sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
sudo apt-get update
sudo apt-get upgrade libstdc++6 -y
Use code with caution.
Bash
(이 단계는 특정 환경 설정에 따라 필요할 수 있습니다.)
터미널로 스크립트 테스트하기
Notebook을 스크립트로 변환한 후 추가로 구체화할 수 있습니다. 스크립트 작업 시 모범 사례 중 하나는 함수를 사용하는 것입니다. 스크립트가 함수로 구성되면 코드를 단위 테스트하기가 더 쉬워집니다. 함수를 사용하면 스크립트는 각각 특정 작업을 수행하는 코드 블록으로 구성됩니다.
Labs/08/src/train-model-parameters.py 스크립트를 열고 내용을 탐색합니다. 네 가지 다른 함수를 포함하는 main 함수가 있습니다.
read_data
split_data
train_model
evaluate_model
main 함수 다음에는 각 함수가 정의되어 있습니다. 각 함수가 예상 입력 및 출력을 어떻게 정의하는지 주목하십시오.
스크립트 편집기 상단의 ▷▷ (스크립트 실행) 아이콘을 선택하여 터미널에서 스크립트를 저장하고 실행합니다. Reading data… 이후 파일 경로가 잘못되어 데이터를 가져올 수 없다는 오류가 발생해야 합니다.
스크립트를 사용하면 입력 데이터나 매개변수를 쉽게 변경할 수 있도록 코드를 매개변수화할 수 있습니다. 이 경우 스크립트는 우리가 제공하지 않은 데이터 경로에 대한 입력 매개변수를 예상합니다. 스크립트 끝의 parse_args() 함수에서 정의되고 예상되는 매개변수를 찾을 수 있습니다.
두 가지 입력 매개변수가 정의되어 있습니다.
--training_data: 문자열을 예상합니다.
--reg_rate: 숫자를 예상하지만 기본값은 0.01입니다.
스크립트를 성공적으로 실행하려면 학습 데이터 매개변수에 대한 값을 지정해야 합니다. 학습 스크립트와 동일한 폴더에 저장된 diabetes.csv 파일을 참조하여 이를 수행해 보겠습니다.
터미널에서 다음 명령을 실행합니다 (현재 터미널 경로가 azure-ml-labs/Labs/08/src/ 디렉터리인지 확인하거나, 아니라면 cd 명령으로 이동합니다):
먼저 해당 디렉토리로 이동합니다 (만약 터미널의 현재 위치가 다르다면).
cd Users/your-user-name/azure-ml-labs/Labs/08/src/
Use code with caution.
Bash
(또는 현재 위치에서 상대 경로 사용)
그리고 스크립트를 실행합니다:
python train-model-parameters.py --training_data diabetes.csv
Use code with caution.
Bash
스크립트가 성공적으로 실행되고 결과적으로 학습된 모델의 정확도(accuracy)와 AUC가 출력에 표시되어야 합니다.
터미널에서 스크립트를 테스트하는 것은 스크립트가 예상대로 작동하는지 확인하는 데 이상적입니다. 코드에 문제가 있으면 터미널에 오류가 표시됩니다.
(선택 사항) 코드를 편집하여 오류를 강제로 발생시키고 터미널에서 명령을 다시 실행하여 스크립트를 실행합니다. 예를 들어, import pandas as pd 줄을 제거하고 저장한 다음 입력 매개변수와 함께 스크립트를 실행하여 오류 메시지를 검토합니다.
스크립트를 명령 작업으로 실행하기
스크립트가 작동하는 것을 알면 명령 작업으로 실행할 수 있습니다. 스크립트를 명령 작업으로 실행하면 스크립트의 모든 입력과 출력을 추적할 수 있습니다.
Labs/08/Run script as command job.ipynb Notebook을 엽니다.
파일(Files) 탐색기에서 azure-ml-labs -> Labs -> 08 폴더로 이동하여 Run script as command job.ipynb 파일을 클릭합니다.
Notebook의 모든 셀을 실행합니다.
이 Notebook은 Python SDK (azure.ai.ml)를 사용하여 train-model-parameters.py 스크립트를 Azure ML 명령 작업(Command Job) 으로 제출하는 코드를 포함합니다.
주요 구성 요소는 다음과 같습니다:
command: 실행할 스크립트와 전달할 인자 (예: python train-model-parameters.py --training_data diabetes.csv --reg_rate 0.1)
code: 스크립트가 포함된 로컬 디렉터리 경로 (예: ./src)
environment: 스크립트 실행에 필요한 Python 환경 (미리 정의된 Azure ML 환경 또는 사용자 지정 환경)
compute: 작업을 실행할 컴퓨팅 대상 (예: 이전에 만든 컴퓨팅 클러스터 이름)
display_name, experiment_name 등
Azure Machine Learning 스튜디오에서 작업(Jobs) 페이지로 이동합니다.
diabetes-train-script 작업(또는 Notebook에서 지정한 display_name)으로 이동하여 실행한 명령 작업의 개요를 탐색합니다.
코드(Code) 탭으로 이동합니다. 명령 작업의 code 매개변수 값이었던 src 폴더의 모든 내용이 여기에 복사됩니다. 명령 작업으로 실행된 학습 스크립트를 검토할 수 있습니다.
출력 + 로그(Outputs + logs) 탭으로 이동합니다.
std_log.txt 파일을 열고 내용을 탐색합니다. 이 파일의 내용은 명령의 출력입니다. 터미널에서 스크립트를 테스트했을 때와 동일한 출력이 표시되었음을 기억하십시오. 스크립트 문제로 인해 작업이 실패하면 오류 메시지가 여기에 표시됩니다.
(선택 사항) 코드를 편집하여 오류를 강제로 발생시키고 Notebook을 사용하여 명령 작업을 다시 시작합니다. 예를 들어, 스크립트에서 import pandas as pd 줄을 제거하고 스크립트를 저장합니다. 또는, 스크립트 대신 작업 구성 자체에 문제가 있을 때 오류 메시지를 탐색하기 위해 명령 작업 구성을 편집합니다.
Azure 리소스 삭제
Azure Machine Learning 탐색을 마치면 불필요한 Azure 비용을 피하기 위해 만든 리소스를 삭제해야 합니다.
Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
Azure Portal의 홈(Home) 페이지에서 리소스 그룹(Resource groups) 을 선택합니다.
rg-dp100-… (스크립트가 생성한 리소스 그룹 이름) 리소스 그룹을 선택합니다.
리소스 그룹의 개요(Overview) 페이지 상단에서 리소스 그룹 삭제(Delete resource group) 를 선택합니다.
삭제를 확인하기 위해 리소스 그룹 이름을 입력하고 삭제(Delete) 를 선택합니다.
이 실습을 통해 Notebook에서 개발한 코드를 Python 스크립트로 변환하고, 터미널에서 테스트한 후, Azure Machine Learning의 명령 작업을 사용하여 보다 체계적이고 재현 가능한 방식으로 실행하는 과정을 경험했습니다. 이는 실험 단계에서 프로덕션 단계로 나아가는 중요한 단계입니다.
