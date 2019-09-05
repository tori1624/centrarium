---
layout: post
title:  "Summarizing AAG conference abstracts and keywords using keywords network analysis (1)"
date:   2019-09-04 11:40:00
author: Youngho Lee
categories: Project
cover:  "/assets/project/keyword-network/wordcloud.PNG"
---

본 포스팅은 과학기술정보통신부와 정보통신기획평가원이 주관하는 "2019년 글로벌 핵심인재 양성지원 사업"의 프로젝트 과정에서 사용한 코드들을 정리하기 위해 작성하였다. 프로젝트 주제는 "컨퍼런스 최신 연구 동향 분석 : 언어네트워크 분석과 빅데이터 분석을 적용한 Generic Platform 개발"로, Association for American Geographers(AAG, http://aag.org) 컨퍼런스에 1998~2019년간 투고된 논문들의 초록과 키워드들을 사례로 지리학의 연구경향을 파악하고자 하는 연구이다. AAG 컨퍼런스는 매년 5일간 약 7천여명의 연구자들이 등록하는 대규모 학술공동체이며, GIS·인문지리·환경지리·지형학·교통지리·원격탐사 등 지리학의 다양한 분야의 연구들이 매년 소개되고 있다. 이번 글은 프로젝트의 첫 번째 세부프로젝트인 "키워드 네트워크 분석을 활용한 AAG 컨퍼런스 요약"에 관한 글로, 첫 단계인 prototype analysis의 데이터 전처리 코드에 대해 정리할 것이다.

<img src = "/assets/project/keyword-network/aagWebsite.PNG" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />

prototype anlysis는 2019년도 AAG 컨퍼런스 데이터를 바탕으로 진행되었으며, 논문의 topic과 keyword를 위주로 분석하였다. 2019년 데이터는 위 그림에서 보이는 AAG 홈페이지에서 크롤링하여 취득할 수 있다.(link - https://aag.secure-abstracts.com/AAG%20Annual%20Meeting%202019/abstracts-gallery)

{% highlight javascript %}
# Data import
test.df <- read.csv("D:/project/2019/aag/new_data/aag2019.csv", header = FALSE)
names(test.df)[1] <- "variable"

head(test.df, 20)
{% endhighlight %}

{% highlight javascript %}                                                variable
1  ####  ‘Moments’ and ‘Butterflies’ in the Geographical Political Economy of
2  Tourism Destinations
3  Assigned to Session
4  Authors: Salvador Anton Clave*, Department of Geography, Faculty of Tourism
5  and Geography, Rovira i Virgili University, Julie Wilson*, Faculty of
6  Economics and Business Studies, Universitat Oberta de Catalunya / Open
7  University of Catalonia, Cinta Sanz-Ib??ez, Department of Geography, Faculty
8  of Tourism and Geography, Rovira i Virgili University
9  Topics: Tourism Geography, Economic Geography
10 Keywords: Evolutionary Economic Geography; Tourism, Regional Development,
11 Moments, Butterfly framework, Socio-Ecological Resilience, Cultural Political
12 Economy, Tourism Spaces
13 Session Type: Paper
14 Day: 4/4/2019
15 Start / End Time: 5:00 PM / 6:40 PM
16 Room: Chairman Boardroom, Omni, East
17 Presentation File:  No File Uploaded
18 ####  ‘As it really is’: Representation, obscurity and visual cultures of
19 creative space in Toronto
20 Assigned to Session
{% endhighlight %}

데이터를 불러온 이후, 데이터의 형태를 살펴보면 위와 같이 출력되는 것을 확인할 수 있다. "####" 뒤에 있는 문장이 Title, 그 외에 Authors, Topics, Keywords 등이 포함되어있었고, prototype analysis를 위해서는 Topics와 Keywords를 추출해야했다. Topics의 경우에는 짧으면 한 줄, 길면 두 줄이었기 때문에, 규칙적으로 추출하는데 있어서 큰 문제가 없었지만, Keywords의 경우에는 길면 짧은 것은 한 줄, 긴 것은 네 줄도 넘었으므로 규칙적으로 추출하는데 어려움이 있었다. 첫 번째 논문의 경우에도 키워드가 세 줄인 것을 볼 수 있다. 따라서 Keywords의 줄 수에 상관없이 Keywords를 추출할 수 있는 함수를 만들었고, 함수 설명은 다음과 같다(설명의 편의를 위해 코드 앞에 숫자를 입력하였다).

{% highlight javascript %}
1  aag2019 <- function(data, a, b, n) {
2    tmp.dif <- data.frame(row1 = grep(a, data[, 1]), row2 = grep(b, data[, 1])-1)
3    tmp.dif$dif <- tmp.dif$row2 - tmp.dif$row1; dif.max <- max(tmp.dif$dif)
4
5    tmp.df <- data.frame(tmp.dif$dif)
6
7    for (i in 1:(dif.max+1)) {
8      tmp <- data.frame(1:n)
9      tmp.df <- data.frame(tmp.df, tmp)
10   }
11
12   names(tmp.df) <- c("dif", paste0("v", 1:(dif.max+1)))
13
14   for (i in 0:dif.max) {
15     if (i == 0) {
16       tmp.df[, i+2] <- test.df$variable[grep(a, data[, 1])+i]
17     } else if (i >= 1) {
18       tmp.df[, i+2] <- test.df$variable[grep(a, data[, 1])+i]
19       tmp.df[tmp.df[, 1] <= i-1, i+2] <- ""
20     }
21   }
22
23   return(tmp.df)
24 }
{% endhighlight %}