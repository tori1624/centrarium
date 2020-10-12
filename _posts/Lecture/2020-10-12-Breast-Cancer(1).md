---
layout: post
title: "Principal Components Analysis - Breast Cancer Wisconsin (Diagnostic) (1)"
author: "Youngho Lee"
date: "2020.10.12"
categories: Lecture
cover: "/assets/Lecture/2020-10-12-Breast-Cancer/breast cancer.jpeg"
---

이번 포스팅은 2020학년도 2학기 지리학과 대학원 수업 중 하나인 홍성연 교수님의 다변량데이터분석의 두 번째 팀과제 결과를 바탕으로 작성되었다. 제시된 자료는 Kaggle 사이트에서 제공되는 유방암 환자 데이터로, 유방암이 양성인지 혹은 악성인지, 종양의 반경, 둘레 면적 등의 정보가 포함되어 있으며, 각 변수들은 평균(mean), 표준편차(se), 상위 3개 값의 평균(worst), 세 개의 값들을 가지고 있다. 유방암은 총 5단계로 구분되는데, 0기는 자연척 치료가 가능하고, 1~2기는 수술을 통한 치료를 진행하며, 3~4기는 생존율이 50% 미만이다. 각 기수마다 치료법의 적용이 달라지는데, 활용 데이터에는 종양이 양성과 악성, 2단계로만 구분되어있다. 따라서 본 분석에서는 주성분 분석(Principal Components Analysis)과 K-Means 군집 분석을 적용하여 종양을 악성과 양성, 2단계보다 많은 4개의 그룹으로 구분하고자 한다.

{% highlight javascript %}
library(factoextra)
library(caret)
library(corrplot)

data_path <- "D:/Study/2020/multivariate/team assign/assign2/"
bcw_raw <- read.csv(paste0(data_path, "data.csv"))

str(bcw_raw)
{% endhighlight %}

{% highlight javascript %}
'data.frame':	569 obs. of  32 variables:
 $ id                     : int  842302 842517 84300903 84348301 84358402 843786 844359 84458202 844981 84501001 ...
 $ diagnosis              : Factor w/ 2 levels "B","M": 2 2 2 2 2 2 2 2 2 2 ...
 $ radius_mean            : num  18 20.6 19.7 11.4 20.3 ...
 $ texture_mean           : num  10.4 17.8 21.2 20.4 14.3 ...
 $ perimeter_mean         : num  122.8 132.9 130 77.6 135.1 ...
 $ area_mean              : num  1001 1326 1203 386 1297 ...
 $ smoothness_mean        : num  0.1184 0.0847 0.1096 0.1425 0.1003 ...
 $ compactness_mean       : num  0.2776 0.0786 0.1599 0.2839 0.1328 ...
 $ concavity_mean         : num  0.3001 0.0869 0.1974 0.2414 0.198 ...
 $ concave.points_mean    : num  0.1471 0.0702 0.1279 0.1052 0.1043 ...
 $ symmetry_mean          : num  0.242 0.181 0.207 0.26 0.181 ...
 $ fractal_dimension_mean : num  0.0787 0.0567 0.06 0.0974 0.0588 ...
 $ radius_se              : num  1.095 0.543 0.746 0.496 0.757 ...
 $ texture_se             : num  0.905 0.734 0.787 1.156 0.781 ...
 $ perimeter_se           : num  8.59 3.4 4.58 3.44 5.44 ...
 $ area_se                : num  153.4 74.1 94 27.2 94.4 ...
 $ smoothness_se          : num  0.0064 0.00522 0.00615 0.00911 0.01149 ...
 $ compactness_se         : num  0.049 0.0131 0.0401 0.0746 0.0246 ...
 $ concavity_se           : num  0.0537 0.0186 0.0383 0.0566 0.0569 ...
 $ concave.points_se      : num  0.0159 0.0134 0.0206 0.0187 0.0188 ...
 $ symmetry_se            : num  0.03 0.0139 0.0225 0.0596 0.0176 ...
 $ fractal_dimension_se   : num  0.00619 0.00353 0.00457 0.00921 0.00511 ...
 $ radius_worst           : num  25.4 25 23.6 14.9 22.5 ...
 $ texture_worst          : num  17.3 23.4 25.5 26.5 16.7 ...
 $ perimeter_worst        : num  184.6 158.8 152.5 98.9 152.2 ...
 $ area_worst             : num  2019 1956 1709 568 1575 ...
 $ smoothness_worst       : num  0.162 0.124 0.144 0.21 0.137 ...
 $ compactness_worst      : num  0.666 0.187 0.424 0.866 0.205 ...
 $ concavity_worst        : num  0.712 0.242 0.45 0.687 0.4 ...
 $ concave.points_worst   : num  0.265 0.186 0.243 0.258 0.163 ...
 $ symmetry_worst         : num  0.46 0.275 0.361 0.664 0.236 ...
 $ fractal_dimension_worst: num  0.1189 0.089 0.0876 0.173 0.0768 ...
{% endhighlight %}

분석을 위한 패키지, 데이터를 불러오고, 구조를 간단하게 살펴보면 위와 같다. 데이터에는 환자 id, 종양이 악성인지 양성인지에 대한 상태, 종양에 대한 다양한 정보가 포함되어 있다.

{% highlight javascript %}
par(mar = c(1, 1, 1, 1))
corrplot(cor(bcw_raw[, 3:32]), method = "color", tl.col = "black")
{% endhighlight %}

<img src = "/assets/Lecture/2020-10-12-Breast-Cancer/correlation.png" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />

우선 변수 간의 상관관계를 시각적으로 표현한 결과는 위와 같다. 결과를 보면, 변수의 수가 많고 일부 변수 간의 상관관계가 높은 것을 확인할 수 있다. 이와 같이 관측치에 비해 변수가 많거나 변수 간의 상관관계가 높은 데이터에 적용하는 것이 주성분 분석이다. 주성분 분석은 다차원의 데이터를 효율적으로 저차원의 데이터로 요약하는 방법이다. R에서 주성분 분석이 가능한 함수로는 `prcomp()`와 `princomp()`가 내장되어 있으며, `princomp()`는 고유값 분해를, `prcomp()`는 특이값 분해를 사용한다. 본 분석에서는 `prcomp()` 함수를 사용하였다.

{% highlight javascript %}
bcw_pcov <- prcomp(bcw_raw[, 3:32]) # covariance matrix
summary(bcw_pcov)
{% endhighlight %}

{% highlight javascript %}
Importance of components:
                           PC1      PC2      PC3     PC4     PC5     PC6   PC7    PC8    PC9
Standard deviation     666.170 85.49912 26.52987 7.39248 6.31585 1.73337 1.347 0.6095 0.3944
Proportion of Variance   0.982  0.01618  0.00156 0.00012 0.00009 0.00001 0.000 0.0000 0.0000
Cumulative Proportion    0.982  0.99822  0.99978 0.99990 0.99999 0.99999 1.000 1.0000 1.0000
                         PC10   PC11    PC12    PC13    PC14    PC15   PC16    PC17    PC18
Standard deviation     0.2899 0.1778 0.08659 0.05623 0.04649 0.03642 0.0253 0.01936 0.01534
Proportion of Variance 0.0000 0.0000 0.00000 0.00000 0.00000 0.00000 0.0000 0.00000 0.00000
Cumulative Proportion  1.0000 1.0000 1.00000 1.00000 1.00000 1.00000 1.0000 1.00000 1.00000
                          PC19    PC20     PC21    PC22     PC23     PC24     PC25     PC26
Standard deviation     0.01359 0.01281 0.008838 0.00759 0.005909 0.005329 0.004018 0.003534
Proportion of Variance 0.00000 0.00000 0.000000 0.00000 0.000000 0.000000 0.000000 0.000000
Cumulative Proportion  1.00000 1.00000 1.000000 1.00000 1.000000 1.000000 1.000000 1.000000
                           PC27     PC28     PC29      PC30
Standard deviation     0.001918 0.001688 0.001416 0.0008379
Proportion of Variance 0.000000 0.000000 0.000000 0.0000000
Cumulative Proportion  1.000000 1.000000 1.000000 1.0000000
{% endhighlight %}

{% highlight javascript %}
bcw_pcor <- prcomp(bcw_raw[, 3:32], scale = TRUE, center = TRUE) # correlation matrix
summary(bcw_pcor)
{% endhighlight %}

{% highlight javascript %}
Importance of components:
                          PC1    PC2     PC3     PC4     PC5     PC6     PC7     PC8    PC9
Standard deviation     3.6444 2.3857 1.67867 1.40735 1.28403 1.09880 0.82172 0.69037 0.6457
Proportion of Variance 0.4427 0.1897 0.09393 0.06602 0.05496 0.04025 0.02251 0.01589 0.0139
Cumulative Proportion  0.4427 0.6324 0.72636 0.79239 0.84734 0.88759 0.91010 0.92598 0.9399
                          PC10   PC11    PC12    PC13    PC14    PC15    PC16    PC17    PC18
Standard deviation     0.59219 0.5421 0.51104 0.49128 0.39624 0.30681 0.28260 0.24372 0.22939
Proportion of Variance 0.01169 0.0098 0.00871 0.00805 0.00523 0.00314 0.00266 0.00198 0.00175
Cumulative Proportion  0.95157 0.9614 0.97007 0.97812 0.98335 0.98649 0.98915 0.99113 0.99288
                          PC19    PC20   PC21    PC22    PC23   PC24    PC25    PC26    PC27
Standard deviation     0.22244 0.17652 0.1731 0.16565 0.15602 0.1344 0.12442 0.09043 0.08307
Proportion of Variance 0.00165 0.00104 0.0010 0.00091 0.00081 0.0006 0.00052 0.00027 0.00023
Cumulative Proportion  0.99453 0.99557 0.9966 0.99749 0.99830 0.9989 0.99942 0.99969 0.99992
                          PC28    PC29    PC30
Standard deviation     0.03987 0.02736 0.01153
Proportion of Variance 0.00005 0.00002 0.00000
Cumulative Proportion  0.99997 1.00000 1.00000
{% endhighlight %}

주성분 분석을 시행하기 이전에, 공분산 행렬을 사용할 것인지, 상관관계 행렬을 사용할 것인지 선택하는 것은 결과에 큰 영향을 미치므로 중요한 문제이다. 위의 결과를 살펴보면, `prcomp()` 함수에 아무 인자도 사용하지 않을 경우에는 공분산 행렬이 사용된다. 하지만 분산의 비율을 살펴봤을 때, 첫 번째 주성분의 분산 비율이 너무 지배적인 것을 확인할 수 있다. 반면에 `prcomp()` 함수에 `scale`과 `center` 인자를 TRUE로 설정하면, 상관관계 행렬이 적용된다. 이 경우의 분산의 비율은 공분산 행렬보다 고르게 분포한 것을 볼 수 있다. 이와 같이 상관관계 행렬은 변수 간의 스케일 차이가 클 때 활용하기 적절한 방법으로, 본 분석에서도 변수 간 스케일 차이가 크므로 상관관계 행렬을 사용하였다.

{% highlight javascript %}
summary(bcw_pcor)
{% endhighlight %}

{% highlight javascript %}
Importance of components:
                          PC1    PC2     PC3     PC4     PC5     PC6     PC7
Standard deviation     3.6444 2.3857 1.67867 1.40735 1.28403 1.09880 0.82172
Proportion of Variance 0.4427 0.1897 0.09393 0.06602 0.05496 0.04025 0.02251
Cumulative Proportion  0.4427 0.6324 0.72636 0.79239 0.84734 0.88759 0.91010
{% endhighlight %}

다음은 주성분의 개수를 선택하는 과정으로, 주성분의 적절한 개수를 선택하는 방법은 4가지 정도가 존재한다. 첫 번째는 주성분의 분산 총합이 70-90%에 해당하는 주성분들을 선택하는 것이다. 위의 결과를 다시보면, 분산의 총합이 70-90%에 해당하는 주성분의 개수는 3~6개인 것을 확인할 수 있다.

{% highlight javascript %}
mean(bcw_pcor$sdev^2)
{% endhighlight %}

{% highlight javascript %}
[1] 1
{% endhighlight %}

{% highlight javascript %}
round(bcw_pcor$sdev^2, 4)
{% endhighlight %}

{% highlight javascript %}
 [1] 13.2816  5.6914  2.8179  1.9806  1.6487  1.2074  0.6752  0.4766  0.4169  0.3507  0.2939
[12]  0.2612  0.2414  0.1570  0.0941  0.0799  0.0594  0.0526  0.0495  0.0312  0.0300  0.0274
[23]  0.0243  0.0181  0.0155  0.0082  0.0069  0.0016  0.0007  0.0001
{% endhighlight %}

두 번째 방법은 주성분의 고유값들의 평균보다 큰 주성분들만을 선택하는 것이다. 주성분의 고유값들의 평균은 1이고, 1보다 큰 주성분은 6번째까지이다. 따라서 이 방법에서는 6개가 적절한 개수로 도출되었다. 또한, 세 번째 방법은 교유값이 0.7이상인 것만 선택하는 것으로, 이 방법도 6개가 적절한 개수인 것을 확인할 수 있다.

{% highlight javascript %}
par(mar = c(5, 5, 5, 0))

plot.new()
plot.window(xlim = c(0, 30), ylim = c(0, 14))
abline(h = seq(0, 14, 2), v = seq(0, 30, 2), col = "grey", lty = 3)

lines(bcw_pcor$sdev^2, lwd = 3)
segments(6.5, 0, 6.5, 14, col = "red", lwd = 2, lty = 3)

axis(side = 1, at = seq(0, 30, 4), cex.axis = 1)
axis(side = 2, at = seq(0, 14, 2), las = 2, cex.axis = 1)

mtext("Component Number", 1, line = 3, cex = 1)
mtext("Component Variance", 2, line = 3, cex = 1)
mtext("Scree Diagram", 3, line = 1, cex = 1.25)
{% endhighlight %}

<img src = "/assets/Lecture/2020-10-12-Breast-Cancer/Scree diagram.png" title = "plot2" alt = "plot2" width = "1008" style = "display: block; margin: auto;" />

마지막은 Scree 다이어그램으로 선에서 급격하게 변하는 지점을 바탕으로 개수를 선정하는 방법으로, 위의 그래프에서는 6~7 정도에서 급격히 변하는 것을 확인할 수 있다. 이 방법에서도 6~7개가 적절한 주성분 개수라고 도출되어, 본 분석에서는 6개의 주성분을 사용하는 것이 적절하다고 판단하였다. 이후 분석 과정은 다음 포스팅에서 설명하고자 한다.