![image](https://user-images.githubusercontent.com/62497897/187019730-030824a4-176c-4a2c-ad3c-d29f495e3d81.png)
<br>
## public 23위, private 34위
https://dacon.io/competitions/official/235927/overview/description   
   
<br><br>

## 데이터 탐구 후 발견.

y_feature_spec_info 데이터를 통해 y에 대한 불량 센서 데이터 행 측정으로 

낮은 loss(nrmse 1.88)를 도출해 내었다.

1. y_feature_spec_info를 train_y 데이터에 적용 후 불량 센서 데이터 행을 뽑아냄
2. train_x에 불량 센서 데이터 행이면 1, 정상 행일 경우 0으로 파생 변수를 하나 넣어줌 (X_57 생성)
train / nrmse -> 1.88

### 결과

1. 불량 센서 데이터 행을 파생 변수로 잘 쓰면 대박이다.
<br>

# 실험1

불량 센서 데이터 행을 파생 변수로 사용할 수 있지 않을까? (정상 센서 : 1, 불량 센서 0, Categorical Variable)
모델을 두 번 사용하는 방법으로 파생 변수를 만들어 보자.

1. train 데이터 셋 => LGBM Model 학습 

=> train_x 데이터 예측 

=> 예측한 y에 대해 불량 센서 범주 적용 (주어진 train_y 아님!)

=> 불량 센서일 경우 x에 파생 변수로 추가 

=> 파생 변수로 추가된 train_x 재 예측

### 결과
train, test 데이터에 대하여 모델을 통과한 예측 값이 
y_spec_info 범주를 벗어나지 않아 파생 변수로 추가할 데이터가 없어졌다.
=> 불량 센서를 예측하는 모델이 아니게 되었음, 예측한 모든 데이터가 정상 센서로 나옴 ... fail

<br>

# 실험2

다른 방법으로 파생변수를 만들어 보려 했다.
LGBM 모델을 통과하면 y값이 불량 센서가 되는 범주보다 작아지는 것 같아서
train_x로 예측한 y에 대해 spec_info를 새로 만들어 보려 했다.

1. train_x, train_y => LGBM Model 생성
2. train_x + LGBM Model => 예측한 새로운 train_y 생성 (기존 train_y 아님)
예측한 train_y에 대하여 정상치 행의 max, min을 측정하여 새로운 y_spec_info를 만들었다.
(주어진 y_spec_info보다 범위가 좁아짐, ex. Min(0.2 -> 0.5), Max(1.0 -> 0.8))
3. 새로운 y_spec_info를 test에 적용하자 예측한 y값이 10개 미만으로 도출되었다.

### 결과

1. test 데이터셋이 모두 정상치 이거나 불량 센서 데이터 수가 매우 적은게 아닐까?
2. test 데이터 셋의 예측치와 제공받은 spec_info 데이터는 아무 상관이 없는 데이터 인가?

<br>

# 실험3

1. 실험2의 실패를 기반으로 train 데이터 셋에서 불량 센서 데이터를 제외하고 실험하였다
2. test 데이터 셋이 정상 센서들로 가득하다면 정상치만 가지고 학습한 모델의 loss가 더 낮을 것 이다.

### 결과

1. loss가 확연히 내려갔다.(nrmse - 1.93698) 
2. 불량 센서 데이터를 100개만 넣어서 학습해볼까?

<br>

# 실험 4

1. 불량 센서 데이터를 0개, 450개, 1500개를 정상 데이터 셋에 넣은 후 학습을 진행하였다.

### 결과

1. 불량 센서 데이터 0개 추가	nrmse	-	1.93698
2. 불량 센서 데이터 450개 추가	nrmse	-	1.9488691857
3. 불량 센서 데이터 1500개 추가	nrmse	-	1.9583680591

<br>

# 실험 5

아무래도 불량 센서 데이터가 무엇인지 맞추는 게 이 대회의 핵심인 것 같다.
불량 센서 데이터 행에 대한 classfication을 진행해보자

1. LGBM Classification 모델을 사용하여 대략 불량 센서  행 3900개, 정상 행 35000개에 대하여 classification을 시도하였다.

### 결과

1. Classification 결과를 train_x에 적용하여 loss를 측정했다. -> nrmse 1.8x,
2. 그대로 대회에 제출 했더니 1.9x로 나온다. overfitting이 엄청 심함 ...fail

<br>

# 실험 6

오버 피팅을 해결하기 위해 정상 데이터와 이상 데이터의 데이터 imbalancing을 해결하고자 했다. SMOTE 모델을 사용하여
개별 Y feature에 대해 각각 SMOTE + LGBM Classification 조합을 사용하여 AUC를 측정하였다.

### 결과

1. AUC 80% 이상이 목표였는데 모든 feature에 대해 AUC 60%를 기록하였다.
2. AUC를 높이기 위해 X_feature를 선택하거나 특정 조합을 여러가지 시도해 봤다 ...fail

<br>

결국 기존 모델을 스태킹 앙상블한 것으로 마무리 했다. <br>
대회 1,2,3등이 전부 정상, 이상치 센서 분류로 잘못 모델링했다가 data leakage로 탈락한 레전드 대회
<br>
덕분에 실제 등수는 훨씬 낮을지도 모른다.
