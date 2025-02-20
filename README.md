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

����ġ ����

```{r}
library(flextable)
# ������ ������ ����
variable_info <- data.frame(
  ������ = c("...1", "age", "gender", "height", "weight", "steps", "hear_rate", "calories", 
           "distance", "entropy_heart", "entropy_setps", "resting_heart", "corr_heart_steps", 
           "norm_heart", "intensity_karvonen", "sd_norm_heart", "steps_times_distance", "device", "activity"),
  �ѱ���_�ؼ� = c("���� �ĺ��� (ID)", "���� (����)", "���� (1: ����, 0: ����)", "Ű (cm)", "ü�� (kg)", "���� ��", 
               "�ɹڼ�", "�Ҹ� Į�θ�", "�̵� �Ÿ� (km)", "�ɹڼ� ��Ʈ���� (�ɹڼ��� ���⼺ �Ǵ� �ұ�Ģ�� ��ǥ)", 
               "������ ��Ʈ���� (���� ������ ���⼺ �Ǵ� �ұ�Ģ�� ��ǥ)", "���� �� �ɹڼ�", "�ɹڼ��� ������ �� ������", 
               "����ȭ�� �ɹڼ�", "Karvonen ������� ���� � ���� ��ǥ", "����ȭ�� �ɹڼ��� ǥ������", 
               "���� ���� �̵� �Ÿ��� ��", "���� ��ġ (��: Apple Watch)", "Ȱ�� ���� (��: Lying, Sitting, Running ��)")
)

# flextable ����
flextable(variable_info) |> 
  autofit() |>  # �ڵ� ũ�� ����
  theme_vanilla() |>  # ������ ���� �׸� ����
  set_caption("explain variable")  # ���̺� ���� �߰�
```


```{r}
summary(aw_fb)
descr::freq(aw_fb$device, plot=F)
descr::freq(aw_fb$activity, plot=F)
skimr::skim(aw_fb)
```

���� 19��, 6264���� ������  
2���� ������ ������ �����ϰ� �������� ������ ����  
device�� ��쿡�� wapple watch�� fitbit �ΰ��� ����  
activity�� ��쿡�� 6���� Lying�� ���� ������ �������� ���� ���� �Ǿ����� �� ������ �Ƹ� Ÿ�ٺ����� �� ��  


���̴� 18~56������ ���� ���� ���� �����Ͱ� ���� ������ ����  
������ ������ ����  
Ű�� 143~191 ���� ������ ���Ժ��� ����  
�����Ե� 43~115 ���� ������ ���Ժ��� ����  
�������� 1~1714�� ���� ������ ���������� �� ������ ����  
�ɹڼ��� 2.22~194.33 ���� ������ �ɰ��ϰ� ���� ��ġ�� �̻�ġ(��Ż��)�� ���։�. ���Ժ��� ����� ��  
Į�θ��� 0.06~97.50 ���������� �� ��������   
�Ÿ��� 0~335.00 ���������� �� ���� ����  
entropy_heart, entropy_steps �� �������� �� ���� ���� ��ġ�� �����  
������ �ɹڼ��� 21.20~155.00 ���Ժ��� ����  
�ɹڼ��� �������� �������� �� �ִ��� �𸣰ڳ�  
����ȭ�� �ɹڼ��� -76~156.32�� �ɹڼ��� ������ ������  
������� -2.71~1.30 ���������� ġ������������ �ణ ���Ժ��� ����  
����ȭ�� �ɹڼ��� ǥ�������� 0~74.46  
������ * �Ÿ��� 0~51520 ���� ����.  

### ������ ���� Ȱ������
```{r}
aw_fb |> 
  mutate(gender = as.factor(gender)) |> 
  group_by(gender) |> 
  summarise(mean_weight = mean(weight),
            mean_height = mean(height),
            mean_hear_rate = mean(hear_rate))
```

ü�߰� ������ ������� �� ���� ���ڰ� ���ں��� ü�߰� Ű�� �����Ƿ� 0�� ���� 1�� ���ڶ�� ���� �غ� �� �ִ�.

```{r}
aw_fb1 <- aw_fb |> 
  mutate(gender = factor(gender, labels=c("female","male")))
descr::freq(aw_fb1$gender,plot=F)
```


#### ������ ���� Ȱ����ǥǥ

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

������ ǥ���� ���Ƽ� �׷��� ��ü������ ���ڰ� ��� �� ���� �ϴ� ������ ��Ÿ����.  
�����ִ� ���� ���� ���������� � �߿����� �� �� 7 meter �޸��Ⱑ ���� ���� ���Դ�.  

### ���̿� ���� ������
```{r}
aw_fb1 |> ggplot() +
  geom_point(aes(x=age, y=steps)) +
  theme_bw() +
  labs(title='steps by age')
```

��ü�� ���� ���� ���� ���� ����. ������ 20�� �Ĺݰ� 30�� �߹ݿ��� �������� ������ Ȯ���� �� �ִ�. �������� ���⶧���ϱ�? 40�� �ʹ� �����ʹ� ���� ������ Ȯ�εȴ�.

�׷� ���̴� ���� Ȱ������ �ٸ��ɱ�?
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

20��� 30���� �����Ͱ� ���� ���� ���� Ȯ���� �� �� �ִ�. ��� ���ɴ뿡�� Lying�� ���� ����. 20�� 30�뿡�� 7 METs�� 5 METs�� �󵵰� ���� ���� self pace walk�� ���� ����. 10�뿡���� 3 METs�� ���� ���� ��Ÿ����. �ݸ鿡 40��� 50��� ��ǥ�� ����� ������ Ȯ�εȴ�. �޸��� ��ǥ�� ���� ���ɴ��� 10��~30���� �������� ���� ���� �翬�� �� ����.

�������� Ȱ�������� ���谡 ������?
```{r}
aw_fb1 |> 
  ggplot() +
  geom_histogram(aes(x=steps, fill=activity),position='dodge') +
  facet_wrap(~activity) +
  theme_bw() +
  theme(legend.position='none') +
  labs(y=NULL, title = 'histogram of steps by activity')
```

Ȱ������ ���� ������ ���� �������� �ִ� ���� �� �� �ִ�.


### ��üȰ�� ���� �𵨸� ������ƽ, ����, knn
�м� ����: ��üȰ�� �����͸� ���� activity�� �����ϴ� ���� �м��� ��ǥ

```{r}
#| message: false
#| warning: false
GGally::ggpairs(aw_fb,mapping = aes(fill=as.factor(activity)))
GGally::ggcorr(aw_fb, label=TRUE,hjust=0.75, size=3, color='grey50')
```

��ü������ �� �������� ������ �������� ġ����������. ǥ��ȭ ��ȯ�� ���Ͽ� ���� ����ȭ�� ���� �м��� ����.  
hear_rate,norm_heart,intensity_karvonen �������� ������谡 ���� ���� ���߰����� ������ ���ؼ� ������ƽ ȸ�ͺм��� ��쿡�� pca��ȯ�� �� �� �ּ��� 1���� ����� ����.



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

���� �����Ϳ� ���ؼ� train data 70% test data 30%, ���Ӻ����� activity ���� ��ȭ������ ���־���.


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

���Ӻ����� activity�� ���ְ� 6���̹Ƿ� ���ߺз� ���� �������Ѵ�. ���� ������ƽ ȸ�ͺм��� ��쿡�� nnet ��Ű���� ����� ���̴�.  
K Nearest Neighbor�� ��쿡�� ������ �̿������� ã�� ���ؼ� �������Ķ���ͷ� �������ش�.  
rand forest ���� ��쿡�� �� Ʈ���� ������ ���������� ���� mtry��, �� ��� ���ҿ� �ʿ��� �ּ����� ������ ���� min_n�� �������Ķ���ͷ� �������ش�.



#resampling
```{r}
set.seed(123)
vfold <- vfold_cv(train, v=5, strata='activity')
control <- control_resamples(save_pred=T,
                             save_workflow=T,
                             event_level = 'first')
```

������ �������Ķ���͸� ã�� ���ؼ� 5���� �����ø��� ���ش�.


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

��� �𵨿� ���ؼ� ...1, device�� ������ ��� �������� ����� ����  
������ƽ ȸ�ͺм��� ��쿡�� ǥ��ȭ ��ȯ�� pca��ȯ  
knn ���� ���ؼ��� ǥ��ȭ ��ȯ�� factor������ numeric��ȯ  
rand forest ���� ���ؼ��� �ƹ��� ��ó���� ���� �ʰ� �м� �����Ѵ�.

#### logistic workflow and fit
```{r}
multinom_wflow <- workflow() |> add_model(multinom_reg_spec) |> add_recipe(rec)
multinom_fit <- fit(multinom_wflow, train)
extract_fit_engine(multinom_fit)

```

���Ӻ����� 6���� Lying�� ���غ����� �Ͽ� ������ƽ ȸ�ͺм��� ������ ����̴�. 

??? �� ���� ��Lying��(���� ����)�� ���Ͽ� ������ �ټ� ���� Ȱ�� ����(Sitting, Self Pace walk, Running 3, 5, 7 METs) ������ ����, �� ���� ����(����, ����, ��ü����ġ, ���� ��ϰ� ��)�� Ȱ�� ���¿� ��ġ�� ����(�α� ���� ��ȭ)�� ��� ���·� ��Ÿ���ϴ�.  
??? ���� ���, ������ ������ Ȱ�� ���¸��� �ٸ��� ��Ÿ����, ��Self Pace walk���� ��Running 5 METs�������� ������ ���� ū ���� ȿ���� ���� Ȱ���� ������ ������ ��ģ�ٰ� �� �� �ֽ��ϴ�.  
??? ����(gendermale)�� ����� ������ ��찡 ����, ������ �� Ư�� Ȱ��(��, Sitting �Ǵ� Running �迭)���� ��Lying���� ��������� �� ���� ��Ÿ�� �� ������ �û��� ���� �ֽ��ϴ�.  
??? ������(steps)�� ���� Ȱ�� ��ǥ�� ���� METs(� ����) ���ֿ��� �� ū ���� ȿ���� ���̸�, �̴� Ȱ���� ������ �����Ҽ��� �ش� ���� ����� ������ �ε巯���ٴ� ���� ��ġ�մϴ�.  
- Į�θ�(calories)�� ����� �ٸ� ������� ������ �� ��������� ���� ����� ������ ������ Sitting�� ���� ���� ����� ���ϰ� ����. Ȯ���� Ȱ���� �ϴ� ������ Į�θ� ����� ������ �̴� Į�θ� �Ҹ� Ȱ�� ���� ������ ���� ������ ��ģ�ٴ� ����.  
??? PCA�� ������ PC1�� ��� ���ֿ��� ���� ����� ������, �̴� ���� ������ �ּ����� � ������ ���� ���迡 ������ �����ݴϴ�.

#### logistic regression predict
```{r}
multinom_pred <- augment(multinom_fit, test)
wear_metric <- metric_set(accuracy, sens,spec,roc_auc,brier_class)
multinom_pred |> wear_metric(activity,estimate=.pred_class, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`, event_level = 'first')
multinom_pred |> conf_mat(truth = activity, estimate = .pred_class)
```

�׽�Ʈ �����Ϳ� ���� ���� ��� Ư�̵��� ������ ��� ��ǥ�� �ſ� ����. ������ ���� ���� �ʴ�.

```{r}
roc_curve(multinom_pred, activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
```

#### knn tune
```{r}
knn_wflow <- workflow() |> add_model(kknn_spec) |> add_recipe(knn_rec)
set.seed(234)
tuning <- tune_grid(knn_wflow, resamples = vfold, grid=15,control=control)
```

knn ���� ��쿡�� ���� grid 15���� �����Ͽ� ������ �������Ķ���͸� ã���ְڴ�.

```{r}
autoplot(tuning)
show_best(tuning,metric = 'accuracy')
best_knn_wflow <- workflow() |> add_model(nearest_neighbor(neighbors = 4) %>%
  set_engine('kknn') %>%
  set_mode('classification')) |> add_recipe(knn_rec)

```

tune_grid�� ���� 15���� �������Ķ������ ���� �򰡸� ���غ��� k=4 ���ؿ��� ���� �پ ��ǥ�� �����ִ� ������ ���δ�. 

```{r}
collect_predictions(tuning) |> group_by(id) |> roc_curve(activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
```

�� activity���� roc_curve�� �׷����� �� ���� Running 3 METs�� Running 7METs���� ���� ������ �����ִ� ���� �� �� �ִ�.

#### knn fit and predict
```{r}
knn_fit <- fit(best_knn_wflow, train)
knn_fit |> extract_fit_engine()
augment(knn_fit, test) |> wear_metric(activity,estimate=.pred_class, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`, event_level = 'first')
augment(knn_fit, test) |> conf_mat(truth = activity, estimate = .pred_class)
```

������ƽ ȸ�ͺм����� ������ Ȯ���� ������ ���� Ȯ�� �� �� �ִ�. 



#### random forest tune
```{r}
rand_wflow <- workflow() |> add_model(rand_forest_spec) |> add_recipe(rand_rec)
set.seed(234)
rand_tuning <- tune_grid(rand_wflow, vfold, grid=10, control=control)
```

����������Ʈ�� ��쿡�� 10�� ���� grid�� ����� ���������� �Ķ���͸� ã�ƺ� ���̴�.


```{r}
autoplot(rand_tuning)
show_best(rand_tuning, metric='accuracy')
collect_predictions(rand_tuning) |> group_by(id) |> roc_curve(activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
best_rand_param <- select_best(rand_tuning,metric = 'accuracy')
best_rand_wflow <- finalize_workflow(rand_wflow, best_rand_param)
best_rand_wflow
```

��Ȯ�� ���� rand forest�� ������ �������Ķ���ʹ� mtry 12, min_n 6���� Ȯ�εǾ���.

#### rand forest fit and predict
```{r}
rand_fit <- last_fit(best_rand_wflow, split)
collect_predictions(rand_fit) |> wear_metric(activity,estimate=.pred_class, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`, event_level = 'first')
collect_predictions(rand_fit)  |> conf_mat(truth = activity, estimate = .pred_class)
collect_predictions(rand_fit)|> roc_curve(activity, .pred_Lying, .pred_Sitting,`.pred_Self Pace walk`,`.pred_Running 3 METs`,`.pred_Running 5 METs`,`.pred_Running 7 METs`) |> autoplot()
```

����������Ʈ�� �̿��Ͽ� test �����Ϳ� ���� ���� ��� �ٸ� �𵨿� ���ؼ� ������ �ξ� ���� ������ �Ǵܵȴ�.

```{r}
vip::vip(extract_fit_parsnip(rand_fit))
```

Ȱ�� ���� �Ǵܿ� ���� ������ ū ������ ������, Į�θ���� ���� �� �� �ִ�.