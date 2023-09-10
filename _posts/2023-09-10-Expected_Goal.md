---
title: "[DP20] 골 결정력 판독기 - 기대득점 xG"
excerpt: "Statsbomb 데이터를 활용한 기대득점 xG 모형 만들기 및 분석"
header: 
    teaser: ""
tags:
  - 축구 모델링
  - 축구 데이터
  - expected goal
  - xG
---

# Preface

*"I walk slowly, but I never walk backward." - Abraham Lincoln*

# Intro 

이번 주제는 꼭 한번 구현해보고 다뤄보고 싶었던 예측 지표들 xG(expected Goal), xA(expected Assist), xPV(expected personal value)이다.
내 기억으로는 2017년 즈음 [Opta](https://www.statsperform.com/opta/) 를 통해서 기대득점 xG 라는 지표를 처음 접했던 것 같다. 
xG 가 나오기 전 까지는 단순 통계정보들 (골, 패스, 점유율, 드리블, … 등) 을 활용한 1차원적인 축구 데이터 분석이 이루어졌다면,
xG 의 등장은 축구에 데이터를 활용하여 더 고도화된 분석이 이루어지게 된 좋은 전환점이 되었다.  

# 기대 득점 (xG)

기대득점(expected Goal)은 슈팅하는 선수의 실력과 골키퍼의 능력, 실수 등을 배제하고 슈팅 위치, 슈팅 방법, 상황, 수비수와의 거리 등을 고려하여
골이 들어갈 확률을 데이터를 기반으로 산출해낸 지표이다.

<br>

쉽게는 *"해당 슈팅상황에서 평균적인 선수가 슈팅했을때 골이 터질 확률"* 로 이해할 수 있다.  

<br>

현재는 공격수들의 슛팅 능력(또는 결정력)을 나타내는 지표로 매우 널리 쓰이고 있고, 
이제는 축구를 좋아하는 사람들이면 모를 수가 없는 아주 대중적인 지표가 되었다. 
처음엔 Opta Sports 에서 xG 지표를 소개하였는 데, 현재는 다양한 곳에서 각자의 방식으로 xG를 모델링하고 있다.

# 모델링

xG의 모델링은 아주 간단하다.

$$y = f(X)$$ 

여기서 $$y$$ 는 골이 될 확률 $$\mathbf{X} = [X_0,X_1,X_2, ...]$$ 는 다양한 슈팅이 발생하는 상황이다.
여기서 골이 될 확률에 영향을 미치는 슈팅 상황 $$X_i$$ 에는 
-	골대와의 거리
- 골대와의 각도
-	골키퍼의 위치
-	가까운 수비수의 위치
-	헤딩 여부
-	공을 차기 직전 공격수의 속도
<br>

등이 있다.
<br>

위 모델을 실제 있었던 슈팅의 상황 및 골 데이터를 활용하여 학습을 진행하면 된다.

학습에 사용될 골 데이터의 Target 값(골 여부)은 Goal, No goal의 binary class discrete value 인 데 반해서
학습을 통해 만들어내고자 하는 xG model 의 예측 결과값 $$\hat{y}$$ 은 0~1 사이의 확률을 나타내는 continues value 이다.
이런 경우에 가장 쉽게 생각해 낼 수 있는 모형이 Logistic Regression 이다.

# Logistic regression

Logistic regression은 가장 널리 사용되는 Classification model 중 하나이다. 이 모형에서 활용되는  logistic function 의 특성상 모형의 output 이 0~1 사이의 확률을 나타내는 값이 되고,
무엇보다 단순한 모형으로 내부 파라미터로 $$X$$ 들과 $$y$$의 상관관계 및 중요도 분석이 용이하다. 
(비슷한 설명력을 가정했을 때, 모형이 단순할 수록 본질에 가까운 정보를 담고 있지 않나..)
만약 슈팅 상황에 대한 풍부한 정보가 갖추어져 있다면 더욱 복잡한 모형을 활용해도 좋을 것 같으나, Statsbomb 에서 제공하는 Event 데이터로는 Logistic regression 이 적당할 것 같다.  

# Dataset

Goal 모형의 학습에 활용할 데이터 셋은 [Statsbomb](https://github.com/statsbomb/open-data) 에서 제공하는 event 데이터를 활용했다.
아쉽게도 최신 경기들의 event 데이터는 부족하지만, 모델링 해보기에는 충분한 것 같다.
감사하게도 [kloppy](https://github.com/PySport/kloppy) 라이브러리에서 기본적인 전처리 및 데이터 로드 방식을 제공하니 훨씬 편하게 데이터를 끌어올 수 있다.

<br>

분석은 Python 을 활용해서 진행했고, 아래와 같은 라이브러리를 활용했다.

```python
import os
import csv
import pandas as pd
import numpy as np
from kloppy import statsbomb
from mplsoccer.pitch import Pitch
import matplotlib.pyplot as plt
```

전처리 과정이 끝난 event 데이터의 형태는 다음과 같다.

```python
df.head()
```

| index | event_type | result | success | period_id | timestamp | end_timestamp | ball_state | ball_owning_team | team_id | ... | coordinates_y | end_coordinates_x | end_coordinates_y | receiver_player_id | set_piece_type | body_part_type | pass_type | competition_id | season_id | match_id |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | PASS | COMPLETE | TRUE | 1 | 0.08 | 0.2088 | alive | AFC Bournemouth | AFC Bournemouth | ... | 40.05 | 59.35 | 42.55 | 3278 | KICK_OFF | RIGHT_FOOT | NaN | 2 | 27 | 3753988 |
| 1 | PASS | COMPLETE | TRUE | 1 | 0.288 | 1.3821 | alive | AFC Bournemouth | AFC Bournemouth | ... | 42.55 | 51.85 | 41.15 | 3344 | NaN | RIGHT_FOOT | NaN | 2 | 27 | 3753988 |
| 2 | PASS | COMPLETE | TRUE | 1 | 1.786 | 3.2996 | alive | AFC Bournemouth | AFC Bournemouth | ... | 41.15 | 36.15 | 55.75 | 3608 | NaN | LEFT_FOOT | NaN | 2 | 27 | 3753988 |
| 3 | PASS | COMPLETE | TRUE | 1 | 3.38 | 5.2381 | alive | AFC Bournemouth | AFC Bournemouth | ... | 55.75 | 26.15 | 38.55 | 3341 | NaN | RIGHT_FOOT | NaN | 2 | 27 | 3753988 |
| 4 | PASS | OUT | FALSE | 1 | 5.318 | 8.4233 | alive | AFC Bournemouth | AFC Bournemouth | ... | 39.75 | 25.75 | 1.05 | NaN | NaN | RIGHT_FOOT | LONG_BALL | 2 | 27 | 3753988 |

이 중에 승부차기를 제외한 슈팅 event를 추리면, 

```python
# except for penalty shotout
df = df[df['period_id'] != 5]
# only for shot event
df_shot = df[df['event_type'] == 'SHOT']
df_shot.head()
```
|        Index | event_type | result | success | period_id | timestamp | end_timestamp | ball_state | ball_owning_team | ... | coordinates_y | end_coordinates_x | end_coordinates_y | receiver_player_id | set_piece_type | body_part_type | pass_type | competition_id | season_id | match_id |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 30 | SHOT | OFF_TARGET | FALSE | 1 | 128.322 | NaN | alive | AFC Bournemouth | ... | 45.15 | NaN | NaN | NaN | NaN | HEAD | NaN | 2 | 27 |  |
| 50 | SHOT | OFF_TARGET | FALSE | 1 | 288.112 | NaN | alive | Swansea City | ... | 45.65 | NaN | NaN | NaN | NaN | RIGHT_FOOT | NaN | 2 | 27 |  |
| 90 | SHOT | OFF_TARGET | FALSE | 1 | 466.642 | NaN | alive | AFC Bournemouth | ... | 42.55 | NaN | NaN | NaN | NaN | LEFT_FOOT | NaN | 2 | 27 |  |
| 104 | SHOT | OFF_TARGET | FALSE | 1 | 577.702 | NaN | alive | AFC Bournemouth | ... | 53.55 | NaN | NaN | NaN | NaN | LEFT_FOOT | NaN | 2 | 27 |  |
| 133 | SHOT | POST | FALSE | 1 | 804.288 | NaN | alive | AFC Bournemouth | ... | 28.75 | NaN | NaN | NaN | NaN | LEFT_FOOT | NaN | 2 | 27 |  |

많은 column 들 중에서 xG 모형의 학습에 사용할 feature는 다음과 같다. 

- 슈팅지점과 골라인 가운데 점 사이의 거리
- 슈팅지점과 양쪽 포스트 사이의 각도
- 슈팅 타입(왼발, 오른발, 그 외)

마음같아선 **골키퍼 위치, 슈팅하는 선수의 순간 속도, 수비수 위치** 등 훨씬 더 많은 지표들을 추가하고 싶지만 statsbomb 데이터는 선수들의 좌표가 없는 Event 데이터라 세 지표로 아주 간단한 xG 모형을 만들어 보았다.

# 데이터 분포

우선, 슈팅 이벤트 위치 분포를 확인해보자, 아래 그래프는 각각 슈팅 위치의 x좌표, y좌표이다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/fe29dd68-d58f-410f-89da-3655c25b0153" align="center" width="40%">
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/4de78d5d-e15b-4e7a-a643-3308f00fc574" align="center" width="40%"/>
</p>

골라인에서 10~20m 위치, pitch 가운데에서 슈팅을 가장 많이 시도했다.

<br>

슈팅 위치와 골 간에 어떤 경향성을 보이는지 무작위로 50개 정도 추려서 그 결과를 확인해보았다.

```python
## sample 100 goals
np.random.seed(10)
sample_idx = np.random.choice(range(len(df_shot)), size=100)

pitch = Pitch(pitch_color='#e7f1fa', line_zorder=1, line_color='black', pitch_type="statsbomb")
fig, ax = pitch.draw()
for i in sample_idx:
    if df_shot.iloc[i]['result'] == 'GOAL':
        col = 'red'
    else :
        col = 'blue'  
    ax.scatter(df_shot.iloc[i]['coordinates_x'], df_shot.iloc[i]['coordinates_y'], color = col)
```
<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/18fffe1b-5169-4ec3-8d39-0a0b7df4d75c" width="300" height="200"/>
</p>

골대 가까운 곳, 중앙 부근에서 골이 많이 터지고, 총 슈팅 대비 골 전환률은 10% 정도 밖에 안된다. 

# 데이터 전처리

좌표 데이터를 골라인과의 거리, 유효 슈팅 각도로 전처리하고, 슈팅 형태는 학습에 사용할 수 있도록 one-hot encoding 한다. 

```python
#statsbomb 데이터 경기장 규격 (120, 80)
#골대 규격 (w, h) = (7.32, 2.44)
#골대 위치 (x1, y1), (x2, y2) = (120, 43.66), (120, 36.34)

def distance(points):
    output = np.sqrt(((np.array([120, 40]) - points)**2).sum(axis=1))
    return output

def on_target_angle(points):
    c = 7.32
    a = np.sqrt(((np.array([120, 43.66]) - points)**2).sum(axis=1))
    b = np.sqrt(((np.array([120, 36.34]) - points)**2).sum(axis=1))
    cos_theta = (a**2 + b**2 - c**2)/(2 * a * b)
    theta = np.arccos(cos_theta)
    return theta 

shot_points = pd.concat([df_shot['coordinates_x'], df_shot['coordinates_y']], axis=1).values
# 거리, 각도 계산
df_shot['shot_distance'] = distance(shot_points)
df_shot['shot_angle'] = on_target_angle(shot_points)
dt = df_shot[['success', 'shot_distance', 'shot_angle', 'body_part_type']]
# y 값 0, 1로 변경
dt['success'] = dt['success'].apply(lambda x:1 if x else 0)
# 슈팅 타입 one-hot encoding
dt = pd.concat([dt.loc[:,~dt.columns.isin(['body_part_type'])],pd.get_dummies(dt['body_part_type'])], axis=1)
```

전처리를 거친 데이터는 아래와 같다.

```python
dt.head()
```

| Index |        success | shot_distance | shot_angle | HEAD | LEFT_FOOT | OTHER | RIGHT_FOOT |
|---|---|---|---|---|---|---|---|
| 30 | 0 | 9.388557 | 0.65538 | 1 | 0 | 0 | 0 |
| 50 | 0 | 21.408993 | 0.327487 | 0 | 0 | 0 | 1 |
| 90 | 0 | 10.756626 | 0.64201 | 0 | 1 | 0 | 0 |
| 104 | 0 | 24.365242 | 0.250102 | 0 | 1 | 0 | 0 |
| 133 | 0 | 25.648879 | 0.256059 | 0 | 1 | 0 | 0 |



# Space Quality

축구경기에서 중요한 공간이란..? 축구경기는 골을 많이 넣으면 승리하는 단순한 경기이니까 단순하게 골을 넣기 좋은 공간, 골 넣을 확률이 높은 공간이 가장 중요한 공간이라고 할 수 있겠다. 최근에는 페널티박스를 지키는 중앙수비수와 측면수비수 사이의 하프스페이스라는 공간을 어떻게 공략할 것인가가 많은 축구팀들의 전술적 화제가 되기도 했다. 

이 논문에서는 아래와 같은 3가지 간단한 아이디어로 공간의 중요도(Space Quality)를 모델링 하였다. 

>1. 공의 위치에 따라서 공간의 중요도가 달라진다.
>2. 수비수들은 항상 중요한 공간들을 지키고 있다.
>3. 상대방의 골대와 가까워질수록 중요도가 높아진다.  

여기서 2번 가정 같은 경우, 빠른 역습과 같이 빠른 공수전환 상황에서 수비수들이 중요한 위치를 점유하고 있지 못하는 경우가 생길 수 있다. 따라서 지공 상황에서의 공간 중요도를 가정했다고 볼 수 있다. (바르셀로나 분석관 아니랄까봐... 최근 핫한 빠른 공수전환을 요하는 경기에서의 공간 중요도는 다르게 산출하는게 좋을 것 같다.)

 논문에서는 공의 위치에 따른 수비 선수들의 pitch control 값을 space quality 로 정의하고, 경기 데이터를 활용하여서 이 함수를 학습하였다. 아주 간단한 1-hidden layer 의 Neural network 로 학습을 진행했다. 모델링에 머신러닝 한 스푼 추가한 것이 참 아이디어가 좋은것 같다.

## Value of location $$V_{k,l}(t)$$

여기 $$V_{k,l}(t)$$ 가 $$t$$ 시점의 위치의 중요도이다. 이 값은 특정 시점의 공의 위치 $$p_b{t}$$ 와 운동장 내의 어떤 위치 $$p_{k,l}(t)$$ 를 input 으로 하는 함수로 구성된다. 이를 식으로 나타내면 아래와 같다. (k, l 은 가로 세로 나타내는 듯) 명확한 식 구성을 위해서 $$(k, l)$$을 location 을 나타내는 $$l$$ 로 바꾸어 표현하겠다.  

$$V_{l}(t) = fn(p_b(t), p_{l}(t);\theta)$$

이 함수를 학습시켜서 output 을 특정 시점, 특정 위치의 Space value로 활용한다.

## Space Quality $$fn$$ 학습데이터 구성

어떤 머신러닝 문제를 풀던지 가장 중요한 것은 어떻게 학습 데이터를 구성하느냐가 아닐까 싶다. 데이터가 좋으면 어떤 모델로 학습하든 쉽게 다 소화시킬 수 있으니. $$fn$$ 을 학습하는 과정에서 $$p_l$$, $$p_b$$는 경기 데이터로 주어진다. 따라서 목표하는 target $$\hat{y}$$, $$\hat{V_{l}}$$ 를 어떻게 모델링 할 것인가 하는 아이디어만 제시하면 된다. 논문에서는 해당 시점, 해당 위치의 수비팀 pitch control 값 $$D_l(t)$$를 $$\hat{V_{l}}$$로 다음과 같이 활용하였다. (pitch control 계산은 이전 글 참조)

$$D_l(t) = \sum_d I_d(p_b(t), p_l(t))$$

$$\hat{V}_l(t) = \begin{cases} 1,& D_l(t)>1 \\ D_l(t), & otherwise \end{cases}$$

실제 학습에 사용되는 $$\hat{V}_l(t)$$ 를 시각화 하면 다음과 같다.
검은 점은 특정 시점의 공의 위치이고 붉게 표현된 부분이 수비팀의 pitch control 값이다. (논문에서는 pitch 위의 모든 점들을 학습 set 으로 활용하지는 않고 가로, 세로를 각각 21, 15등분 하여 한 시점당 315 개의 학습 데이터셋이 구성되도록 했다.)
<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/18fffe1b-5169-4ec3-8d39-0a0b7df4d75c" width="300" height="200"/>
</p>

## Space Quality $$fn$$ 학습

학습은 아래외 같이 Mean Squared error 를 loss function $$\mathcal{L}$$ 의 값을 최소화 하는 파라미터 $$\theta$$ 를 찾는 방식으로 진행된다

$$\mathcal{L}(\theta) = \arg_\theta\min {1 \over n} \sum^n_{e=1}(\hat{y} - y)^2$$ 

최적화 방법은 여러 Backpropagation 의 종류 중 하나인 Adam 을 활용하였다. 

## 학습된 Space Quality model

학습된 Space Quailty model은 공의 위치에 따라 다른 공간 중요도를 나타낸다. 이를 시각화 하면 아래와 같다. 공에서 가까운 공격방향의 공간 중요도가 크게 나타난다. 공격자 입장에서 쉽게 공을 전달하여 공격을 전개할 수 있는 공간들이기 때문이다.


<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/5a9001a4-ffc1-4e85-b9bd-407736b67842" width="500" height="300"/>
</p>

저자는 여기서 *아이디어 3. 상대방의 골대와 가까워질수록 중요도가 높아진다.* 를 모델에 반영하기 위해서 아래 그림 (a) 와 같이 상대방 골대로 가까워질수록 공간의 중요도에 가중치를 부여한다. 나머지 그림은 가중치가 부여된 공간 중요도이다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/a949ca88-53c4-4681-9b33-20e9fa006421" width="500" height="300"/>
</p>

## Quality of owned space $$Q$$

이제 특정 선수가 점유하는 공간을 pitch control $$PC_i(t)$$로 모델링하였고, 특정 공간의 중요도를 space quality $$V(t)$$로 구현하였다. 특정 선수가 점유하는 공간의 중요도 $$Q_i(t)$$ 는 다음과 같이 표현할 수 있다.

$$Q_i(t) = PC_i(t)V(t)$$ 

이제 이 선수가 점유하고 있는 공간의 중요도를 수치화하였으니, 이 수치가 선수의 움직임에 따라 어떻게 변하는지를 통해 해당 선수를 평가할 수 있다.

# Space Occupation Gain

위의 $$Q_i$$ 가 특정시점 $$t$$ 부터 $$t+w$$ 까지 변한다고 할때, 이 값의 시간에 따른 변화량 $$G_i(t)$$ 룰 아래와 같이 정의하고 

$$G_i(t) = \Delta Q_i$$

이 값에 일종의 $$\epsilon$$ 를 도입하여 최종적으로 Space Occupation Gain $$SOG$$ 를 구현한다.

$$SOG_i(t) = \begin{cases} G_i(t) & if \space G_i(t)  \ge \epsilon \\ 0 & otherwise \end{cases}$$

 여기서 $$\epsilon$$ 는 연속적인 Space Occupation Gain 를 Discrete 한 값으로 변환 시켜주는 역할을 한다. 아래 왼쪽 그림에서는 H4, H6, H12 선수가 수비수의 방해가 없는 넓은 공간을 찾아서 나오는 장면이다. 상대방의 견제를 멀리 하면서 공격 작업을 쉽게 할 수 있는 좋은 공간으로의 움직임이다. 오른쪽 그림에서는 H8, H9 두 공격수가 상대 수비수와 미드필더 사이의 빈 공간으로 파고들면서 더 좋은 공간을 점유하기 위해 움직이면서 $$\epsilon$$ 보다 높은 SOG 값을 창출하고 있다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/f7762aa2-6053-4bd3-880c-5e1ca4716664" align="center" width="40%">
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/34baa5cc-4d49-4f07-9dc0-73f05957895f" align="center" width="40%"/>
</p>

한 경기에서 각 선수가 발생시킨 SOG의 횟수, 평균 등을 보면 각 선수의 움직임이 얼마나 효과적이었는가를 나타낼 수 있다. 

# Space Generation Gain

축구경기에서 공간을 창출하는 방법은 자신이 직접 좋은 공간을 파고들고 점유하는 선수도 분명 가치 있는 선수이지만, 수비수들을 끌고 들어가면서 다른 선수에게 더 좋은 공간을 만들어 주는 선수들도 있다. 한 때 유행했던 *가짜 9번 - False 9* 유형의 선수들이 바로 이런 유형의 움직임을 가져간다. (개인적으로 피르미누와 같이 이 위치에서 이타적인 선수들 너무 좋아함.) 이 움직임은 다음과 같이 모델링 할 수 있다.

> 1. A팀 선수 a1와 B팀 선수 b1이 가까운 거리에 있다. $$(t)$$
> 2. b1이 A팀의 다른 선수 a2와 가까워지면서, a1과 멀어진다. $$(t+w)$$
> 3. a1가 b1과 멀어지면서 $$\epsilon$$ 보다 높은 SOG를 갖는다. $$(t+w)$$

아래는 실제로 모델링을 통해 얻은 결과로 공격수 A10 가 파고드는 움직임을 통해 수비수 H2를 끌어당기고 측면공격수 A7 에게 공간을 만들어주는 Space Generation이 발생하는 장면이다. (그래도 H1이 빠르게 열린 공간을 커버해주고 있다.) 
<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/a9b183b9-dfc9-4286-b8ea-a040300dc84b" align="center" width="30%">
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/6d1113a5-f294-4e29-bcca-06a8cfc0154a" align="center" width="30%"/>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/e517052a-9b3c-4473-b7eb-7f0f856c3fd8" align="center" width="30%"/>
</p>

직접 구현한 모델들을 바탕으로 Metrica sports가 제공하는 Sample Game1 의 선수들의 SOG, SGG를 정리하면 다음과 같다. 

| player | generation  SGG | receive SGG | active SOG | passive SOG |
|-------:|-----------:|--------:|-------:|--------:|
|     A1 |          3 |       3 |      8 |       0 |
|    A10 |          5 |       7 |     73 |      10 |
|    A11 |          0 |       0 |      0 |       0 |
|    A12 |          1 |       2 |     23 |       1 |
|    A13 |          0 |       1 |      4 |       0 |
|    A14 |          2 |       2 |      7 |       1 |
|     A2 |          1 |       6 |     10 |       0 |
|     A3 |         11 |       3 |     18 |       3 |
|     A4 |          3 |       8 |     37 |       5 |
|     A5 |          9 |       1 |     42 |       5 |
|     A6 |          1 |       9 |     27 |       1 |
|     A7 |          5 |       4 |     50 |       5 |
|     A8 |          0 |       0 |     17 |       7 |
|     A9 |         17 |      12 |     99 |       5 |
|     H1 |          1 |       2 |      3 |       0 |
|    H10 |          3 |       3 |     42 |       9 |
|    H11 |          0 |       0 |      0 |       0 |
|    H12 |          5 |       4 |     35 |       4 |
|    H13 |          0 |       0 |     17 |       6 |
|    H14 |          0 |       0 |     11 |       1 |
|     H2 |          0 |       2 |      8 |       0 |
|     H3 |          7 |       3 |     16 |       0 |
|     H4 |          0 |       1 |     18 |       4 |
|     H5 |          2 |       2 |     42 |       6 |
|     H6 |          0 |       3 |     25 |       0 |
|     H7 |          0 |       3 |     16 |       1 |
|     H8 |         11 |       8 |     46 |       9 |
|     H9 |         12 |      10 |     69 |      16 |

각 팀의 중앙공격수 H9, A9의 지표가 가장 눈에 띈다. 아무래도 가장 위협적인 공간에서 많은 움직임을 가져가기 때문인 듯 하다. 또 주목해보아야 할 값들은 A2, A4의 SGG 값이다. 이 두 측면 수비수는 다른 선수의 움직임을 통해 공간을 얻은 횟수가 만들어준 횟수보다 많다. 다른 선수들의 이타적인 움직임을 잘 활용할 수 있도록 측면의 부분전술이 좋은 팀이라고 볼 수 있겠다.

# 모델 구현

이 논문에 나오는 여러가지 지표나 모델들을 구현하는 과정에서 단순 구현은 그리 어려운 일이 아니지만, RAW 상태의 데이터 전처리, 머신러닝 학습셋 구성, 시각화 등 부수적인 부분들이 훨씬 품이 많이 드는 일이었다.... 오랜만에 python 공부하고 오히려 좋아 🤣. 하지만 이 글에 겪었던 고생을 하나하나 늘어 놓을수는 없으니 넘어가도록 하고 중요 구현 부분만 소개하고자 한다. (자세한 전처리 코드는 [Wide-Open-Space](https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model)에 정리해 두었음)

## Space Quality model 학습
저자가 어떤 tool을 활용해서 개발했는지는 모르겠지만, 나는 다루기 편한 pytorch 를 활용해서 학습을 진행했다. (학습 데이터 구성은 너무 복잡하니 스킵..)

### Torch 환경 확인

```python
device = torch.device('mps:0' if torch.backends.mps.is_available() else 'cpu')
print (f"PyTorch version:{torch.__version__}") # 1.12.1 이상
print(f"MPS device is built: {torch.backends.mps.is_built()}") # True 여야 합니다.
print(f"MPS device is available: {torch.backends.mps.is_available()}") # True 여야 합니다.
!python -c 'import platform;print(platform.platform())'
```

### Custom dataset 구성
```python
class custom_dataset(Dataset):
    def __init__(self, file_path):
        data = pd.read_csv(file_path)
        data['Ball_x'] = data['Ball_x']/104
        data['loc_x'] = data['loc_x']/104
        data['Ball_y'] = data['Ball_y']/68
        data['loc_y'] = data['loc_y']/68
        self.columns = data.columns.to_list()
        self.y = data['v_kl'].values.reshape(-1, 1)
        self.x = data[['Ball_x', 'Ball_y', 'loc_x', 'loc_y']].values
        self.length = len(data)
    
    def __getitem__(self, index):
        x = torch.FloatTensor(self.x[index])
        y = torch.FloatTensor(self.y[index])
        return x, y
    
    def __len__(self):
        return self.length
    
    def column(self):
        return self.column
```

### Train - Test Split
이미 ./Data/dataset.csv 에 학습 데이터셋 구성되어 있을때를 가정한다.
```python
dataset = custom_dataset('./Data/dataset.csv')
dataset_size = len(dataset)

train_size = int(dataset_size * 0.8)
test_size = dataset_size - train_size

generator1 = torch.Generator().manual_seed(1)
train_dataset, test_dataset = random_split(dataset, [train_size, test_size])

train_dataloader = DataLoader(train_dataset, batch_size=1024, shuffle=True)
test_dataloader = DataLoader(test_dataset, batch_size=1024)
```

### Simple Multi-layer neural network model
논문에서는 1-hidden layer model 이었는 데, 내부 hidden node 갯수나 activation function 과 같은 세부정보가 없어서 새로 모델링 하였다.  

```python
class nnModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.hidden1 = nn.Linear(4, 24, bias=True)
        self.hidden2 = nn.Linear(24, 16, bias= True)
        self.hidden3 = nn.Linear(16, 8, bias= True)
        self.output = nn.Linear(8, 1, bias=True)
        #self.dropout = nn.Dropout(0.2)

    def forward(self, x):
        x = F.relu(self.hidden1(x))
        x = F.relu(self.hidden2(x))
        x = F.relu(self.hidden3(x))
        x = F.relu(self.output(x))

        return x
    
    def reset_parameters(self):
        nn.init.kaiming_normal_(self.hidden1.weight)
        self.hidden1.bias.data.fill_(0.01)
        nn.init.kaiming_normal_(self.hidden2.weight)
        self.hidden2.bias.data.fill_(0.01)
        nn.init.kaiming_normal_(self.hidden3.weight)
        self.hidden3.bias.data.fill_(0.01)
        nn.init.kaiming_normal_(self.output.weight)
        self.output.bias.data.fill_(0.01)
```

### parameter 초기화
```python
model = nnModel()
model.reset_parameters()
```

### 구조 확인
```python
summary(model, (128, 4))
```
```

----------------------------------------------------------------
        Layer (type)               Output Shape         Param #
================================================================
            Linear-1              [-1, 128, 24]             120
            Linear-2              [-1, 128, 16]             400
            Linear-3               [-1, 128, 8]             136
            Linear-4               [-1, 128, 1]               9
================================================================
Total params: 665
Trainable params: 665
Non-trainable params: 0
----------------------------------------------------------------
Input size (MB): 0.00
Forward/backward pass size (MB): 0.05
Params size (MB): 0.00
Estimated Total Size (MB): 0.05
----------------------------------------------------------------
```

### Traning 
```python
optimizer = optim.Adam(model.parameters(), lr=0.001)

# loss function
loss_fn = nn.MSELoss()

# training
train_loss_list=[]
test_loss_list=[]

n = len(train_dataloader)

epoch = 50
for i in range(epoch):
    model.train()
    train_loss = 0

    #load data
    for idx, data in enumerate(train_dataloader):
        x, y = data
        x = x.to(device)
        y = y.to(device)

        optimizer.zero_grad()

        #forward propagation
        y_pred = model(x)
        loss = loss_fn(y_pred,y)
        
        #backpropagation
        loss.backward()
        optimizer.step()

        train_loss += loss.mean()
        print('\r' + ''*100, end="")
        print('\r {}/{} batch_loss : {:.5f},  epochs: {}'.format(idx, n, loss.mean(), (i+1)), end="  ")
    
    train_loss /= n
    train_loss_list.append(train_loss.item())

    if (i+1)%10 == 1:
        print('\r' + ''*100, end="")
        print('\n epoch {}/{} train_loss : {:.5f}'.format(i+1, epoch, train_loss))

    with torch.no_grad():
        model.eval()
        test_loss = 0
        for x, y in test_dataloader:
            x = x.to(device)
            y = y.to(device)
            
            y_pred = model(x)
            loss = loss_fn(y_pred,y)
            
            test_loss += loss.mean()

        test_loss /= len(test_dataloader)
        test_loss_list.append(test_loss.item())

        if (i+1)%10 == 1:
            print('\r' + ''*100, end="")
            print('epoch {}/{} test_loss : {:.5f} \n'.format(i+1, epoch, test_loss))

        if min(test_loss_list) >= test_loss.item():
            torch.save(model.state_dict(), './SpaceValueModel/best_svmodel_sdict.pt')
```

### Verifying model
공의 위치에 따른 수비 선수들의 위치를 학습셋으로 활용하였기때문에 경기 흐름에 따라서 실제 y값 자체의 분산이 존재한다. 같은 위치에서 공격이 진행되더라도 역습상황에서의 수비선수들의 위치는 지공상황의 수비수들의 위치보다 분명 높게 형성될 것이기 때문이다. 따라서 모델의 정확도 보다는 경향성 즉, 실제 공격상황에서 수비수들이 지키고 있을만한 좋은 공간을 잘표현하고 있는가가 중요하다. 그래서 모델 학습이 잘 되었는지 다음과 같은 3가지 방법으로 확인하였다. 

> 1. Learning curve
> 2. 실제값-예측값 plot
> 3. 시각화

#### Learning curve
Train loss와 Test loss가 epoch에 따라서 비슷하게 줄어들고 있다. (학습이 그럭저럭 진행되고 있다.)

<p align=center>
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/742e1ffc-7b12-4891-80f7-9f9b82aa56e9" width="500" height="250"/>
</p>

#### 실제값-예측값 plot

x축은 Target space quality 값 $$V_l(t)$$ 이고, y축은 예측 space quality 값 $$\hat{V}_l(t)$$ 이다. 모델이 데이터를 아주 잘 설명하고 있다고 보긴 어렵지만, 우리가 학습시키고자 하는 방향성은 어느정도 학습 된 것 같다. (공의 위치에 따른 수비선수들의 위치를 학습 시키다보니 운동장 위의 상황에 따른 데이터의 분산 자체가 존재한다. 예를 들어서 역습상황시의 수비위치와 지공상황의 수비위치는 분명 다르기에 학습 데이터의 target 값에 분산이 존재하고 모델링의 설명력 자체 한계가 존재할 수 밖에 없다.)

<p align=center>
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/7babe5e7-6f70-4982-a26d-7cb17249a8e3" width="500", height="250">
</p>

#### 예측값 plot

학습된 모델로 부터 산출한 공에 위치에 따른 Space quality 를 시각화하면 왼쪽 아래 그림과 같다. 공격방향, 공 근처에 있는 공간들의 중요도가 높게 나타난다. 여기서 상대방 골대와 가까워질수록 더 중요하다는 도메인 지식을 추가하여(방법은 위에서 소개한 상대방 골대에 까워질 수록 가중치 부여) 최종적으로 오른쪽 아래 형태의 Space quality 모델을 구현하였다.

<p align=center>
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/6c737076-1838-42d0-b611-19a6e15ab73a" align="center" width="40%">
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/cd42e7b6-b6e2-4ce7-ba2a-2408eea7ed68" align="center" width="40%"/>
</p>

아래 코드는 ETPS 데이터와 축구경기 event 데이터를 가지고, 특정시점에 pitch control, space quality, space value, 공을 소유하고 있는 팀을 구하는 함수를 구현한 것이다.   

```python
import numpy as np
import pandas as pd
import math
import torch
from collections import Counter
import warnings
from scipy.stats import multivariate_normal
from pandas.errors import SettingWithCopyWarning
import matplotlib.pyplot as plt 
from matplotlib.patches import Ellipse
from matplotlib.ticker import FormatStrFormatter
from torch import optim
from torch.utils.data import Dataset
from torch.utils.data import DataLoader
from torch.utils.data import random_split
from src.nn_model import nnModel
from src.plot_utils import *
from src.Influence_function import *
from matplotlib import animation
warnings.simplefilter(action="ignore", category=SettingWithCopyWarning)
warnings.simplefilter(action="ignore", category=DeprecationWarning)

class space_model():
    def __init__(self, df, e_df):
        self.df = self.preprocess(df, e_df)
        self.velocities = None
        self.positions = None
        self.points = None
        self.players = None
        self.ball_x = None
        self.ball_y = None
        self.set_frame_flag = False

    def set_frame(self, frame): 
        t_df = self.df[self.df['Frame'] == frame]
        self.period = t_df['Period'].values[0]
        t_df = t_df.drop(['Period', 'Ball_x', 'Ball_y', 'Ball_v_abs'], axis=1).iloc[0,:]
        self.positions = t_df[[i for i in t_df.index if (('_x' in i) or ('_y' in i)) and 'v' not in i]]
        self.positions.dropna(inplace=True)
        self.velocities = t_df[[i for i in t_df.index if '_v' in i]]
        self.velocities.dropna(inplace=True)
        self.points = np.array([[self.positions[2*i], self.positions[2*i+1]] for i in range(len(self.positions)//2)])
        self.velocities = np.array([[self.velocities[3*i], self.velocities[3*i+1]] for i in range(len(self.velocities)//3)])
        self.players = np.array([self.positions.index[2*i].split('_')[0] for i in range(len(self.points))])
        self.ball_x, self.ball_y = self.df.loc[self.df['Frame'] == frame, ['Ball_x', 'Ball_y']].values[0]

        if math.isnan(self.ball_x) :
            self.set_frame_flag = False
            return 
        else :
            self.set_frame_flag = True
            return
        
    def pitch_control(self, locations):
        s_h, s_a = 0, 0
        if not self.set_frame_flag:
            print('need to set frame')
            return
        for i, j, k in zip(self.players, self.points, self.velocities):
            if 'H' in i:
                s_h += influence_function2(j, locations, k, (self.ball_x, self.ball_y))
            else :
                s_a += influence_function2(j, locations, k, (self.ball_x, self.ball_y))
                
        z = 1 / (1 + np.exp(- s_h + s_a))
        return z
    
    
    def space_value(self, locations):
        nn_model = nnModel()
        nn_model.load_state_dict(torch.load('./SpaceValueModel/best_svmodel_sdict.pt'))

        x = locations.reshape(-1, 2)[:,0].reshape(-1, 1) /104
        y = locations.reshape(-1, 2)[:,1].reshape(-1, 1) /68
    
        ball_x = self.ball_x * np.ones_like(x) / 104
        ball_y = self.ball_y * np.ones_like(y) / 68

        input = torch.FloatTensor(np.concatenate([ball_x, ball_y, x, y], axis=1))
        output = nn_model(input)
        output = output.detach().numpy()

        z = self.distance_from_goal(x*104, y*68)
        output = output.reshape(-1) * z.reshape(-1) 

        return output
    
    def space_quality(self, attacking_direction):
        if not self.set_frame_flag:
            print('need to set frame')
            return
        
        space_quality = {}

        for i,j in zip(self.players, self.points):
            pc = self.pitch_control(j)
            if i.startswith('A'):
                pc = 1 - pc
                if attacking_direction == 1:
                    sv = self.space_value(np.array([104 - j[0], j[1]]))
                else :
                    sv = self.space_value(j)
            else :
                if attacking_direction == 1:
                    sv = self.space_value(j)
                else :
                    sv = self.space_value(np.array([104 - j[0], j[1]]))
            key = i
            key = key + '_sq'
            space_quality[key] = float(pc*sv)

        return space_quality


    def distance_from_goal(self, x, y):
        dist = np.sqrt((104-x)**2 + (34-y)**2)
        max_v = np.sqrt(104**2 + 34**2)
        return (max_v - dist)/ (max_v)


    def preprocess(self, df, e_df):
        t_df = e_df[['Team', 'Type', 'Start Frame', 'End Frame', 'Period']]
        t_df = pd.concat([t_df, t_df.shift(-1).rename(columns={'Team': 'Next Team', 
                                                            'Type' : 'Next Type',
                                                            'Start Frame' : 'Next Start Frame',
                                                            'End Frame' : 'Next End Frame',
                                                            'Period' : 'Next Period'})], axis=1)
        t_df = t_df[(t_df['Start Frame'] < t_df['Next End Frame']) &
            (t_df['Team'] == t_df['Next Team']) & 
            (t_df['Period'] == t_df['Next Period']) &
            ((t_df['Type'] == 'PASS') | (t_df['Type'] == 'RECOVERY'))]
        t_df.reset_index(drop=True, inplace=True)

        n_list = np.zeros(len(df))
        for i in range(len(t_df)):
            s_frame, f_frame = np.array(t_df.loc[i, ['Start Frame', 'Next End Frame']])
            if math.isnan(s_frame*f_frame) == False:
                if t_df.loc[i, 'Team'] == 'Home':
                    n_list[int(s_frame) :int(f_frame)] = 1
                if t_df.loc[i, 'Team'] == 'Away':
                    n_list[int(s_frame):int(f_frame)] = 2
        for i in range(len(t_df)):
            if t_df.loc[i,'Type'] == 'RECOVERY':
                s_frame = t_df.loc[i, 'Start Frame']
                n_list[int(s_frame-25):int(s_frame + 25*5)] = 0
        df['owning'] = n_list
        return df
```

# Wrap Up

처음 이 논문을 읽고 한번 구현해봐야지.. 라는 생각을 하고 정리를 해서 글을 쓰기까지 약 한달정도 걸렸다. 여유 시간이 있을때마다 조금씩 구현했는데 정리하고보니 별 내용이 없는 거 같지만 하는 과정에서는 꽤나 품이 많이 들었다. 처음 생각은 2주정도면 충분하다고 생각했는데 내 여유시간과 능력을 냉정하게 돌아보게 되었다. 축구를 좋아하는 팬으로서, 축구 데이터에 굉장히 관심이 많은 사람으로서, 모델링을 통해서 축구 선수의 움직임을 지표화하고 모델링하는 과정이 정말 흥미로웠다. 세계최고의 축구팀인 바르셀로나 데이터 분석팀을 맡았던 분의 연구결과이기에 실제 최고 레벨의 축구를 구사하는 팀이 어떻게 데이터를 활용하고 있는 가를 간접적으로나 체험할 수 있었다. 역시 데이터는 누가 어떻게 다루느냐에 다라서 그 가치가 결정되는 것 같다.

[Github: metrica-sports](https://github.com/metrica-sports) <br>
[Github: kloppy](https://github.com/PySport/kloppy)
<br>
[파이썬 축구 데이터 분석-김현성님](https://class101.net/ko/search?query=%EC%B6%95%EA%B5%AC)
<br>
[Metrica-pitch- control](https://github.com/anenglishgoat/Metrica-pitch-control)
<br>
[Wide Open Spaces: A statistical technique for measuring space creation in professional soccer](https://static.capabiliaserver.com/frontend/clients/barca/wp_prod/wp-content/uploads/2018/05/Wide-Open-Spaces.pdf)