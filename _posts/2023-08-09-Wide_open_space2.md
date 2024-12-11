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

축구경기에서 중요한 공간이란..? 축구경기는 골을 많이 넣으면 승리하는 단순한 경기이니까 단순하게 골 넣을 확률이 높은 페널티 박스 근처라 할 수 있겠다. 최근에는 페널티박스를 지키는 중앙수비수와 측면수비수 사이의 하프스페이스라는 공간을 어떻게 공략할 것인가가 많은 축구팀들의 전술적 화제가 되기도 했다. 

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
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/edae0b1f-d53f-4ab5-8c90-f0b73bd7ac03" width="300" height="200"/>
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

### Train - Test Split
이미 ./Data/dataset.csv 에 학습 데이터셋 구성되어 있을때를 가정한다.


<details>
<summary> CODE
</summary>
<div markdown="1">

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

</div>
</details>
<br>

### Simple Multi-layer neural network model
논문에서는 1-hidden layer model 이었는 데, 내부 hidden node 갯수나 activation function 과 같은 세부정보가 없어서 새로 모델링 하였다.  


<details>
<summary> CODE
</summary>
<div markdown="1">

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

</div>
</details>
<br>

### 구조 확인

<details>
<summary> CODE </summary>
<div markdown="1">

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

</div>
</details>
<br>

### Traning

<details>
<summary> CODE
</summary>
<div markdown="1">

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

</div>
</details>

<br>

# Evaluation 
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

x축은 Target space quality 값 $$V_l(t)$$ 이고, y축은 예측 space quality 값 $$\hat{V}_l(t)$$ 이다. 모델이 데이터를 아주 잘 설명하고 있다고 보긴 어렵지만, 우리가 학습시키고자 하는 방향성은 어느정도 학습 된 것 같다. (공의 위치에 따른 수비선수들의 위치를 학습 시키다보니 운동장 위의 상황에 따른 데이터의 분산이 존재한다. 예를 들어서 역습상황시의 수비위치와 지공상황의 수비위치는 분명 다르기에 학습 데이터의 target 값에 분산이 존재하고 모델링의 설명력 자체의 한계가 존재할 수 밖에 없다.)

<p align=center>
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/7babe5e7-6f70-4982-a26d-7cb17249a8e3" width="500", height="250">
</p>

#### 예측값 plot

학습된 모델로 부터 산출한 공에 위치에 따른 Space quality 를 시각화하면 왼쪽 아래 그림과 같다. 공격방향, 공 근처에 있는 공간들의 중요도가 높게 나타난다. 여기서 상대방 골대와 가까워질수록 더 중요하다는 도메인 지식을 추가하여(방법은 위에서 소개한 상대방 골대에 까워질 수록 가중치 부여) 최종적으로 오른쪽 아래 형태의 Space quality 모델을 구현하였다.

<p align=center>
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/6c737076-1838-42d0-b611-19a6e15ab73a" align="center" width="40%">
<img src="https://github.com/jmlee8939/Wide-Open-Space_Pitch_Control_Model/assets/58785929/cd42e7b6-b6e2-4ce7-ba2a-2408eea7ed68" align="center" width="40%"/>
</p> 

# Wrap Up

처음 이 논문을 읽고 한번 구현해봐야지.. 라는 생각을 하고 정리를 해서 글을 쓰기까지 약 한 달정도 걸렸다. 여유 시간이 있을때마다 조금씩 구현했는데 정리하고보니 별 내용이 없는 거 같지만 하는 과정에서는 꽤나 품이 많이 들었다. 세계최고의 축구팀인 바르셀로나 데이터 분석팀을 맡았던 분의 연구결과이기에 실제 최고 레벨의 축구를 구사하는 팀이 어떻게 데이터를 활용하고 있는 가를 간접적으로나 체험할 수 있었다. 역시 데이터는 누가 어떻게 다루느냐에 다라서 그 가치가 결정되는 것 같다.

[Github: metrica-sports](https://github.com/metrica-sports) <br>
[Github: kloppy](https://github.com/PySport/kloppy)
<br>
[파이썬 축구 데이터 분석-김현성님](https://class101.net/ko/search?query=%EC%B6%95%EA%B5%AC)
<br>
[Metrica-pitch- control](https://github.com/anenglishgoat/Metrica-pitch-control)
<br>
[Wide Open Spaces: A statistical technique for measuring space creation in professional soccer](https://static.capabiliaserver.com/frontend/clients/barca/wp_prod/wp-content/uploads/2018/05/Wide-Open-Spaces.pdf)