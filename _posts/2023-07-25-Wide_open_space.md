---
layout: post
mathjax: true
title: "[DP20] 축구경기에서 공간창출을 지표화 - 1"
excerpt: "Voronoi Diagram 부터 Pitch control 모형 + Wide Open Spaces:A statistical technique for measuring space creation in professional soccer 논문 리뷰 및 구현하기"
header:
tags:
  - Soccer
  - Voronoi Diagram
  - Pitch control
---

# Preface
### *"Long time no see"* <br>

참 오랜만에 글을 써 본다. 머릿속에 데이터 및 축구에 대한 아이디어는 자꾸 떠오르는데 현실적으로 구현해볼 여유가 없는 것 같다. 억지로라도 꾸역꾸역 풀어놓아야겠다. 최근에 다시 축구 분야 연구들에 관심에 불이 붙어서 여기 저기 기웃하였다. 그 중에 CLASS 101에서 김현성님의 축구 데이터를 활용한 데이터 사이언스 강의를 들었는 데 정말 흥미로웠다. (대학원 졸업 이후에 핏투게더 지원을 진지하게 고민했었는 데, 해볼껄 그랬다.) 데이터 분석가 or 사이언티스트는 축구데이터를 어떻게 다루는지 궁금한 사람은 한 번 들어보는 것을 추천한다.  

# Intro 
이번 포스팅의 주제는 **"축구경기에서 공간을 지표화"** 이다.
2008년 펩과르디올라의 바르셀로나가 "티키타카"를 앞세워 트레블을 달성한 이후로 "공간" 과 "점유" 가 현대 축구의 가장 중요한 요소가 되었다. 최근에는 이러한 공간 창출 및 점유의 트렌드 위에 강력한 압박과 빠른 공수전환속도가 가미되고 있다. 골, 어시스트, 패스, 드리블, 슛 등 통게적인 지표와는 다르게 해당 선수가 얼만큼의 공간을 점유하고 있는가?, 좀 더 나아가서 해당 선수의 움직임을 어떻게 평가할 것인가? 와 같이 선수가 창출하는 공간과 움직임에 대한 평가는 단순하게 지표화 하기 대단히 어렵다. 주변 선수들과의 상호작용 및 상황에 따라 완전히 달라지기 때문이다. 하지만, 이처럼 선수의 플레이를 단순히 표현하기 어렵기 때문에 축구가 더 재밌고, 분석해 볼만한 가치가 많기도 하다.   

# EPTS
과거에는 선수들의 통계지표만 활용하여 플레이를 분석했다면, 최근에는 EPTS(Electronic Performance and Tracking Systems) 기술을 통해서 실시간으로 선수들의 좌표 데이터를 생성해낸다. GPS 기기 또는 Computer Vision 기술을 활용해서 이러한 데이터들을 뽑아내는데, 이러한 좌표 데이터가 매우 정교해 지면서 이를 기반으로 선수들의 플레이를 모델링하는 것이 가능해졌다. 너무나 감사하게도 Metrica_sports 에서 3경기 분량의 ETPS 예시 경기 데이터를 무료로 공개하고 있고, kloppy package 에서 축구 데이터 분석을 위한 다양한 전처리 함수 및 시각화 함수를 제공하고 있어서 데이터를 이것 저것 다뤄볼 수 있다.

[metrica-sports](https://github.com/metrica-sports) <br>
[kloppy](https://github.com/PySport/kloppy)

# Voronoi Diagram
Voronoi Diagram 은 평면을 특정 점까지의 거리가 가장 가까운 점의 집합으로 분할한 그림이다. 들로네 삼각분할과 쌍대관계이며, 머신러닝의 관점에서 1-nn 모델의 decision boundary 라고 볼 수 도 있겠다. 이는 축구경기에서 선수들의 공간점유를 표현하하는 방법으로도 활용되었다. 선수의 현재 속도, 신체 능력과 별개로 현재 위치만 고려했을 때, 특정 공간과 가장 가까운 선수가 공간을 차지할 것이라는 단순한 가정을 한다면 Voronoi Diagram 으로 공간 점유를 표현할 수 있다.

![out](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/bbb1652d-e483-4bf1-b143-ed0f4e779713)
> Sample match 1의 Voronoi Diagram 시각화 예시

선수들이 점유하는 공간을 단순하고 직관적으로 표현하기에는 좋지만, 선수들의 움직임이나 능력치가 전혀 반영되어 있지 않기 때문에 선수들의 움직임과 실제 선수가 점유하고 있는 공간을 표현하기 위해서는 좀 더 디테일한 모형들이 필요하다.

# Pitch control
말그대로 "경기장 점유" 모형이다. 특정 공간이 어떤 선수 또는 팀으로부터 점유되고 있는지 확률적으로 나타내는 모형이다. Voronoi Diagram 의 개념을 확장해서 선수들의 속도, 공과의 거리, 공을 컨트롤 할 확률 등 여러가지를 고려한 다양한 모형들이 제시되고있다. 이번에 구현해볼 Pitch control 모형은 Javier Fernandez 의 Wide Open Spaces.. 논문에 나오는 모형이다. (논문을 읽고 나서 찾아보니 바르셀로나 FC 에서 스포츠 분석팀 리더까지 맡았던 대단한 분이었다...)
해당 모형은 선수의 속도, 공까지의 거리까지 고려한 Pitch control 모형이다.

![pitchcontrol](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/75bcd197-3066-4851-8078-1312e48cc2e9)
> Wide open space : pitch control model

# Wide Open space / Pitch control
본 논문에서 구현한 pitch control 모형은 2차원의 운동장 공간 위에 하나의 선수가 만들어내는 공간의 점유를 하나의 bivariate gaussian distribution 으로 표현하는 것을 기본으로 한다. 여기서 선수의 움직임을 반영하여, distribution 의 중심의 위치와 covariance matrix 의 크기와 방향을 조정한다. 

## Influence function 
$t$ 시점에서 플레이어 $i$ 의 pitch 위 $p$지점의 영향력을 나타내는 함수를  Influence function을 $I_i(p,t)$ 라고 정의한다.

이 $I_i(p,t)$ 값은 아래와 같이 정의된다.

$$ I_i(p,t) = {f_i(p,t)\over f_i(p_i(t), t)}$$

<br>

$p_i(t)$는 해당 시점의 플레이어 
$i$의 위치이고,
<br>
$f_i(p, t)$ 는 bivariate gaussian distribution의 PDF로 표현된 함수이다.  

$$  \mathbf{X_t} \sim \mathcal{N}(\mu_t, \Sigma_t) $$

$$f_i(p, t) = \mathbf{pdf} \space of \space \mathbf{X}$$



![image](https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/360af4a4-3f7d-4c2c-9d1b-b8e4309aeba6)
> L) 단순 bivariate normal distribution 기반의  $f_i(p,t)$, <br>
R) 플레이어의 속도와 방향이 고려된 Wide open space 의 $f_i(p,t)$   

###  1) Simple $f$ modeling
왼쪽 $f$ 는 플레이어의 속도를 고려하지 않고 거리만을 가지고 설계한 것이다.

$$  \mu_t = p_i(t),  \Sigma_t = k * \begin{vmatrix} 1 & 0 \\ 0 & 1\end{vmatrix}$$

위 모델은 선수가 뛰어가고 있는 방향에 대한 고려가 없다. 선수의 움직임과 반대방향으로 공이 전달되었을 때, 대부분의 선수들은 역동작에 걸리고 안정적인 볼 컨트롤이 어려워진다. 따라서 조금만 생각해 보아도, 왼쪽의 모델보다는 오른쪽 형태의 gaussian distributon 이 더 적합해보인다. 

### 2) Wide open space $f$ modeling
오른쪽 그림은 저자가 제안한 $f$ 이다. 
$$\mu_t =  p_i(t) + \vec{s}_i(t)*0.5 $$
$\vec{s}_i(t)$ 는 플레이어 $i$의 속도이고, distribution 의 중심을 속도 방향으로 평행이동시켜 주는 역할을 한다.



