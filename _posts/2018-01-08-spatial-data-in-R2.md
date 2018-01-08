---
layout: post
title:  "R과 공간 데이터 (2)"
date:   2018-01-08 23:40:00
author: Youngho Lee
categories: Dummy
cover:  "/assets/sptialdata_inR_1/spatial.jpg"
---

## 1. 공간 데이터의 시각화 (polygon 데이터)

이전에는 point 데이터의 시각화에 대해 설명했다면, 이번에는 polygon 데이터의 시각화에 대해 설명하고자 한다. 우선 시각화를 위한 기본적인 패키지들을 호출한다. 

{% highlight javascript %}
library(rgdal)
library(RColorBrewer)
library(classInt)
library(ggplot2)
{% endhighlight %}

다음으로 시각화를 위한 데이터들을 불러온다. 공간 데이터는 서울시 동별 행정경계를 나타내며, WGS 좌표체계로 변형하였다. csv 데이터는 서울시 동별 폭염일수, 노인인구밀도, 기초생활보장 수급자 수, 녹지면적의 속성 정보를 가지고 있다.

{% highlight javascript %}
seoul.sp <- readOGR("D:/Study/spatial_data_R/data/seoul/Seoul_dong.shp")
seoul.wgs <- spTransform(seoul.sp, CRS("+proj=longlat +datum=WGS84 
                                       +no_defs +ellps=WGS84 +towgs84=0,0,0"))
seoul.heat <- read.csv("D:/Study/spatial_data_R/data/seoul/seoul_heat.csv")
{% endhighlight %}

{% highlight javascript %}
OGR data source with driver: ESRI Shapefile 
Source: "D:/Study/spatial_data_R/data/seoul/Seoul_dong.shp", layer: "Seoul_dong"
with 423 features
It has 3 fields
{% endhighlight %}

point 데이터의 경우에는 polygon 데이터를 시각화한 다음에 add = TRUE 인자를 사용하여 point 데이터를 추가적으로 시각화하였다. 하지만 polygon 데이터의 경우에는 polygon 데이터를 시각화할 때, `plot()` 함수의 인자들을 활용하면 되기 때문에 오히려 더 간편하다. 클래스를 구분하고 색을 지정해주는 과정은 R과 공간데이터 (1)의 공간 데이터의 시각화 (point 데이터)부분에서 설명한 것을 참고하면 된다. 서울시 동별 폭염일수를 시각화한 결과는 다음과 같다.

{% highlight javascript %}
reds <- brewer.pal(5, "Reds")
heatclass <- classIntervals(seoul.heat$heat_wave, n = 5, style = "jenks")

plot(seoul.wgs, col = findColours(heatclass, reds), border = "Grey 50",
     main = "서울시 동별 폭염일수")
{% endhighlight %}

<img src = "/assets/2018-01-08-sptialdata/rplot1.png" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />

polygon 데이터 시각화에 대해 설명하는 부분이 짧았기 때문에 추가적으로 하나의 그림에 여러 개의 지도르 나타내는 방법에 대해 설명하고자 한다. 우선 폭염일수를 제외한 다른 속성 정보들도 클래스를 구분하고 색을 지정해준다.

{% highlight javascript %}
brown <- brewer.pal(5, "YlOrBr")
elderclass <- classIntervals(seoul.heat$elder_dens, n = 5, style = "jenks")

blues <- brewer.pal(5, "Blues")
receiverclass <- classIntervals(seoul.heat$receiver, n = 5, style = "jenks")

greens <- brewer.pal(5, "Greens")
greenclass <- classIntervals(seoul.heat$green_area, n = 5, style = "jenks")
{% endhighlight %}

이 부분에서는 R에 기본적으로 내장되어 있는 `par()` 함수를 사용할 것이다. `par()` 함수의 인자에는 여러 가지가 있는데, 하나의 그림에 여러 개의 그림을 나타내기 위해서는 `mfrow`나 `mfcol` 인자를 사용하면 된다. `mfrow`의 경우에는 행 위주로 그림이 들어가고, `mfcol`의 경우에는 열 위주로 그림이 들어간다. `mfrow = c(2, 2)`로 지정을 하게 되면, 2x2 형태로 총 4개의 그림이 들어갈 수 있게 된다. `mar` 인자는 여백을 지정해주는 기능을 한다. `par.origin` 객체를 따로 지정해주는 이유는 `mfrow = c(2, 2)` 인자를 한 번 지정해주게 되면, 계속해서 적용되는 오류가 발생하는 것을 방지하고자 하는 것이다. 결과는 다음과 같다.

{% highlight javascript %}
par.origin <- par(no.readonly = TRUE)

par(mfrow = c(2, 2), mar = c(0.1, 0.1, 0.1, 0.1))
plot(seoul.wgs, col = findColours(heatclass, reds), border = "Grey 50",
     main = "서울시 동별 폭염일수")
plot(seoul.wgs, col = findColours(elderclass, brown), border = "Grey 50",
     main = "서울시 동별 노인인구밀도")
plot(seoul.wgs, col = findColours(receiverclass, blues), border = "Grey 50",
     main = "서울시 동별 기초생활보장 수급자 수")
plot(seoul.wgs, col = findColours(greenclass, greens), border = "Grey 50",
     main = "서울시 동별 녹지면적")
par(par.origin)
{% endhighlight %}

<img src = "/assets/2018-01-08-sptialdata/rplot2.png" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />