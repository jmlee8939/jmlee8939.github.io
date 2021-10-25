---
title: "Crwaling Soccer Data"
excerpt: "Chrome webdriver로 축구 경기 웹사이트(Whoscored.com)에서 경기 DATA 크롤링 하기"
header:
  teaser: "http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_c.jpg"
tags: 
  - web crwaling
  - data collecting
  - soccer data
---
## Intro
#### *"Action is the foundational key to all success." - Pablo Picasso* 

드디어 미루고 미뤘던 첫 포스팅을 시작한다. 
평범한 공대생이었던 내가 데이터 마이닝과 머신러닝으로 진로를 바꾸어 공부한지도 어연 2년이 되었다.
아직 부족하고 모르는 것 투성이지만, 용감하게 블로그를 시작하려고 한다.
이 블로그를 통해 앞으로 쭉 즐겁게 데이터 마이닝과 롱런하고 싶다.

---
## Topic
첫 포스팅의 주제는 바로 *"축구경기 데이터 crawling"* 이다.
평소 축구경기에 죽고 사는 나는 자주 방구석 전력분석관이 되곤 한다.
능력있는 방구석 전력분석관으로 거듭나기 위해선 데이터 수집이 필수적이다.
다행히도 다양한 축구 통계 사이트에서 경기 결과 정보를 쉽게 찾을 수 있다.
이번 포스팅에서는 Chrome webdriver를 활용하여 웹상의 경기 Data를 Crwaling하는 방법을 소개하고자 한다.

---
## Crawling/Crawler

Crwaling은 웹 페이지를 그대로 가져와서 데이터를 추출하는 행위를 말한다. 
이때, 이 Crawling 작업을 해주는 소프트웨어는 crwaler 라고 불린다. 
쉽게 말해서 Crawler 는 우리가 검색을 통해 얻을 수 있는 정보를 우리 대신에 빠르고 반복적으로 수행해서 수집해주는 아주 유용한 놈이라는 것이다.

---
## Software
- Python
- Selenium
- Chrome webdriver

웹 상의 데이터를 crawling 하기 위해서 활용한 소프트웨어는 Python, Selenium, 

Here are some examples of what a post with images might look like. If you want to display two or three images next to each other responsively use `figure` with the appropriate `class`. Each instance of `figure` is auto-numbered and displayed in the caption.

### Figures (for images or video)

#### One Up

<figure>
	<a href="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_b.jpg"><img src="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_c.jpg"></a>
	<figcaption><a href="http://www.flickr.com/photos/80901381@N04/7758832526/" title="Morning Fog Emerging From Trees by A Guy Taking Pictures, on Flickr">Morning Fog Emerging From Trees by A Guy Taking Pictures, on Flickr</a>.</figcaption>
</figure>

Vero laborum commodo occupy. Semiotics voluptate mumblecore pug. Cosby sweater ullamco quinoa ennui assumenda, sapiente occupy delectus lo-fi. Ea fashion axe Marfa cillum aliquip. Retro Bushwick keytar cliche. Before they sold out sustainable gastropub Marfa readymade, ethical Williamsburg skateboard brunch qui consectetur gentrify semiotics. Mustache cillum irony, fingerstache magna pour-over keffiyeh tousled selfies.

#### Two Up

Apply the `half` class like so to display two images side by side that share the same caption.

```html
<figure class="half">
    <a href="/assets/images/image-filename-1-large.jpg"><img src="/assets/images/image-filename-1.jpg"></a>
    <a href="/assets/images/image-filename-2-large.jpg"><img src="/assets/images/image-filename-2.jpg"></a>
    <figcaption>Caption describing these two images.</figcaption>
</figure>
```

And you'll get something that looks like this:

<figure class="half">
	<a href="http://placehold.it/1200x600.JPG"><img src="http://placehold.it/600x300.jpg"></a>
	<a href="http://placehold.it/1200x600.jpeg"><img src="http://placehold.it/600x300.jpg"></a>
	<figcaption>Two images.</figcaption>
</figure>

#### Three Up

Apply the `third` class like so to display three images side by side that share the same caption.

```html
<figure class="third">
	<img src="/images/image-filename-1.jpg">
	<img src="/images/image-filename-2.jpg">
	<img src="/images/image-filename-3.jpg">
	<figcaption>Caption describing these three images.</figcaption>
</figure>
``` 

And you'll get something that looks like this:

<figure class="third">
	<img src="http://placehold.it/600x300.jpg">
	<img src="http://placehold.it/600x300.jpg">
	<img src="http://placehold.it/600x300.jpg">
	<figcaption>Three images.</figcaption>
</figure>
