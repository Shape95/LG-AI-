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
