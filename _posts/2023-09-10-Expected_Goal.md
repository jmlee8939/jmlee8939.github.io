---
title: "[DP20] 골 결정력 판독기 - 기대득점 xG"
excerpt: "Statsbomb 데이터를 활용한 기대득점 xG 모형 만들기 및 분석"
header: 
    teaser: "https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/a706cdb7-8896-4a4e-8d56-d7bf1fc1589e"
tags:
  - 축구 모델링
  - 축구 데이터
  - expected goal
  - xG
---

# Intro 

이번 주제는 꼭 한번 구현해보고 다뤄보고 싶었던 예측 지표들 xG(expected Goal), xA(expected Assist), xPV(expected personal value) 중 가장 맏형인 xG 이다.
내 기억으로는 2017년 즈음 [Opta](https://www.statsperform.com/opta/) 를 통해서 기대득점 xG 라는 지표를 처음 접했던 것 같다. 
xG 가 나오기 전 까지는 단순 통계정보들 (골, 패스, 점유율, 드리블, … 등) 을 활용한 1차원적인 축구 데이터 분석이 이루어졌다면,
xG 의 등장은 축구에 데이터를 활용하여 더 고도화된 분석이 이루어지게 된 전환점이 되었다.  

# 기대 득점 (xG)

기대득점(expected Goal)은 슈팅하는 선수의 실력과 골키퍼의 능력, 실수 등을 배제하고 슈팅 위치, 슈팅 방법, 상황, 수비수와의 거리 등을 고려하여
골이 들어갈 확률을 데이터를 기반으로 산출해낸 지표이다.

<br>

쉽게는 *"해당 슈팅상황에서 평균적인 선수가 슈팅했을때 골이 터질 확률"* 로 이해할 수 있다.  

<br>

현재는 공격수들의 슛팅 능력(또는 결정력)을 나타내는 지표로 매우 널리 쓰이고 있고, 
이제는 축구를 좋아하는 사람들이면 모를 수가 없는 아주 대중적인 지표가 되었다. 
처음엔 Opta Sports 에서 xG 지표를 개발 및 소개하였는 데, 현재는 다양한 곳에서 각자의 방식으로 xG를 모델링하고 있다.

# 모델링

xG의 모델링은 아주 간단하다.

$$y = f(X)$$ 

여기서 $$y$$ 는 골이 될 확률 $$\mathbf{X} = [X_0,X_1,X_2, ...]$$ 는 다양한 슈팅이 발생하는 상황이다.
여기서 골이 될 확률에 영향을 미치는 슈팅 상황 $$X_i$$ 에는 
-	골대와의 거리
-   골대와의 각도
-	골키퍼의 위치
-	가까운 수비수의 위치
-	슈팅 방법 (왼발, 오른발, 헤딩, 그 외)
-	공을 차기 직전 공격수의 속도
-   어시스트 여부
-   어시스트 패스 타입
-   세트피스(페널티킥, 프리킥, 코너킥)
<br>

등이 있다.
<br>

위 모델을 실제 있었던 슈팅의 상황 및 골 데이터를 활용하여 학습을 진행하면 된다.

학습에 사용될 골 데이터의 Target 값(골 여부)은 Goal, No goal의 binary class discrete value 인 데 반해서
학습을 통해 만들어내고자 하는 xG model 의 예측 결과값 $$\hat{y}$$ 은 0~1 사이의 확률을 나타내는 continues value 이다.
이런 경우에 가장 쉽게 생각해 낼 수 있는 모형이 Logistic Regression 이다.

# Dataset

Goal 모형의 학습에 활용할 데이터 셋은 [Statsbomb](https://github.com/statsbomb/open-data) 에서 제공하는 event 데이터를 활용했다.
아쉽게도 최신 경기들의 event 데이터는 부족하지만, 모델링 해보기에는 충분한 것 같다.
감사하게도 [kloppy](https://github.com/PySport/kloppy) 및 [mplsoccer](https://github.com/andrewRowlinson/mplsoccer) 에서 기본적인 전처리 및 데이터 로드 기능을 제공하니 훨씬 편하게 데이터를 끌어올 수 있다.

<br>

분석은 Python 을 활용해서 진행했다(전처리 과정은 가볍게 생략) 전처리 과정이 끝난 event 데이터의 형태는 다음과 같다.


<details>
<summary> 전처리 완료된 데이터
</summary> 
<div markdown="1">

```python
df.head()
```

| index | match_id | id | type_name | sub_type_name_x | player_name | team_name | body_part_name | outcome_name | x | y | shot_statsbomb_xg | sub_type_name_y | pass_height_name |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 18243 | 129d6ccd-ef7d-44e2-87e1-b55d018f5e24 | Shot | Open Play | Jorge Resurrección Merodio | Atlético Madrid | Right Foot | Saved | 108.0 | 52.0 | 0.069306 | NaN | High Pass |
| 1 | 18243 | 855d5fee-a32c-4818-b5c8-9c98bb17bac5 | Shot | Open Play | Karim Benzema | Real Madrid | Left Foot | Saved | 115.0 | 41.0 | 0.418073 | Free Kick | Low Pass |
| 2 | 18243 | f17c2cb5-cc03-4991-987c-fae2518a21c9 | Shot | Open Play | Marcelo Vieira da Silva Júnior | Real Madrid | Left Foot | Off T | 96.0 | 32.0 | 0.032653 | NaN | NaN |
| 3 | 18243 | 158ca4ad-9fd2-4f86-94f9-c711e4b2993f | Shot | Open Play | Toni Kroos | Real Madrid | Left Foot | Blocked | 97.0 | 25.0 | 0.023983 | NaN | Ground Pass |
| 4 | 18243 | 2bf5883e-1d85-4412-b1e9-9f03ff8dafa0 | Shot | Open Play | Sergio Ramos García | Real Madrid | Left Foot | Goal | 117.0 | 40.0 | 0.562398 | NaN | High Pass |

</div>
</details>
<br>


많은 column 들 중에서 xG 모형의 학습에 사용할 feature는 다음과 같다. 

- 슈팅지점과 골라인 가운데 점 사이의 거리
- 슈팅지점과 양쪽 포스트 사이의 각도
- 슈팅 타입(왼발, 오른발, 그 외)
- 슈팅 상황(Open play, 페널티킥, 프리킥)
- 어시스트 패스 여부
- 어시스트 패스 높이(땅볼, 낮게, 높게)

마음같아선 **골키퍼 위치, 슈팅하는 선수의 순간 속도, 수비수 위치** 등 훨씬 더 많은 지표들을 추가하고 싶지만 statsbomb 데이터는 선수들의 좌표가 없는 Event 데이터라 세 지표로 간단한 xG 모형을 만들어 보았다. (좌표가 있는 데이터도 있지만 특정 대회(월드켭, 유로)에만 제공)

# 데이터 분포

우선, 슈팅 이벤트 위치 분포를 확인해보자, 아래 그래프는 각각 슈팅 위치의 x좌표, y좌표이다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/fe29dd68-d58f-410f-89da-3655c25b0153" align="center" width="40%">
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/4de78d5d-e15b-4e7a-a643-3308f00fc574" align="center" width="40%"/>
</p>

골라인에서 10~20m 위치, pitch 가운데에서 슈팅을 가장 많이 시도했다.

<br>

슈팅 위치와 골 간에 어떤 경향성을 보이는지 무작위로 50개 정도 추려서 그 결과를 확인해보았다.


<details>
<summary> 시각화 코드
</summary>
<div markdown="1">


```python
## sample 100 goals
np.random.seed(10)
sample_idx = np.random.choice(range(len(df_shot)), size=50)

pitch = Pitch(pitch_color='grass', line_zorder=1, line_color='white', pitch_type="statsbomb", stripe=True)
fig, ax = pitch.draw()
for i in sample_idx:
    if df.iloc[i]['outcome'] == 'Goal':
        col = 'red'
    else :
        col = 'blue'
    ax.scatter(df.iloc[i]['x'], df.iloc[i]['y'], color = col)
```

</div>
</details>
<br>


<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/8d9703a7-c091-4020-85a2-1de2a35fedba" width="300" height="200"/>
</p>

50개의 샘플링된 슈팅 중에 골은 4골이 터졌고 모두, 페널티 박스 내, 중앙 부근에서 골이 터졌다. 어떤 위치, 어떤 슈팅 상황에서 골이 터질 확률이 높을지 xG 모형 학습을 통해 가볍게 확인해보았다. 

# Logistic regression

Logistic regression은 가장 널리 사용되는 Classification model 중 하나이다. 아래는 Logistic regression 의 구조이다.

$$ y = {e^{(\beta_0 + \beta_1x_1 + \beta_2x_2 + ...)}\over1+ e^{(\beta_0 + \beta_1x_1 + \beta_2x_2 + ...)}}$$

여기서 $$y$$ 모델링 하고자 하는 종속변수이고, $$x$$는 슈팅 각도, 슈팅 거리 등 각각의 독립변수 이다. $$ \beta $$ 는 데이터를 통해서 학습되는 모형의 회귀 계수이다.  

이 모형에서 활용되는  logistic function 의 특성상 모형의 output 이 0~1 사이의 확률을 나타내는 값이 되고,
무엇보다 단순한 모형으로 내부 파라미터로 $$X$$ 들과 $$y$$의 상관관계 및 중요도 분석이 용이하다. 
(비슷한 설명력을 가정했을 때, 모형이 단순할 수록 본질에 가까운 정보를 담고 있지 않나..)
만약 슈팅 상황에 대한 풍부한(+ 복잡한) 정보가 갖추어져 있다면 더욱 복잡한 모형을 활용해도 좋을 것 같으나, Statsbomb 에서 제공하는 Event 데이터로는 Logistic regression 이면 충분해 보인다.  


# 데이터 전처리

좌표 데이터를 골라인과의 거리, 유효 슈팅 각도로 전처리하고, 슈팅 형태는 학습에 사용할 수 있도록 one-hot encoding 한다. 


<details>
<summary> 코드 </summary>
<div markdown="1">

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

shot_points = pd.concat([df['x'], df['y']], axis=1).values
# 거리, 각도 계산
df['shot_distance'] = distance(shot_points)
df['shot_angle'] = on_target_angle(shot_points)
# y 값 0, 1로 변경
df['success'] = [1 if i=='Goal' else 0 for i in df['outcome']]
# 슈팅 타입 one-hot encoding
numeric_columns = ['success', 'shot_distance', 'shot_angle']
categorical_columns = ['body_part', 'shot_type', 'pass_height', 'pass_type']
dt = pd.concat([df.loc[:,df.columns.isin(numeric_columns)],pd.get_dummies(df[categorical_columns])], axis=1)
# 최종 데이터 컬럼 정리
dt = dt[['success', 'shot_distance', 'shot_angle', 'body_part_Head',
       'body_part_Left Foot', 'body_part_Other',
       'body_part_Right Foot', 'shot_type_Free Kick',
       'shot_type_Penalty', 'pass_height_Ground Pass',
       'pass_height_High Pass', 'pass_height_Low Pass',
       'pass_type_Corner', 'pass_type_Free Kick']]
```

</div>
</details>
<br>

전처리를 거친 데이터는 아래와 같다.

```python
dt.head()
```

| success | shot_distance | shot_angle | body_part_Head | body_part_Left Foot | body_part_Other | body_part_Right Foot | shot_type_Free Kick | shot_type_Penalty | pass_height_Ground Pass | pass_height_High Pass | pass_height_Low Pass | pass_type_Corner | pass_type_Free Kick |  |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| 0 | 0 | 16.970563 | 0.309593 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| 1 | 0 | 5.099020 | 1.239135 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 1 |
| 2 | 0 | 25.298221 | 0.273350 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 3 | 0 | 27.459060 | 0.223529 | 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| 4 | 1 | 3.000000 | 1.768350 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |


# 모델 학습

sckit-learn 라이브러리를 활용해서 Logistic regression 모형을 학습했다.


<details>
<summary> 코드 </summary>
<div markdown="1">

```python
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import MinMaxScaler
from sklearn.model_selection import train_test_split
```
feature(x) 와 output(y) 값을 구분.

```python
x = dt.loc[:, dt.columns != 'success'].values
y = dt['success'].values
```

모델 학습에 사용할 feature 들을 0~1 값으로 normalize.
```python
scaler = MinMaxScaler()
X = scaler.fit_transform(x)
```
train-set, test-set 을 나누고
```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
```
학습. 
```python
lr = LogisticRegression()
lr.fit(X_train, y_train)
```

</div>
</details>
<br>


# Model validation

## unbalance dataset
위에서 학습한 데이터의 테스트셋의 정확도(accuracy)는 
```python
print(lr.score(X_test, y_test))
# = 0.9018...
```
90%가 넘는다. 하지만 실제 예측 결과를 보면,
```python
np.random.seed(50)
sample_idx2 = np.random.choice(range(len(dt)), 100)
sample_x = X[sample_idx2,:]
sample_y = y[sample_idx2]
print(lr.predict(sample_x))

# array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0])
```
xG 모형의 Decision Boundary


<details>
<summary> 코드 </summary>
<div markdown="1">

```python
from scipy.special import expit
x_ = np.linspace(-6, 6, 121)
y_ = expit(x_)
plt.plot(x_, y_)
for i in range(len(sample_y)):
    if sample_y[i] == 1:
        color = 'red'
    else :
        color = 'blue'
    plt.scatter(predicted_score[i], sample_y[i], color = color)
plt.vlines(x=0, ymin=-0.1, ymax=1.1, linestyles='--', colors='red')
plt.grid()
plt.xlim(-6, 6)
plt.ylim(-0.1, 1.1)
plt.xlabel('logistic(x)')
plt.title('샘플 데이터의 logistic(x) 값')
plt.fill_between([0, 0, 6, 6], [-1, 1, 1, -1], alpha=0.1, color='red')
plt.text(x=2.5, y=0.5, s='예측 골')
plt.show()
```

</div>
</details>
<br>

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/2fbddfb2-6d68-4e36-bbdf-a925d88ce657" width="300" height="200"/>
</p>

50슈팅 중 2골을 예측했고, (그 마저도 하나는 틀렸다...) 정확도는 no goal이 90%인 *class imbalance* 상황에서는 큰 의미가 없다. (만약 모든 슈팅이 실패한다고 예측한다고 하더라도 정확도 accuracy는 90%가 넘을 것이기 때문에..) 
<br>


따라서 이와 같은 *class imbalance* 상황에서는 모델이 얼마나 설명력을 가지는 가를 나타내는 지표로 정확도가 아닌, AUC 나 f1-score 등의 지표를 활용하여야 한다. 나는 [AUC(Area under curce)](https://en.wikipedia.org/wiki/Receiver_operating_characteristic)를 활용해서 모델의 설명력을 확인해보았다. AUC(Area Under Curve)는 binary class 를 구분하는 treshold 의 값이 변할때 TPR(True Positive Rate)와 FPR(False Positive Rate)의 값으로 그린 그래프의 아래 면적으로, class imbalance 비율에 관계없이 쉽게 모형이 얼마나 두 클래스의 분포를 잘 구분하고 있는 지를 나타내는 지표이다.

## AUC-ROC curve


<details>
<summary> 코드 </summary>
<div markdown="1">

```python
y_pred_proba = lr.predict_proba(X_test)[::,1]
auc = roc_auc_score(y_test, y_pred_proba)
print(auc)
#0.80334..
```

```python
plt.figure(figsize = (5,5))
plt.plot(model_fpr, model_tpr, marker = '.', label = "Logistic")
plt.plot([0, 1], [0, 1], 'red')
plt.xlabel('FPR')
plt.ylabel('TPR')
```

</div>
</details>
<br>

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/ce947773-9670-41ae-bd58-2b9e3f91e9e0" width="300" height="300"/>
</p>

AUC-ROC curve의 형태 및 AUC 값으로 보아 xG 모형이 골의 경향성을 잘 설명하고 있음을 확인할 수 있다. 하지만 이 모형이 정말 xG 모형으로 쓸만한 놈인가 좀 더 다각도로 확인해보고자 아래 3가지를 더 들여다보았다.
 
> 1. 회귀 계수를 통한 변수 중요도 Check. 
> 2. 총 기대득점(xG)의 합
> 3. statsbomb 자체 xG

### 1. 회귀 계수와 변수 중요도 Check.

학습된 Logistic regression 의 계수는 아래와 같다.


<details>
<summary> 코드 </summary>
<div markdown="1">

```python
coef_dict = pd.DataFrame(columns = dt.loc[:, dt.columns != 'success'].columns, data= lr.coef_).transpose()
coef_dict
```

</div>
</details>
<br>

| X | Beta |
|---:|---|
| shot_distance | -9.779051 |
| shot_angle | 5.213936 |
| body_part_Head | -0.682454 |
| body_part_Left Foot | 0.222211 |
| body_part_Other | 0.122994 |
| body_part_Right Foot | 0.319869 |
| shot_type_Free Kick | 1.023703 |
| shot_type_Penalty | 2.473718 |
| pass_height_Ground Pass | 0.288567 |
| pass_height_High Pass | -0.092236 |
| pass_height_Low Pass | -0.091065 |
| pass_type_Corner | -0.796904 |
| pass_type_Free Kick | -0.208160 |

우리가 학습한 아래의 logistic regression 모형에서
$$ P_{goal} = {e^{(\beta_0 + \beta_1x_1 + \beta_2x_2 + ...)}\over1+ e^{(\beta_0 + \beta_1x_1 + \beta_2x_2 + ...)}}$$

골이 발생할 확률 $$P_{goal}$$ 에 특정 $$x$$ 가 얼마나 영향을 미칠지 결정해 주는 값이 $$\beta$$ 이다. 특정 $$x_i$$ 의 유무에 따라서 결과가 바뀔 확률을 나타내는 값이 odd ratio 이고, logistic regression 에서는 이 값이 회기 계수 $$\exp(\beta_i)$$ 이다. 따라서 페널티킥의 경우 그렇지 않은 슈팅의 경우보다 골 확률이 10% 이상 상승한다. (다른 요소가 모두 같다고 가정했을 때) *logistic regression odd ratio의 자세한 내용은 여기 [참고](https://stats.oarc.ucla.edu/other/mult-pkg/faq/general/faq-how-do-i-interpret-odds-ratios-in-logistic-regression/). 
<br>
복잡하게 생각하지 않고 1차원적으로 선형 관계식에서 계수의 절대값이 클 수록 결과 값에 많은 큰 영향을 끼친다고 쉽게 이해해도 좋을 것 같다. 그런 관점에서 봤을 때, xG 값에 가장 영향을 크게 미치는 요소는 역시나 슛거리와 각도이다. 슛거리가 작아질수록, 골대와의 각도는 커질 수록 xG 값이 커진다. 또한 페널티킥, 프리킥의 경우 골 기대값이 더 높아진다. 수비수의 도전 없이, 충분히 준비한 상태에서 슈팅을 할 수 있다는 장점이, 벽을 세우는 수비수와 방어하는 골키퍼가 충분히 대비를 할 수 있다는 단점보다 강력하게 작용하는 것으로 보인다.  

### 총 기대득점(xG)의 합

이 모형을 통해서 얻고 싶은 값은 '기대 득점(xG)' 즉, 골이 들어갈 확률이다. 기대 득점의 분포가 적절하게 구성되어 있음을 확인하기 위해서 모든 슈팅의 기대득점의 합과 실제 들어간 골의 개수를 비교해 보았다. 물론 logistc regression 모델을 학습하는 과정에서 이 두 값이 최소화하도록 목적함수가 구성되기 때문에 (당연히...) 차이가 적어야 하지만, 좀 더 직관적으로 기대 득점의 분포를 이해할 수 있기에 확인해보았다. 


```python
print(np.sum(y))
print(np.sum(lr.predict_proba(X)[::,1]))
# 4716
# 4741.6
```

실제 4716 골이 들어갔고, 모델을 통해 계산한 슈팅의 xG 값의 총합은 4741.6이다. 학습된 모델의 xG 값이 실제 골이 터질 확률을 잘 표현하고 있다고 볼 수 있다. 

### Statsbomb xG 와 비교

Statsbomb 에서도 모든 슈팅에 대한 자체적인 xG 값을 제공하고 있다 [Statsbomb xG](https://statsbomb.com/soccer-metrics/expected-goals-xg-explained/). statsbomb xG(줄여서 s-xG 라고 하자)와 모델링을 통해서 구한 xG 값을 비교해보고 그 차이를 확인해보았다.
일단 제일 먼저, 그래프를 그려보았다.


<details>
<summary> 코드 </summary>
<div markdown="1">

```python
plt.figure(figsize=(5,5 ))
plt.scatter(df_shot['shot_statsbomb_xg'], lr.predict_proba(X)[::,1])
plt.plot([-0.01, 1.01],[-0.01, 1.01], c='red')
plt.xlabel('s-xG')
plt.ylabel('xG')
```

</div>
</details>
<br>

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/22fe6489-8f7a-4d9c-a74a-5f83d52a80b1" width="500" height="500"/>
</p>


그래프로 보면 두 값이 분명히 차이가 나는 점들이 많이 있는 것 같다. (0.8 부근에 1자로 보이는 점들은 아마 페널티킥 일 듯.) 궁금증 해결을 위해서 두 값의 차이가 가장 큰 5개 슈팅을 추려보았다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/a6e1d3e0-cdbf-43de-aa74-eeaf181bb44b" width="500" height="500"/>
</p>

| match_id | type_name | sub_type_name | player_name | team_name | body_part_name | outcome_name | x | y | shot_statsbomb_xg | xG_diff |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 3753979 | Shot | Open Play | Jermain Defoe | Sunderland | Left Foot | Goal | 105.6 | 35.7 | 0.983099 | 0.869473 |
| 3900512 | Shot | Open Play | Souleymane Camara | Montpellier | Left Foot | Goal | 110.1 | 30.5 | 0.948656 | 0.833520 |
| 3890269 | Shot | Open Play | Daniel Ginczek | VfB Stuttgart | Left Foot | Goal | 106.1 | 38.8 | 0.954949 | 0.826498 |
| 3829464 | Shot | Open Play | Alassane Pléa | OGC Nice | Left Foot | Off T | 109.3 | 30.9 | 0.960538 | 0.812035 |
| 3829413 | Shot | Open Play | Lucas Rodrigues Moura da Silva | Paris Saint-Germain | Right Foot | Goal | 109.2 | 49.4 | 0.964086 | 0.809019 |


골대와 비슷한 거리 및 각도에 있다는 점 말고, 내가 구성한 학습데이터 셋으로는 다른 특이점을 찾기가 어렵다. (역시나..) 예상컨데 앞선 모델에서 수비수의 위치 및 골키퍼 위치 정보를 추가하지 않았으니, 수비수나 골키퍼 위치에 따른 차이라 본다. (아마 1대1 상황이나 골키퍼를 제친 상황이지 않을까?) 고민할 필요 없이 영상을 찾아봤다. 다행히 5개 슈팅 중에 4개를 유튜브에서 찾았다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/d84963c6-fd8c-4b6c-8995-09063ad5f8cd" align="center" width="40%">
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/01ce86d2-8d6d-4044-abaf-471fca93e8a3" align="center" width="40%">
</p>
<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/5829d73b-3e32-4e03-85f1-d5b040dc242b" align="center" width="40%">
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/5ec8f064-9571-4f7c-baeb-c289b6c9861a" align="center" width="40%">

</p>

네 슈팅 모두 골키퍼까지 제친 빈 골대를 향한 슈팅이었다. 역시 수비수와 골키퍼 위치정보 없이 학습된 xG 모형은 한계가 있는 것 같다. 처음부터 슈팅하는 상황에서 다른 선수들의 위치 데이터까지 포함하여서 모델링을 할껄 그랬다. 모델링을 쭉 이어서 하고 나서야 깨달았다 🤣.

# Wrap Up

수준 높은 모델링은 아니지만, 축구경기 데이터 다루는 것은 항상 즐겁다. xG 지표가 나온 이후로 여러 지표가 함께 등장하면서 축구 분석들이 훨씬 풍부해진것 같다. 요즘에는 유럽 클럽중에 데이터 분석팀 없는 곳을 찾기 힘들 정도로 중요성이 커지고 있는 것 같다. 한가지 바램이 있다면 나와 같은 취미 분석가들도 잘 놀 수 있도록 데이터를 더 뿌려줬으면 한다.

# Reference



[logistic regression odd ratio](https://stats.oarc.ucla.edu/other/mult-pkg/faq/general/faq-how-do-i-interpret-odds-ratios-in-logistic-regression/) <br>
[Github: metrica-sports](https://github.com/metrica-sports) <br>
[Github: kloppy](https://github.com/PySport/kloppy)
<br>
[파이썬 축구 데이터 분석](https://class101.net/ko/search?query=%EC%B6%95%EA%B5%AC)
