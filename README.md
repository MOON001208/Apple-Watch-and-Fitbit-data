# Apple-Watch-and-Fitbit-data
kaggle apple watch, fitbit data analysis

https://www.kaggle.com/datasets/aleespinosa/apple-watch-and-fitbit-data


```{r}
#| message: false
#| warning: false
library(tidyverse)
aw_fb <- read_csv('aw_fb_data.csv') |> select(-2)
colSums(is.na(aw_fb))
head(aw_fb)
```

결측치 없음

```{r}
library(flextable)
# 데이터 프레임 생성
variable_info <- data.frame(
  변수명 = c("...1", "age", "gender", "height", "weight", "steps", "hear_rate", "calories", 
           "distance", "entropy_heart", "entropy_setps", "resting_heart", "corr_heart_steps", 
           "norm_heart", "intensity_karvonen", "sd_norm_heart", "steps_times_distance", "device", "activity"),
  한국어_해석 = c("고유 식별자 (ID)", "나이 (연령)", "성별 (1: 남성, 0: 여성)", "키 (cm)", "체중 (kg)", "걸음 수", 
               "심박수", "소모 칼로리", "이동 거리 (km)", "심박수 엔트로피 (심박수의 복잡성 또는 불규칙성 지표)", 
               "걸음수 엔트로피 (걸음 패턴의 복잡성 또는 불규칙성 지표)", "안정 시 심박수", "심박수와 걸음수 간 상관계수", 
               "정규화된 심박수", "Karvonen 방식으로 계산된 운동 강도 지표", "정규화된 심박수의 표준편차", 
               "걸음 수와 이동 거리의 곱", "사용된 장치 (예: Apple Watch)", "활동 유형 (예: Lying, Sitting, Running 등)")
)

# flextable 적용
flextable(variable_info) |> 
  autofit() |>  # 자동 크기 조정
  theme_vanilla() |>  # 가독성 좋은 테마 적용
  set_caption("explain variable")  # 테이블 제목 추가
```


```{r}
summary(aw_fb)
descr::freq(aw_fb$device, plot=F)
descr::freq(aw_fb$activity, plot=F)
skimr::skim(aw_fb)
```

변수 19개, 6264개의 데이터  
2개의 범주형 변수가 존재하고 나머지는 숫자형 변수  
device의 경우에는 wapple watch와 fitbit 두가지 존재  
activity의 경우에는 6가지 Lying이 가장 많으며 나머지는 고르게 분포 되어있음 이 변수가 아마 타겟변수가 될 듯  


나이는 18~56세까지 분포 젊은 층의 데이터가 많은 것으로 보임  
성별의 비율은 동일  
키는 143~191 까지 있으며 정규분포 형태  
몸무게도 43~115 까지 있으며 정규분포 형태  
걸음수는 1~1714보 까지 있으며 오른쪽으로 긴 꼬리의 형태  
심박수는 2.22~194.33 까지 있으며 심각하게 낮은 수치는 이상치(오탈자)로 간주됌. 정규분포 모양을 띔  
칼로리는 0.06~97.50 오른쪽으로 긴 꼬리형태   
거리는 0~335.00 오른쪽으로 긴 꼬리 형태  
entropy_heart, entropy_steps 는 왼쪽으로 긴 꼬리 형태 수치도 비슷함  
안정시 심박수는 21.20~155.00 정규분포 형태  
심박수와 걸음수의 상관관계는 왜 있는지 모르겠네  
정규화된 심박수는 -76~156.32로 심박수와 분포가 유사함  
운동강도는 -2.71~1.30 오른쪽으로 치우쳐져있지만 약간 정규분포 형태  
정규화된 심박수의 표준편차는 0~74.46  
걸음수 * 거리는 0~51520 으로 있음.  

### 성별에 따른 활동유형
```{r}
aw_fb |> 
  mutate(gender = as.factor(gender)) |> 
  group_by(gender) |> 
  summarise(mean_weight = mean(weight),
            mean_height = mean(height),
            mean_hear_rate = mean(hear_rate))
```

체중과 신장을 고려했을 때 보통 여자가 남자보다 체중과 키가 작으므로 0은 여자 1은 남자라고 추측 해볼 수 있다.

```{r}
aw_fb1 <- aw_fb |> 
  mutate(gender = factor(gender, labels=c("female","male")))
descr::freq(aw_fb1$gender,plot=F)
```


#### 성별에 따른 활동지표표

```{r}
aw_fb1 |> 
  group_by(gender,activity) |> 
  summarise(count=n(),.groups='drop') |> 
  ggplot(aes(x=activity,y=count,fill=gender)) +
  geom_bar(stat='identity',position='dodge') +
  theme_bw() +
  labs(title='activity by gender',y=NULL)

aw_fb1 |> 
  group_by(gender) |> 
  summarise(count=n())
```

여성의 표본이 많아서 그런지 전체적으로 여자가 운동을 더 많이 하는 것으로 나타났다.  
누워있는 것이 가장 많긴하지만 운동 중에서는 그 중 7 meter 달리기가 가장 많이 나왔다.  

### 나이에 따른 걸음수
```{r}
aw_fb1 |> ggplot() +
  geom_point(aes(x=age, y=steps)) +
  theme_bw() +
  labs(title='steps by age')
```

대체로 젊을 수록 걸음 수가 많다. 하지만 20대 후반과 30대 중반에서 걸음수가 적은걸 확인할 수 있다. 직장인이 많기때문일까? 40대 초반 데이터는 없는 것으로 확인된다.

그럼 나이대 별로 활동량이 다른걸까?
```{r}
aw_fb1 |> mutate(age = if_else(age <20, '10~19',
                               if_else(age<30, '20~29',
                                       if_else(age<40, '30~39',
                                               if_else(age<50, '40~49','50~59'))))) |> 
  ggplot() +
  geom_bar(aes(x=age, fill=activity),position='dodge') +
  theme_bw() +
  labs(y=NULL)
```

20대와 30대의 데이터가 가장 많은 것을 확인해 볼 수 있다. 모든 연령대에서 Lying이 가장 높다. 20대 30대에서 7 METs와 5 METs의 빈도가 많이 높고 self pace walk가 가장 적다. 10대에서는 3 METs가 가장 높게 나타났다. 반면에 40대와 50대는 지표가 비슷한 것으로 확인된다. 달리는 지표가 많은 연령대인 10대~30대의 걸음수가 많은 것은 당연한 것 같다.

걸음수와 활동유형의 관계가 있을까?
```{r}
aw_fb1 |> 
  ggplot() +
  geom_histogram(aes(x=steps, fill=activity),position='dodge') +
  facet_wrap(~activity) +
  theme_bw() +
  theme(legend.position='none') +
  labs(y=NULL, title = 'histogram of steps by activity')
```

활동량이 많아 질수록 높은 걸음수가 있는 것을 알 수 있다.


### 생체활동 예측 모델링 로지스틱, 랜포, knn
분석 목적: 생체활동 데이터를 토대로 activity를 예측하는 것이 분석의 목표

```{r}
#| message: false
#| warning: false
GGally::ggpairs(aw_fb,mapping = aes(fill=as.factor(activity)))
GGally::ggcorr(aw_fb, label=TRUE,hjust=0.75, size=3, color='grey50')
```

전체적으로 각 변수들의 분포는 한쪽으로 치우쳐져있음. 표준화 변환을 통하여 변수 안정화를 만들어서 분석할 것임.  
hear_rate,norm_heart,intensity_karvonen 변수끼리 상관관계가 높은 편임 다중공선성 방지를 위해서 로지스틱 회귀분석의 경우에는 pca변환을 한 후 주성분 1개만 사용할 것임.



#### split data
```{r}
aw_fb1 <- aw_fb1 |> mutate(activity = as_factor(activity))
split <- initial_split(aw_fb1, strata = 'activity')
train <- training(split)
test <- testing(split)


aw_fb1|> group_by(activity) |> summarise(n()/nrow(aw_fb1))
train |> group_by(activity) |> summarise(n()/nrow(train))
test |> group_by(activity) |> summarise(n()/nrow(test))


```

원본 데이터에 대해서 train data 70% test data 30%, 종속변수인 activity 기준 층화추출을 해주었다.


#### parsnip
```{r}
library(tidymodels)

multinom_reg_spec <-
  multinom_reg() %>%
  set_engine('nnet')

kknn_spec <-
  nearest_neighbor(neighbors = tune()) %>%
  set_engine('kknn') %>%
  set_mode('classification')

rand_forest_spec <-
  rand_forest(mtry = tune(), min_n = tune()) %>%
  set_engine('ranger', importance='permutation') %>%
  set_mode('classification')

```

종속변수인 activity의 범주가 6개이므로 다중분류 모델을 만들어야한다. 따라서 로지스틱 회귀분석의 경우에는 nnet 패키지를 사용할 것이다.  
K Nearest Neighbor의 경우에는 최적의 이웃개수를 찾기 위해서 하이퍼파라미터로 설정해준다.  
rand forest 모델의 경우에는 각 트리당 무작위 예측변수의 수인 mtry와, 각 노드 분할에 필요한 최소한의 데이터 수인 min_n를 하이퍼파라미터로 설정해준다.



#resampling
```{r}
set.seed(123)
vfold <- vfold_cv(train, v=5, strata='activity')
control <- control_resamples(save_pred=T,
                             save_workflow=T,
                             event_level = 'first')
```

최적의 하이퍼파라미터를 찾기 위해서 5개의 리샘플링을 해준다.


#### recipe
```{r}
rec <- recipe(activity~., data= train) |> 
  update_role(`...1`,device, new_role='id') |>
  step_normalize(all_numeric_predictors()) |> 
  step_pca(hear_rate,norm_heart,intensity_karvonen, num_comp = 1)

knn_rec <- recipe(activity~., data=train) |> 
  update_role(`...1`,device, new_role='id') |>
  step_normalize(all_numeric_predictors()) |> 
  step_mutate(gender = as.numeric(gender))

rand_rec <- recipe(activity~., data=train) |> 
  update_role(`...1`,device, new_role='id')

prep(rec) |> bake(train) |> summary()
prep(knn_rec) |> bake(train) |> summary()



```

모든 모델에 대해서 ...1, device를 제외한 모든 설명변수를 사용할 것임  
로지스틱 회귀분석의 경우에는 표준화 변환과 pca변환  
knn 모델의 대해서는 표준화 변환과 factor변수의 numeric변환  
rand forest 모델의 대해서는 아무런 전처리를 하지 않고 분석 진행한다.

#### logistic workflow and fit
```{r}
multinom_wflow <- workflow() |> add_model(multinom_reg_spec) |> add_recipe(rec)
multinom_fit <- fit(multinom_wflow, train)
extract_fit_engine(multinom_fit)

```

종속변수가 6개로 Lying을 기준변수로 하여 로지스틱 회귀분석을 진행한 결과이다. 

??? 이 모델은 “Lying”(기준 범주)과 비교하여 나머지 다섯 가지 활동 상태(Sitting, Self Pace walk, Running 3, 5, 7 METs) 각각에 대해, 각 예측 변수(연령, 성별, 신체계측치, 센서 기록값 등)가 활동 상태에 미치는 영향(로그 오즈 변화)을 계수 형태로 나타냅니다.  
??? 예를 들어, 연령의 영향은 활동 상태마다 다르게 나타나며, “Self Pace walk”나 “Running 5 METs”에서는 연령이 보다 큰 양의 효과를 보여 활동에 긍정적 영향을 미친다고 볼 수 있습니다.  
??? 성별(gendermale)의 계수는 음수인 경우가 많아, 남성일 때 특정 활동(예, Sitting 또는 Running 계열)보다 “Lying”이 상대적으로 더 쉽게 나타날 수 있음을 시사할 수도 있습니다.  
??? 걸음수(steps)와 같은 활동 지표는 높은 METs(운동 강도) 범주에서 더 큰 양의 효과를 보이며, 이는 활동의 강도가 증가할수록 해당 센서 기록의 증가가 두드러진다는 점과 일치합니다.  
- 칼로리(calories)의 계수는 다른 변수들과 비교했을 때 상대적으로 높은 계수를 가지고 있으며 Sitting이 가장 낮은 계수를 지니고 있음. 확실히 활동을 하는 상태의 칼로리 계수가 높으며 이는 칼로리 소모가 활동 상태 결정에 많은 영향을 끼친다는 것임.  
??? PCA로 생성된 PC1도 모든 범주에서 양의 계수를 가지며, 이는 여러 변수의 주성분이 운동 강도와 양의 관계에 있음을 보여줍니다.

#### logistic regression predict
```{r}
multinom_pred <- augment(multinom_fit, test)
wear_metric <- metric_set(accuracy, sens,spec,roc_auc,brier_class)
multinom_pred |> wear_metric(activity,estimate=.pred_class, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`, event_level = 'first')
multinom_pred |> conf_mat(truth = activity, estimate = .pred_class)
```

테스트 데이터에 대한 예측 결과 특이도를 제외한 모든 지표가 매우 낮음. 성능이 별로 좋지 않다.

```{r}
roc_curve(multinom_pred, activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
```

#### knn tune
```{r}
knn_wflow <- workflow() |> add_model(kknn_spec) |> add_recipe(knn_rec)
set.seed(234)
tuning <- tune_grid(knn_wflow, resamples = vfold, grid=15,control=control)
```

knn 모델의 경우에는 랜덤 grid 15개를 생성하여 최적의 하이퍼파라미터를 찾아주겠다.

```{r}
autoplot(tuning)
show_best(tuning,metric = 'accuracy')
best_knn_wflow <- workflow() |> add_model(nearest_neighbor(neighbors = 4) %>%
  set_engine('kknn') %>%
  set_mode('classification')) |> add_recipe(knn_rec)

```

tune_grid를 통한 15개의 하이퍼파라미터의 성능 평가를 비교해보면 k=4 기준에서 가장 뛰어난 지표를 보여주는 것으로 보인다. 

```{r}
collect_predictions(tuning) |> group_by(id) |> roc_curve(activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
```

각 activity별로 roc_curve를 그려보면 이 모델은 Running 3 METs와 Running 7METs에서 좋은 성능을 보여주는 것을 볼 수 있다.

#### knn fit and predict
```{r}
knn_fit <- fit(best_knn_wflow, train)
knn_fit |> extract_fit_engine()
augment(knn_fit, test) |> wear_metric(activity,estimate=.pred_class, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`, event_level = 'first')
augment(knn_fit, test) |> conf_mat(truth = activity, estimate = .pred_class)
```

로지스틱 회귀분석보다 성능이 확실히 좋아진 것을 확인 할 수 있다. 



#### random forest tune
```{r}
rand_wflow <- workflow() |> add_model(rand_forest_spec) |> add_recipe(rand_rec)
set.seed(234)
rand_tuning <- tune_grid(rand_wflow, vfold, grid=10, control=control)
```

랜덤포레스트의 경우에는 10개 랜덤 grid를 만들어 최적하이퍼 파라미터를 찾아볼 것이다.


```{r}
autoplot(rand_tuning)
show_best(rand_tuning, metric='accuracy')
collect_predictions(rand_tuning) |> group_by(id) |> roc_curve(activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
best_rand_param <- select_best(rand_tuning,metric = 'accuracy')
best_rand_wflow <- finalize_workflow(rand_wflow, best_rand_param)
best_rand_wflow
```

정확도 기준 rand forest의 최적의 하이퍼파라미터는 mtry 12, min_n 6으로 확인되었다.

#### rand forest fit and predict
```{r}
rand_fit <- last_fit(best_rand_wflow, split)
collect_predictions(rand_fit) |> wear_metric(activity,estimate=.pred_class, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`, event_level = 'first')
collect_predictions(rand_fit)  |> conf_mat(truth = activity, estimate = .pred_class)
collect_predictions(rand_fit)|> roc_curve(activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
```

랜덤포레스트를 이용하여 test 데이터에 대한 예측 결과 다른 모델에 비해서 성능이 훨씬 좋은 것으로 판단된다.

```{r}
vip::vip(extract_fit_parsnip(rand_fit))
```

활동 상태 판단에 가장 영향이 큰 변수는 걸음수, 칼로리라는 것을 알 수 있다.