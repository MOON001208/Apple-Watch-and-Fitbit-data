# 프로젝트의 배경 및 목표

평소 전자기기에 관심이 많아 워치로 수집 된 생체 데이터를 분석해보고 싶은 열망이 있었음.

워치로 수집된 생체 데이터를 통하여 기본적인 바이오 데이터에 대한 지식을 확보하기 위해서 이 프로젝트를 시작하였다.

분석의 최종 목표는 사이트에 그들이 먼저 머신러닝을 통하여 도출해놓은 정확도만큼 내 모델의 정확도를 끌어올리는 것이다.

---

<aside>

### 활용한 데이터셋

</aside>

| 출처 | 데이터셋 | 데이터 주소 |
| --- | --- | --- |
| kaggle | apple and fitbit watch data | https://www.kaggle.com/datasets/aleespinosa/apple-watch-and-fitbit-data |

---

<aside>

# 프로젝트 진행 과정

</aside>

데이터 분석은 분석 과정의 문서 자동화를 위하여 R의 Quarto 패키지를 이용하여 markdown 형식으로 데이터 분석을 진행하였다. 자세한 분석 내용은 맨위에 링크를 참고하면 된다.

데이터 수집 후 데이터가 영어로 되어있어 한글로 바꾼 뒤 표 형식으로 만들어 정리해 둘 필요가 있어보였다. R flextable 패키지를 이용하여 간단하게 보기좋은 표 형식으로 데이터를 정리해주었다.

| **변수명** | **한국어_해석** |
| --- | --- |
| ...1 | 고유 식별자 (ID) |
| age | 나이 (연령) |
| gender | 성별 (1: 남성, 0: 여성) |
| height | 키 (cm) |
| weight | 체중 (kg) |
| steps | 걸음 수 |
| hear_rate | 심박수 |
| calories | 소모 칼로리 |
| distance | 이동 거리 (km) |
| entropy_heart | 심박수 엔트로피 (심박수의 복잡성 또는 불규칙성 지표) |
| entropy_setps | 걸음수 엔트로피 (걸음 패턴의 복잡성 또는 불규칙성 지표) |
| resting_heart | 안정 시 심박수 |
| corr_heart_steps | 심박수와 걸음수 간 상관계수 |
| norm_heart | 정규화된 심박수 |
| intensity_karvonen | Karvonen 방식으로 계산된 운동 강도 지표 |
| sd_norm_heart | 정규화된 심박수의 표준편차 |
| steps_times_distance | 걸음 수와 이동 거리의 곱 |
| device | 사용된 장치 (예: Apple Watch) |
| activity | 활동 유형 (예: Lying, Sitting, Running 등) |

데이터의 경우에는 결측값은 존재하지 않았으며 분포가 대부분 한쪽으로 치우친 경향이 있었다. 

분석의 목적은 종속변수 activity인 활동유형을 예측하는 다중분류 모형을 만들어야 한다.

그래서 총 로지스틱 회귀분석, K-Nearest-Neighbor, Random Forest 모형 3가지를 사용했다.

각 모델 당 전처리를 약간 씩 다르게 하여 분석을 진행해주었다. 자세한 내용은 밑에서 설명하겠다.

모델 분석을 시작하기 전에 활동유형에 가장 많이 영향을 미칠 것 같은 성별과 걸음수에 관하여 시각화를 진행해주었다.

## 데이터 시각화

### 성별에 따른 활동지표

![Image](https://github.com/user-attachments/assets/e09e9591-e235-4121-ab7f-b8ff869f8040)

여성의 표본이 많아서 그런지 전체적으로 여자가 운동을 더 많이 하는 것으로 나타났다.

누워있는 것이 가장 많긴하지만 운동 중에서는 그 중 7 meter 달리기가 가장 많이 나왔다.

### 나이에 따른 걸음수

![image.png](attachment:48e51c7a-1e99-4948-90f5-7bcfdd5a02e9:image.png)

대체로 젊을 수록 걸음 수가 많다. 하지만 20대 후반과 30대 중반에서 걸음수가 적은걸 확인할 수 있다. 직장인이 많기때문일까? 40대 초반 데이터는 없는 것으로 확인된다.

### 그래서 나이대별로 활동량이 다른지 확인을 해보고싶었다.

![image.png](attachment:5a021c11-d50f-4eb7-833b-3bdafa535f60:image.png)

20대와 30대의 데이터가 가장 많은 것을 확인해 볼 수 있다. 모든 연령대에서 Lying이 가장 높다. 20대 30대에서 7 METs와 5 METs의 빈도가 많이 높고 self pace walk가 가장 적다. 10대에서는 3 METs가 가장 높게 나타났다. 반면에 40대와 50대는 지표가 비슷한 것으로 확인된다. 달리는 지표가 많은 연령대인 10대~30대의 걸음수가 많은 것은 당연한 것 같다.

### 걸음수와 활동유형의 관계가 있을까?

![image.png](attachment:48c50e76-46b1-44d9-86d1-ecea28bc782a:image.png)

활동량이 많아 질수록 높은 걸음수가 있는 것을 알 수 있다.

## 예측 모델링 수립
![Image](https://github.com/user-attachments/assets/05ce3795-bf37-4bc3-aa55-108d0bcfd896)

![Image](https://github.com/user-attachments/assets/8e6314c2-2f91-4a4c-88ac-ca4ab7f8d1c6)

전체적으로 각 변수들의 분포는 한쪽으로 치우쳐져있음을 확인, 표준화 변환을 통하여 변수 안정화를 만들어서 분석할 것임.

hear_rate,norm_heart,intensity_karvonen 변수끼리 상관관계가 높은 편임 다중공선성 방지를 위해서 로지스틱 회귀분석의 경우에는 pca변환을 한 후 주성분 1개만 사용할 것임.

모든 모델에 대해서 고유 ID와 device 변수는 activity에 영향이 없으므로 분석 모델에서 제거하고 나머지 변수에 대해서 예측변수를 수립한다.

로지스틱 회귀분석, K-nn, random forest 모델 3가지를 사용하였으며 하이퍼파라미터가 필요한 모델의 경우에는 5-folds resampling 기법을 이용하여 5개의 리샘플링과 랜덤 grid search 기법으로 최적 하이퍼 파라미터를 탐색했다.

---

![Image](https://github.com/user-attachments/assets/2136be23-6cd3-4cbb-821a-7f30edbfffc8)

![Image](https://github.com/user-attachments/assets/5f4b1a82-d9ad-4ab8-b5d9-435a8b42a6b7)


### **로지스틱 회귀분석 모델 예측결과**

테스트 데이터에 대한 예측 결과 특이도를 제외한 모든 지표가 매우 낮음. 성능이 별로 좋지 않다.

### **로지스틱 회귀분석 결과**

- 전처리 : 데이터의 치우침으로 인하여 숫자형 변수에 대한 표준화 변환과 성별의 더미변수 처리, 상관관계가 높은 hear_rate,norm_heart,intensity_karvonen 들의 pca 변환 후 주성분 1개 사용

– 이 모델은 “Lying”(기준 범주)과 비교하여 나머지 다섯 가지 활동 상태(Sitting, Self Pace walk, Running 3, 5, 7 METs) 각각에 대해, 각 예측 변수가 활동 상태에 미치는 영향(로그 오즈 변화)을 계수 형태로 나타냅니다.

– 예를 들어, 연령의 영향은 활동 상태마다 다르게 나타나며, “Self Pace walk”나 “Running 5 METs”에서는 연령이 보다 큰 양의 효과를 보여 활동에 긍정적 영향을 미친다고 볼 수 있습니다.

– 성별의 계수는 음수인 경우가 많아, 남성일 때 특정 활동(예, Sitting 또는 Running 계열)보다 “Lying”이 상대적으로 더 쉽게 나타날 수 있음을 시사할 수도 있습니다.

– 걸음수와 같은 활동 지표는 높은 METs범주에서 더 큰 양의 효과를 보이며, 이는 활동의 강도가 증가할수록 해당 센서 기록의 증가가 두드러진다는 점과 일치합니다.

– 칼로리의 계수는 다른 변수들과 비교했을 때 상대적으로 높은 계수를 가지고 있으며 Sitting이 가장 낮은 계수를 지니고 있음. 확실히 활동을 하는 상태의 칼로리 계수가 높으며 이는 칼로리 소모가 활동 상태 결정에 많은 영향을 끼친다는 것임.

– PCA로 생성된 PC1도 모든 범주에서 양의 계수를 가지며, 이는 여러 변수의 주성분이 운동 강도와 양의 관계에 있음을 보여줍니다.

---

![Image](https://github.com/user-attachments/assets/4d6fa30a-520a-43cf-b2d7-255aee3faeb8)


### **K-Nearest-Neighbor 결과**

- 전처리 : 데이터 간의 거리의 개념을 이용한 만큼 데이터끼리의 scale을 동일하게 만들기 위해서 표준화 변환과 성별의 더미변수 처리.

![Image](https://github.com/user-attachments/assets/6fdfbcd3-e4a3-4de8-9bf2-9cfdb6c1f345)

knn 모델의 경우에는 이웃의 개수인 k를 하이퍼 파라미터로 지정하였으며 랜덤 grid 15(1~15)개를 생성하여 최적의 하이퍼파라미터를 찾아주었다.  그 중 accuracy 기준 k가 4일 때 성능이 가장 좋은 것으로 나왔으며 각 activity별로 roc_curve를 그려보면 이 모델은 Running 3 METs와 Running 7METs에서 좋은 성능을 보여주는 것을 볼 수 있다.

![image.png](attachment:d43b9584-8ab5-4f9e-aa10-84d04a588a37:image.png)


### **K-Nearest-Neighbor 모델 예측결과**

test데이터에 대하여 예측 결과 모든 지표가 로지스틱 회귀분석보다 좋아진 것을 확인할 수 있다.

---

![Image](https://github.com/user-attachments/assets/f6a19cbc-69aa-47c6-b55d-71d98a01d089)


![Image](https://github.com/user-attachments/assets/aeb59209-b87c-448d-8e1e-508e04bdeaca)

### **랜덤포레스트 분석결과**

- 전처리 : 아무런 조치를 하지 않아도 모델의 성능이 준수하게 나오므로 전처리 안함.

랜덤 포레스트의 경우에는 트리당 무작위 예측변수의 수인 mtry와, 각 노드 분할에 필요한 최소한의 데이터 수인 min_n를 하이퍼파라미터로 설정해준다. 모델 실행시간을 고려하여 10개의 grid만 생성하여 분석을 진행하였다.

| mtry<int> | min_n<int> | .metric<chr> | .estimator<chr> | mean<dbl> | n<int> | std_err<dbl> | .config<chr> |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 12 | 6 | accuracy | multiclass | 0.8340739 | 5 | 0.005037365 | Preprocessor1_Model03 |
| 16 | 5 | accuracy | multiclass | 0.8298154 | 5 | 0.005309401 | Preprocessor1_Model06 |
| 11 | 12 | accuracy | multiclass | 0.8172458 | 5 | 0.005510324 | Preprocessor1_Model05 |
| 14 | 15 | accuracy | multiclass | 0.8046817 | 5 | 0.004260502 | Preprocessor1_Model02 |
| 8 | 20 | accuracy | multiclass | 0.7929739 | 5 | 0.004817018 | Preprocessor1_Model01 |

정확도 기준 rand forest의 최적의 하이퍼파라미터는 mtry 12, min_n 6으로 확인되었다.

### **랜덤 포레스트 모델 예측결과**

![Image](https://github.com/user-attachments/assets/ccae2740-db85-431f-bb8c-f31513d3afc5)

![Image](https://github.com/user-attachments/assets/6fe43794-7937-46b2-83cd-34369bc64c44)


랜덤포레스트를 이용하여 test 데이터에 대한 예측 결과 다른 모델에 비해서 성능이 훨씬 좋은 것으로 판단된다.

![Image](https://github.com/user-attachments/assets/cdb63205-f1e9-44a2-a9c8-fc70f64ede1a)

마지막으로 랜덤 포레스트 모델에 대하여 각 예측변수의 영향력을 시각화 해보면 활동 상태를 예측하는 것에 가장 영향력이 큰 것은 steps 걸음수 이며 그다음 calories, distance corr_heart_steps 로 나왔다. 

# 분석 결과

로지스틱 회귀분석, K-Nearest-Neighbor, random forest 모델의 사용자의 활동 상태 예측 분석 결과 성능이 가장 좋은 것은 random forest 모형으로 나왔으며 정확도는 84.8%로 분석 목적인 데이터 세트에 나온 정확도와 근사한 수치를 도출하게 되었다.

최적 하이퍼 파라미터는 mtry = 12, min_n=6으로 나왔으며 활동 상태에 가장 영향력이 큰 변수는 걸음수와 칼로리로 도출 되었다.

---

# 프로젝트를 통해 얻은 인사이트

간단한 데이터를 통하여 바이오 데이터에 대한 머신러닝 기법을 사용할 수 있게 되었다.

다중 분류에 대한 모형 평가 지표에 대해서 공부할 수 있는 계기가 되었으며 앞으로 있을 바이오 데이터에 대한 딥러닝과 머신러닝 분석의 시작점으로 다양한 분석 기법을 적용하여 바이오 데이터 분석가로 성장 할 수 있는 계기가 될 것이다.































