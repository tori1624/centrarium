---
layout: post
title:  "R과 공간 데이터 (3)"
date:   2018-02-06 14:00:00
author: Youngho Lee
categories: Spatial-Data
cover:  "/assets/spatialdata/2017-12-31-spatialdata/spatial.jpg"
---

## 6. Point 데이터 간 거리 산정

이전까지는 공간 데이터의 사각화에 대해서 살펴보았다면, 이제부터는 공간 데이터를 활용하는 방법에 대해 설명할 계획이다. 첫번째로 가장 기본적인 point 데이터 간의 거리를 산정하는 방법에 대해 알아보고자 한다. point 데이터 간의 거리를 산정하는 방법에 대해서는 황소걸음 아카데미에서 출판한 'R 프로그램에 기반한 지리공간정보 자료 분석' 책의 내용을 참고하여 작성하였다. 이 책에서는 서울 남동지역 구청의 위치 정보를 예시로 들어 설명하였으므로, 이 포스팅에서도 이 예시를 활용하고자 하였다. 우선 경도와 위도로 구성된 지리좌표계에서의 point 데이터 간 거리 산정 방법에 대해 알아볼 것이다.

`sp` 패키지를 호출하고, 좌표를 직접 입력하여 데이터를 만든다. `Spatialpoints()` 함수는 일반적인 숫자를 proj4string 인자를 통해 지정한 좌표체계로 바꿔주는 역할을 한다.

{% highlight javascript %}
library(sp)
la <- c(37.5124500, 37.4781547, 37.4837493, 37.5104969, 37.5142013,
        37.5638433, 37.5325270, 37.5634557, 37.5386167, 37.5295204)
lo <- c(126.9395002, 126.9514850, 127.0325954, 127.0452259, 127.1066864,
        126.9976021, 126.9904904, 127.0368207, 127.0823747, 127.1234640)
ll <- data.frame(longtitude = lo, latitude = la)
cs <- CRS("+proj=longlat + datum=WGS84")
ll.sp <- SpatialPoints(ll, proj4string = cs)
name <- c("동작구청", "관악구청", "서초구청", "강남구청", "송파구청",
          "중구청", "용산구청", "성동구청", "광진구청", "강동구청")
{% endhighlight %}

입력한 데이터를 확인하면 다음과 같다.

{% highlight javascript %}
ll.sp
{% endhighlight %}

{% highlight javascript %}
SpatialPoints:
      longtitude latitude
 [1,]   126.9395 37.51245
 [2,]   126.9515 37.47815
 [3,]   127.0326 37.48375
 [4,]   127.0452 37.51050
 [5,]   127.1067 37.51420
 [6,]   126.9976 37.56384
 [7,]   126.9905 37.53253
 [8,]   127.0368 37.56346
 [9,]   127.0824 37.53862
[10,]   127.1235 37.52952
Coordinate Reference System (CRS) arguments: +proj=longlat +ellps=WGS84
{% endhighlight %}

`spDists()` 함수는 데이터의 모든 지점 간의 직선 거리를 km 단위로 산정하는 역할을 한다. `round()` 함수를 통해 소수점 둘째 자리까지만 나타낸 결과는 다음과 같다.

{% highlight javascript %}
round(spDists(ll.sp), 2)
{% endhighlight %}

{% highlight javascript %}
      [,1]  [,2] [,3] [,4]  [,5]  [,6]  [,7]  [,8]  [,9] [,10]
 [1,]  0.00  3.95 8.83 9.35 14.78  7.67  5.03 10.30 12.96 16.37
 [2,]  3.95  0.00 7.20 9.03 14.30 10.35  6.95 12.11 13.38 16.24
 [3,]  8.83  7.20 0.00 3.17  7.37  9.41  6.57  8.85  7.51  9.51
 [4,]  9.35  9.03 3.17 0.00  5.45  7.26  5.42  5.92  4.53  7.23
 [5,] 14.78 14.30 7.37 5.45  0.00 11.10 10.47  8.25  3.46  2.26
 [6,]  7.67 10.35 9.41 7.26 11.10  0.00  3.53  3.47  8.00 11.76
 [7,]  5.03  6.95 6.57 5.42 10.47  3.53  0.00  5.34  8.15 11.76
 [8,] 10.30 12.11 8.85 5.92  8.25  3.47  5.34  0.00  4.88  8.53
 [9,] 12.96 13.38 7.51 4.53  3.46  8.00  8.15  4.88  0.00  3.77
[10,] 16.37 16.24 9.51 7.23  2.26 11.76 11.76  8.53  3.77  0.00
{% endhighlight %}

다음으로는 UTM 좌표계에서의 point 데이터 간 거리 산정 방법에 대해 알아볼 것이다. 먼저 `rgdal` 패키지를 호출하고, `spTransform()` 함수를 통해 좌표계를 변환한다.

{% highlight javascript %}
round(spDists(ll.sp), 2)
{% endhighlight %}

{% highlight javascript %}
library(rgdal)
cs2 <- CRS("+proj=utm +zone=51 +ellps=WGS84 +units=km")
ll.sp2 <- spTransform(ll.sp, cs2)
{% endhighlight %}

UTM 좌표계로 변환한 데이터는 다음과 같다.

{% highlight javascript %}
ll.sp2
{% endhighlight %}

{% highlight javascript %}
SpatialPoints:
      longtitude latitude
 [1,]   848.2264 4159.019
 [2,]   849.4463 4155.256
 [3,]   856.5956 4156.182
 [4,]   857.5852 4159.200
 [5,]   863.0025 4159.847
 [6,]   853.1214 4164.941
 [7,]   852.6408 4161.438
 [8,]   856.5891 4165.047
 [9,]   860.7347 4162.463
[10,]   864.4115 4161.612
Coordinate Reference System (CRS) arguments: +proj=utm +zone=51 +ellps=WGS84
+units=km
{% endhighlight %}

UTM 좌표계에서는 `dist()` 함수를 통해 거리를 산정하는데, `method` 인자를 활용하여 유클리디안 거리(직선거리), 혹은 맨하튼 거리로 산정할 것인지 지정해줄 수 있다. 유클리디안 거리와 맨하튼 거리의 결과는 다음과 같다.

{% highlight javascript %}
round(dist(data.frame(ll.sp2), method = "euclidean"), 3)
{% endhighlight %}

{% highlight javascript %}
        1      2      3      4      5      6      7      8      9
2   3.955                                                        
3   8.837  7.209                                                 
4   9.361  9.044  3.176                                          
5  14.799 14.312  7.381  5.456                                   
6   7.684 10.359  9.423  7.273 11.117                            
7   5.034  6.958  6.578  5.428 10.483  3.536                     
8  10.309 12.119  8.865  5.931  8.257  3.469  5.349              
9  12.974 13.393  7.522  4.536  3.463  8.006  8.159  4.885       
10 16.392 16.259  9.517  7.240  2.259 11.771 11.772  8.543  3.774
{% endhighlight %}

{% highlight javascript %}
round(dist(data.frame(ll.sp2), method = "manhattan"), 3)
{% endhighlight %}

{% highlight javascript %}
        1      2      3      4      5      6      7      8      9
2   4.983                                                        
3  11.206  8.075                                                 
4   9.539 12.082  4.007                                          
5  15.604 18.147 10.071  6.064                                   
6  10.817 13.360 12.233 10.206 14.976                            
7   6.834  9.376  9.211  7.183 11.953  3.984                     
8  14.390 16.933  8.871  6.843 11.614  3.573  7.557              
9  15.953 18.496 10.420  6.413  4.885 10.091  9.119  6.729       
10 18.778 21.321 13.246  9.239  3.175 14.619 11.945 11.257  4.528
{% endhighlight %}

마지막으로는 실제 도로망을 고려한 거리를 계산하고자 한다. 출발지점은 동작구청이며, 목적지는 관악구청이다. 먼저 지리좌표께에서 데이터를 만든 방법과 같이 데이터를 만들고, 이후에 UTM 좌표계로 변환을 해준다.

{% highlight javascript %}
library(sp)
la <- c(37.5124500, 37.5059546, 37.4964553, 37.4958369,
        37.4900359, 37.4823951, 37.4812365, 37.4781547)
lo <- c(126.9395002, 126.9403928, 126.9405994, 126.9413620,
        126.9450080, 126.9465463, 126.9526738, 126.9514850)
ll <- data.frame(longtitude = lo, latitude = la)
cs <- CRS("+proj=longlat + datum=WGS84")
ll.sp <- SpatialPoints(ll, proj4string = cs)
library(rgdal)
cs2 <- CRS("+proj=utm +zone=51 +ellps=WGS84 +units=km")
ll.sp2 <- spTransform(ll.sp, cs2)
{% endhighlight %}

그리고 `Line()` 함수를 통해 점들을 잇는 직선의 형태로 만들어준다.

{% highlight javascript %}
ss <- Line(data.frame(ll.sp2))
{% endhighlight %}

실제 도로망을 고려한 결과는 다음과 같다.

{% highlight javascript %}
LineLength(ss)
{% endhighlight %}

{% highlight javascript %}
[1] 4.374461
{% endhighlight %}

최종적으로 결과들을 살펴보면, 동작구청과 관악구청 간의 유클리디안 거리(직선 거리)는 3.955km, 맨하튼 거리는 4.983km, 도로상의 실제 거리는 4.375km이다.