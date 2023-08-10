---
title: "[DP20] 축구경기에서 공간창출을 지표화 한다면? - 2"
excerpt: "Wide Open Spaces:A statistical technique for measuring space creation in professional soccer 논문 리뷰 및 구현하기"
header: 
    teaser: "https://github.com/jmlee8939/jmlee8939.github.io/assets/58785929/086475a8-01db-43ac-abb9-37b898b26e3b"
tags:
  - 축구 모델링
  - 축구 데이터
  - Space quality
  - Space occupation gain
  - Space generation gain
---

# Preface

이전 글에서 축구경기 EPTS 데이터를 활용해서, Javier Fernandez가 제시한 Pitch control 모형을 만들고 선수 개개인과 팀 단위의 공간에 대한 점유도를 수치화 하였다. 이 글에서는 공간의 중요도를 나타내는 Space quality 값을 계산하고, pitch control 값과 함께 활용하여 특정 시간에 각 선수가 얼마나 중요한 공간을 얼마나 잘 점유하고 있는가를 수치화한다. 이 수치를 활용해서 선수가 얼마나 중요한 공간을 잘 찾아가는지 또 다른 선수들에게 공간을 얼마나 잘 만들어주는지 나타내는 지표를 만들어 낸다. 

"*선수들은 한 경기당 3분정도 공을 소유한다. 중요한 것은 87분동안 어떻게 뛰어다니는지이다. 이것이 좋은 선수와 나쁜 선수를 가른다.*"

 크루이프의 유명한 명언이다. 이 논문의 핵심은 크루이프가 말하는 좋은 선수를 어떻게 객관적으로 수치화 하여 구분해 낼 수 있는가에 대한 방법을 제시하는 것이다. 다시 한번 느끼는 바지만, 바르셀로나라는 축구팀은 참 대단한 것 같다. (명언 + 모델링이라니... 가슴이 웅장해진다.) 

 # Space Quality

축구경기에서 중요한 공간이란..? 축구경기는 골을 많이 넣으면 승리하는 단순한 경기이니까 단순하게 골을 넣기 좋은 공간, 골 넣을 확률이 높은 공간이 가장 중요한 공간이라고 할 수 있겠다. 최근에는 페널티박스를 지키는 중앙수비수와 측면수비수 사이의 하프스페이스라는 공간을 어떻게 공략할 것인가가 많은 축구팀들의 전술적 화제가 되기도 했다. 

이 논문에서는 간단한 아이디어로 공간의 중요도(Space Quality)를 모델링 하였다. 

>1. 공의 위치에 따라서 공간의 중요도가 달라진다.
>2. 수비수들은 항상 중요한 공간들을 지키고 있다.
>3. 상대방의 골대와 가까워 질 수록 중요도가 높아진다.  

여기서 2번 가정 같은 경우, 빠른 역습과 같이 빠른 공수전환 상황에서 수비수들이 중요한 위치를 점유하고 있지 못하는 경우가 생길 수 있다. 따라서 지공 상황에서의 공간 중요도를 가정했다고 볼 수 있다. (바르셀로나 분석관 아니랄까봐... 최근 핫한 빠른 공수전환을 요하는 경기에서의 공간 중요도는 다르게 산출하는게 좋을 것 같다.)

 논문에서는 공의 위치에 따른 수비 선수들의 pitch control 값을 space quality 로 정의하고, 경기 데이터를 활용하여서 이 함수를 학습하였다. 아주 간단한 1-hidden layer 의 Neural network 로 학습을 진행했다. 모델링에 머신러닝 한 스푼 추가한 것이 연구에 흥미를 더 들게 만든다.

## Value of location $$V_{k,l}(t)$$

여기 $$V_{k,l}(t)$$ 가 $$t$$ 시점의 위치의 중요도이다. 이 값은 특정 시점의 공의 위치 $$p_b{t}$$ 와 운동장 내의 어떤 위치 $$p_{k,l}(t)$$ 를 input 으로 하는 함수로 구성된다. 이를 식으로 나타내면 아래와 같다. (k, l 은 가로 세로 나타내는 듯) 명확한 식 구성을 위해서 $$(k, l)$$을 location 을 나타내는 $$l$$ 로 바꾸어 표현하겠다.  

$$V_{l}(t) = fn(p_b(t), p_{l}(t);\theta)$$

이 함수를 학습시켜서 output 을 특정 시점, 특정 위치의 Space value로 활용한다.

## $$fn$$ 학습데이터 구성

어떤 머신러닝 문제를 풀던지 가장 중요한 것은 어떻게 학습 데이터를 구성하느냐가 아닐까 싶다. 데이터가 좋으면 어떤 모델로 학습하든 쉽게 다 소화시킬 수 있으니. $$fn$$ 을 학습하는 과정에서 $$p_l$$, $$p_b$$는 경기 데이터로 주어진다. 따라서 목표하는 target $$\hat{y}$$, $$\hat{V_{l}}$$ 를 어떻게 모델링 할 것인가 하는 아이디어만 제시하면 된다. 논문에서는 해당 시점, 해당 위치의 수비팀 pitch control 값 $$D_l(t)$$를 $$\hat{V_{l}}$$로 다음과 같이 활용하였다. (pitch control 계산은 이전 글 참조)

$$D_l(t) = \sum_d I_d(p_b(t), p_l(t))$$

$$\hat{V}_l(t) = \begin{cases} 1,& D_l(t)>1 \\ D_l(t), & otherwise \end{cases}$$

실제 학습에 사용되는 $$\hat{V}_l(t)$$ 를 시각화 하면 다음과 같다.
검은 점은 특정 시점의 공의 위치이고 붉게 표현된 부분이 수비팀의 pitch control 값이다. (논문에서는 pitch 위의 모든 점들을 학습 set 으로 활용하지는 않고 가로, 세로를 각각 21, 15등분 하여 한 시점당 315 개의 학습 데이터셋이 구성되도록 했다.)
<p align=center>
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/edae0b1f-d53f-4ab5-8c90-f0b73bd7ac03" width="300" height="200"/>
</p>

## $$fn$$ 학습

학습은 아래외 같이 Mean Squared error 를 loss function $$\mathcal{L}$$ 의 minimzation 하는 파라미터 $$\theta$$ 를 찾는 방식으로 진핸된다. 함수가 Neural Network 이기 때문에,  

$$\mathcal{L} = arg$$ 




