---
layout: post
title:  "R과 공간 데이터 (2)"
date:   2018-01-08 23:40:00
author: Youngho Lee
categories: Spatial-Data
cover:  "/assets/spatialdata/2017-12-31-spatialdata/spatial.jpg"
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

<img src = "/assets/spatialdata/2018-01-08-spatialdata/rplot1.png" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />

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

<img src = "/assets/spatialdata/2018-01-08-spatialdata/rplot2.png" title = "plot2" alt = "plot2" width = "1008" style = "display: block; margin: auto;" />

## 공간 데이터의 시각화 (point density)

다음은 point density의 시각화에 대해 설명하고자 한다. point density 시각화 부분에서 사용하고자 하는 데이터는 서울시 wifi 위치 정보를 가지고 있는 데이터이다. 이 부분에서는 이전처럼 공간 데이터 시각화의 기본적인 요소들로 시각화를 하는 방법을 아직 찾지 못했기 때문에, `ggplot2` 패키지의 함수들을 활용하고자 한다. 우선 `ggplot2` 패키지를 호출하고 필요한 데이터들을 불러온다. 

{% highlight javascript %}
library(ggplot2)

wifi.sp <- readOGR("D:/Study/spatial_data_R/data/etc/wifi.shp")
wifi <- read.csv("D:/Data/Public_data/Wifi/seoul_wifi_location.csv")
{% endhighlight %}

{% highlight javascript %}
OGR data source with driver: ESRI Shapefile 
Source: "D:/Study/spatial_data_R/data/etc/wifi.shp", layer: "wifi"
with 2994 features
It has 6 fields
{% endhighlight %}

먼저 polygon 데이터를 나타낸 후, 여기서는 `xlim()`과 `ylim()` 함수를 통해 x축과 y축의 범위를 지정해주었는데, 이는 wifi 데이터에 oulier가 존재하기 때문이다. 다음으로 point 데이터 시각화 부분과 동일하게 point 데이터를 나타내준다. 이후에 사용되는 함수들이 point density와 관련된 함수들이다. `geom_density2d()` 함수는 point density를 등고선의 형태로 보여주고, `stat_density2d()` 함수는 point density를 색으로 보여준다. 기본적인 인자를 지정해준 결과는 다음과 같다.

{% highlight javascript %}
ggplot() +
  geom_polygon(data = seoul.wgs, aes(x = long, y = lat, group = group),
               fill = 'white', color = 'Grey 50') +
  xlim(126.75, 127.2) + ylim(37.42, 37.7) +
  geom_point(data = wifi, aes(x = lon, y = lat, alpha = 0.5), 
             color = 'Grey 50') +
  geom_density2d(data = wifi, aes(x = lon, y = lat), size = 0.3, 
                 color = 'Grey50') +
  stat_density2d(data = wifi, aes(x = lon, y = lat, 
                                  fill = ..level.., alpha = ..level..), 
                 size = 0.3, geom = "polygon")
{% endhighlight %}

<img src = "/assets/spatialdata/2018-01-08-spatialdata/rplot3.png" title = "plot3" alt = "plot3" width = "1008" style = "display: block; margin: auto;" />

다음으로는 위의 지도의 요소들을 바꿔 꾸미는 방법에 대해 설명하고자 한다. `scale_fill_gradient()` 함수는 `stat_density2d()` 함수뿐만 아니라 다양한 시각화 함수에서 나타날 색을 지정해줄 수 있다. `scale_alpha()` 함수는 투명도의 범위를 지정해줄 수 있고, `guide = FALSE` 인자를 통해 범례에서 생략할 수 있다. `theme_classic()`는 테마를 지정할 수 있는 함수로, 이 함수 외에도 `theme_void()` 등 다양한 테마들이 있다. 마지막으로 `theme()` 함수는 범례, x축, y축 등 세부적인 요소들을 지정해줄 수 있는 기능을 한다. 여기서는 범례와 범례 글씨의 크기를 지정하였다. 지금까지의 결과는 다음과 같다. 

{% highlight javascript %}
ggplot() +
  geom_polygon(data = seoul.wgs, aes(x = long, y = lat, group = group),
               fill = 'white', color = 'Grey 50') +
  xlim(126.75, 127.2) + ylim(37.42, 37.7) +
  geom_point(data = wifi, aes(x = lon, y = lat, alpha = 0.5), 
             color = 'Grey 50') +
  geom_density2d(data = wifi, aes(x = lon, y = lat), size = 0.3, 
                 color = 'Grey50') +
  stat_density2d(data = wifi, aes(x = lon, y = lat, 
                                  fill = ..level.., alpha = ..level..), 
                 size = 0.3, geom = "polygon") + 
  scale_fill_gradient(low = "Light Green", high = "Indian Red") + 
  scale_alpha(range = c(0.2, 0.4), guide = FALSE) +
  theme_classic() + theme(legend.key.heigh = unit(1.3,'cm'), 
                          legend.key.width = unit(1.3,'cm'),
                          legend.title = element_text(size = 17),
                          legend.text = element_text(size = 15))
{% endhighlight %}

<img src = "/assets/spatialdata/2018-01-08-spatialdata/rplot4.png" title = "plot4" alt = "plot4" width = "1008" style = "display: block; margin: auto;" />

추가적으로 `ggplot2` 패키지에서 방위와 축척을 추가하는 방법에 대해 살펴볼 것이다. 이를 위해서는 `ggsn` 패키지를 설치하고 호출해야만 한다. 그리고 polygon 데이터를 `fortify()` 함수를 통해 data frame 형태의 객체로 따로 만들어야 한다. 

{% highlight javascript %}
# install.packages("ggsn")
library(ggsn)

seoul.fortify <- fortify(seoul.wgs)
{% endhighlight %}

`north()`는 방위를 추가해주는 함수이며, `scalebar()`는 축척을 추가해주는 함수이다. 각 함수에 `fortify()` 함수를 통해 만든 객체를 넣어주고 위치와 같은 인자들을 지정해주면 된다. 결과는 다음과 같다. 

{% highlight javascript %}
ggplot() +
  geom_polygon(data = seoul.wgs, aes(x = long, y = lat, group = group),
               fill = 'white', color = 'Grey 50') +
  xlim(126.75, 127.2) + ylim(37.42, 37.7) +
  geom_point(data = wifi, aes(x = lon, y = lat, alpha = 0.5), 
             color = 'Grey 50') +
  geom_density2d(data = wifi, aes(x = lon, y = lat), size = 0.3, 
                 color = 'Grey50') +
  stat_density2d(data = wifi, aes(x = lon, y = lat, 
                                  fill = ..level.., alpha = ..level..), 
                 size = 0.3, geom = "polygon") + 
  scale_fill_gradient(low = "Light Green", high = "Indian Red") + 
  scale_alpha(range = c(0.2, 0.4), guide = FALSE) +
  theme_classic() + theme(legend.key.heigh = unit(1.3,'cm'), 
                          legend.key.width = unit(1.3,'cm'),
                          legend.title = element_text(size = 17),
                          legend.text = element_text(size = 15)) +
  north(seoul.fortify, location = 'topright', symbol = 3, scale = 0.1) +
  scalebar(seoul.fortify, dist = 2.5, dd2km = TRUE, model = 'WGS84', 
           location = 'bottomleft')
{% endhighlight %}

<img src = "/assets/spatialdata/2018-01-08-spatialdata/rplot5.png" title = "plot5" alt = "plot5" width = "1008" style = "display: block; margin: auto;" />