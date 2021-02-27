---
layout: post
title: "topic 별 실시간 issue 집계 후 나열 웹페이지 제작 프로젝트"
comments: true
image: assets/images/2021-02-27-naver_news.png
---

# Front Matter

This is a project to create a web page that lists current real-time issues (topics) about things happening in Korea by ranking.

## 한국에서 발생하는 일들에 대한 현재 실시간 이슈(토픽)을 집계하여, 순위별로 나열된 웹페이지를 만드는 프로젝트입니다.
---
프로젝트 핵심은 다음과 같습니다 
기존 Naver 실시간 검색어 서비스 폐지(2021.02.25)에 따른 불편함을 해소해보고자 직접 구현한 실시간 토픽 서비스

+ page1 = 카테고리
	+ 주식종목 & 금융 20개 (네이버금융 많이본 + 장중특징주 + 주요뉴스)
	+ 영화드라마 & 연예인 20개 (네이버연예뉴스)
	+ 스포츠 20 개 (종목별 네이버 스포츠 뉴스)
	+ 경제 & 정치 & 사회 20개 (네이버뉴스)

+ page 2
  + 데이터랩 같은 지표형식
  + 사람들이 납득할 수 있게 이슈별로 그래프 하나씩 추가
 
+ 진행방식
  + 랭킹 차트(조회수 / comment 수를 통해 시작)
