---
title: "[DP20] 축구 데이터 크롤링하기"
excerpt: "Chrome webdriver 로 축구 경기 웹사이트(Whoscored.com)에서 경기 DATA 크롤링 하기"
header:
  teaser: "http://drive.google.com/uc?export=view&id=1-atf2hBTNCbA7c3MoSaFDiqI0EqWGzzp"
tags: 
  - Web Crawling
  - Soccer
---
  
# Preface
### *"Action is the foundational key to all success." - Pablo Picasso* <br>  

드디어 미루고 미뤘던 첫 포스팅을 시작한다. 
평범한 공대생이었던 내가 데이터 마이닝과 머신러닝으로 진로를 바꾸어 공부한지도 어연 2년이 되었다.
아직 부족하고 모르는 것 투성이지만, 용감하게 블로그를 시작하려고 한다.
이 블로그를 통해 앞으로 쭉 즐겁게 데이터 마이닝과 롱런하고 싶다.


# Intro
첫 포스팅의 주제는 바로 **"축구경기 데이터 Crawling"** 이다.
평소 축구경기에 죽고 사는 나는 자주 방구석 전력분석관이 되곤 한다. 
능력있는 방구석 전력분석관으로 거듭나기 위해선 데이터 수집이 필수적이다.
다행히도 다양한 축구 통계 사이트에서 경기 결과 정보를 쉽게 찾을 수 있다.
이번 포스팅에서는 Chrome webdriver 를 활용하여 웹상의 경기 Data를 Crawling 하는 방법을 소개하고자 한다.
> 자세한 크롤링 코드는 [whoscored_crawling](https://github.com/jmlee8939/whoscored_crawling) 을 참고


# Crawling/Crawler

Crwaling은 웹 페이지를 그대로 가져와서 데이터를 추출하는 행위를 말한다. 
이때, 이 Crawling 작업을 해주는 소프트웨어가 바로 *Crawler* 이다. 
*Crawler* 는 검색을 통해 얻을 수 있는 정보를 우리 대신에 빠르고 반복적으로 수집해주는 아주 유용한 놈이다.


# Software
이 유용한 친구(*Crawler*) 에 활용되는 소프트웨어는 Python, Selenium, Chrome webdriver 이다.
> - Python 
> - Selenium 
> - Chrome webdriver 

Selenium은 웹 자동화 테스트(버튼 클릭, 스크롤 조작 등)에 사용되는 프레임워크로 
자동적으로 원하는 정보가 있는 페이지를 찾아서, 필요한 소스에 접근하게 해준다. 
Chrome browser 를 자동화 하기 위해서 Chrome webdriver 가 필요하고,
python 환경에서 쉽게 구현 가능하다.



# Setting up
### 1. Selenium 설치 <br>
  - Python 환경에서 pip 로 쉽게 설치 가능하다.

```python
pip  install selenium
```

### 2. Chrome webdriver 설치

  - [ChromeDriver](https://chromedriver.chromium.org/downloads) 사이트에서 쉽게 설치 가능하다.
  - 사용하려는 Chrome browser 와 버전이 동일해야 한다.** <br>
    - chrome version 확인 : 크롬 브라우저 검색창에 *chrome://version* 입력해서 확인가능<br>
    
   1. 현재 Chrome browser 버전을 확인하고,<br>
   2. 버전에 맞는 ChromeDriver를 설치한다. <br>
   3. ChromeDriver 가 설치된 디렉토리를 확인한다.


# Whoscored.com
후스코어드닷컴은 여러 축구 통계사이트(opta, transfermakt, 등..) 중 하나로 다양한 경기정보를 제공한다. 
유럽 각 국의 1부리그 뿐만 아니라 하부리그 경기 정보까지 제공하며,
경기에서 발생하는 세부 지표들을 상세하게 정리해 놓아서 많은 축구팬들이 참고하는 사이트이다.
나와 같은 방구석 분석가에게 꼭 필요한 사이트라 할 수 있다.
본 포스팅에서는 2021시즌 Premier League table 을 Crawling 하는 방법을 간단히 소개하고자 한다.<br>
내가 처음으로 Crawling 하고자 한 정보는 바로 이 [2021 League Table](https://1xbet.whoscored.com/Regions/252/Tournaments/2/Seasons/8228/England-Premier-League)
 이다.

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/Whoscored/PL2122 League Table.png?raw=true" width = 500 height = 300>
</p>

위 League table은 2021 시즌의 순위와 각 팀의 경기 결과들을 한눈에 볼 수 있는 표로 참고할 만한 여러 정보를 담고있다.
머신러닝 및 데이터 분석에 활용할 수 있도록 python 이나 R 환경으로 세부 정보들을 옮겨보고자 한다.

# Crawling process
### 1. webdriver 연결 확인
python 환경에서 로컬에 저장된 Chromedriver 를 연결해서 브라우저 제어 가능한지 확인하는 코드는 다음과 같다.

```python
# 디렉토리에 저장된 chromedriver 에 연결
driver = webdriver.Chrome('./chromedriver')
# 웹 브라우저 열기    
driver.get('https://1xbet.whoscored.com/')
# 일시 정지 (3초)
time.sleep(3)
# 웹 브라우저 닫기 
driver.close()
```

chromedriver 가 잘 연결되었다면, 위 코드를 실행했을 때 Whoscored.com 사이트가 열렸다가 3초 후 브라우저가 종료될 것이다. 
### 2. 웹 상의 소스코드 구조 확인
웹페이지의 소스코드는 F12를 누르면 확인할 수 있고,
*Control + Shift + C* 를 활용하면 마우스 커서에 해당하는 Element 의 소스 코드를 쉽게 찾을 수 있다.
<br>
이를 통해서 Crawling 하고자 하는 Element 의 소스 코드 구조를 확인할 수 있다.     

<p align="center">
<img src= "https://github.com/jmlee8939/jmlee8939.github.io/blob/master/assets/images/Whoscored/Web sourcecode.png?raw=true" width = 500 height = 300>
</p>

### 3. CSS selector
CSS selector 는 웹상에 존재하는 모든 요소들을 선택하게 해 주는 네비게이션 역할을 한다.
이 CSS selector 를 통해서 원하는 소스 코드에 접근하고 이를 가져오는 방식으로 Crawling 이 이루어진다.

# Selenium elements finding function

Crawling 하고자하는 페이지의 html 의 구조를 확인하였으면,
Selenium 를 활용하여 해당 element 에 접근하여야 한다.
Selenium 에서는 아래와 같은 함수들를 통해 element 에 접근한다.

| 함수 이름                    | 기능                       |
|------------------------------|----------------------------|
| find_element_by_id           | id 속성을 사용하여 접근    |
| find_element_by_name         | name 속성을 사용하여 접근  |
| find_element_by_tag_name     | 태그를 사용하여 접근       |
| find_element_by_class_name   | 클래스를 사용하여 접근     |
| find_element_by_css_selector | CSS 선택자를 사용하여 접근 |

다수의 elements 에 접근하는 함수는 다음과 같다. (리스트 반환)

| 함수 이름                     	| 기능                       	|
|-------------------------------	|----------------------------	|
| find_elements_by_id           	| id 속성을 사용하여 접근    	|
| ind_elements_by_name          	| name 속성을 사용하여 접근  	|
| find_elemenst_by_tag_name     	| 태그를 사용하여 접근       	|
| find_elements_by_class_name   	| 클래스를 사용하여 접근     	|
| find_elements_by_css_selector 	| CSS 선택자를 사용하여 접근 	|

# Python Code
내가 Crawling 하고자 하는 table 의 class 는 "standings" 이고 
다수의 하위 요소 "td" 에 팀이름, 승, 무, 패, 승점 등의 정보가 담겨 있다.
위 함수들을 를 활용하여 class 이름으로 table 에 접근하고, 다수의 "td" 에 해당하는 정보들을 차례로 불러오고자 한다.

*Crawler* 를 구성한 코드는 다음과 같다.

<details>
<summary> Crawler code
</summary>
<div markdown="1">

```python
import time
import pandas as pd
from selenium import webdriver

def league_table_crawler(URL, api_delay_term=3):
    """
    cwaling league table(seasonal statistics) 
    Args : 
        URL : league table URL 
    Output :
        leauge table(dataframe)
    """
    url = str(URL)
    driver = webdriver.Chrome('./chromedriver')
    driver.get(url)
    
    time.sleep(api_delay_term)
    
    league_table_df = pd.DataFrame(columns=[
        "team_number", "team_name", "P", "W", "D", "L", "GF", "GA", "GD", "Pts"])
    
    elements = driver.find_elements_by_class_name("standings")[0]
    elements = elements.find_elements_by_css_selector("tr")
    
    for element in elements:
        league_table_dict = { 
            "team_number": element.find_elements_by_css_selector("a")[0].get_attribute("href").split("/")[4], 
            "team_name": element.find_elements_by_css_selector("a")[0].text,     
            "P": element.find_elements_by_css_selector("td")[1].text,
            "W": element.find_elements_by_css_selector("td")[2].text, 
            "D": element.find_elements_by_css_selector("td")[3].text,
            "L": element.find_elements_by_css_selector("td")[4].text, 
            "GF": element.find_elements_by_css_selector("td")[5].text, 
            "GA": element.find_elements_by_css_selector("td")[6].text,
            "GD": element.find_elements_by_css_selector("td")[7].text,
            "Pts": element.find_elements_by_css_selector("td")[8].text,
        }
        league_table_df.loc[len(league_table_df)] = league_table_dict
    # close webdriver
    driver.close()
    
    return league_table_df
```

</div>
</details>
<br>

그 다음 Premier League 2021 시즌의 League table 의 [URL](https://1xbet.whoscored.com/Regions/252/Tournaments/2/Seasons/8228/England-Premier-League)
과 *Crawler* 를 연결시키고 Crawling 을 진행한다.<br>
```python
URL = 'https://1xbet.whoscored.com/Regions/252/Tournaments/2/Seasons/8228/England-Premier-League'
df = league_table_crawler(URL, api_delay_term=3)
df.head()
```

그리고 그 결과는 다음과 같다.

| team_number |               team_name |  P |  W |  D |  L | GF | GA |  GD | Pts |
|------------:|------------------------:|---:|---:|---:|---:|---:|---:|----:|----:|
| 167         | Manchester City         | 38 | 27 | 5  | 6  | 83 | 32 | +51 | 86  ㄴ|
| 32          | Manchester United       | 38 | 21 | 11 | 6  | 73 | 44 | +29 | 74  |
| 26          | Liverpool               | 38 | 20 | 9  | 9  | 68 | 42 | +26 | 69  |
| 15          | Chelsea                 | 38 | 19 | 10 | 9  | 58 | 36 | +22 | 67  |
| 14          | Leicester               | 38 | 20 | 6  | 12 | 68 | 50 | +18 | 66  |

20개의 모든 팀의 정보가 Crawling 됨을 확인하였다. 하지만
.head() 함수는 디폴트로 상위 5개 항목만을 보여주므로, 안타깝께도 토트넘 아스날은 여기에 끼지 못했다.
(앞으로도 한동안 여기 못 낄 듯 하다. 어서 손흥민 선수가 탈 토트넘 하길...) 

# Wrap up
방구석 축구전문가로 거듭나기 위한 첫번째 단계이자, 데이터 분석을 더욱 재밌게 즐기기 위한 첫 포스팅을 마무리하고자 한다.
*Crawler* 는 많은 데이터의 반복적인 수집을 필요로 할 때 효과적으로 활용할 수 있을것 같다. 
이 편리한 친구는 앞으로도 나의 데이터 수집 하청업자로서 역할을 톡톡히 해내리라 믿는다.

이 포스팅에서 소개한 Crawling 방법 이외에도 다양하고 훨씬 효율적인 Crawling 방법들이 많이 있지만,
소개하였던 Crawling 방법을 조금 응용하는 것만으로도 간단한 *Crawler* 를 만드는 데에는 큰 무리가 없지 않을까. 
League table 이 외에도 매 경기의 경기 결과 정보, 선수 정보도 Crawling 하는 코드들을 [요기](https://github.com/jmlee8939/whoscored_crawling) 에 정리해 두었으니 필요하면 참고해 두면 좋을 것 같다. 
