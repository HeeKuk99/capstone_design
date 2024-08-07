# capstone_design
프로젝트 주제 : 유방암 HRA(Health Risk Appraisal) 모델의 구축 및 서비스 앱 구현
프로젝트 기간 : 2024.3 - 2024.6

프로젝트 목적 : 유방암은 여성들 사이에서 가장 흔한 종양성 질환 중 하나로, 예방과 함께 조기 치료의 중요성이 강조되고 있다. 본 프로젝트에서는 국내 데이터를 활용하여 유방암 위험도 예측 모델의 구축과 서비스 어플리케이션 프로토타입을 개발하였다.

활용 데이터 : 국립 암센터의 가상 데이터와 직접 수집한 대조군 데이터를 전처리

프로젝트 내용 : 랜덤 포레스트와 그래디언트 부스팅 모델을 활용하여 임계값 0.01 과 0.05를 기준으로 두 개의 위험 요인 집합을 선택하였다. 그 후, 선정된 요인을 기반으로 유방암 위험도 예측을 위해 랜덤 포레스트,그래디언트 부스팅, 인공신경망, 로지스틱 회귀 네 가지 모델을 학습시켰다. 모델의 변수 중요도를 고려하여 각각의 인스턴스에 대한 예측 확률을 계산하고, 최종적으로 유방암 위험 점수를 산출하였다. 실험 결과, 임계값 0.01 이상의 모델들은 그래디언트 부스팅이 높은 Accuracy, Precision, F1 Score, ROC AUC를 보였으며, 임계값 0.05 이상의 모델들에 대해서는 랜덤 포레스트가 우수한 결과를 나타냈다. 마지막으로, 최종 선정된 모델을 기반으로 국내 이용자의 편의를 고려하여 한국어 유방 건강 어플 프로토타입을 개발하였다. 

## 데이터 수집
 유방암 환자군 데이터로는 국립암센터의 유방암 환자군의 Fake Data를, 대조군 데이터는 자체 설문조사를 진행하여 확보한 151명의 데이터를 선정하였다.
A. 환자군 데이터
 Fake Data는 국립 암센터에서 제공하는 암라이브러리 데이터 체험을 위한 가상 데이터이다. 국립암센터는 원천 데이터 사용의 제약성으로 인해 데이터 개방 및 활용이 어렵기 때문에, 실제 데이터에 기반해 만들어낸 가상 데이터로 인공지능의 개발 활성화를 지향한다고 서술한 바 있다.
B. 대조군 데이터
 설문지는 유방암 발생 위험요인에 관한 문항과 개인 특성에 관한 문항으로 이루어져 있으며 본 프로젝트 팀원들이 작성하였다. 나이, 키 등 개인 특성 6문항, 인식 조사 11문항, 초경 연령, 출산 여부 등 위험요인 18 문항등을 포함하여 전체 37 문항으로 구성되어있다. 해당 설문은 14일동안 내국인을 대상으로 다른 제약조건 없이 인터넷 설문을 통해 진행되었고, 그 결과 151개의 응답을 수집할 수 있었다.
 
## 데이터 전처리
 최종 데이터는 다음과 같은 방법으로 구성되었다.
A. 환자군 데이터 구성 
 제공받은 Fake Data내에서 입원일자, 유방암 수술경력등 유방암을 진단 받은 이후의 정보와 관련된 칼럼들을 
제외하였으며, 대조군 데이터를 수집할 때 ‘호르몬 치료를 받은 적 있냐’에 대한 응답을 받은 것을 기반으로, 환자군 데이터에도 호르몬, 방사선 치료 ‘여부’에 대한 칼럼을 임의로 추가하였다.  데이터 완전성을 위해 환자군 데이터 내 결측치가 다수인 행을 제거했고, 데이터의 고유성을 위하여 중복된 행 역시 제거했다.
B.  대조군 데이터 구성 
 설문조사를 통해 얻은 데이터들을 TGAN 모델에 학습시켜 12000개의 데이터를 만들었다. 이 때, 설문조사 
응답이 극소수로 존재하는 폐경, 암 유경험 등의 데이터는 학습이 불가해 제외하고 학습하였다. 최종적으로 증강과정 데이터에 포함된 칼럼은 나이, 체중, 흡연여부 등 14 개 칼럼이다.
 이후, 환자군 데이터와 대조하여 없는 칼럼을 생성하고 환자군 데이터에 합친 뒤, K-최근접 이웃법을 이용하여 호르몬 치료 기간, 개인 병력 여부 등 대조군 데이터에 존재하지 않는 칼럼의 데이터들을 대체했다. 

 이로부터 TARGET 여부 미포함 총 65개 칼럼의 23807개 데이터로 최종 데이터를 구성하였다.

## 유방암 위험요인분류
  최종 데이터를 Random Forest(RF), Gradient Boosting(GB) 모델에 각각 학습시켰다. 이 때, 학습률, 트리의 최대 깊이 등 모델의 하이퍼 파라미터를 실험적으로 조정하며 모델의 학습 정확도를 높이고자 하였다.
  모델 학습을 위한 train set과 test set을 나눌 때, 각 칼럽에 대해서 나이, BMI 등의 분포를 동일하게 구성하기 위해서 StratifiedKFold 교차검증을 수행하여 클래스의 분포를 유지했다.
  학습 결과로 각 모델에 의해 선정되 요인을 합친 뒤, 임계값 0.01/0.05를 기준으로 두개의 위험요인집합을 만들었다. 임계값 0.05를 기준으로 한 위험 요인은 흡연량, 흡연시작연령, 흡연기간 등을 포함한 10개 칼럼이며, 임계값 0.01을 기준으로 한 위험 요인은 첫 출산 연령, HRT 사용 여부 등을 포함한 19개 칼럼이다.
