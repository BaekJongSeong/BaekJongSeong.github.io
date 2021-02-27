---
layout: post
title: "topic 별 실시간 issue 집계 후 나열 웹페이지 제작 프로젝트"
comments: true
image: assets/images/2021-02-27-naver_news.png
---

This is a project to create a web page 

that lists current real-time issues (topics) 

about things happening in Korea by ranking.

## 한국에서 발생하는 일들에 대한 현재 실시간 이슈(토픽)을 집계하여, 

순위별로 나열된 웹페이지를 만드는 프로젝트입니다.
---
프로젝트 핵심은 다음과 같습니다 

기존 Naver 실시간 검색어 서비스 폐지(2021.02.25)에 따른 

불편함을 해소해보고자 직접 구현한 실시간 토픽 서비스


+ page1 = 카테고리
	+ 주식종목 & 금융 20개 (네이버금융 많이본 + 장중특징주 + 주요뉴스)
	+ 영화드라마 & 연예인 20개 (네이버연예뉴스)
	  + 네이버 연예 뉴스 => 랭킹 => 많이 본 연애 뉴스 + 공감 많은 뉴스 (6 category) 토대로 작성
	+ 스포츠 20 개 (종목별 네이버 스포츠 뉴스)
	  + 네이버 스포츠 뉴스 => 랭킹 => 많이 본 뉴스 순위 토대로 작성
	+ 경제 & 정치 & 사회 20개 (네이버뉴스)
	  + 네이버 뉴스 => 랭킹뉴스 탭 => 해당 언론사의 순위 토대로 작성


+ page 2 (예정)
  + 데이터랩 같은 지표형식
  + 사람들이 납득할 수 있게 이슈별로 그래프 하나씩 추가

+ 진행방식
  + 방식은 총 3개의 카테고리 뉴스들 (정치 / 연애 / 스포츠)로 시작
  
  + 3개에서 랭킹 차트에 주목(조회수 / comment 수)
  
  + 각 랭킹 차트의 뉴스 기사들을 통해 핵심어 분석
  
  + 핵심어 노출(검색어 순위와는 다른 개념!!!)
  
    

먼저 마운트를 해주고

```python
from google.colab import drive
drive.mount('/content/drive')
```

## 1번 카테고리 -정치/사회를 먼저 구성해보면

"네이버 뉴스" 페이지에서 언론사별 뉴스를 탐색했을때

나오는 id 번호들을 고려해서 press_ID를 작성해주고

request를 통해서 url에 대한 html 정보를 얻어오면

soup에서 해당 기사들 중에서 rank가 메겨진 부분들에

대한 정보를 담고 있는 tag를 찾아서 값을 넘겨준다

### 우리는 랭킹 뉴스에 대한 정보를 원하기 때문!

```python
#import os
#os.chdir(r"/content/drive/MyDrive/Naver/")
import re
import requests
import pandas as pd
import time
from bs4 import BeautifulSoup

press_ID = {"연합뉴스": "001","JTBC" : "437", "KBS" : "056"}

def get_ranking_news(date):
  for press in press_ID:
    url = "https://news.naver.com/main/ranking/office.nhn?officeId=" + press_ID[press] + "&date=" + str(date)
    headers = {"User-Agent": "Mozilla/5.0(Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"}
    resp = requests.get(url , headers = headers)
    soup = BeautifulSoup(resp.text , "html.parser")
    ranking_box = soup.find_all(class_= "rankingnews_box_inner")
    I = []
    
#이때 rankingnews_box_inner라는 태그안에서 오늘날짜로의

#rank가 매겨진 모든 기사들이 순위로 나오게 되는데 이것을 기준으로

#list_content를 하나씩 list에 담아서 url_list 변수로 받고

#(find_all과 find는 html 태그에 대해서만 작동하기 떄문에 조심)

#get_text()는 해당 태그에서 다른 주소들말고 제대로 된 문자열만 반환해주기 때문에

#title과 view / comment, rank 값을 받아서 넘겨준다

    for ranking_type in range(2):   #그 안에 찾고싶은 class 갯수가 밑에처럼 두개가 있어서 2구나~!
      ranking = ranking_box[ranking_type].find_all(class_ = "list_ranking_num")
      url_list = ranking_box[ranking_type].find_all(class_ = "list_content")
      for rank in range(20):
        d = {}
        d['Date'] = int(date)
        d['Press'] = press
        d['Rank'] = ranking[rank].get_text()
        d['URL'] = url_list[rank].find('a')['href']
        d['Title'] = url_list[rank].find('a').get_text()
        if (ranking_type == 0):
          d['View'] = url_list[rank].find(class_="list_view").get_text()
        elif (ranking_type == 1):
          d['Comment'] = url_list[rank].find(class_="list_comment nclicks('RBP.dcmtnwscmt')").get_text()
        I.append(d)
        
#후에 받아온 url 값에 들어있는 oid와 aid 값을 토대로

#해당 게시물에 들어가서 contents를 받아오고

#(이때 get_text를 통해서 위처럼 받아오는데 해당 게시물에

#코드 문자들을 ex)▶▽♡◀ 전부 치환하고 가져오기 위해서 re.sub사용)

#데이터프레임으로 묶어서 csv 파일로 저장하면 된다.

    for news in I:
      resp = requests.get("https://news.naver.com" + news['URL'], headers = headers)
      soup = BeautifulSoup(resp.text,"html.parser")
      contents = soup.find(id="articleBodyContents").get_text()
      news['Content'] = re.sub('[\{\}\[\]\/?\(\);:|*~`!^\-_+<>▶▽♡◀ㅡ@\#$&\\\=\'\"ⓒ(\n)(\t)]', '', contents) 
    df = pd.DataFrame(I)
    title = press + "/" + str(date) + "_" + press + "_ranking_news.csv"
    df.to_csv(title,sep=",", index = False, encoding = "utf-8-sig")   
    
  print("======================================")
  
get_ranking_news(20210226)
```



## 그 다음에 카테고리2 - 연애 뉴스 역시 똑같은 방식으로 진행되는데

먼저 해당 url 에서 html로 받아오고

이때 페이지 형식이 이전의 정치/사회 뉴스들과는 조금 다른 부분이 있는데

네이버 연애 뉴스를 들어가보면 

많이 본 연애 뉴스라는 우측의 순위가 가장 먼저

html 문서 안에 태그되어있다.

### 각 뉴스 란 마다 문서 형식이 다름

또한 연애 뉴스란에서는 우리가 많이 본 뉴스 10 + 공감 많은(각 공감별 6개 카테고리) 뉴스 10을 묶어서

총 70개의 뉴스로 분석하기로 했으므로 먼저 많이 본 뉴스부터 시작한다

해당 태그는 rank_lst이다.

```python
import re
import requests
import pandas as pd
import time
from bs4 import BeautifulSoup

url = "https://entertain.naver.com/ranking#type=hit_total&date=2021-02-26"
headers = {"User-Agent": "Mozilla/5.0(Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"}
resp = requests.get(url, headers = headers)
soup = BeautifulSoup(resp.text, "html.parser")
ranking_box = soup.find_all(class_= "rank_lst")
date = "20210226"
I=[]
url_rank = ranking_box[0].find_all(class_ = "blind")
a = ranking_box[0].find_all("a")
for rank in range(10):
  d = {}
  d['Date'] = int(date)
  d['Rank'] = url_rank[rank].get_text()
  d['URL'] = a[rank]['href']
  d['Title'] = a[rank].get_text()
  I.append(d)
```





rank_lst로 가져왔으면 이 내용들에서

위에서 진행한 방식과 같이 하나씩 dictionary에 넣어준다

## 6개 공감 카테고리에 대해서 반복되는 코드는 생략.

## 그리고 마지막 스포츠 카테고리 역시 같은 방식으로 진행

이때 스포츠 카테고리는 

### <script type="text/javascript"></script>


태그로 내용을 감쌈

즉, 랭킹뉴스에 관한 정보가 자바스크립트형태로 감싸져 있어서

해당 내용을 추출하려면 먼저 script를 고르고

script안에서 랭킹 내용이 dictionary로 되어있으므로

tuple로 감싸서 dictionary 형태로 뽑아내기

```python
#%cd /content/drive/MyDrive/Naver/
import re
import requests
import pandas as pd
import time
from bs4 import BeautifulSoup

url = "https://sports.news.naver.com/ranking/index.nhn"
headers = {"User-Agent": "Mozilla/5.0(Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"}
resp = requests.get(url, headers = headers) 
soup = BeautifulSoup(resp.text, "html.parser")
#date = '20210226' 안에 date 있어서 이거 없어도 돼
url_list=soup.find_all("script")
url_list=re.split("{",url_list[-5].text)
url_list2=[]
I=[]
for item in url_list:
  if "\"section\":" in item and "subSection" in item:
    url_list2.append(item.split("}")[0])
url_list=url_list2.copy()
url_list2.clear()
url_list
for item in url_list:
  item=item.split(',"').copy()    #url_list2.append(tuple(map(str,item.split(','))))
  url_list3=[]
  for i in range(len(item)):
    url_list3.append(tuple(map(str,item[i].split('":'))))
  url_list2.append(url_list3)
i=0
for item in url_list2:
  url_list2[i]=dict(item)
  i+=1

for rank in range(20):
  d = {}
  d['Date'] = (url_list2[rank]['date']).split('"')[1]
  d['Rank'] = url_list2[rank]['rank']
  d['URL'] = 'oid=' +  (url_list2[rank]['oid']).split('"')[1] + '&' + 'aid=' + (url_list2[rank]['aid']).split('"')[1]
  d['Title'] = url_list2[rank]['title']
  d['subContent'] = url_list2[rank]['subContent']
  d['totalCount'] = url_list2[rank]['totalCount']
  I.append(d)
for news in I:
      resp = requests.get("https://sports.news.naver.com/news.nhn?" + news['URL'], headers = headers)
      soup = BeautifulSoup(resp.text,"html.parser")
      contents = soup.find(id="newsEndContents").get_text()
      news['Content'] = re.sub('[\{\}\[\]\/?\(\);:|*~`!^\-_+<>▶▽♡◀ㅡ@\#$&\\\=\'\"ⓒ(\n)(\t)]', '', contents)
    
df = pd.DataFrame(I)

title = "Sport/" + str(date) + "_" + "sport" + "_ranking_news.csv"

df.to_csv(title,sep=",", index = False, encoding = "utf-8-sig")

print("=======================================")
```

