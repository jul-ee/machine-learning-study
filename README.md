# 📋 XGBoost & LightGBM 하이퍼파라미터 튜닝 실험

캐글 [Santander Customer Satisfaction](https://www.kaggle.com/competitions/santander-customer-satisfaction/data) 데이터를 기반으로

하이퍼 파라미터 튜닝을 통해 XGBoost, LightGBM 모델의 성능을 개선하고, 최종적으로 ROC AUC 성능이 가장 높은 하이퍼 파라미터 조합을 도출하는 것을 목표로 진행하였습니다.

> 실험 환경: &nbsp;Python, Scikit-learn, XGBoost, LightGBM, Jupyter Notebook

> 성능 기준: &nbsp;ROC AUC
> 

<br>

#### 0. Baseline 성능 확인

```python
XGBClassifier(n_estimators=500, learning_rate=0.05, eval_metric='auc')
```

- 초기 ROC AUC: 0.8413
- 기본 설정만으로도 높은 성능을 보였으나, 모델 복잡도 제어(max_depth, min_child_weight)와 데이터 불균형 대응을 통한 성능 개선의 여지가 있으므로 추가 실험이 필요하다.

<br>

## Part 1. XGBoost 하이퍼파라미터 튜닝

#### HyperOpt 기반 튜닝 결과

```python
# Best params from HyperOpt
{
    'n_estimators': 500,
    'colsample_bytree': 0.6849,
    'learning_rate': 0.0709,
    'max_depth': 6,
    'min_child_weight': 5
}
```

- 초기 ROC AUC: 0.8465
- 전반적으로 안정적인 성능을 보인다. 이 설정을 기준으로 세부 실험 진행하였다.

<br>

#### 실험 1. max_depth & min_child_weight

|  | max_depth | min_child_weight | ROC AUC |
| --- | --- | --- | --- |
| 1-1 | 5 | 4 | 0.8441 |
| 1-2 | 5 | 5 | 0.8456 |
| 1-3 | 6 | 5 | **0.8465** |
| 1-4 | 6 | 6 | 0.8455 |
| 1-5 | 7 | 5 | 0.8463 |

- max_depth=6, min_child_weight=5일 때, ROC-AUC가 0.8465로 가장 높은 성능을 보인다.
- 너무 낮거나 높은 depth는 성능 저하 또는 과적합 문제가 발생할 수 있으므로 주의가 필요하다.

<br>

#### 실험 2. colsample_bytree

|  | colsample_bytree | ROC AUC |
| --- | --- | --- |
| 2-1 | 0.60 | 0.8464 |
| 2-2 | 0.65 | **0.8469** |
| 2-3 | 0.70 | 0.8457 |
| 2-4 | 0.80 | 0.8446 |

- colsample_bytree=0.65에서 ROC-AUC가 0.8469로 미세한 성능 향상을 보인다.
- 너무 높거나 낮은 경우 모두 성능 저하 경향이 있으므로, 적절한 피처 샘플링 비율이 필요하다.

<br>

#### 실험 3. learning_rate & n_estimators

|  | learning_rate | n_estimators | ROC AUC |
| --- | --- | --- | --- |
| 3-1 | 0.05 | 500 | 0.8468 |
| 3-2 | 0.03 | 700 | 0.8464 |
| 3-3 | 0.06 | 500 | **0.8471** |
| 3-4 | 0.07 | 500 | 0.8462 |

- learning_rate=0.06에서 ROC-AUC가 0.8471로 미세한 성능 향상을 보인다.

<br>

#### 최종 하이퍼파라미터 조합 (XGBoost)

```python
XGBClassifier(
    n_estimators=500,
    learning_rate=0.06,
    max_depth=6,
    min_child_weight=5,
    colsample_bytree=0.65,
    early_stopping_rounds=100,
    eval_metric='auc',
    use_label_encoder=False
)
```

- 최종 ROC AUC: 0.8471

<br>
<br>

## Part 2. LightGBM 하이퍼파라미터 튜닝

#### 1. HyperOpt 기반 튜닝 결과

```python
# Best params from HyperOpt
{
    'n_estimators': 500,
    'learning_rate': 0.0594,
    'max_depth': 115,
    'min_child_samples': 64,
    'num_leaves': 38,
    'subsample': 0.8226
}
```
- 초기 ROC AUC: 0.8423

<br>

#### 실험 1. max_depth

|  | max_depth | ROC AUC |
| --- | --- | --- |
| 1-1 | 115 | 0.8423 |
| 1-2 | 50 | 0.8423 |
| 1-3 | 30 | 0.8423 |
| 1-4 | 10 | **0.8430** |
| 1-5 | 130 | 0.8423 |

- max_depth=30에서 ROC-AUC가 0.8430으로 미세한 성능 향상을 보인다.
- max_depth=10~30 범위에서도 충분한 표현력을 확인할 수 있으며, 너무 깊은 트리는 오히려 과적합의 원인이 될 수 있다.

<br>

#### 실험 2. num_leaves

|  | num_leaves | ROC AUC |
| --- | --- | --- |
| 2-1 | 38 | 0.8430 |
| 2-2 | 64 | 0.8424 |
| 2-3 | 20 | 0.8420 |
| 2-4 | 32 | 0.8419 |
| 2-5 | 37 | **0.8437** |

- num_leaves=37에서 ROC-AUC가 0.8437로 미세한 성능 향상을 보인다.
- num_leaves=37이 가장 높은 AUC를 기록했지만, 전반적인 성능 차이가 미세하여 실험 결과의 통계적 유의성은 낮을 수 있다.

<br>

#### 실험 3. subsample

|  | subsample | ROC AUC |
| --- | --- | --- |
| 3-1 | 0.82 | 0.8437 |
| 3-2 | 0.90 | 0.8437 |
| 3-3 | 0.70 | 0.8437 |
| 3-4 | 1.00 | 0.8437 |

- 모든 실험에서 성능 동일함이 확인되어 subsample=0.82를 유지한다.

<br>

#### 실험 4. learning_rate

|  | learning_rate | ROC AUC |
| --- | --- | --- |
| 4-1 | 0.0594 | **0.8443** |
| 4-2 | 0.06 | 0.8414 |
| 4-3 | 0.05 | 0.8414 |
| 4-4 | 0.04 | 0.8415 |

- learning_rate는 Best params인 0.0594에서 가장 높은 성능을 보인다.

<br>

#### 최종 하이퍼파라미터 조합 (LightGBM)

```python
LGBMClassifier(
    n_estimators=500,
    learning_rate=0.0594,
    max_depth=30,
    num_leaves=37,
    min_child_samples=64,
    subsample=0.82
)

```

- 최종 ROC AUC: 0.8443

<br>

## 결론

| Model | ROC AUC | 주요 설정 |
| --- | --- | --- |
| XGBoost | 0.8471 | max_depth=6, learning_rate=0.06, colsample_bytree=0.65 |
| LightGBM | 0.8443 | max_depth=30, learning_rate=0.0594, num_leaves=37 |
- XGBoost가 전체적으로 미세하게 더 높은 성능을 보여준다.
- 각 실험을 통해 성능에 민감한 하이퍼파라미터를 구분하고, 불필요한 복잡도나 과적합을 유발하는 설정을 배제함으로써 성능을 향상시킬 수 있다.

<br>
<br>

## 인사이트 및 회고

이번 실험을 통해 하이퍼 파라미터가 모델 성능에 미치는 영향과 그 민감도를 확인할 수 있었다.

XGBoost 실험에서는 max_depth, min_child_weight, colsample_bytree, learning_rate와 같은 주요 파라미터를 변경할 때마다 ROC AUC 점수가 미세하게 변동되었고, 이 중 learning_rate=0.06 조합이 가장 높은 성능(0.8471)을 보여주었다. colsample_bytree와 같이 상대적으로 간과하기 쉬운 파라미터도 성능 향상에 유의미한 영향을 줄 수 있음을 확인하였다.

LightGBM 실험에서는 HyperOpt가 도출한 max_depth=115가 과도한 복잡도로 인해 오히려 성능 개선에 도움이 되지 않았고, max_depth=30 수준에서 더 안정적인 성능을 보여주었다. 이를 통해 LightGBM이 빠르고 효율적인 구조를 갖고 있기 때문에, 적절한 깊이와 리프 수 조절이 성능과 학습 시간 모두에 긍정적인 영향을 준다는 것을 확인할 수 있었다.

실험 중 subsample, num_leaves, learning_rate 등은 일정 범위 내에서는 성능에 큰 영향을 주지 않는 구간이 존재한다는 점을 알 수 있었다. 추후 모델 튜닝 시에는 민감도가 높은 파라미터부터 우선 조정하는 것이 효율적이겠다는 생각을 할 수 있었다.

실험 결과와는 관련 없는 이야기지만, 로그 관리(verbosity, early stopping, log_evaluation 등)와 GPU 활용을 통해 모델 학습 시간을 줄이고 불필요한 출력 로그를 제거하는 방법을 찾아보며 효율적으로 모델링하는 방법을 정리해가는 시간이 되었다.


<br>

>본 실험은 책 *『파이썬 머신러닝 완벽가이드』* 를 참고하며 진행하였으며, 책에서 제안된 하이퍼파라미터 튜닝 순서를 따르되 실험 결과를 바탕으로 어떤 조합이 실제 성능에 어떤 영향을 주는지를 실험하는 데 중점을 두었습니다.

<br>
