---
layout: post
title:  "NaverCafe Comment Crawling and Data Handling (네이버카페 댓글 크롤링 및 데이터 핸들링)"
image: 'https://cdn-images-1.medium.com/max/1600/1*HJpcblBvD8MpqAEZZXWVgg.png'
date:   2018-05-07 00:12:00
tags:
- NaverCafe
- Crawling
- Selenium
- Highcharter
description: 'Python + Selenium for Crawl and R + Highcharter for Visualization'
categories:
- Crawling
- Visualization
---

##Abstract (개요)

There was a costume voting event in a mobile SRPG game called Browndust. The voting was held in Browndust Official NaverCafe
and people were able to vote in the format '[username] / [unitname]'. There were over 1800 comments and I thought to myself that
making a web crawler here wouldn't be so bad. There was a slight problem when trying to crawl in NaverCafe because the url are
hidden, so I cannot use a orthodox method of going reading html of each unique url. This is where I approached **Selenium**.
After crawling with **Python** and **Selenium**, and making a simple dataframe, I used **R** and **Highcharter** to visualize the
result!

브라운더스트 (모바일 SRPG게임)에서 코스튬 투표 이벤트가 있었다. 브라운더스트 공식 네이버카페에서 '[유저이름] / [용병이름]' 유형으로 댓글 투표가 있었는데, 댓글이 약
1800개가 넘어가서 웹 크롤러가 있으면 좋을 것 같다고 해서 시작했다. 처음부터 네이버카페 url이 숨긴 상태여서 평시 사용되는 웹 크롤러 (유일한 url로 html 읽어오기)는
사용할수가 없어서 **셀레늄** 을 사용하게 되었다. 그래서 큰그림으로 보면, 웹 크롤링은 **파이썬** 과 **셀레늄** 으로 처리를 했고, **R** 과 **하이차터** 로 데이터
시각화를 했다!

##Web Crawling (웹 크롤링)

I used Chrome as Webdriver (웹 드라이버는 크롬을 사용했습니다).

**Setting up for Crawling (크롤링 준비과정)**
{% highlight python %}
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd
import re
import time

def get_n_child_index(lastCounter, currentCounter):
    nextPageChildPos = -1
    currentSector = int(currentCounter/10)
    if(currentSector == 0):
        nextPageChildPos = currentCounter + 1
    else:
        nextPageChildPos = ((currentCounter%10) + 2) + 1
    return(nextPageChildPos)

df = pd.DataFrame(columns=['user_ID','unit_voted'])
dfIndexCounter = 1
r = re.compile('.*\/.*')
# Could've used User-Defined Method, but this was cleaner and simplier
# 사용자 정의 함수를 사용해도 됬지만, 그냥 유저에게 받는게 더 깔끔하고 간략했
lastIndex = int(input('Last Comment Page : ')) # 19
{% endhighlight %}

There isn't anything particularly special until here. User defined method is specific to NaverCafe comment page sector,
so just keep that in mind. *If you are interested... the comment page sector is labelled as nchild, and the nchild is slightly different depending on how many pages are*

여기까진 뭐 특별한 것 없다. 사용자 정의 함수는 네이버카페 댓글 페이지 분할 시스템에 적용된 함수라, 네이버카페 댓글에서만 사용 가능한걸 알고 있으시면 됩니다.
*관심 있으시면... 페이지 분할 paginator를 보시면 nchild로 구분되있는데, 총 페이지 숫자에 맞춰서 지정된다*

**Crawling with Selenium (셀레늄으로 크롤링)**
{% highlight python %}
driver = webdriver.Chrome()

driver.get('http://cafe.naver.com/browndust/ArticleRead.nhn?clubid=28708849&menuid=16&articleid=217084')
time.sleep(3)
driver.switch_to_frame('cafe_main')

for pageIndex in range(0, lastIndex):
    time.sleep(3)
    commentList = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CSS_SELECTOR, '#cmt_list')))
    splitCommentList = (commentList.text).split('\n')
    splitCommentList = list(filter(r.match, splitCommentList))

    # parsing for every legitimate comment
    # 각각 댓글 텍스트 parsing  
    for elements in splitCommentList:
        splitElement = elements.split('/')
        userID = splitElement[0].replace(' ','')
        unitVoted = splitElement[1].replace(' ','')\
        data = pd.DataFrame({'user_ID': userID, 'unit_voted': unitVoted}, index=[dfIndexCounter])
        df = pd.concat([df,data])
        dfIndexCounter += 1

    print('comment Page ' + str(pageIndex+1) + ' complete...')

    # Current page crawling is complete, so preparing next comment page
    # 인제 현재 페이지 크롤링 완료되어서, 다음페이지 준비하는 과정입니다
    if(pageIndex != (lastIndex - 1)):
        nextPageChildPos = get_n_child_index(lastIndex, pageIndex)
        nextPageXPath = '//*[@id="cmt_paginate"]/a[' + str(nextPageChildPos) + ']'
        nextPageCrawl = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, nextPageXPath)))
        nextPageCrawl.send_keys(webdriver.common.keys.Keys.SPACE)
        nextPageCrawl.click()
{% endhighlight %}

If you are a programmer with a bit of knowledge in selenium, you might have some issues on how this code was built...
Here are some explanations on why such code was inserted in some areas. If I see any additional questions in comments,
I will add them when I have the time! :^)

> 1. Why do you need time.sleep when you're already using WebDriverWait?
>
> The xPath for current comment page and the next comment page are the same, so before it goes on to the next comment page, it checks xPath for current comment page (which will return True and continue that results as an error). So in order to wait for not checking for xPath in the current page, added time.sleep.
>
> 2. Why do you need send_keys, when you only need to do click?
>
> If the clicking element is not currently viewed on the browser (physically), it returns an error. I've googled about this problem, and other people also had this problem. So, the send_keys will scroll the browser to actually see the element that's about to be clicked, which solves the error.

셀레늄을 다뤄보신분들이나 따라하시면서 '이걸 왜하지?'라는 생각이 들게 몇개 있어서 미리 FAQ처럼 밑에 설명했습니다. 만약 다른 궁금한거 있으시면 댓글로 달아주세요,
나중에 시간되면 추가하겠습니다! :^)

> 1. WebDriverWait를 사용하는데, 왜 time.sleep이 필요한가?
>
> 지금 현재 페이지와 다음페이지 xPath가 같아서, 만약 time.sleep을 안넣으면, 다음 페이지 가기전에 현재 페이지 에서 확인하게 됩니다. xPath가 같아서, 기달리지 않고 바로 실행하려다가 에러나서, time.sleep를 추가하게 되었습니다.
>
> 2. click만 하면 되는데, 왜 send_keys까지 필요한가?
>
> 이건 확실히 왜 바로 click하면 에러가 안뜨는데 인터넷에서 다른 유사한 문제를 읽어보니, 현재 화면이 element에 없으면 에러가 나는 경우가 있다고 합니다. 그래서 send_keys를 사용해서 지금 화면을 다음페이지 가는 element에 고정 후 click을 하는겁니다.

**Exporting Dataframe as csv (csv로 데이터프레임 추출)**
{% highlight python %}
df.to_csv('~/Desktop/browndust-related/browndust-costume-event-crawled-comment.csv', sep=',', encoding='utf-8')
{% endhighlight %}

##Data Handling and Visualization (데이터 핸들링 및 시각화)

**Data Handling (데이터 핸들링)**
{% highlight r %}
library(pacman)
pacman::p_load(readr,dplyr,highcharter)

df.comment <- readr::read_csv('~/Desktop/browndust-related/browndust-costume-event-crawled-comment.csv')
df.comment$X1 <- NULL
df.unit <- readr::read_csv('~/Desktop/browndust-related/browndust-unit-db.csv')

# User Defined Function (사용자 정의 함수)
reduce_duplicates <- function(dup.userID) {
  all.dup.index.list <- lapply(dup.userID, function(x) which(df.formal$user_ID %in% x))
  all.dup.index <- unlist(all.dup.index.list, use.names=FALSE)
  keeping.index <- vapply(all.dup.index.list, tail, n = 1L, FUN.VALUE = numeric(1))
  removing.index <- all.dup.index[!(all.dup.index %in% keeping.index)]
  return(removing.index)
}

df.formal <- df.comment[df.comment$unit_voted %in% df.unit$용병이름,]
df.informal <- df.comment[!(df.comment$unit_voted %in% df.unit$용병이름),]

df.duplicate <- df.formal[duplicated(df.formal$user_ID),]

removing.index <- reduce_duplicates(df.duplicate$user_ID)
df.formal <- df.formal[-removing.index,]
{% endhighlight %}

There are some users who didn't write just unitname, but some extra texts such as 'unit x in maid costume', since those
comments did not follow the rules, I've separated them into a formal / informal dataframe (it's possible to distinguish them
with grep and apply, so if needed this is possible)

어느 유저들은 규칙을 어긋나게 추가로 용병이름에 다른 텍스트를 넣는 케이스도 있었습니다. (예: '메이드 코스튬 입은 용병 x'). 규칙을 어긋나서 일단 유형 / 비유형
으로 나눴고 만약 필요하면 추가로 grep하고 apply 함수로 필요한 추출이 가능합니다.

**Data Visualization (데이터 시각화)**

Since I uploaded the results in the NaverCafe, results are in korean, but those are just unitnames and titles.

{% highlight r %}
df.formal.pie.data <- table(df.formal$unit_voted)
highchart() %>%
 hc_chart(type = "pie") %>%
 hc_add_series_labels_values(labels = attributes(df.formal.pie.data)$dimnames[[1]], values = df.formal.pie.data) %>%
 hc_tooltip(pointFormat = paste('{point.y} 표<br/><b>{point.percentage:.1f}%</b>')) %>%
 hc_title(text = "코스튬 투표 현황 (유형 맞춘 투표만 적용)")
{% endhighlight %}

<img src="uploads/naver-cafe-crawl-and-visualization-pie-1.png">

Due to too much data, removing unit votes less than 10

데이터가 너무 많아서 투표 10개보다 낮은 투표는 무시

{% highlight r %}
df.formal.pie.data.reformat <- df.formal.pie.data[df.formal.pie.data > 10]
highchart() %>%
 hc_chart(type = "pie") %>%
 hc_add_series_labels_values(labels = attributes(df.formal.pie.data.reformat)$dimnames[[1]], values = df.formal.pie.data.reformat) %>%
 hc_tooltip(pointFormat = paste('{point.y} 표<br/><b>{point.percentage:.1f}%</b>')) %>%
 hc_title(text = "코스튬 투표 현황 (유형 맞춘 투표만 적용 + 투표 < 10 용병은 무시)")
{% endhighlight %}

<img src="uploads/naver-cafe-crawl-and-visualization-pie-2.png">

Due to one unit getting overwhemling votes from users, there was no need to go through the informal dataframe to extract
votes. (better for me XD)

'라피나' 용병이 압도적으로 투표를 많이 가져가서, 비유형 데이터프레임에서 데이터 추출은 필요가 없게되었네요... (저한테 좋죠 ㅎㅎ)
