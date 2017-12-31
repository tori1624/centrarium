---
layout: post
title:  "R과 공간 데이터"
date:   2017-12-31 16:40:00
author: Youngho Lee
categories: Dummy
---

## 1. 공간 데이터 불러오기와 좌표체계

R 자체에는 일반 숫자와 좌표를 구분하는 것과 같이 공간 데이터와 관련된 기능이 내장되어있지 않다. 이에 따라 많은 사용자들이 공간 데이터를 다루기 위한 패키지들을 개발하였지만, 과거에는 이러한 패키지들이 서로 다른 가정을 가지고 만들어져 서로 호환되지 않는 문제가 발생하였다. 현재에는 `sp` 패키지가 공간 데이터를 다루기 위한 표준 역할을 하게 되면서 공간 데이터의 클래스는  하나로 통일되었으며, 공간 데이터와 관련된 패키지들은 대부분 `sp` 패키지를 기반으로 작동하게 되었다. 공간 데이터를 불러오기 위해 많이 사용되는 함수는 `readOGR()`로 `rgdal` 패키지에 내장되어 있다.

{% highlight javascript %}
# install.packages("rgdal")
library(rgdal)

goyang.sp <- readOGR("D:/Study/spatial_data_R/data/goyang/koyang_WGS.shp")
seoul.sp <- readOGR("D:/Study/spatial_data_R/data/seoul/seoul_tm.shp")
{% endhighlight %}

공간 데이터의 좌표체계를 확인하기 위해서는 `sp` 패키지에 내장되어 있는 `proj4string()` 함수를 이용하면 된다. `goyang.sp`와 `seoul.sp`의 좌표체계를 확인한 결과, `goyang.sp`는 WGS, `seoul.sp`는 TM인 것을 알 수 있다. 

{% highlight javascript %}
proj4string(goyang.sp)
proj4string(seoul.sp)
{% endhighlight %}

좌표체계를 변환하기 위해서는 `sp` 패키지에 내장되어 있는 `spTransform()` 함수를 이용하면 된다. 첫 번째 코드와 같이 `CRS projection`을 직접 입력해도 되지만, 두 번째 코드와 같이 좌표체계가 다른 데이터를 활용해서 변환하는 것도 가능하다.

{% highlight javascript %}
goyang.tm <- spTransform(goyang.sp, CRS("+proj=tmerc +lat_0=38 +lon_0=127 +k=1 
                                        +x_0=200000 +y_0=500000 +ellps=bessel 
                                        +units=m +no_defs"))
goyang.tm2 <- spTransform(goyang.sp, CRS(proj4string(seoul.sp)))
{% endhighlight %}

## 2. 공간 데이터의 시각화 (1)

공간 데이터의 시각화는 `plot()` 함수를 이용하면 된다. 만약 나타난 지도 위에 다른 지도를 나타내고 싶다면, add = TRUE 인자를 활용하면 된다. 여기서 주의할 점은 `add = TRUE` 인자는 R에 기본적으로 내장된 함수인 `plot()` 함수에는 존재하지 않고, `sp` 패키지에 내장된 `plot()` 함수에 존재한다는 것을 알고 있어야 한다. 위에서 사용한 데이터들을 같은 지도 위에 나타냈을 때, 경계가 정확히 맞지 않았으므로 공간데이터의 시각화 부분에서는 다른 데이터들을 활용하였으며, 좌표체계는 WGS로 설정하였다. (R markdown에서는 add = TRUE 인자를 활용할 때, 따로 실행하면 실행이 되지 않는다. 따라서 나타내고자 하는 모든 지도들을 동시에 실행시켜야 한다.)

{% highlight javascript %}
goyang.tm3 <- readOGR("D:/Study/spatial_data_R/data/goyang/koyang_dong.shp")
goyang.wgs <- spTransform(goyang.tm3, CRS(proj4string(goyang.sp)))
seoul.tm <- readOGR("D:/Study/spatial_data_R/data/seoul/Seoul_dong.shp")
seoul.wgs <- spTransform(seoul.tm, CRS(proj4string(goyang.sp)))

plot(seoul.wgs, border = "darkgray")
plot(goyang.wgs, add = TRUE, border = "darkgray")
{% endhighlight %}

위의 지도에서는 나타난 지도가 서울시를 중심으로 되어있기 때문에, 고양시 일부분이 생략된 것을 볼 수 있다. 이와 같은 문제를 해결하기 위해서는 `sp` 패키지에 내장되어 있는 `bbox()` 함수를 이용하면 된다. `bbox()` 함수는 공간 데이터의 x,y 좌표의 최댓값과 최솟값을 보여주기 때문에, 이 함수를 통해 우선 x,y 좌표의 범위를 확인하고 `plot()` 함수의 xlim, ylim 인자를 활용하면 전체적인 지도를 확인할 수 있게 된다. `plot()` 함수는 xlim, ylim 인자 외에도 border, col, pch, cex 등과 같이 R에 내장된 `plot()` 함수의 인자들과 유사한 인자들을 포함하고 있다.

{% highlight javascript %}
bbox(seoul.wgs)
bbox(goyang.wgs)
{% endhighlight %}```

{% highlight javascript %}
plot(seoul.wgs, xlim = c(126.66, 127.2), ylim = c(37.4, 37.76), 
     border = "white", col = "gray")
plot(goyang.wgs, border = "white", col = "gray", add = TRUE)
{% endhighlight %}