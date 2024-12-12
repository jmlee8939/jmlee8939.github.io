---
title: "손흥민 선수는 PL에서 얼마나 잘하고 있는 걸까?"
excerpt: "프리미어리그 선수 데이터 visualization 및 간단한 ranking system 으로 알아보는 손흥민 선수."
categories: "Football"
permalink: "/Football/sony_pl/"
header:
  teaser: "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/puskas son.png?raw=true"
tags: 
  - Soccer
  - Premier League
  - Son Heung-min
---
  
# Intro
이번 포스팅의 주제는 **"손흥민 선수는 프리미어리그에서 얼마나 잘하고 있는가?"** 이다.
당연하게도 세계에서 가장 치열한 리그인 프리미어리그에서 뛰고 있다는 것만으로도 엄청난 일이다.
손흥민 선수는 뛰어난 프리미어리그 선수들 중에서도 단연 빛나는 선수가 되었다.
최근 몇 년간 손흥민 선수는 이미 대한민국의 **국뽕**이 담기엔 너무 큰 선수가 된 듯 하다. <br>
(프리미어리그 PFA Best 11에다, 푸스카스 수상)<br>

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/puskas son.png?raw=true" width = 500 height = 300>
</p>


<br>

21-22 시즌 현재, 토트넘이 흔들리며 팀과 함께 어려운 시즌을 보내고 있지만, 
손흥민 선수는 15-16 데뷔 이후 20-21 시즌 까지 꾸준히 발전하는 모습을 보여주었다. 
본 포스팅에서는 손흥민 선수와 다른 PL 공격수들의 각 시즌별 활약을 데이터로 살펴보고자 한다.
- Data 는 후스스코어드에서 크롤링 [참조](https://github.com/jmlee8939/whoscored_crawling) <br>
- Visualization 은 python matplotlib package 활용 [참조](https://github.com/jmlee8939/Sonny_stats_visualization)

# 축구선수를 평가할 때 어떤 지표를 활용해야 할까?

축구는 골을 넣고 그 골로 승부가 결정나는 단순한 게임이다.
공격수의 가치는 골을 얼마나 많이 생산해 내는지로 결정된다. 축구 기사를 보다보면 **'공격수는 골로 말한다'** 라는 말이 자주 등장하는 것도 그 때문이다.

하지만, 꼭 골이나 어시스트로 직접 득점을 만들어 내지 않고 활동량이나 움직임 처럼 다양한 방법으로
팀의 승리에 기여하는 선수도 있을 것이다. 예를 들어 해버지 박지성 선수처럼 
많은 공격포인트를 기록하지는 못하지만 엄청난 활동량, 수비 가담, 영리한 움직임으로 팀의 승리를 이끄는 타입도 있다.

축구선수, 특히 공격수의 능력을 쉽게 비교하기 위해서 다음과 같은 2가지 지표에 주목하였다.
> - 공격포인트(골 + 어시스트)
> - 평점 

공격포인트는 그 선수의 **생산력**을 나타내는 지표이고, 
평점은 공격포인트 만으로 표현되지 않는 **경기 영향력**이 포함된 지표라고 생각하였다.<br>
물론 분명히 공격포인트와 평점의 관계는 독립적이지 않고,
공격포인트를 기록한 경기에서 그 선수의 평점이 높게 책정되었을 것이다.
하지만, 평점은 공격포인트로 표현되지 않은 플레이도 분명 반영된 값으로 
축구 전문가들이 제시하는 **고오급 정보**로서 활용하지 않을 수 없었다.


# 공격포인트 & 평점 plot

> - 프리미어리그 선수 데이터 : whoscored.com 에서 crawling
> - visualization method : python matplotlib, seaborn package 활용
> - [github/sonny_stats_visualziation](https://github.com/jmlee8939/Sonny_stats_visualization) 참고

리버풀이 30년만에 리그 우승을 하였던 19-20 시즌의 공격포인트 & 평점 plot은 다음과 같다.

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/son_19_20_PL.png?raw=true" width = 500 height = 300>
</p>

눈에 띄는 건 역시 **Kevin De Bruyne** 선수이다. <br> 
공격포인트와 평점에서 독보적인 시즌을 보냈고, 실제 올해의 선수상 수상자 이기도 하다.<br>
또 눈에 띄는 선수는 **Trent Alexander-Arnold** 로 웬만한 공격수 보다 높은 공격포인트를 기록하고 있다. <br>
**손흥민** 선수도 이 시즌 리그 탑급 퍼포먼스를 보낸 것으로 나타난다.

# 손흥민의 20-21 season

저번 시즌(2020-2021 season) **손흥민**은 **케인**과 함께 엄청난 시즌을 보냈다.
아쉽게도 팀은 유로파 컨퍼런스 진출에 그쳤지만, 두 선수는 커리어 하이 시즌을 보냈다.
손흥민은 17골 10도움, 케인은 23골 14도움(득점왕 + 도움왕)으로 나란히 리그 베스트 11에 들었다.

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/son_2021_PL_no_line.png?raw=true" width = 500 height = 300>
</p>

그림에도 나타나듯 손흥민과 케인은 다른 프리미어리그 공격수들에 비해 독보적인 시즌을 보낸 것을 확인할 수 있다.<br>
(2021 시즌 한정 손흥민 > 마네 = 스털링)

# Productive & Influential player

공격포인트 & 평점 plot 을 활용해서 선수의 특성을 구분할 수 있다.
plot 에서 보면 공격수(AM, FW)들의 공격포인트와 평점은 마치 선형관계처럼 보인다.
평점(선수들의 퍼포먼스)를 독립변수, 공격포인트를 종속변수로 하여 `단순 선형회귀선`을 그어보았다.

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/son_2021_PL.png?raw=true" width = 500 height = 400>
</p>

선형 회귀선은 PL 선수들의 평점과 공격포인트의 상관관계를 표현하고, 
이 선형 회귀선 위쪽의 선수들은 퍼포먼스 대비 공격포인트 생산성이 좋은 선수들,
회귀선 아래쪽의 선수들은 상대적으로 공격포인트 생산량보다 경기 영향력이 좋은 선수들이라고 할 수 있다.

- Productive player : 살라, 손흥민, 케인
- Influential player : 잭 그릴리쉬, 케빈 데브라위너

# Simple forward ranking system

공격포인트와 평점을 통해서 간단한 ranking system 을 구성하고자 한다.
너무나 당연하게도 공격포인트가 많을 수록, 평점이 높을 수록 좋은 선수이다.
현재 2차원 평면으로 표현되어 있는 선수들의 퍼포먼스를 어떻게 **한 줄**로 세울 것인가?<br>

### PCA
>  행렬의 고유벡터와 고유값을 활용한 차원 축소 방법으로, 데이터를 가장 잘 표현하는(해당 축에 대해서 variance 가 최대가 되는) 축들을 찾아준다.

데이터 차원 축소법 중 가장 널리 활용되는 PCA(Principal Component Analysis)를 활용하여,
1차원에 선수들의 퍼포먼스를 맵핑하고 순위를 정해보았다.<br>
PCA 를 통해 차원 축소를 진행하면 다음 그림과 같다.

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/PCA_forward.png?raw=true" width = 500 height = 200>
</p>

PCA 결과는 다음과 같다.

| Ranking | Player_name     | Position | Goal_assist | Rating | Norm_goal_assist | Norm_Rating | PCA score|
|---------|-----------------|----------|-------------|--------|------------------|-------------|----------|
| 1       | Harry Kane      | AM/FW    | 37          | 7.79   | 0.735245         | 0.605729    | 0.939420 |
| 2       | Kevin De Bruyne | AM/FW    | 18          | 7.65   | 0.221732         | 0.521392    | 0.539073 |
| 3       | Son Heung-Min   | AM/FW    | 27          | 7.27   | 0.464975         | 0.292476    | 0.525691 |
| 4       | Jack Grealish   | AM       | 16          | 7.56   | 0.167678         | 0.467175    | 0.462695 |
| 5       | Mohamed Salah   | AM/FW    | 27          | 7.07   | 0.464975         | 0.171994    | 0.434715 |
| 6       | Jamie Vardy     | AM/FW    | 24          | 7.11   | 0.383894         | 0.196090    | 0.399752 |
| 7       | Sadio Mané      | AM/FW    | 18          | 7.33   | 0.221732         | 0.328621    | 0.393511 |
| 9       | Patrick Bamford | AM/FW    | 24          | 7.03   | 0.383894         | 0.147898    | 0.363362 |
| 9       | Ollie Watkins   | AM/FW    | 19          | 7.18   | 0.248759         | 0.238259    | 0.342998 |

PL 공격수의 ranking top 5는 
1. Harry Kane
2. Kevin De Bruyne
3. 손흥민
4. Jack Grealish
5. Mohamed Salah 순이다.

손흥민 선수가 모든 PL 공격수를 통틀어 3번째로 좋은 시즌을 보낸 것으로 볼 수 있다.
(미친 시즌을 보낸 케인, 손흥민을 데리고 7위한 토트넘은 대체...)

# 최근 6년간 손흥민의 퍼포먼스

손흥민은 15-16 시즌 독일 분데스리가 레버쿠젠에서 토트넘으로 이적했다.
그 이후 손흥민의 seasonal ranking 이 어떻게 변화했는지 확인하기 위해서 bump chart 를 그려보았다.

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/sony_visualization/Forward_player_rank.png?raw=true" width = 500 height = 400>
</p>

그래프를 보면, 손흥민은 데뷔 시즌을 제외하고는 항상 리그 탑급 퍼포먼스를 보여왔다.
직전 시즌에는 심지어 한 단계 업그레이드 해서 리그 최고 수준의 공격수로 거듭났다.
위 그래프를 보면 시즌이 거듭 될 수록 폼이 떨어지는 선수들이 보이는 데, (ex. 스털링, 아게로, 피르미누)
손흥민의 퍼포먼스는 확실하게 상승 곡선이 그려지고 있다.


# Wrap up

프리미어리그 선수들의 스탯을 활용해서 손흥민의 퍼포먼스가 얼마나 대단했는지 확인해보았다.
퍼포먼스를 평가하기 위해서 활용할 수 있는 스탯들이 더 많았지만, 단순한 평가지표가 오히려 더욱 강력하다고 생각한다.<br>
손흥민은 이미 프리미어리그 최고 스타가 되었고, 토트넘 레전드이며, 월클임을 충분히 증명해 왔다.
하지만..... 아직 우승컵을 못 들었다는 것이 팬으로서 너무 마음이 아프다.
최근 콘테가 토트넘에 부임했지만, 경쟁이 치열한 프리미어리그에서 타이틀을 따는 것은 쉽지 않을 것 같다.
그래도 손흥민 선수 덕분에 정말 즐겁게 축구를 볼 수 있었고, 앞으로도 계속 좋은 퍼포먼스를 보여줬으면 좋겠다.<br>
(마음 같아선 탈토트넘....ㅜㅜ)












