---
title: "[DP20] 축구경기에서 공간창출을 지표화 한다면? - 1"
excerpt: "Wide Open Spaces:A statistical technique for measuring space creation in professional soccer 논문 리뷰 및 구현하기"
header: 
    teaser: "https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/086475a8-01db-43ac-abb9-37b898b26e3b"
tags:
  - 축구 모델링
  - 축구 데이터
  - Voronoi Diagram
  - Pitch control
---
 
# Preface
### *"Long time no see"* <br>

들여다보고 싶은 축구 데이터 분석 주제는 자꾸 떠오르는데 현실적으로 구현해볼 여유가 없다. 억지로 시간내어 꾸역꾸역 풀어놓아야겠다. 최근에 다시 축구 분야 연구들에 관심에 불이 붙어서 여기 저기 기웃하였다. 그 중에 CLASS 101에서 김현성님의 \<파이썬 축구 데이터 분석\> 강의를 우연히 들었는 데 정말 흥미로웠다. (대학원 졸업 이후에 핏투게더 지원을 진지하게 고민했었는 데, 해볼껄 그랬다.) 데이터 분석가 or 사이언티스트는 축구데이터를 어떻게 다루는지 궁금한 사람은 한 번 들어보는 것을 추천한다.

# Intro 
이번 포스팅의 주제는 **"축구경기에서 공간을 지표화"** 이다.
2008년 펩과르디올라의 바르셀로나가 "티키타카"를 앞세워 트레블을 달성한 이후로 "공간" 과 "점유" 가 현대 축구의 가장 중요한 요소가 되었다. 최근에는 이러한 공간 창출 및 점유의 트렌드 위에 강력한 압박과 빠른 공수전환속도가 가미되고 있다. 골, 어시스트, 패스, 드리블, 슛 등 통계적인 지표와는 다르게 해당 선수가 얼만큼의 공간을 점유하고 있는가? 좀 더 나아가서 해당 선수의 움직임을 어떻게 평가할 것인가? 와 같이 선수가 창출하는 공간과 움직임에 대한 평가는 단순하게 지표화 하기 대단히 어렵다. 주변 선수들과의 상호작용 및 상황에 따라 완전히 달라지기 때문이다. 하지만, 이처럼 선수의 플레이를 단순히 표현하기 어렵기 때문에 축구가 더 재밌고, 분석해 볼만한 가치가 많기도 하다.   

# EPTS
과거에는 선수들의 통계지표만 활용하여 플레이를 분석했다면, 최근에는 EPTS(Electronic Performance and Tracking Systems) 기술을 통해서 실시간으로 선수들의 좌표 데이터를 생성해낸다. GPS 기기 또는 Computer Vision 기술을 활용해서 이러한 데이터들을 뽑아내는데, 이러한 좌표 데이터가 매우 정교해 지면서 이를 기반으로 선수들의 플레이를 모델링하는 것이 가능해졌다. 너무나 감사하게도 Metrica_sports 에서 3경기 분량의 ETPS 예시 경기 데이터를 무료로 공개하고 있고, kloppy package 에서 축구 데이터 분석을 위한 다양한 전처리 함수 및 시각화 함수를 제공하고 있어서 데이터를 이것 저것 다뤄볼 수 있다.

[metrica-sports](https://github.com/metrica-sports) <br>
[kloppy](https://github.com/PySport/kloppy)

# Voronoi Diagram
Voronoi Diagram 은 평면을 특정 점까지의 거리가 가장 가까운 점의 집합으로 분할한 그림이다. 들로네 삼각분할과 쌍대관계이며, 머신러닝의 관점에서 1-nn 모델의 decision boundary 라고 볼 수 도 있겠다. 이는 축구경기에서 선수들의 공간점유를 표현하는 방법으로도 활용되었다. 선수의 현재 속도, 신체 능력과 별개로 현재 위치만 고려했을 때, 특정 공간과 가장 가까운 선수가 공간을 차지할 것이라는 단순한 가정을 한다면 Voronoi Diagram 으로 공간 점유를 표현할 수 있다.

![out](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/bbb1652d-e483-4bf1-b143-ed0f4e779713)
> Sample match 1의 Voronoi Diagram 시각화 예시

선수들이 점유하는 공간을 단순하고 직관적으로 표현하기에는 좋지만, 선수들의 움직임이나 능력치가 전혀 반영되어 있지 않기 때문에 선수들의 움직임과 실제 선수가 점유하고 있는 공간을 표현하기 위해서는 좀 더 디테일한 모형들이 필요하다.

# Pitch control
말그대로 "경기장 점유" 모형이다. 특정 공간이 어떤 선수 또는 팀으로부터 점유되고 있는지 확률적으로 나타내는 모형이다. Voronoi Diagram 의 개념을 확장해서 선수들의 속도, 공과의 거리, 공을 컨트롤 할 확률 등 여러가지를 고려한 다양한 모형들이 제시되고있다. 이번에 구현해볼 Pitch control 모형은 Javier Fernandez 의 Wide Open Spaces.. 논문에 나오는 모형이다. (논문을 읽고 나서 찾아보니 바르셀로나 FC 에서 스포츠 분석팀 리더까지 맡았던 대단한 분이었다...)
해당 모형은 선수의 속도, 공까지의 거리까지 고려한 Pitch control 모형이다.

![pitchcontrol](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/75bcd197-3066-4851-8078-1312e48cc2e9)
> Wide open space : pitch control model

# Wide Open space / Pitch control
본 논문에서 구현한 pitch control 모형은 2차원의 운동장 공간 위에 하나의 선수가 만들어내는 공간의 점유를 하나의 bivariate gaussian distribution 으로 표현하는 것을 기본으로 한다. 그 과정에 선수의 움직임, 공의 위치 등을 반영하여 distribution 의 중심의 위치와 covariance matrix 의 크기와 방향을 조정한다. 

## Influence function 
특정 시간 $$t$$ 에서 플레이어 $$i$$의 pitch 위 $$p$$지점의 영향력을 나타내는 함수를  Influence function을 $$I_i(p,t)$$ 라고 정의한다.

이 $$I_i(p,t)$$ 값은 아래와 같이 정의된다.

$$ I_i(p,t) = {f_i(p,t)\over f_i(p_i(t), t)}$$

해당시점의 위치 $$ p_i(t)$$ 는 해당 시점의 플레이어 $$i$$ 의 위치이고, $$f_i(p, t)$$ 는 bivariate gaussian distribution의 PDF로 표현된 함수이다.  

$$  \mathbf{X_t} \sim \mathcal{N}(\mu_t, \Sigma_t) 
$$

$$f_i(p, t) = \mathbf{pdf} \space of \space \mathbf{X}$$



![image](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/360af4a4-3f7d-4c2c-9d1b-b8e4309aeba6)
> L) 단순 bivariate normal distribution 기반의  $$f_i(p,t)$$, <br>
R) 플레이어의 속도와 방향이 고려된 Wide open space 의 $$f_i(p,t)$$   

###  1) Simple $$f$$ modeling
왼쪽 $$f$$ 는 플레이어의 속도를 고려하지 않고 거리만을 가지고 설계한 것이다.

$$  \mu_t = p_i(t),  \Sigma_t = k * \begin{vmatrix} 1 & 0 \\ 0 & 1\end{vmatrix}$$

위 모델은 선수가 뛰어가고 있는 방향에 대한 고려가 없다. 단순히 선수의 현재 위치와 가까울수록 공간에 대한 점유도가 높아진다. 하지만, 실제 경기장위에서는 선수의 움직임과 반대방향으로 공이 전달되었을 때, 많은 선수들이 역동작에 걸려 안정적인 볼 컨트롤에 여러움을 겪는 모습을 쉽게 볼 수 있다. 따라서 조금만 생각해 보아도, 왼쪽의 모델보다는 진행방향에 있는 공간들에 대해 가중치가 포함된 오른쪽 형태의 gaussian distributon 이 더 적합해보인다. 

### 2) Wide open space $$f$$ modeling
오른쪽 그림은 저자가 제안한 $$f$$ 이다. 

$$\mu_t =  p_i(t) + \vec{s}_i(t)*0.5$$ 

플레이어 $$i$$의 속도 $$\vec{s}_i(t)$$ 는 distribution 의 중심을 그 방향으로 평행이동시켜 주는 역할을 한다. 
이때, $$f$$ 의 covariance matrix $$\Sigma$$ 를 다음과 같이 decomposition 하면, eigenvenctor 로 구성된 $$V$$ 오 eigenvalues 값의 diagonal vector $$L$$ 로 표현가능하다.  


$$\Sigma = \mathbf{VLV}^{-1}$$

이때, $$ \sqrt{\mathbf{L}} =  \mathbf{S} $$ 이라 하면 다음과 같이 회전행렬 $$\mathbf{R}$$ 과 scale factor $$\mathbf{S}$$ 로 표현가능하다.

$$\Sigma = \mathbf{RSSR}^{-1}$$

$$\mathbf{R} = \begin{vmatrix} cos(\theta) & -sin(\theta) \\ sin(\theta) & cos(\theta)\end{vmatrix}$$

특정 시점 $$t$$ 에 플레이어 $$i$$ 의 scale factor $$S_i(t)$$ 는 다음과 같이 표현된다.

$$Srat_i(s) = {s^{2}\over13^{2}}$$

$$S_i(t) = \begin{vmatrix} {R_i(t) + (R_i(t)Srat_i(\vec{s}(t)))\over 2} & 0 \\ 0 & {R_i(t) - (R_i(t)Srat_i(\vec{s}(t)))\over 2}\end{vmatrix}$$

$$\Sigma_i(t) = R(\theta,t)S_i(t)S_i(t)R(\theta_i(t), t)^{-1}$$

여기서, $$Srat_i(s)$$ 는 일반적인 선수들의 최고 속도를 13m/s 로 고정하고 현재 속도 $$s$$ 와 최고속력의 비율을 나타낸다. 이 값이 커지면 커질 수록 속도 방향으로 길고 가는 형태의 gaussian 분포가 나타나게 된다. $$R_i(t)$$ 는 선수-공과의 거리와 영향력 범위를 나타내는 값인데 본 연구에서는 도메인 지식을 활용해서 [4,10] 의 값을 가지는 함수로 지정하였다. 공과의 거리가 가까울수록 선수가 점유하는 공간은 좁아진다는 개념을 기초로 하고 있다. 공이 가까이 있는 경우에는 볼을 컨트롤 한다거나, 볼을 가진 선수들을 마크하는 상황이 많기 때문에 오히려 점유하는 공간이 제한된다. 거리-영향력 범위 함수는 아래 그래프와 같은 값을 가진다.

<p align=center>
<img src="https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/6fe3c80a-8140-4e66-9669-2f818d8310a7" width="300" height="200"/>
</p>

## 최종 Pitch Control 모형

드디어 선수, 시점, 공의 위치에 따른 Inflence function $$I_i(p,t)$$ 에 대한 설명을 마무리했다. 이를 활용해서 모든 경기장 위의 점, 시점에 대한 pitch control $$PC(p,t)$$
값은 다음과 같이 구성한다. 

$$PC(p,t) = \sigma(\Sigma_i I(p,t)-\Sigma_j I(p,t))$$

여기서 $$\sigma$$ 는 logistic function 으로 logistic regression, neural network 의 activation function 에서 자주 활용되는 함수이다. 여기서는  홈팀의 Influence 합과 어웨이 팀의 Influence 합을 뺀 값을  0-1사이의 값으로 변환시켜주는 역할을 한다. 이 PC의 값을 활용하여서 0.5 이상/미만으로 공간을 구분한다면 특정 공간이 어떤 팀에 의해서 점유되고 있는지를 표현하는 classifier 로 이해할 수도 있을 것이다. 최종적으로 구성된 특정 시점의 PC를 시각화하면 다음과 같다.


![image](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/31e78d4f-32d2-40be-bfad-57a77fc4db37)

이 모형을 통해서 매 시점마다의 선수들이 점유하는 공간을 표현할 수 있고, 이 논문에서는 이를 활용해서 선수들의 공간 창출 움직임을 수치화 하는 방법까지 제시한다. 그 내용은 다음 글에서 계속해서 설명하려고 한다. (쓰다보니까 글이 너무 길어짐.)

# 모델 구현

## 데이터 불러오기 

우선 데이터는 [metrica-sports](https://github.com/metrica-sports), 또는 [kloppy](https://github.com/PySport/kloppy) 의 ETPS, event 데이터 셋을 사용한다. 가장 간단한 방법은 klopy package를 통해 불러오는 것이다.


<details>
<summary> 데이터 불러오기 코드 </summary>
<div markdown="1">

```python
from kloppy import metrica

dataset = metrica.load_open_data(
    match_id=1,
    
    # Optional arguments
    limit=2000,
    coordinates="metrica"
)

dataset.to_df().head()
```

</div>
</details>


<details>
<summary> 불러온 데이터 보기 
</summary>
<div markdown="1">

| period_id | timestamp | frame_id | ball_state | ball_owning_team_id | ball_x |  ball_y |  ball_z | home_11_x | home_11_y |     ... | away_22_d | away_22_s | away_23_x | away_23_y | away_23_d | away_23_s | away_24_x | away_24_y | away_24_d | away_24_s |      |
|----------:|----------:|---------:|-----------:|--------------------:|-------:|--------:|--------:|----------:|----------:|--------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|------|
|         0 |         1 |     0.00 |          1 |                None |   None | 0.45472 | 0.61291 |      None |   0.00082 | 0.51762 |       ... |      None |      None |   0.43693 |   0.94998 |      None |      None |   0.37833 |   0.72617 |      None | None |
|         1 |         1 |     0.04 |          2 |                None |   None | 0.49645 | 0.59344 |      None |   0.00096 | 0.51762 |       ... |      None |      None |   0.43693 |   0.94998 |      None |      None |   0.37833 |   0.72617 |      None | None |
|         2 |         1 |     0.08 |          3 |                None |   None | 0.53716 | 0.57444 |      None |   0.00114 | 0.51762 |       ... |      None |      None |   0.43693 |   0.94998 |      None |      None |   0.37833 |   0.72617 |      None | None |
|         3 |         1 |     0.12 |          4 |                None |   None | 0.55346 | 0.57769 |      None |   0.00121 | 0.51762 |       ... |      None |      None |   0.43644 |   0.94962 |      None |      None |   0.37756 |   0.72527 |      None | None |
|         4 |         1 |     0.16 |          5 |                None |   None | 0.55512 | 0.59430 |      None |   0.00129 | 0.51762 |       ... |      None |      None |   0.43580 |   0.95023 |      None |      None |   0.37663 |   0.72457 |      None | None |ß

</div>
</details>

<br>

## 전처리

사실 모델링보다 데이터 전처리가 훨씬 오래걸리는 일이지만... 안그래도 긴 글이 더 길어지니까 스킵! 프레임마다 제공되는 선수들의 위치 정보를 활용해서 프레임 마다의 속도를 계산하고 홈, 어웨이 팀 선수들을 보기 좋게 라벨링 했다. 그리고 이벤트 데이터까지 조인하여서 각 프레임마다 어떤 플레이가 일어났는지 어느 팀이 공을 소유하고 있는지가 함께 나타나는 데이터 셋을 구성했다.

전처리가 끝난 데이터셋에서 특정 프레임(1000)에서의 선수의 위치, 속도, 공의 위치를 별도의 변수로 선언해둔다. (의식의 흐름대로 진행하다보니 코드가 깔끔하지 않으니 주의... 자세한 전처리 코드는 [Wide-Open-Space](https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model)에 정리해 두었음)


<details>
<summary> 전처리 코드 </summary>
<div markdown="1">

```python
frame = 1000

df1 = df.loc[:, [i for i in df.columns if 'v' not in i]]
df2 = df.loc[:, [i for i in df.columns if 'v' in i]]
positions = df1[df1['Frame'] == frame].iloc[:,3:].drop(['Ball_x', 'Ball_y'], axis=1).dropna(axis=1).iloc[0,:]
velocities = np.array(df2[df1['Frame'] == frame].drop(['Ball_x_v', 'Ball_y_v', 'Ball_v_abs'], axis=1).dropna(axis=1).iloc[0,:])
points = np.array([[positions[2*i], positions[2*i+1]] for i in range(len(positions)//2)])
velocities = np.array([[velocities[3*i], velocities[3*i+1]] for i in range(len(velocities)//3)])
players = np.array([positions.index[2*i].split('_')[0] for i in range(len(points))])
ball_x, ball_y = df.loc[df['Frame'] == frame, ['Ball_x', 'Ball_y']].values[0]
ball = np.array([ball_x, ball_y])
```

</div>
</details>

<details>
<summary> 전처리 후 최종 데이터 </summary>
<div markdown="1">

```py
#... 전처리 끝난 Dataframe df
df.head()
```

| Period | Frame | Time [s] | H11_x |   H11_y |     H1_x |     H1_y |     H2_x |     H2_y |     H3_x |      ... | A10_v_abs |  A12_x_v | A12_y_v | A12_v_abs | A13_x_v | A13_y_v | A13_v_abs | A14_x_v | A14_y_v | A14_v_abs |     |
|-------:|------:|---------:|------:|--------:|---------:|---------:|---------:|---------:|---------:|---------:|----------:|---------:|--------:|----------:|--------:|--------:|----------:|--------:|--------:|----------:|-----|
|      2 |     1 |        1 |  0.04 | 0.08528 | 32.80184 | 33.95392 | 44.41896 | 35.04904 | 33.22684 | 32.16408 |       ... |      NaN |     NaN |       NaN |     NaN |     NaN |       NaN |     NaN |     NaN |       NaN | NaN |
|      3 |     1 |        2 |  0.08 | 0.09984 | 32.80184 | 33.95392 | 44.41896 | 35.04904 | 33.22684 | 32.16408 |       ... | 2.910703 |     NaN |       NaN |     NaN |     NaN |       NaN |     NaN |     NaN |       NaN | NaN |
|      4 |     1 |        3 |  0.12 | 0.11856 | 32.80184 | 33.95392 | 44.41896 | 35.04904 | 33.22684 | 32.16408 |       ... | 3.028292 |     NaN |       NaN |     NaN |     NaN |       NaN |     NaN |     NaN |       NaN | NaN |
|      5 |     1 |        4 |  0.16 | 0.12584 | 32.80184 | 33.92688 | 44.41556 | 35.03448 | 33.31184 | 32.18176 |       ... | 3.144990 |     NaN |       NaN |     NaN |     NaN |       NaN |     NaN |     NaN |       NaN | NaN |
|      6 |     1 |        5 |  0.20 | 0.13416 | 32.80184 | 33.90088 | 44.38292 | 35.01056 | 33.33224 | 32.18592 |       ... | 3.163107 |     NaN |       NaN |     NaN |     NaN |       NaN |     NaN |     NaN |       NaN | NaN |

</div>
</details>

<br>

## Pitch control model

논문에서 정의한 Influence radius 와 Influence function 을 구성한다.
<details>
<summary> Influence radius, Influence function 코드 </summary>
<div markdown="1">

```py
def influence_radius(ball, position):
    distance = np.linalg.norm(ball - position)
    output = np.minimum(3/200*(distance)**2 + 4, 10)
    return output
```

```py
def influence_function(position, locations, velocity, ball):
    mu = position + 0.5*velocity
    srat = (velocity[0]**2 + velocity[1]**2)/13**2
    theta = np.arctan(velocity[1]/(velocity[0]+1e-7))
    R = np.array([[np.cos(theta), -np.sin(theta)],[np.sin(theta), np.cos(theta)]])
    R_inv = np.array([[np.cos(theta), np.sin(theta)],[-np.sin(theta), np.cos(theta)]])
    Ri = influence_radius(ball, position)
    S = np.array([[(1 + srat)*Ri/2, 0],[0, (1-srat)*Ri/2]])
    Cov = np.matmul(np.matmul(np.matmul(R, S), S), R_inv)
    new_gaussian = multivariate_normal(mu, Cov)
    out = new_gaussian.pdf(locations)
    out /= new_gaussian.pdf(position)
    return out
```

</div>
</details>

이를 시각화 하면,
<details>
<summary> 시각화 코드 </summary>
<div markdown="1">

```py
x, y = np.mgrid[0:104:0.1, 0:68:0.1]
locations = np.concatenate([x.reshape(-1,1), y.reshape(-1,1)], axis=1)

s_h, s_a = 0, 0
for i, j, k in zip(players, points, velocities):
    if 'H' in i:        
        s_h += influence_function(j, locations, k, ball)
    else :
        s_a += influence_function(j, locations, k, ball)

z = 1 / (1 + np.exp(- s_h + s_a))

fig, ax = draw_pitch('white', 'black')
ax.contourf(x, y, z.reshape(1040, 680), alpha=0.8)
for t, p, v in zip(players, points, velocities):
    if 'H' in t:
        color = 'red'
    else:
        color = 'blue'
    ax.scatter(p[0], p[1], c=color, s=20)
    ax.arrow(p[0], p[1], v[0], v[1], color='green', head_width = 1)

ax.scatter(ball_x, ball_y, color='black')
```

</div>
</details>

![image](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/086475a8-01db-43ac-abb9-37b898b26e3b)



# Wrap up 
 역시 바르셀로나가 축구를 괜히 잘하는 게 아니구나... 이정도로 디테일한 모델링과 분석을 하니까 잘할 수 밖에. 생각보다 너무 시간이 많이 들었다. 이렇게 진지모드로 글을 쓰려고 했던 것은 아닌데 말이지. 정리하다 보면 점점 요령이 쌓이겠지.

# References
[metrica-sports](https://github.com/metrica-sports) <br>
[kloppy](https://github.com/PySport/kloppy)
<br>
[파이썬 축구 데이터 분석-김현성님](https://class101.net/ko/search?query=%EC%B6%95%EA%B5%AC)
<br>
[Metrica-pitch- control](https://github.com/anenglishgoat/Metrica-pitch-control)
<br>
[Wide Open Spaces: A statistical technique for measuring space creation in professional soccer](https://static.capabiliaserver.com/frontend/clients/barca/wp_prod/wp-content/uploads/2018/05/Wide-Open-Spaces.pdf)