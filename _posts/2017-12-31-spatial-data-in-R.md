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

goyang.sp <- readOGR("D:/Study/spatial_data_R/data/koyang_WGS.shp")
seoul.sp <- readOGR("D:/Study/spatial_data_R/data/seoul_tm.shp")
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