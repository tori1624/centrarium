---
layout: post
title: "Development prediction model of monthly rent using multiple regression analysis - focused on seoul"
author: "Young Ho Lee"
date: "2017.11.18"
categories: Contest
---




{% highlight r %}
# Basic Packages
library(dplyr)
library(ggplot2)
library(readr)
library(stringr)
library(gridExtra)
library(openxlsx)
{% endhighlight %}

# 1. Data Import

{% highlight r %}
seoul2016 <- read.xlsx("D:/Data/Public_data/real_transaction_price_2017/2016/github/seoul2016.xlsx")

split_gu <- function(x){
  strsplit(x, split = ',')[[1]][2]
}

split_dong <- function(x){
  strsplit(x, split = ',')[[1]][3]
}
{% endhighlight %}


{% highlight r %}
seoul2016$gu <- sapply(seoul2016$sigungu, split_gu)
seoul2016$dong <- sapply(seoul2016$sigungu, split_dong)

seoul2016$sale_day <- as.factor(seoul2016$sale_day)
seoul2016$buliding_type <- as.factor(seoul2016$buliding_type)
seoul2016$gu <- as.factor(seoul2016$gu)
seoul2016$dong <- as.factor(seoul2016$dong)

seoul2016 <- seoul2016 %>%
  select(-sigungu)

str(seoul2016)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	99510 obs. of  12 variables:
##  $ exclusive_area: num  72.8 60 39.5 60 39.5 ...
##  $ sale_month    : num  12 4 1 2 2 2 2 3 3 3 ...
##  $ sale_day      : Factor w/ 3 levels "1~10","11~20",..: 2 3 2 1 2 2 3 1 1 2 ...
##  $ deposit       : num  5000 10000 2000 35000 12000 17000 20000 30000 2000 10000 ...
##  $ monthly_rent  : num  120 80 90 50 60 70 50 35 90 55 ...
##  $ floors        : num  4 5 14 12 8 4 11 13 10 14 ...
##  $ yr_built      : num  2002 2001 1992 1992 1992 ...
##  $ x             : num  127 127 127 127 127 ...
##  $ y             : num  37.5 37.5 37.5 37.5 37.5 ...
##  $ buliding_type : Factor w/ 3 levels "아파트","연립/다세대",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ gu            : Factor w/ 25 levels "강남구","강동구",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ dong          : Factor w/ 366 levels "가락동","가리봉동",..: 10 10 10 10 10 10 10 10 10 10 ...
{% endhighlight %}



{% highlight r %}
summary(seoul2016)
{% endhighlight %}



{% highlight text %}
##  exclusive_area     sale_month      sale_day        deposit      
##  Min.   :  0.00   Min.   : 1.000   1~10 :30917   Min.   :     0  
##  1st Qu.: 29.41   1st Qu.: 3.000   11~20:33325   1st Qu.:  2000  
##  Median : 51.43   Median : 6.000   21~31:35268   Median :  5000  
##  Mean   : 55.51   Mean   : 6.286                 Mean   : 13161  
##  3rd Qu.: 83.06   3rd Qu.: 9.000                 3rd Qu.: 20000  
##  Max.   :280.01   Max.   :12.000                 Max.   :200000  
##                                                                  
##   monthly_rent        floors         yr_built          x        
##  Min.   :  0.00   Min.   :-2.00   Min.   :1961   Min.   :126.8  
##  1st Qu.: 35.00   1st Qu.: 3.00   1st Qu.:1996   1st Qu.:126.9  
##  Median : 50.00   Median : 5.00   Median :2004   Median :127.0  
##  Mean   : 61.79   Mean   : 7.07   Mean   :2003   Mean   :127.0  
##  3rd Qu.: 70.00   3rd Qu.:10.00   3rd Qu.:2012   3rd Qu.:127.1  
##  Max.   :900.00   Max.   :68.00   Max.   :2017   Max.   :127.2  
##                                                                 
##        y             buliding_type         gu             dong      
##  Min.   :37.43   아파트     :53040   송파구 :10475   상계동 : 2478  
##  1st Qu.:37.50   연립/다세대:29924   강남구 : 9621   잠실동 : 2334  
##  Median :37.53   오피스텔   :16546   강서구 : 7408   구로동 : 2247  
##  Mean   :37.54                       서초구 : 6366   화곡동 : 2123  
##  3rd Qu.:37.57                       마포구 : 5982   봉천동 : 1998  
##  Max.   :37.69                       노원구 : 5846   마곡동 : 1956  
##                                      (Other):53812   (Other):86374
{% endhighlight %}

# 2. Data Exploration
## 1) Visualization
### 1-1) Exclusive Area

{% highlight r %}
seoul2016 %>%  
  ggplot(aes(x = seoul2016$exclusive_area, y = log(seoul2016$monthly_rent), 
             color = seoul2016$exclusive_area)) +
  geom_point(shape = 21) + geom_line() +
  xlab("Exclusive Area") + ylab("log(Monthly Rent)") + 
  scale_color_gradient(low = "deepskyblue", high = "hotpink",
                       name = "Exclusive Area") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-4-1.png)

### 1-2) Sale Month

{% highlight r %}
seoul2016 %>%
  ggplot(aes(x = factor(seoul2016$sale_month), y = log(seoul2016$monthly_rent), 
             fill = factor(seoul2016$sale_month))) +
  geom_jitter(color = 'grey', alpha = .2) +
  geom_violin(alpha = .7) + xlab("Sale Month") + ylab("log(Monthly Rent)") +
  scale_fill_discrete(name = "Sale Month") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-5](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-5-1.png)

### 1-3) Sale Day

{% highlight r %}
seoul2016 %>%
  ggplot(aes(x = seoul2016$sale_day, y = log(seoul2016$monthly_rent), 
             fill = seoul2016$sale_day)) +
  geom_jitter(color = 'grey', alpha = .2) +
  geom_violin(alpha = .7) + xlab("Sale day") + ylab("log(Monthly Rent)") +
  scale_fill_discrete(name = "Sale Day") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-6](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-6-1.png)

### 1-4) Deposit

{% highlight r %}
seoul2016 %>%  
  ggplot(aes(x = log(seoul2016$deposit), y = log(seoul2016$monthly_rent), 
             color = log(seoul2016$deposit))) +
  geom_point(shape = 21) + geom_line() +
  xlab("Deposit") + ylab("log(Monthly Rent)") +
  scale_color_gradient(low = "deepskyblue", high = "hotpink",
                       name = "Deposit") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-7-1.png)

### 1-5) Floors

{% highlight r %}
seoul2016 %>%  
  ggplot(aes(x = factor(seoul2016$floors), y = log(seoul2016$monthly_rent), 
             fill = factor(seoul2016$floors))) +
  geom_boxplot() +
  xlab("Floors") + ylab("log(Monthly Rent)") +
  scale_fill_discrete(name = "Floors") + theme(legend.position = "none") +
  theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1)) +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-8-1.png)

### 1-6) Built year

{% highlight r %}
seoul2016 %>%
  mutate(built_year = cut(seoul2016$yr_built, seq(1960, 2020, by = 10),
                          labels = paste0(seq(1960, 2010, by = 10), "s"))) %>%
  ggplot(aes(x = factor(built_year), y = log(seoul2016$monthly_rent),
             fill = factor(built_year))) +
  geom_jitter(color = 'grey', alpha = .2) +
  geom_violin(alpha = .7) + xlab("Built Year") + ylab("log(Monthly Rent)") +
  scale_fill_discrete(name = "Built Year")  +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-9-1.png)

### 1-7) Building Type

{% highlight r %}
seoul2016 %>%
  ggplot(aes(x = seoul2016$buliding_type, y = log(seoul2016$monthly_rent), 
             fill = seoul2016$buliding_type)) +
  geom_jitter(color = 'grey', alpha = .2) +
  geom_violin(alpha = .7) + xlab("Building Type") + ylab("log(Monthly Rent)") +
  scale_fill_discrete(name = "Building Type") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-10-1.png)

### 1-8) Gu

{% highlight r %}
seoul2016 %>%
  ggplot(aes(x = seoul2016$gu, y = log(seoul2016$monthly_rent), 
             fill = seoul2016$gu)) +
  geom_jitter(color = 'grey', alpha = .2) +
  geom_violin(alpha = .7) + xlab("Gu") + ylab("log(Monthly Rent)") +
  theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1)) +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16)) + 
  theme(legend.position = "none")
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-11-1.png)

### 1-9) Monthly Rent

{% highlight r %}
library(rgdal)
library(RColorBrewer)
library(classInt)
{% endhighlight %}


{% highlight r %}
seoul.sp <- readOGR("D:/Study/spatial_data_R/data/seoul/Seoul_dong.shp")
{% endhighlight %}



{% highlight text %}
## OGR data source with driver: ESRI Shapefile 
## Source: "D:/Study/spatial_data_R/data/seoul/Seoul_dong.shp", layer: "Seoul_dong"
## with 423 features
## It has 3 fields
{% endhighlight %}



{% highlight r %}
seoul.wgs <- spTransform(seoul.sp, CRS("+proj=longlat +datum=WGS84 
                                       +no_defs +ellps=WGS84 +towgs84=0,0,0"))
{% endhighlight %}


{% highlight r %}
orrd <- brewer.pal(5, "OrRd")
montlyclass <- classIntervals(seoul2016$monthly_rent, n = 5)
{% endhighlight %}


{% highlight r %}
ggplot() + 
  geom_polygon(data = seoul.wgs, aes(x = long, y = lat, group = group), 
               fill = 'white', color = 'Grey 50') +
  geom_point(data = seoul2016, aes(x = x, y = y, alpha = .075,
                                      color = factor(findCols(montlyclass)))) +
  scale_color_brewer(palette = "OrRd", name = "Montly rent",
                     labels = c("Less than 30", "30 - 45", "46 - 57", 
                                "58 - 80", "81 - 900")) +
  guides(alpha = "none")
{% endhighlight %}

![plot of chunk unnamed-chunk-15](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-15-1.png)

# 3. Modeling

{% highlight r %}
seoul2016$monthly_rent <- ifelse(seoul2016$monthly_rent == 0, 1, seoul2016$monthly_rent)

set.seed(1234)
trainIndex <- sample(x = 1:nrow(seoul2016), size = 0.7 * nrow(seoul2016))
train <- seoul2016[trainIndex, ]
test <- seoul2016[-trainIndex, ]
{% endhighlight %}

## 1) Raw Model

{% highlight r %}
monthlyRent_model <- lm(monthly_rent ~ ., data = train[, -12])
summary(monthlyRent_model)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = monthly_rent ~ ., data = train[, -12])
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -268.90  -17.01    0.20   14.86  740.08 
## 
## Coefficients:
##                            Estimate Std. Error  t value Pr(>|t|)    
## (Intercept)              -2.686e+03  1.145e+03   -2.346   0.0190 *  
## exclusive_area            1.267e+00  6.159e-03  205.702   <2e-16 ***
## sale_month                4.278e-03  3.473e-02    0.123   0.9020    
## sale_day11~20            -1.294e-01  3.084e-01   -0.420   0.6747    
## sale_day21~31            -7.102e-01  3.034e-01   -2.341   0.0192 *  
## deposit                  -1.911e-03  1.066e-05 -179.282   <2e-16 ***
## floors                    1.218e+00  2.619e-02   46.496   <2e-16 ***
## yr_built                  1.418e+00  1.429e-02   99.260   <2e-16 ***
## x                        -6.298e+01  7.643e+00   -8.240   <2e-16 ***
## y                         2.108e+02  1.028e+01   20.496   <2e-16 ***
## buliding_type연립/다세대 -1.251e+01  3.803e-01  -32.880   <2e-16 ***
## buliding_type오피스텔    -5.787e+00  4.401e-01  -13.149   <2e-16 ***
## gu강동구                 -5.828e+01  1.158e+00  -50.305   <2e-16 ***
## gu강북구                 -9.475e+01  1.723e+00  -54.995   <2e-16 ***
## gu강서구                 -8.469e+01  1.722e+00  -49.188   <2e-16 ***
## gu관악구                 -5.443e+01  1.199e+00  -45.412   <2e-16 ***
## gu광진구                 -4.777e+01  1.031e+00  -46.349   <2e-16 ***
## gu구로구                 -6.858e+01  1.592e+00  -43.083   <2e-16 ***
## gu금천구                 -6.133e+01  1.774e+00  -34.565   <2e-16 ***
## gu노원구                 -8.673e+01  1.694e+00  -51.201   <2e-16 ***
## gu도봉구                 -9.441e+01  1.847e+00  -51.126   <2e-16 ***
## gu동대문구               -7.261e+01  1.213e+00  -59.845   <2e-16 ***
## gu동작구                 -5.368e+01  1.103e+00  -48.668   <2e-16 ***
## gu마포구                 -6.017e+01  1.226e+00  -49.090   <2e-16 ***
## gu서대문구               -7.395e+01  1.368e+00  -54.043   <2e-16 ***
## gu서초구                 -1.174e+01  7.207e-01  -16.283   <2e-16 ***
## gu성동구                 -4.967e+01  9.615e-01  -51.658   <2e-16 ***
## gu성북구                 -8.380e+01  1.327e+00  -63.141   <2e-16 ***
## gu송파구                 -3.432e+01  7.098e-01  -48.354   <2e-16 ***
## gu양천구                 -6.447e+01  1.616e+00  -39.887   <2e-16 ***
## gu영등포구               -6.141e+01  1.330e+00  -46.181   <2e-16 ***
## gu용산구                 -3.935e+01  1.128e+00  -34.890   <2e-16 ***
## gu은평구                 -9.155e+01  1.535e+00  -59.627   <2e-16 ***
## gu종로구                 -5.938e+01  1.455e+00  -40.828   <2e-16 ***
## gu중구                   -5.420e+01  1.277e+00  -42.446   <2e-16 ***
## gu중랑구                 -8.808e+01  1.637e+00  -53.794   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 32.58 on 69621 degrees of freedom
## Multiple R-squared:  0.5339,	Adjusted R-squared:  0.5336 
## F-statistic:  2278 on 35 and 69621 DF,  p-value: < 2.2e-16
{% endhighlight %}

## 2) Log model

{% highlight r %}
log_model <- lm(log(monthly_rent) ~ ., data = train[, -12])
summary(log_model)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = log(monthly_rent) ~ ., data = train[, -12])
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -4.3071 -0.2294  0.0975  0.3267  3.5756 
## 
## Coefficients:
##                            Estimate Std. Error  t value Pr(>|t|)    
## (Intercept)               3.547e+00  1.815e+01    0.195  0.84508    
## exclusive_area            1.511e-02  9.762e-05  154.812  < 2e-16 ***
## sale_month               -3.135e-04  5.504e-04   -0.569  0.56903    
## sale_day11~20            -5.212e-03  4.888e-03   -1.066  0.28630    
## sale_day21~31            -1.441e-02  4.809e-03   -2.997  0.00273 ** 
## deposit                  -2.542e-05  1.689e-07 -150.479  < 2e-16 ***
## floors                    1.563e-02  4.151e-04   37.664  < 2e-16 ***
## yr_built                  1.877e-02  2.265e-04   82.885  < 2e-16 ***
## x                        -8.396e-01  1.211e-01   -6.931 4.22e-12 ***
## y                         1.852e+00  1.630e-01   11.360  < 2e-16 ***
## buliding_type연립/다세대 -2.186e-01  6.029e-03  -36.263  < 2e-16 ***
## buliding_type오피스텔    -6.771e-02  6.976e-03   -9.706  < 2e-16 ***
## gu강동구                 -7.100e-01  1.836e-02  -38.669  < 2e-16 ***
## gu강북구                 -1.100e+00  2.731e-02  -40.275  < 2e-16 ***
## gu강서구                 -1.041e+00  2.729e-02  -38.142  < 2e-16 ***
## gu관악구                 -7.406e-01  1.900e-02  -38.984  < 2e-16 ***
## gu광진구                 -5.343e-01  1.634e-02  -32.709  < 2e-16 ***
## gu구로구                 -8.761e-01  2.523e-02  -34.727  < 2e-16 ***
## gu금천구                 -8.841e-01  2.812e-02  -31.437  < 2e-16 ***
## gu노원구                 -9.830e-01  2.685e-02  -36.611  < 2e-16 ***
## gu도봉구                 -1.054e+00  2.927e-02  -36.022  < 2e-16 ***
## gu동대문구               -8.208e-01  1.923e-02  -42.684  < 2e-16 ***
## gu동작구                 -6.479e-01  1.748e-02  -37.061  < 2e-16 ***
## gu마포구                 -6.790e-01  1.943e-02  -34.952  < 2e-16 ***
## gu서대문구               -8.485e-01  2.169e-02  -39.120  < 2e-16 ***
## gu서초구                 -1.427e-01  1.142e-02  -12.490  < 2e-16 ***
## gu성동구                 -5.489e-01  1.524e-02  -36.017  < 2e-16 ***
## gu성북구                 -9.474e-01  2.104e-02  -45.037  < 2e-16 ***
## gu송파구                 -3.862e-01  1.125e-02  -34.327  < 2e-16 ***
## gu양천구                 -7.885e-01  2.562e-02  -30.778  < 2e-16 ***
## gu영등포구               -7.374e-01  2.108e-02  -34.987  < 2e-16 ***
## gu용산구                 -4.643e-01  1.788e-02  -25.969  < 2e-16 ***
## gu은평구                 -1.072e+00  2.434e-02  -44.030  < 2e-16 ***
## gu종로구                 -6.460e-01  2.306e-02  -28.018  < 2e-16 ***
## gu중구                   -5.916e-01  2.024e-02  -29.230  < 2e-16 ***
## gu중랑구                 -1.039e+00  2.595e-02  -40.043  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5165 on 69621 degrees of freedom
## Multiple R-squared:  0.4154,	Adjusted R-squared:  0.4151 
## F-statistic:  1414 on 35 and 69621 DF,  p-value: < 2.2e-16
{% endhighlight %}

# 4. Model Evaluation
## 1) Raw model

{% highlight r %}
rmse <- function(actual, predict){
  if(length(actual) != length(predict))
      stop("The length of two vectors are different")

  length <- length(actual)
  errorSum <- sum((actual - predict)^2)
  
  return(sqrt(errorSum / length))
}
{% endhighlight %}


{% highlight r %}
predict_price <- predict(monthlyRent_model, test)

plot(predict_price)
{% endhighlight %}

![plot of chunk unnamed-chunk-20](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-20-1.png)


{% highlight r %}
rmse(test$monthly_rent, predict_price)
{% endhighlight %}



{% highlight text %}
## [1] 32.13349
{% endhighlight %}

## 2) Log model

{% highlight r %}
rmsle <- function(pred, act) {
    if(length(pred) != length(act))
        stop("The length of two vectors are different")
    
    len <- length(pred)
    pred <- log(pred + 1)
    act <- log(act + 1)
    
    msle <- mean((pred - act)^2)
    
    return(sqrt(msle))
}
{% endhighlight %}


{% highlight r %}
log_pred <- predict(log_model, test)
log_pred <- exp(log_pred)

plot(log_pred)
{% endhighlight %}

![plot of chunk unnamed-chunk-23](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-23-1.png)


{% highlight r %}
rmsle(log_pred, test$monthly_rent)
{% endhighlight %}



{% highlight text %}
## [1] 0.4986278
{% endhighlight %}

# 5. Model Improvement
## 1) Data Import

{% highlight r %}
final <- read.xlsx("D:/Data/Public_data/real_transaction_price_2017/2016/github/seoul2016_final.xlsx")
{% endhighlight %}

## 2) Data Handling

{% highlight r %}
# delete unnecessary columns
final <- final[, -c(1, 13:18)]

# categorize distance columns
final <- final %>%
  dplyr::mutate(theater_c = ifelse(theater_dist < 300, 5, 
                            ifelse(theater_dist > 300 & theater_dist < 600, 4,
                            ifelse(theater_dist > 600 & theater_dist < 900, 3, 
                            ifelse(theater_dist > 900 & theater_dist < 1200, 2,
                            ifelse(theater_dist > 1200, 1, NA)))))) %>%
  dplyr::mutate(subway_c = ifelse(subway_dist < 300, 5, 
                           ifelse(subway_dist > 300 & subway_dist < 600, 4,
                           ifelse(subway_dist > 600 & subway_dist < 900, 3, 
                           ifelse(subway_dist > 900 & subway_dist < 1200, 2,
                           ifelse(subway_dist > 1200, 1, NA)))))) %>%
  dplyr::mutate(univ_c = ifelse(univ_dist < 300, 5, 
                         ifelse(univ_dist > 300 & univ_dist < 600, 4,
                         ifelse(univ_dist > 600 & univ_dist < 900, 3, 
                         ifelse(univ_dist > 900 & univ_dist < 1200, 2,
                         ifelse(univ_dist > 1200, 1, NA)))))) %>%
  dplyr::mutate(host_c = ifelse(host_dist < 300, 5, 
                         ifelse(host_dist > 300 & host_dist < 600, 4,
                         ifelse(host_dist > 600 & host_dist < 900, 3, 
                         ifelse(host_dist > 900 & host_dist < 1200, 2,
                         ifelse(host_dist > 1200, 1, NA)))))) %>%
  dplyr::mutate(police_c = ifelse(police_dist < 300, 5, 
                           ifelse(police_dist > 300 & police_dist < 600, 4,
                           ifelse(police_dist > 600 & police_dist < 900, 3, 
                           ifelse(police_dist > 900 & police_dist < 1200, 2,
                           ifelse(police_dist > 1200, 1, NA)))))) %>%
  select(-c(theater_dist, subway_dist, univ_dist, host_dist, police_dist))

# one-hot encoding
final$sale_day <- as.factor(final$sale_day)
final$gu <- as.factor(final$gu)
final$buliding_type <- as.factor(final$buliding_type)
{% endhighlight %}

## 3) Visualization
### 3-1) Theater

{% highlight r %}
final %>%
  ggplot(aes(x = factor(theater_c), y = log(monthly_rent), 
             color = factor(theater_c))) +
  geom_point(position = "jitter", alpha = 0.1) +
  xlab("Distance from Theater(Weight)") + scale_color_discrete(name = "Weight") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-27](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-27-1.png)

### 3-2) Subway Station

{% highlight r %}
final %>%
  ggplot(aes(x = factor(subway_c), y = log(monthly_rent), 
             color = factor(subway_c))) +
  geom_point(position = "jitter", alpha = 0.1) +
  xlab("Distance from Subway Station(Weight)") + scale_color_discrete(name = "Weight") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-28](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-28-1.png)

### 3-3) University

{% highlight r %}
final %>%
  ggplot(aes(x = factor(univ_c), y = log(monthly_rent), 
             color = factor(univ_c))) +
  geom_point(position = "jitter", alpha = 0.1) +
  xlab("Distance from University(Weight)") + scale_color_discrete(name = "Weight") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-29](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-29-1.png)

### 3-4) General Hospital

{% highlight r %}
final %>%
  ggplot(aes(x = factor(host_c), y = log(monthly_rent), 
             color = factor(host_c))) +
  geom_point(position = "jitter", alpha = 0.1) +
  xlab("Distance from General Hospital(Weight)") + scale_color_discrete(name = "Weight") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-30](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-30-1.png)

### 3-5) Police Office

{% highlight r %}
final %>%
  ggplot(aes(x = factor(police_c), y = log(monthly_rent), 
             color = factor(police_c))) +
  geom_point(position = "jitter", alpha = 0.1) +
  xlab("Distance from Police Office(Weight)") + scale_color_discrete(name = "Weight") +
  theme(axis.title.x = element_text(size = 16),
        axis.title.y = element_text(size = 16))
{% endhighlight %}

![plot of chunk unnamed-chunk-31](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-31-1.png)

## 4) Final model

{% highlight r %}
final$monthly_rent <- ifelse(final$monthly_rent == 0, 1, final$monthly_rent)

tr.final <- final[trainIndex, ]
te.final <- final[-trainIndex, ]
{% endhighlight %}


{% highlight r %}
final.model <- lm(log(monthly_rent) ~ ., data = tr.final)
summary(final.model)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = log(monthly_rent) ~ ., data = tr.final)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -4.2667 -0.2187  0.0927  0.3163  3.8447 
## 
## Coefficients:
##                            Estimate Std. Error  t value Pr(>|t|)    
## (Intercept)               3.040e+01  1.786e+01    1.702  0.08871 .  
## exclusive_area            1.571e-02  9.660e-05  162.616  < 2e-16 ***
## sale_month               -1.926e-04  5.400e-04   -0.357  0.72136    
## sale_day11~20            -5.188e-03  4.795e-03   -1.082  0.27936    
## sale_day21~31            -1.266e-02  4.718e-03   -2.683  0.00731 ** 
## deposit                  -2.672e-05  1.680e-07 -159.053  < 2e-16 ***
## floors                    1.391e-02  4.090e-04   33.998  < 2e-16 ***
## yr_built                  1.964e-02  2.229e-04   88.110  < 2e-16 ***
## x                        -1.039e+00  1.195e-01   -8.698  < 2e-16 ***
## y                         1.755e+00  1.636e-01   10.725  < 2e-16 ***
## buliding_type연립/다세대 -2.348e-01  5.927e-03  -39.616  < 2e-16 ***
## buliding_type오피스텔    -8.509e-02  6.928e-03  -12.283  < 2e-16 ***
## gu강동구                 -7.118e-01  1.806e-02  -39.411  < 2e-16 ***
## gu강북구                 -1.057e+00  2.809e-02  -37.626  < 2e-16 ***
## gu강서구                 -1.086e+00  2.727e-02  -39.830  < 2e-16 ***
## gu관악구                 -7.929e-01  1.884e-02  -42.095  < 2e-16 ***
## gu광진구                 -5.857e-01  1.673e-02  -35.006  < 2e-16 ***
## gu구로구                 -9.421e-01  2.498e-02  -37.716  < 2e-16 ***
## gu금천구                 -8.592e-01  2.768e-02  -31.040  < 2e-16 ***
## gu노원구                 -1.012e+00  2.725e-02  -37.151  < 2e-16 ***
## gu도봉구                 -1.020e+00  2.932e-02  -34.770  < 2e-16 ***
## gu동대문구               -8.444e-01  1.962e-02  -43.039  < 2e-16 ***
## gu동작구                 -7.147e-01  1.751e-02  -40.815  < 2e-16 ***
## gu마포구                 -7.047e-01  1.977e-02  -35.640  < 2e-16 ***
## gu서대문구               -8.776e-01  2.278e-02  -38.524  < 2e-16 ***
## gu서초구                 -1.358e-01  1.146e-02  -11.855  < 2e-16 ***
## gu성동구                 -5.923e-01  1.543e-02  -38.377  < 2e-16 ***
## gu성북구                 -9.369e-01  2.197e-02  -42.641  < 2e-16 ***
## gu송파구                 -3.591e-01  1.113e-02  -32.256  < 2e-16 ***
## gu양천구                 -7.806e-01  2.532e-02  -30.832  < 2e-16 ***
## gu영등포구               -8.248e-01  2.084e-02  -39.574  < 2e-16 ***
## gu용산구                 -4.712e-01  1.803e-02  -26.138  < 2e-16 ***
## gu은평구                 -1.102e+00  2.457e-02  -44.837  < 2e-16 ***
## gu종로구                 -7.286e-01  2.393e-02  -30.443  < 2e-16 ***
## gu중구                   -7.001e-01  2.039e-02  -34.343  < 2e-16 ***
## gu중랑구                 -1.070e+00  2.580e-02  -41.477  < 2e-16 ***
## theater_c                 1.061e-02  1.645e-03    6.450 1.13e-10 ***
## subway_c                  6.827e-02  2.021e-03   33.781  < 2e-16 ***
## univ_c                    9.408e-03  2.151e-03    4.373 1.23e-05 ***
## host_c                    4.031e-02  1.775e-03   22.708  < 2e-16 ***
## police_c                  2.073e-02  1.844e-03   11.240  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5067 on 69616 degrees of freedom
## Multiple R-squared:  0.4374,	Adjusted R-squared:  0.437 
## F-statistic:  1353 on 40 and 69616 DF,  p-value: < 2.2e-16
{% endhighlight %}


{% highlight r %}
final.pred <- predict(final.model, te.final)
final.pred <- exp(final.pred)
plot(final.pred)
{% endhighlight %}

![plot of chunk unnamed-chunk-34](/assets/contest/2017-11-18-Monthly-Rent/unnamed-chunk-34-1.png)


{% highlight r %}
rmsle(final.pred, te.final$monthly_rent)
{% endhighlight %}



{% highlight text %}
## [1] 0.4880586
{% endhighlight %}
