---
layout: post
title: "Principal Components Analysis - Breast Cancer Wisconsin (Diagnostic) (2)"
author: "Youngho Lee"
date: "2020.11.09"
categories: Lecture
cover: "/assets/Lecture/2020-10-12-Breast-Cancer/breast-cancer.jpg"
---

이번 포스팅은 저번 포스팅에 이어 작성되었으므로, 본 포스팅을 처음보는 분들에게는 이전 포스팅을 참고하는 것을 추천드린다. 전체적인 분석을 다시 간단히 요약하면, 활용 데이터인 유방암 환자 데이터에는 종양이 양성과 악성, 2단계로만 구분되어있지만, 유방암은 총 5단계로 구분되어 0기는 자연적 치료가 가능하고, 1~4기는 각기 다른 치료법이 적용된다. 이에 따라 본 분석에서는 주성분 분석(Principal Components Analysis)과 K-Means 군집 분석을 적용하여 종양을 악성과 양성, 2단계보다 많은 4개의 그룹으로 구분하고자 하였다. 이전 포스팅까지는 데이터를 살펴보고 최적의 주성분 개수를 찾는 과정으로, 최적의 주성분 개수는 6개로 도출되었다. 이번 포스팅에서는 우선 (1) 원본 데이터와 (2) 주성분 분석을 진행한 데이터를 바탕으로 비지도 학습인 K-Means 군집 분석을 진행하고, 두 데이터에 대해 정확도를 비교하고자 한다.

{% highlight javascript %}
set.seed(1234)
kmeans_model <- kmeans(bcw_raw[, 3:32], 2, iter.max = 10, nstart = 1)
print(kmeans_model)
bcw_raw$kmeans_raw2 <- kmeans_model$cluster
{% endhighlight %}

위의 코드는 원본 데이터를 바탕으로 k를 2로 설정하고 K-Means 군집 분석을 진행한 것이며, 아래 코드와 그림은 군집 분석 결과를 시각화한 것이다.

{% highlight javascript %}
par(mar = c(5, 5, 5, 0))

plot.new()
plot.window(xlim = c(-20, 10), ylim = c(-15, 10))
abline(h = seq(-15, 10, 5), v = seq(-20, 10, 5), col = "grey", lty = 3)

points(bcw_raw[bcw_raw$kmeans_raw2 == 2, "pc1"], 
       bcw_raw[bcw_raw$kmeans_raw2 == 2, "pc2"], pch = 16, col = "#FC766AFF",
       cex = 0.75)
points(bcw_raw[bcw_raw$kmeans_raw2 == 1, "pc1"], 
       bcw_raw[bcw_raw$kmeans_raw2 == 1, "pc2"], pch = 16, col = "#5B84B1FF",
       cex = 0.75)

axis(side = 1, at = seq(-20, 10, 5), cex.axis = 1)
axis(side = 2, at = seq(-15, 10, 5), las = 2, cex.axis = 1)

mtext("PC1", 1, line = 3, cex = 1)
mtext("PC2", 2, line = 3, cex = 1)

legend(-20, 10, pch = 16, col = c("#5B84B1FF", "#FC766AFF"), 
       legend = c("B", "M"))
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Kmeans2_raw.png" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />

두 개의 그룹으로 잘 나뉜 것을 확인할 수 있다.

{% highlight javascript %}
set.seed(1234)
kmeans_model <- kmeans(bcw_pcor$x[, 1:6], 2, iter.max = 10, nstart = 1)
print(kmeans_model)
bcw_raw$kmeans_pca2 <- kmeans_model$cluster
{% endhighlight %}

위의 코드는 주성분 분석이 진행된 데이터를 바탕으로 k를 2로 설정하고 K-Means 군집 분석을 진행한 것이며, 아래 코드와 그림은 군집 분석 결과를 시각화한 것이다.

{% highlight javascript %}
par(mar = c(5, 5, 5, 0))

plot.new()
plot.window(xlim = c(-20, 10), ylim = c(-15, 10))
abline(h = seq(-15, 10, 5), v = seq(-20, 10, 5), col = "grey", lty = 3)

points(bcw_raw[bcw_raw$kmeans_pca2 == 2, "pc1"], 
       bcw_raw[bcw_raw$kmeans_pca2 == 2, "pc2"], pch = 16, col = "#FC766AFF",
       cex = 0.75)
points(bcw_raw[bcw_raw$kmeans_pca2 == 1, "pc1"], 
       bcw_raw[bcw_raw$kmeans_pca2 == 1, "pc2"], pch = 16, col = "#5B84B1FF",
       cex = 0.75)

axis(side = 1, at = seq(-20, 10, 5), cex.axis = 1)
axis(side = 2, at = seq(-15, 10, 5), las = 2, cex.axis = 1)

mtext("PC1", 1, line = 3, cex = 1)
mtext("PC2", 2, line = 3, cex = 1)

legend(-20, 10, pch = 16, col = c("#5B84B1FF", "#FC766AFF"), 
       legend = c("B", "M"))
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Kmeans2_pca.png" title = "plot2" alt = "plot2" width = "1008" style = "display: block; margin: auto;" />

원본 데이터와 마찬가지로 두 개의 그룹으로 잘 나뉘었지만, 분포는 다른 것을 확인할 수 있다.

{% highlight javascript %}
par(mar = c(5, 5, 5, 0))

plot.new()
plot.window(xlim = c(-20, 10), ylim = c(-15, 10))
abline(h = seq(-15, 10, 5), v = seq(-20, 10, 5), col = "grey", lty = 3)

points(bcw_raw[bcw_raw$diagnosis == "M", "pc1"], 
       bcw_raw[bcw_raw$diagnosis == "M", "pc2"], pch = 16, col = "#FC766AFF",
       cex = 0.75)
points(bcw_raw[bcw_raw$diagnosis == "B", "pc1"], 
       bcw_raw[bcw_raw$diagnosis == "B", "pc2"], pch = 16, col = "#5B84B1FF",
       cex = 0.75)

axis(side = 1, at = seq(-20, 10, 5), cex.axis = 1)
axis(side = 2, at = seq(-15, 10, 5), las = 2, cex.axis = 1)

mtext("PC1", 1, line = 3, cex = 1)
mtext("PC2", 2, line = 3, cex = 1)

legend(-20, 10, pch = 16, col = c("#5B84B1FF", "#FC766AFF"), 
       legend = c("B", "M"))
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Kmeans2_actual.png" title = "plot3" alt = "plot3" width = "1008" style = "display: block; margin: auto;" />

위의 코드와 그림은 실제값인 양성(B)과 악성(M)을 시각화 한 것으로, 주성분 데이터를 바탕으로 진행된 군집 분석의 결과와 더 유사해보이는 것으로 나타났다. 하지만 시각적으로 확인하는 것은 주관적일 수 있으므로, 혼동행렬을 통해 객관적인 정확도 검증을 진행하였다.

{% highlight javascript %}
bcw_raw$diagnosis_n <- ifelse(bcw_raw$diagnosis == "M", 1, 2)

confusionMatrix(factor(bcw_raw$kmeans_raw2), factor(bcw_raw$diagnosis_n))
{% endhighlight %}

{% highlight javascript %}
Confusion Matrix and Statistics

          Reference
Prediction   1   2
         1 130   1
         2  82 356
                                          
               Accuracy : 0.8541          
                 95% CI : (0.8224, 0.8821)
    No Information Rate : 0.6274          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.6618          
                                          
 Mcnemar's Test P-Value : < 2.2e-16       
                                          
            Sensitivity : 0.6132          
            Specificity : 0.9972          
         Pos Pred Value : 0.9924          
         Neg Pred Value : 0.8128          
             Prevalence : 0.3726          
         Detection Rate : 0.2285          
   Detection Prevalence : 0.2302          
      Balanced Accuracy : 0.8052          
                                          
       'Positive' Class : 1
{% endhighlight %}

{% highlight javascript %}
confusionMatrix(factor(bcw_raw$kmeans_pca2), factor(bcw_raw$diagnosis_n))
{% endhighlight %}

{% highlight javascript %}
Confusion Matrix and Statistics

          Reference
Prediction   1   2
         1 176  18
         2  36 339
                                         
               Accuracy : 0.9051         
                 95% CI : (0.878, 0.9279)
    No Information Rate : 0.6274         
    P-Value [Acc > NIR] : <2e-16         
                                         
                  Kappa : 0.7934         
                                         
 Mcnemar's Test P-Value : 0.0207         
                                         
            Sensitivity : 0.8302         
            Specificity : 0.9496         
         Pos Pred Value : 0.9072         
         Neg Pred Value : 0.9040         
             Prevalence : 0.3726         
         Detection Rate : 0.3093         
   Detection Prevalence : 0.3409         
      Balanced Accuracy : 0.8899         
                                         
       'Positive' Class : 1
{% endhighlight %}

(1) 원본 데이터를 바탕으로 진행된 군집 분석의 정확도는 약 85.4%이고, (2) 주성분 데이터를 바탕으로 진행된 군집 분석의 정확도는 약 90.5%로, 주성분 데이터의 군집 분석 결과가 더 우수한 것을 확인할 수 있었다. 이러한 결과를 바탕으로 주성분 데이터를 바탕으로 k가 4인 K-Means 군집 분석을 진행하였고, 시각화 코드와 결과는 아래와 같다.

{% highlight javascript %}
set.seed(1234)
kmeans_model <- kmeans(bcw_pcor$x[, 1:6], 4, iter.max = 10, nstart = 1)
print(kmeans_model)
bcw_raw$kmeans_pca4 <- kmeans_model$cluster
{% endhighlight %}

{% highlight javascript %}
mycol1 <- c(rgb(47, 93, 140, 255, maxColorValue = 255),
            rgb(170, 19, 66, 255, maxColorValue = 255),
            rgb(221, 178, 71, 255, maxColorValue = 255),
            rgb(60, 140, 76, 255, maxColorValue = 255)) 

par(mar = c(5, 5, 5, 0))

plot.new()
plot.window(xlim = c(-20, 10), ylim = c(-15, 10))
abline(h = seq(-15, 10, 5), v = seq(-20, 10, 5), col = "grey", lty = 3)

points(bcw_raw[bcw_raw$kmeans_pca4 == 1, "pc1"], 
       bcw_raw[bcw_raw$kmeans_pca4 == 1, "pc2"], pch = 16, col = mycol1[1],
       cex = 0.75)
points(bcw_raw[bcw_raw$kmeans_pca4 == 2, "pc1"], 
       bcw_raw[bcw_raw$kmeans_pca4 == 2, "pc2"], pch = 16, col = mycol1[2],
       cex = 0.75)
points(bcw_raw[bcw_raw$kmeans_pca4 == 3, "pc1"], 
       bcw_raw[bcw_raw$kmeans_pca4 == 3, "pc2"], pch = 16, col = mycol1[3],
       cex = 0.75)
points(bcw_raw[bcw_raw$kmeans_pca4 == 4, "pc1"], 
       bcw_raw[bcw_raw$kmeans_pca4 == 4, "pc2"], pch = 16, col = mycol1[4],
       cex = 0.75)

axis(side = 1, at = seq(-20, 10, 5), cex.axis = 1)
axis(side = 2, at = seq(-15, 10, 5), las = 2, cex.axis = 1)

mtext("PC1", 1, line = 3, cex = 1)
mtext("PC2", 2, line = 3, cex = 1)

legend(-20, 10, pch = 16, col = mycol1, legend = c(1:4))
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Kmeans4_pca.png" title = "plot4" alt = "plot4" width = "1008" style = "display: block; margin: auto;" />

분류된 군집을 바탕으로 유방암 진단에 가장 큰 근거가 되는 종양의 크기(radius)를 boxplot을 통해 군집별로 비교하였으며, 코드와 그림은 아래와 같다.

{% highlight javascript %}
par(mar = c(5, 5, 0, 0))

boxplot(radius_mean ~ kmeans_pca4, data = bcw_raw, axes = F, col = mycol2)

axis(1); axis(2)

mtext("Group", 1, line = 3, cex = 1)
mtext("Radius-Mean", 2, line = 3, cex = 1)
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Boxplot_radius_mean.png" title = "plot5" alt = "plot5" width = "1008" style = "display: block; margin: auto;" />

{% highlight javascript %}
par(mar = c(5, 5, 0, 0))

boxplot(radius_se ~ kmeans_pca4, data = bcw_raw, axes = F, col = mycol2)

axis(1); axis(2)

mtext("Group", 1, line = 3, cex = 1)
mtext("Radius-Standard Error", 2, line = 3, cex = 1)
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Boxplot_radius_se.png" title = "plot6" alt = "plot6" width = "1008" style = "display: block; margin: auto;" />

{% highlight javascript %}
par(mar = c(5, 5, 0, 0))

boxplot(radius_worst ~ kmeans_pca4, data = bcw_raw, axes = F, col = mycol2)

axis(1); axis(2)

mtext("Group", 1, line = 3, cex = 1)
mtext("Largest Radius", 2, line = 3, cex = 1)
{% endhighlight %}

<img src = "/assets/Lecture/2020-11-09-Breast-Cancer/Boxplot_radius_worst.png" title = "plot7" alt = "plot7" width = "1008" style = "display: block; margin: auto;" />

시각화 결과를 통해 군집 1과 3이 양성에 가까운 군집이고, 군집 2와 4가 악성에 가까운 군집인 것을 파악할 수 있었다. 종양 크기의 평균과 상위 3개 값의 평균인 worst에 관해서는 유사한 결과를 보여주었고, 표준오차와 관련하여서 군집 1 ~ 3까지의 차이는 존재하지 않았지만, 악성 종양으로 파악되는 군집 4에서는 차이가 비교적 큰 것으로 나타났다. 본 분석에서는 주성분 분석과 K-Means 군집 분석을 활용하여 데이터에 나타나있는 양성과 악성의 범주보다 더 세부적인 군집으로 나눌 것을 고려하고자 하였다. 분석 결과, 원본 데이터보다는 주성분 분석을 진행한 데이터로 군집 분석을 진행하는 것이 정확도가 더 우수하였으며, 세부적인 군집으로 나뉜 결과를 살펴봤을 때, 각 군집별로 유의미한 차이가 나타나는 것을 확인할 수 있었다. 이러한 분석의 활용 및 구체화는 각 유방암 기수별 치료법 및 치료제 적용에 관한 도움이 될 수 있을 것으로 예상된다.
