# 서도형

# EDA

### 방법 1 : Feature를 분류해서 학습하면 Feature별 중요도를 알 수 있지 않을까?

Train Set에 대하여 Feature 별로 그룹 지어 → 학습한 결과 값 정리 

(validation set 에 대한 NRMSE 기록)]

[feature test.xlsx](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a3f2d6d7-8766-4dfb-acce-808b12ee76ea/feature_test.xlsx)

### EDA 결과

1. y에 대한 정상 spec 범위에 대한 Feature를 새로 만듦 (1 = 정상 센서 (91%), 0 = 비정상 (9%))
2. Validation Error 결과가 좋음
3. test set에 대한 y값이 없으니 x에 대해 정상 spec 범위를 추론하는 변수를 새로 만들어야 함

### Pandas Profiling 결과 자료

[pr_report.html](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/978c3094-ee11-4998-9f77-5acc673a35bb/pr_report.html)

[trainSet Pandas Profiling.xlsx](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/55110ef4-091a-4567-b9cc-99e6745492b7/trainSet_Pandas_Profiling.xlsx)

---

## 파생 변수 생성

- y_feature_spec_info의 spec 범위를 통과한 feature 생성

### x에 대한 정상 spec 범위 추론 기준 선정

1. 데이터의 describe 결과 값에서 25%, 75% 편차가 높은 데이터들을 대상으로 IQR을 진행한다.
2. y와 연관성이 높은 변수 값 들을 선정하여 나열한다. (기준이 되는 연관성 %는 아직 미정)

(IQR 진행 후 이상치 결과 데이터의 수가 전체 데이터의 9~10% 미만인 데이터인지 확인한다.)

위 1, 2 에 대한 모든 feature 변수에 대하여 IQR을 진행하고 이상치 데이터의 수가 10% 미만이 되는지 확인한다

### 결과

- y_spec_info에 대한 이상치 행을 제작했다.

10%에 해당되는 4천 개에 대한 이상치 데이터와 

90%의 정상 데이터로 분류했다.

- 두 데이터 셋의 min, max 차이를 기준으로 x_spec_info를 제작해 분류를 시도했다.

1400개의 이상치 데이터가  분류되었고 나머지 2600개의 데이터는 각각의 feature에 대해 정상 데이터의 분포 사이에 존재한다.

⇒ 정상, 이상 변수 분류 관련 파생 변수 추가에 대하여

이상치 데이터 분류를 위해 classification 모델을 설계 하는 것이 가장 효과적으로 결론 지었음.

### Classification 모델

**관찰 결과**

1. 데이터의 분포가 이상치 데이터 10%, 정상치 데이터 90%로 **Imbalancing 문제**가 있음
2. AUC 판정 기준 **80%** 이상의 이상치 데이터를 새로 생성해야 한다.
3. **Overfitting**이 심하게 측정 되었음

## Imbalancing & Overfitting 해결 시도

### Q. 이상 / 정상 데이터를 분리하는 중요 Feature를 찾을 수 있을까?

**실험 1 : clustering 시각화로 데이터를 분리 해보자!**

**1-1** 정상치/이상치 각 feature에 대해 kmeans 클러스터링 진행 

→ 시각적으로 분리되는 Feature를 발견함

**1-2** Clustering을 기반으로 개별 Y Feature에 대해 예측(SMOTE + Classification) 을 시도함 

**1-3** Y_06에 대한 이상치 개수 10개에 대해 AUC값이 높음, 

(Y_06 제외한 다른 y Feature에 대해서 70% 이상의 AUC를 관측하지 못함)

**실험 1 결과 ⇒ 중첩되는 데이터의 수가 많음, 해결하지 못함**

---

**실험 2 : 모든 개별 x에 대하여 y와 연관성을 관찰 할 수 있을까?**

2-1 모든 개별 x feature를 y에 대해 예측을 시도 (y1 ← x(1 ~ 56), y2 ← x(1 ~ 56), ….

2-2 이상치 비중이 높은 y1~5에 대하여 AUC값이 낮음

**실험 2 결과 ⇒ Y_06을 제외한 feature에 대해 연관성 있는 x feature를 관측하지 못함**

---

## EDA 진행 후 발견.

y_feature_spec_info 데이터를 통해 y에 대한 이상치 측정으로 

낮은 loss(nrmse 1.88)를 도출해 내었다.

1. y_feature_spec_info를 train_y 데이터에 적용 후 이상치 행을 뽑아냄
2. train_x에 이상치 행이면 1, 정상 행일 경우 0으로 파생 변수를 하나 넣어줌 (X_57 생성)
train / nrmse -> 1.88

### 결론

1. 이상치 행을 파생 변수로 잘 쓰면 대박이다.

# 실험1

이상치 행을 파생 변수로 사용할 수 있지 않을까?
모델을 두 번 사용하는 방법으로 파생 변수를 만들어 보자.

1. train 데이터 셋 => LGBM Model 학습 

=> train_x 데이터 예측 

=> 예측한 y에 대해 이상치 범주 적용 (주어진 train_y 아님!)

=> 이상치 일 경우 x에 파생 변수로 추가 

=> 파생 변수로 추가된 train_x 재 예측

### 실험 1 - 결론
train, test 데이터에 대하여 모델을 통과한 예측 값이 y_spec_info 범주를 벗어나지 않아 

파생 변수로 추가할 데이터가 없어졌다.
=> 이상치를 예측하는 모델이 아니게 되었음, 예측한 모든 데이터가 정상치로 나옴 ... fail

# 실험2

다른 방법으로 파생변수를 만들어 보려 했다.
LGBM 모델을 통과하면 y값이 이상치 범주보다 작아지는 것 같아서
train_x로 예측한 y에 대해 spec_info를 새로 만들어 보려 했다.

1. train_x, train_y => LGBM Model 생성
2. train_x + LGBM Model => 예측한 새로운 train_y 생성 (기존 train_y 아님)
예측한 train_y에 대하여 정상치 행의 max, min을 측정하여 새로운 y_spec_info를 만들었다.
(주어진 y_spec_info보다 범위가 좁아짐, ex. Min(0.2 -> 0.5), Max(1.0 -> 0.8))
3. 새로운 y_spec_info를 test에 적용하자 예측한 y값이 10개 미만으로 도출되었다.

### 실험 2 - 결론

1. test 데이터셋이 모두 정상치 이거나 이상치가 매우 적은게 아닐까?
2. test 데이터 셋의 예측치와 제공받은 spec_info 데이터는 아무 상관이 없는 데이터 인가?

# 실험3

1. 실험1의 실패를 기반으로 train 데이터 셋에서 이상치 데이터를 제외하고 실험하였다
2. test 데이터 셋이 정상 센서들로 가득하다면 정상치만 가지고 학습한 모델의 loss가 더 낮을 것 이다.

### 실험 3 -결론

1. loss가 확연히 내려갔다.(nrmse - 1.93698) 
2. 이상치를 100개만 넣어서 학습해볼까?

# 실험 4

1. 이상치를 0개, 450개, 1500개를 정상 데이터 셋에 넣은 후 학습을 진행하였다.

### 실험 4 - 결론

1. 이상치 0개 추가	nrmse	-	1.93698
2. 이상치 450개 추가	nrmse	-	1.9488691857
3. 이상치 1500개 추가	nrmse	-	1.9583680591

# 실험 5

아무래도 이상치 행이 무엇인지 맞추는 게 이 대회의 핵심인 것 같다.
이상치 행에 대한 classfication을 진행해보자

1. LGBM Classification 모델을 사용하여 대략 이상치 행 3900개, 정상 행 35000개에 대하여 classification을 시도하였다.

### 실험 5 - 결론

1. Classification 결과를 train_x에 적용하여 loss를 측정했다. -> nrmse 1.8x,
2. 제출 했더니 1.9x로 나온다. overfitting이 엄청 심함 ...fail

# 실험 6

오버 피팅을 해결하기 위해 imbancing을 진행해봤다. SMOTE 모델을 사용하여
개별 Y feature에 대해 각각 SMOTE + LGBM Classification 조합을 사용하여 AUC를 측정하였다.

### 실험 6 - 결론

1. AUC 80% 이상이 목표였는데 모든 feature에 대해 ACU 60%를 기록하였다.
2. AUC를 높이기 위해 X_feature를 선택하거나 특정 조합을 여러가지 시도해 봤다 ...fail

결국 기존 모델을 앙상블한 것으로 마무리 했다. 

시간이 더 있었다면.. 파생 변수에 성공했을까?

# 코드

### ! 코드는 구글 드라이브에 업로드 했습니다!
