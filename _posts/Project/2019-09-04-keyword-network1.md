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

prototype anlysis는 2019년도 AAG 컨퍼런스 데이터를 바탕으로 진행되었으며, 논문의 topic과 keyword를 위주로 분석하였다. 2019년 데이터는 위 그림에서 보이는 AAG 홈페이지에서 크롤링하여 취득할 수 있다(link - https://aag.secure-abstracts.com/am2019).

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

데이터를 불러온 이후, 데이터의 형태를 살펴보면 위와 같이 출력되는 것을 확인할 수 있다. "####" 뒤에 있는 문장이 Title, 그 외에 Authors, Topics, Keywords 등이 포함되어있었고, prototype analysis를 위해서는 Topics와 Keywords를 추출해야했다. Topics의 경우에는 짧으면 한 줄, 길면 두 줄이었기 때문에, 규칙적으로 추출하는데 있어서 큰 문제가 없었지만, Keywords의 경우에는 길면 짧은 것은 한 줄, 긴 것은 네 줄도 넘었으므로 규칙적으로 추출하는데 어려움이 있었다. 첫 번째 논문의 경우에도 키워드가 세 줄인 것을 볼 수 있다. 따라서 Keywords의 줄 수에 상관없이 Keywords를 추출할 수 있는 함수를 만들었고, 전체적인 함수의 코드는 다음과 같다(설명의 편의를 위해 코드 앞에 숫자를 입력하였다).

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

함수의 전체를 한 번에 설명하기보다는 함수의 인자부터 차례대로 자세하게 설명하고자 한다.

{% highlight javascript %}
1  aag2019 <- function(data, a, b, n) {
...
24 }
{% endhighlight %}

우선, 함수의 인자에는 데이터(data), 데이터에서 뽑아내고자 하는 정보를 구분할 수 있는 단어들(a, b), 데이터에 포함된 논문의 수(n), 총 4가지가 포함되도록 만들었다. 헷갈릴 수 있는 a, b에 대해 예를 들어 설명하면, keywords만을 추출하고자 하는 경우에는 keywords가 위치한 항목에 해당하는 "Keywords:"를 a에, keywords의 다음 항목에 해당하는 "Session Type:"을 b에 입력하면 된다.

{% highlight javascript %}
2    tmp.dif <- data.frame(row1 = grep(a, data[, 1]), row2 = grep(b, data[, 1])-1)
3    tmp.dif$dif <- tmp.dif$row2 - tmp.dif$row1; dif.max <- max(tmp.dif$dif)
{% endhighlight %}

함수의 첫 부분에서 진행되는 작업은 각 논문별로 a, b에 입력된 항목이 몇 번째 줄에 있는지 data frame으로 만들고, 두 값의 차이를 계산하는 것을 통해 각 논문별로 keywords나 title 등이 몇 줄에 걸쳐 작성되어 있는지 파악하는 과정이다. 이 때, 차이의 최대값은 다음 단계에서 활용할 것이므로 `dif.max`라는 객체에 지정해주었다.

{% highlight javascript %}
head(tmp.dif, 10)
{% endhighlight %}

{% highlight javascript %}
   row1 row2 dif
1     9   11   2
2    21   22   1
3    32   33   1
4    43   43   0
5    55   55   0
6    65   65   0
7    80   80   0
8    91   92   1
9   103  104   1
10  115  115   0
{% endhighlight %}

2019년 데이터를 활용한 결과는 위와 같다. row1에는 keywords가 처음으로 나오는 row 위치, row2에는 keywords가 마지막으로 나오는 row 위치, dif에는 이 둘의 차이가 입력된다. dif가 2이면 3줄에 걸쳐서 keywords가 작성되어있다는 것을 의미하고, 0이면 keywords가 1줄에만 작성되어있다는 것을 의미한다.

{% highlight javascript %}
5    tmp.df <- data.frame(tmp.dif$dif)
6
7    for (i in 1:(dif.max+1)) {
8      tmp <- data.frame(1:n)
9      tmp.df <- data.frame(tmp.df, tmp)
10   }
11
12   names(tmp.df) <- c("dif", paste0("v", 1:(dif.max+1)))
{% endhighlight %}

다음으로 진행되는 작업은 여러 줄에 걸쳐 작성되어있는 Title, Authors, Keywords 등을 한 줄로 만들기 위해 틀을 만드는 과정이다. 우선 기존에 만들었던 data frame에서 dif 변수를 추후에 활용할 것이므로, dif 변수를 `tmp.df`의 첫 번째 열에 입력해 놓는다. 그리고 for 구문을 사용하여 차이의 최대값(여기서 `dif.max` 객체를 활용)만큼 새로운 열이 생성되도록 한다. 이 때, data frame에 들어갈 데이터들로 1:n을 사용하였는데, 이는 나중에 다른 값들이 입력될 것이기 때문에, 데이터에 포함된 논문의 수만큼 어떤 값들이 입력되어도 상관이 없다. data frame이 만들어진 이후에는 변수 이름을 "dif", "v1", "v2" 등으로 정리해준다.