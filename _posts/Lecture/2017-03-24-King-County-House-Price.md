---
layout: post
title: "King County House Price"
author: "Young Ho Lee"
date: "2017.03.24"
categories: Lecture
cover: "/assets/Lecture/2017-03-24-King-County-House-Price/kingcounty.jpg"
---

이번 포스팅은 2017년 1월에 경희대학교 소셜네트워크과학과의 재윤님이 진행하셨던 R을 이용한 머신러닝 특강의 자료를 활용한 것이다. 당시에는 R을 배운지 얼마되지 않았었기 때문에, 강의 시간에 배운 코드들을 위주로 분석을 진행하였다. 자료는 2014년 5월부터 2015년 5월까지 매매된 King Conunty의 주택 매매가를 포함하고 있으며, 머신러닝의 방법 중 하나인 다중회귀분석을 활용하여 King Conunty의 주택 매매가를 예측하고자 하였다.


{% highlight javascript %}
# Bacis Packages
library(ggplot2)
library(readr)
library(dplyr)
library(gridExtra)
{% endhighlight %}

# 1. Data Import

{% highlight javascript %}
train <- read.csv("D:/Data/Machine_Learning/KC_House/train.csv")
test <- read.csv("D:/Data/Machine_Learning/KC_House/test.csv")

head(train)
{% endhighlight %}



{% highlight javascript %}
##     price bedrooms bathrooms sqft_living sqft_lot floors waterfront view
## 1  175003        3      1.50        1390     1882      2          0    0
## 2  705000        6      2.75        2830    10579      1          0    0
## 3  800000        3      1.75        1890    10292      1          0    0
## 4  300000        2      1.00        1290     2482      2          0    0
## 5  467000        3      2.00        1840     3432      2          0    0
## 6 1150000        3      2.50        2100    15120      1          0    0
##   condition grade sqft_above sqft_basement yr_built yr_renovated zipcode
## 1         3     7       1390             0     2014            0   98108
## 2         4     8       1430          1400     1967            0   98005
## 3         4     8       1890             0     1969            0   98040
## 4         3     7       1290             0     2008            0   98053
## 5         3     7       1840             0     2012            0   98155
## 6         4     8       2100             0     1953            0   98004
##       lat     long sqft_living15 sqft_lot15 sale_year sale_month
## 1 47.5667 -122.297          1490       2175      2014         12
## 2 47.6360 -122.171          2060      10745      2014         10
## 3 47.5350 -122.234          2630      10625      2014         12
## 4 47.6972 -122.025          1290       2482      2015          4
## 5 47.7368 -122.295          1280       7573      2015          3
## 6 47.6201 -122.222          3070      16078      2015          1
{% endhighlight %}



{% highlight javascript %}
str(train)
{% endhighlight %}



{% highlight javascript %}
## 'data.frame':	15129 obs. of  21 variables:
##  $ price        : num  175003 705000 800000 300000 467000 ...
##  $ bedrooms     : int  3 6 3 2 3 3 3 4 2 3 ...
##  $ bathrooms    : num  1.5 2.75 1.75 1 2 2.5 2 2.25 1.75 2 ...
##  $ sqft_living  : int  1390 2830 1890 1290 1840 2100 2070 1800 1370 2168 ...
##  $ sqft_lot     : int  1882 10579 10292 2482 3432 15120 9000 7200 4495 4000 ...
##  $ floors       : num  2 1 1 2 2 1 1 1 1 1.5 ...
##  $ waterfront   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ view         : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ condition    : int  3 4 4 3 3 4 4 3 4 3 ...
##  $ grade        : int  7 8 8 7 7 8 7 7 8 8 ...
##  $ sqft_above   : int  1390 1430 1890 1290 1840 2100 1450 1230 1370 2168 ...
##  $ sqft_basement: int  0 1400 0 0 0 0 620 570 0 0 ...
##  $ yr_built     : int  2014 1967 1969 2008 2012 1953 1969 1979 1975 1907 ...
##  $ yr_renovated : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ zipcode      : int  98108 98005 98040 98053 98155 98004 98023 98177 98198 98105 ...
##  $ lat          : num  47.6 47.6 47.5 47.7 47.7 ...
##  $ long         : num  -122 -122 -122 -122 -122 ...
##  $ sqft_living15: int  1490 2060 2630 1290 1280 3070 1630 2260 1370 1770 ...
##  $ sqft_lot15   : int  2175 10745 10625 2482 7573 16078 7885 7498 4686 4000 ...
##  $ sale_year    : num  2014 2014 2014 2015 2015 ...
##  $ sale_month   : num  12 10 12 4 3 1 12 3 5 9 ...
{% endhighlight %}



{% highlight javascript %}
summary(train)
{% endhighlight %}



{% highlight javascript %}
##      price            bedrooms        bathrooms      sqft_living   
##  Min.   :  80000   Min.   : 0.000   Min.   :0.000   Min.   :  370  
##  1st Qu.: 323800   1st Qu.: 3.000   1st Qu.:1.750   1st Qu.: 1430  
##  Median : 450000   Median : 3.000   Median :2.250   Median : 1920  
##  Mean   : 540778   Mean   : 3.371   Mean   :2.116   Mean   : 2082  
##  3rd Qu.: 648000   3rd Qu.: 4.000   3rd Qu.:2.500   3rd Qu.: 2550  
##  Max.   :7700000   Max.   :33.000   Max.   :8.000   Max.   :12050  
##     sqft_lot           floors        waterfront            view       
##  Min.   :    520   Min.   :1.000   Min.   :0.000000   Min.   :0.0000  
##  1st Qu.:   5085   1st Qu.:1.000   1st Qu.:0.000000   1st Qu.:0.0000  
##  Median :   7641   Median :1.500   Median :0.000000   Median :0.0000  
##  Mean   :  15438   Mean   :1.492   Mean   :0.007601   Mean   :0.2399  
##  3rd Qu.:  10800   3rd Qu.:2.000   3rd Qu.:0.000000   3rd Qu.:0.0000  
##  Max.   :1164794   Max.   :3.500   Max.   :1.000000   Max.   :4.0000  
##    condition         grade          sqft_above   sqft_basement   
##  Min.   :1.000   Min.   : 3.000   Min.   : 370   Min.   :   0.0  
##  1st Qu.:3.000   1st Qu.: 7.000   1st Qu.:1200   1st Qu.:   0.0  
##  Median :3.000   Median : 7.000   Median :1570   Median :   0.0  
##  Mean   :3.408   Mean   : 7.661   Mean   :1792   Mean   : 290.5  
##  3rd Qu.:4.000   3rd Qu.: 8.000   3rd Qu.:2210   3rd Qu.: 560.0  
##  Max.   :5.000   Max.   :13.000   Max.   :8860   Max.   :4820.0  
##     yr_built     yr_renovated        zipcode           lat       
##  Min.   :1900   Min.   :   0.00   Min.   :98001   Min.   :47.16  
##  1st Qu.:1951   1st Qu.:   0.00   1st Qu.:98033   1st Qu.:47.47  
##  Median :1975   Median :   0.00   Median :98065   Median :47.57  
##  Mean   :1971   Mean   :  85.51   Mean   :98078   Mean   :47.56  
##  3rd Qu.:1996   3rd Qu.:   0.00   3rd Qu.:98118   3rd Qu.:47.68  
##  Max.   :2015   Max.   :2015.00   Max.   :98199   Max.   :47.78  
##       long        sqft_living15    sqft_lot15       sale_year   
##  Min.   :-122.5   Min.   : 460   Min.   :   659   Min.   :2014  
##  1st Qu.:-122.3   1st Qu.:1490   1st Qu.:  5100   1st Qu.:2014  
##  Median :-122.2   Median :1840   Median :  7649   Median :2014  
##  Mean   :-122.2   Mean   :1988   Mean   : 12986   Mean   :2014  
##  3rd Qu.:-122.1   3rd Qu.:2370   3rd Qu.: 10125   3rd Qu.:2015  
##  Max.   :-121.3   Max.   :6210   Max.   :858132   Max.   :2015  
##    sale_month    
##  Min.   : 1.000  
##  1st Qu.: 4.000  
##  Median : 7.000  
##  Mean   : 6.607  
##  3rd Qu.: 9.000  
##  Max.   :12.000
{% endhighlight %}

# 2. Data Exploration
## 2-1) Colum Explanation

{% highlight javascript %}
names(train)
{% endhighlight %}



{% highlight javascript %}
##  [1] "price"         "bedrooms"      "bathrooms"     "sqft_living"  
##  [5] "sqft_lot"      "floors"        "waterfront"    "view"         
##  [9] "condition"     "grade"         "sqft_above"    "sqft_basement"
## [13] "yr_built"      "yr_renovated"  "zipcode"       "lat"          
## [17] "long"          "sqft_living15" "sqft_lot15"    "sale_year"    
## [21] "sale_month"
{% endhighlight %}



{% highlight javascript %}
unique(train$bedrooms)
{% endhighlight %}



{% highlight javascript %}
##  [1]  3  6  2  4  5  8  1  0  7 33  9 10
{% endhighlight %}



{% highlight javascript %}
unique(train$bathrooms)
{% endhighlight %}



{% highlight javascript %}
##  [1] 1.50 2.75 1.75 1.00 2.00 2.50 2.25 3.00 3.50 3.75 4.50 3.25 4.00 4.25
## [15] 5.00 4.75 1.25 0.75 6.00 5.50 5.75 8.00 7.50 0.50 0.00 5.25 6.50 7.75
## [29] 6.75
{% endhighlight %}



{% highlight javascript %}
unique(train$floors)
{% endhighlight %}



{% highlight javascript %}
## [1] 2.0 1.0 1.5 3.0 2.5 3.5
{% endhighlight %}



{% highlight javascript %}
unique(train$waterfront)
{% endhighlight %}



{% highlight javascript %}
## [1] 0 1
{% endhighlight %}



{% highlight javascript %}
unique(train$view)
{% endhighlight %}



{% highlight javascript %}
## [1] 0 1 2 3 4
{% endhighlight %}



{% highlight javascript %}
unique(train$condition)
{% endhighlight %}



{% highlight javascript %}
## [1] 3 4 5 2 1
{% endhighlight %}



{% highlight javascript %}
unique(train$grade)
{% endhighlight %}



{% highlight javascript %}
##  [1]  7  8  9  6 11 10  5  4 12 13  3
{% endhighlight %}



{% highlight javascript %}
unique(train$yr_built)
{% endhighlight %}



{% highlight javascript %}
##   [1] 2014 1967 1969 2008 2012 1953 1979 1975 1907 1970 1922 1973 1964 2004
##  [15] 1999 1956 2010 2006 1960 1986 1974 2001 1978 1963 1959 1954 1965 1905
##  [29] 1991 1968 1949 1958 2005 1913 1966 1994 1972 1900 1943 1977 1997 1928
##  [43] 2003 2007 1924 1961 1992 1988 1940 1962 1987 1976 1927 1990 2013 1926
##  [57] 1945 1908 1921 1995 1981 1984 1989 1944 1931 2009 1983 1910 1996 1971
##  [71] 1948 2000 1942 1919 1903 1902 1951 1980 1911 1998 1950 1955 1985 1946
##  [85] 1952 1915 1909 1957 2002 2011 1918 1930 1932 1938 1947 1925 1916 1941
##  [99] 1982 1901 1912 1993 1939 1920 1937 1917 1936 1904 1923 1929 1914 1935
## [113] 1933 1906 2015 1934
{% endhighlight %}



{% highlight javascript %}
unique(train$yr_renovated)
{% endhighlight %}



{% highlight javascript %}
##  [1]    0 1997 2000 1971 2005 2008 2003 1998 2011 2013 1983 1984 1995 2002
## [15] 1940 1989 2001 1991 2006 1993 1992 2014 2009 1999 1985 1979 2007 1994
## [29] 1996 1986 1978 1988 1968 1981 1953 1987 2004 1990 1980 1965 1964 1982
## [43] 1976 1956 2010 1962 1977 2015 2012 1955 1945 1963 1970 1974 1975 1957
## [57] 1969 1973 1959 1972 1967 1958 1946 1948 1960 1934 1954
{% endhighlight %}



{% highlight javascript %}
"unique(train$sqft_basement)"
{% endhighlight %}



{% highlight javascript %}
## [1] "unique(train$sqft_basement)"
{% endhighlight %}



{% highlight javascript %}
"unique(train$sqft_above)"
{% endhighlight %}



{% highlight javascript %}
## [1] "unique(train$sqft_above)"
{% endhighlight %}



{% highlight javascript %}
"unique(train$sqft_lot)"
{% endhighlight %}



{% highlight javascript %}
## [1] "unique(train$sqft_lot)"
{% endhighlight %}



{% highlight javascript %}
"unique(train$sqft_living)"
{% endhighlight %}



{% highlight javascript %}
## [1] "unique(train$sqft_living)"
{% endhighlight %}

## 2-2) Visualization
### (1) bedrooms

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(bedrooms), y = log(price), fill = factor(bedrooms))) + 
  geom_boxplot() + 
  xlab("bedrooms") + 
  scale_fill_discrete(name = "bedrooms")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-7-1.png" title = "plot1" alt = "plot1" width = "1008" height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(bedrooms) %>%
  summarise(avg_price = mean(price)) %>%
  ggplot(aes(x = factor(bedrooms), y = avg_price, fill = factor(bedrooms))) + 
  geom_col() +
  xlab("bedrooms") + 
  scale_fill_discrete(name = "bedrooms")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-7-2.png" title = "plot2" alt = "plot2" width = "1008" height = "500" style = "display: block; margin: auto;" />

### (2) bathrooms

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(bathrooms), y = log(price), fill = factor(bathrooms))) + 
  geom_boxplot() + 
  xlab("bathrooms") + 
  scale_fill_discrete(name = "bathrooms") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, hjust = 1))
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-8-1.png" title = "plot3" alt = "plot3" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(bathrooms) %>%
  summarise(avg_price = mean(log(price))) %>%
  ggplot(aes(x = factor(bathrooms), y = avg_price, fill = factor(bathrooms))) +
  geom_col() +
  xlab("bathrooms") +
  scale_fill_discrete(name = "bathrooms") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, hjust = 1))
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-8-2.png" title = "plot4" alt = "plot4" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (3) floors

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(floors), y = log(price), fill = factor(floors))) +
  geom_boxplot() +
  xlab("floors") +
  scale_fill_discrete(name = "floors")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-9-1.png" title = "plot5" alt = "plot5" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(floors) %>%
  summarise(avg_price = mean(price)) %>%
  ggplot(aes(x = factor(floors), y = avg_price, fill = factor(floors))) +
  geom_col() +
  xlab("floors") +
  scale_fill_discrete(name = "floors")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-9-2.png" title = "plot6" alt = "plot6" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (4) Water Front & View

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(waterfront), y = log(price), fill = factor(waterfront))) +
  geom_boxplot() + 
  xlab("waterfront") +
  scale_fill_discrete(name = "waterfront")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-10-1.png" title = "plot7" alt = "plot7" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(view), y = log(price), fill = factor(view))) +
  geom_boxplot() + 
  xlab("view") +
  scale_fill_discrete(name = "view")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-10-2.png" title = "plot8" alt = "plot8" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(waterfront, view) %>%
  summarise(avg_price = mean(price)) %>%
  ggplot(aes(x = factor(view), y = avg_price, fill = factor(waterfront))) +
  geom_col(position = "dodge") +
  xlab("view") +
  scale_fill_discrete("waterfront")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-10-3.png" title = "plot9" alt = "plot9" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (5) Condition & Grade

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(condition), y = log(price), fill = factor(condition))) +
  geom_boxplot() +
  xlab("condition") +
  scale_fill_discrete("condition")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-11-1.png" title = "plot10" alt = "plot10" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(grade), y = log(price), fill = factor(grade))) +
  geom_boxplot() +
  xlab("grade") +
  scale_fill_discrete("grade")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-11-2.png" title = "plot11" alt = "plot11" width = "1008"  height = "500" style = "display: block; margin: auto;" />
{% highlight javascript %}
train %>%
  group_by(condition, grade) %>%
  summarise(avg_price = mean(log(price))) %>%
  ggplot(aes(x = factor(condition), y = avg_price, fill = factor(grade))) +
  geom_col(position = "dodge") +
  xlab("condition") +
  scale_fill_discrete(name = "grade")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-11-3.png" title = "plot12" alt = "plot12" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (6) Built Year

{% highlight javascript %}
train %>%
  mutate(yr_built = cut(yr_built, breaks = seq(1899, 2020, by = 10),
                        labels = paste0(seq(1900, 2010, by = 10), "s"))) %>%
  ggplot(aes(x = factor(yr_built), y = log(price), fill = factor(yr_built))) +
  geom_boxplot() +
  xlab("yr_built") +
  scale_fill_discrete(name = "yr_built")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-12-1.png" title = "plot13" alt = "plot13" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  mutate(yr_built = cut(yr_built, breaks = seq(1899, 2020, by = 10),
                        labels = paste0(seq(1900, 2010, by = 10), "s"))) %>%
  group_by(yr_built) %>%
  summarise(avg_price = mean(price)) %>%
  ggplot(aes(x = factor(yr_built), y = avg_price, fill = factor(yr_built))) +
  geom_col() +
  xlab("yr_built") +
  scale_fill_discrete(name = "yr_built")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-12-2.png" title = "plot14" alt = "plot14" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (7) Renovated Year

{% highlight javascript %}
train %>%
  filter(yr_renovated != 0) %>%
  mutate(yr_renovated = cut(yr_built, seq(1899, 2020, by = 10),
                            labels = paste0(seq(1900, 2010, by = 10), "s"))) %>%
  ggplot(aes(x = factor(yr_renovated), y = log(price), fill = factor(yr_renovated))) +
  geom_boxplot() +
  xlab("yr_renovated") +
  scale_fill_discrete(name = "yr_renovted")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-13-1.png" title = "plot15" alt = "plot15" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  filter(yr_renovated != 0) %>%
  mutate(yr_renovated = cut(yr_built, seq(1899, 2020, by = 10),
                            labels = paste0(seq(1900, 2010, by = 10), "s"))) %>%
  group_by(yr_renovated) %>%
  summarise(avg_price = mean(price)) %>%
  ggplot(aes(x = factor(yr_renovated), y = avg_price, fill = factor(yr_renovated))) +
  geom_col() +
  xlab("yr_renovated") +
  scale_fill_discrete(name = "yr_renovted")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-13-2.png" title = "plot16" alt = "plot16" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  mutate(renovated = ifelse(yr_renovated > 0, 1, 0)) %>%
  ggplot(aes(x = factor(renovated), y = log(price), fill = factor(renovated))) +
  geom_boxplot() +
  xlab("renovated") +
  scale_fill_discrete(name = "renovated")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-13-3.png" title = "plot17" alt = "plot17" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (8) Sale Year / Month

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(sale_year), y = log(price), fill = factor(sale_year))) +
  geom_boxplot() + 
  xlab("sale_year") +
  scale_fill_discrete(name = "sale_year")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-14-1.png" title = "plot18" alt = "plot18" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(sale_month), y = log(price), fill = factor(sale_month))) +
  geom_boxplot() + 
  xlab("sale_month") +
  scale_fill_discrete(name = "sale_month")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-14-2.png" title = "plot19" alt = "plot19" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(sale_year, sale_month) %>%
  summarise(avg_price = mean(price)) %>%
  ggplot(aes(x = factor(sale_month), y = avg_price, fill = factor(sale_year))) +
  geom_col(position = "dodge") +
  xlab("sale_month") + scale_fill_discrete(name = "sale_year")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-14-3.png" title = "plot20" alt = "plot20" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (9) Lat / Long

{% highlight javascript %}
train %>%
  ggplot(aes(x = lat, y = log(price), color = lat)) +
  geom_point(shape = 21) + 
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-15-1.png" title = "plot21" alt = "plot21" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  ggplot(aes(x = long, y = log(price), color = long)) +
  geom_point(shape = 21) + 
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-15-2.png" title = "plot22" alt = "plot22" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (10) Zipcode

{% highlight javascript %}
train %>%
  ggplot(aes(x = factor(zipcode), y = log(price), fill = factor(zipcode))) +
  geom_boxplot() +
  theme(legend.position="none") +
  xlab("zipcode")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-16-1.png" title = "plot23" alt = "plot23" width = "1008"  height = "500" style = "display: block; margin: auto;" />

### (11) Sqft

{% highlight javascript %}
train %>%
  ggplot(aes(x = sqft_living, y = log(price), color = sqft_living)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink") +
  theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1)) +
  ggtitle("sqft_living") +
  theme(plot.title = element_text(hjust = 0.5)) -> g1

train %>%
  ggplot(aes(x = sqft_lot, y = log(price), color = sqft_lot)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink") +
  ggtitle("sqft_lot") +
  theme(plot.title = element_text(hjust = 0.5))-> g2

train %>%
  ggplot(aes(x = sqft_living15, y = log(price), color = sqft_living15)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink") +
  ggtitle("sqft_living15") +
  theme(plot.title = element_text(hjust = 0.5))-> g3

train %>%
  ggplot(aes(x = sqft_lot15, y = log(price), color = sqft_lot15)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink") +
  ggtitle("sqft_lot15") +
  theme(plot.title = element_text(hjust = 0.5))-> g4

grid.arrange(g1, g2, g3, g4, nrow = 2)
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-17-1.png" title = "plot24" alt = "plot24" width = "1008"  height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  ggplot(aes(x = sqft_above, y = log(price), color = sqft_above)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink") +
  ggtitle("sqft_above") +
  theme(plot.title = element_text(hjust = 0.5))-> m1

train %>%
  filter(sqft_basement != 0) %>%
  ggplot(aes(x = sqft_basement, y = log(price), color = sqft_basement)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink") +
  ggtitle("sqft_basement") +
  theme(plot.title = element_text(hjust = 0.5))-> m2

grid.arrange(m1, m2)
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-24-King-County-House-Price/unnamed-chunk-17-2.png" title = "plot25" alt = "plot25" width = "1008"  height = "500" style = "display: block; margin: auto;" />

# 3. Feature Engineering

{% highlight javascript %}
FeatureEngineering <- function(data){
  data %>%
    # NA
    select(-sqft_basement) %>%
    
    # Renovated or not
    mutate(renovated = ifelse(yr_renovated > 0, 1, 0))
}

#Apply FeatureEngineering
train <- FeatureEngineering(train)
test <- FeatureEngineering(test)

train$renovated <- as.factor(train$renovated)
test$renovated <- as.factor(test$renovated)

train$zipcode <- as.factor(train$zipcode)
test$zipcode <- as.factor(test$zipcode)

train$waterfront <- as.factor(train$waterfront)
test$waterfront <- as.factor(test$waterfront)
{% endhighlight %}

# 4. Modeling

{% highlight javascript %}
house_model <- lm(log(price) ~ ., data = train)
summary(house_model)
{% endhighlight %}



{% highlight javascript %}
## 
## Call:
## lm(formula = log(price) ~ ., data = train)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.34238 -0.09607  0.00863  0.10476  0.99102 
## 
## Coefficients:
##                 Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   -1.905e+02  1.338e+01 -14.239  < 2e-16 ***
## bedrooms       3.572e-03  2.067e-03   1.728 0.083991 .  
## bathrooms      3.469e-02  3.612e-03   9.603  < 2e-16 ***
## sqft_living    1.287e-04  4.860e-06  26.473  < 2e-16 ***
## sqft_lot       5.691e-07  5.069e-08  11.226  < 2e-16 ***
## floors        -2.556e-02  4.308e-03  -5.934 3.03e-09 ***
## waterfront1    4.815e-01  1.926e-02  25.001  < 2e-16 ***
## view           5.917e-02  2.358e-03  25.092  < 2e-16 ***
## condition      6.427e-02  2.666e-03  24.109  < 2e-16 ***
## grade          8.923e-02  2.491e-03  35.813  < 2e-16 ***
## sqft_above     7.046e-05  4.974e-06  14.166  < 2e-16 ***
## yr_built      -2.188e-04  8.866e-05  -2.468 0.013584 *  
## yr_renovated   3.798e-03  4.916e-04   7.726 1.18e-14 ***
## zipcode98002  -1.201e-02  1.966e-02  -0.611 0.541162    
## zipcode98003  -1.360e-03  1.738e-02  -0.078 0.937654    
## zipcode98004   9.305e-01  3.198e-02  29.098  < 2e-16 ***
## zipcode98005   5.919e-01  3.404e-02  17.389  < 2e-16 ***
## zipcode98006   5.109e-01  2.810e-02  18.179  < 2e-16 ***
## zipcode98007   5.192e-01  3.497e-02  14.849  < 2e-16 ***
## zipcode98008   5.200e-01  3.376e-02  15.403  < 2e-16 ***
## zipcode98010   3.230e-01  2.945e-02  10.967  < 2e-16 ***
## zipcode98011   2.222e-01  4.373e-02   5.081 3.79e-07 ***
## zipcode98014   2.687e-01  4.865e-02   5.522 3.40e-08 ***
## zipcode98019   1.839e-01  4.751e-02   3.871 0.000109 ***
## zipcode98022   1.880e-01  2.609e-02   7.204 6.12e-13 ***
## zipcode98023  -6.566e-02  1.653e-02  -3.972 7.17e-05 ***
## zipcode98024   3.890e-01  4.291e-02   9.067  < 2e-16 ***
## zipcode98027   4.451e-01  2.886e-02  15.421  < 2e-16 ***
## zipcode98028   1.842e-01  4.235e-02   4.350 1.37e-05 ***
## zipcode98029   5.397e-01  3.307e-02  16.319  < 2e-16 ***
## zipcode98030   4.480e-02  1.989e-02   2.253 0.024302 *  
## zipcode98031   4.454e-02  2.001e-02   2.225 0.026068 *  
## zipcode98032  -6.957e-02  2.410e-02  -2.887 0.003897 ** 
## zipcode98033   5.910e-01  3.638e-02  16.246  < 2e-16 ***
## zipcode98034   3.253e-01  3.895e-02   8.351  < 2e-16 ***
## zipcode98038   2.100e-01  2.209e-02   9.507  < 2e-16 ***
## zipcode98039   1.055e+00  4.269e-02  24.722  < 2e-16 ***
## zipcode98040   7.204e-01  2.830e-02  25.458  < 2e-16 ***
## zipcode98042   7.373e-02  1.866e-02   3.951 7.81e-05 ***
## zipcode98045   4.046e-01  4.100e-02   9.869  < 2e-16 ***
## zipcode98052   4.737e-01  3.720e-02  12.734  < 2e-16 ***
## zipcode98053   4.568e-01  3.993e-02  11.442  < 2e-16 ***
## zipcode98055   8.450e-02  2.240e-02   3.773 0.000162 ***
## zipcode98056   2.144e-01  2.439e-02   8.789  < 2e-16 ***
## zipcode98058   1.129e-01  2.128e-02   5.307 1.13e-07 ***
## zipcode98059   2.633e-01  2.387e-02  11.028  < 2e-16 ***
## zipcode98065   3.886e-01  3.758e-02  10.341  < 2e-16 ***
## zipcode98070   1.899e-01  2.833e-02   6.704 2.11e-11 ***
## zipcode98072   2.789e-01  4.365e-02   6.389 1.72e-10 ***
## zipcode98074   4.396e-01  3.545e-02  12.401  < 2e-16 ***
## zipcode98075   4.569e-01  3.404e-02  13.424  < 2e-16 ***
## zipcode98077   2.530e-01  4.557e-02   5.551 2.89e-08 ***
## zipcode98092   6.811e-02  1.768e-02   3.853 0.000117 ***
## zipcode98102   7.369e-01  3.748e-02  19.665  < 2e-16 ***
## zipcode98103   5.755e-01  3.509e-02  16.400  < 2e-16 ***
## zipcode98105   7.168e-01  3.613e-02  19.839  < 2e-16 ***
## zipcode98106   1.629e-01  2.573e-02   6.332 2.49e-10 ***
## zipcode98107   5.902e-01  3.625e-02  16.282  < 2e-16 ***
## zipcode98108   1.909e-01  2.844e-02   6.714 1.97e-11 ***
## zipcode98109   7.884e-01  3.709e-02  21.259  < 2e-16 ***
## zipcode98112   8.534e-01  3.319e-02  25.712  < 2e-16 ***
## zipcode98115   5.855e-01  3.562e-02  16.435  < 2e-16 ***
## zipcode98116   5.641e-01  2.894e-02  19.496  < 2e-16 ***
## zipcode98117   5.562e-01  3.612e-02  15.399  < 2e-16 ***
## zipcode98118   3.041e-01  2.533e-02  12.004  < 2e-16 ***
## zipcode98119   7.548e-01  3.527e-02  21.402  < 2e-16 ***
## zipcode98122   6.110e-01  3.130e-02  19.520  < 2e-16 ***
## zipcode98125   3.317e-01  3.855e-02   8.603  < 2e-16 ***
## zipcode98126   3.708e-01  2.668e-02  13.898  < 2e-16 ***
## zipcode98133   1.895e-01  3.974e-02   4.768 1.88e-06 ***
## zipcode98136   5.041e-01  2.761e-02  18.261  < 2e-16 ***
## zipcode98144   4.795e-01  2.908e-02  16.488  < 2e-16 ***
## zipcode98146   1.270e-01  2.411e-02   5.267 1.41e-07 ***
## zipcode98148   9.579e-02  3.431e-02   2.792 0.005242 ** 
## zipcode98155   1.617e-01  4.139e-02   3.906 9.41e-05 ***
## zipcode98166   1.858e-01  2.250e-02   8.256  < 2e-16 ***
## zipcode98168  -4.322e-02  2.349e-02  -1.840 0.065802 .  
## zipcode98177   3.215e-01  4.138e-02   7.769 8.43e-15 ***
## zipcode98178   3.753e-02  2.435e-02   1.541 0.123241    
## zipcode98188   2.514e-02  2.515e-02   1.000 0.317466    
## zipcode98198  -1.825e-02  1.899e-02  -0.961 0.336592    
## zipcode98199   6.275e-01  3.443e-02  18.222  < 2e-16 ***
## lat            5.616e-01  8.668e-02   6.479 9.51e-11 ***
## long          -3.170e-01  6.359e-02  -4.986 6.24e-07 ***
## sqft_living15  9.018e-05  3.947e-06  22.850  < 2e-16 ***
## sqft_lot15     1.877e-07  8.019e-08   2.341 0.019233 *  
## sale_year      6.781e-02  5.175e-03  13.103  < 2e-16 ***
## sale_month     3.010e-03  7.728e-04   3.895 9.88e-05 ***
## renovated1    -7.486e+00  9.815e-01  -7.628 2.53e-14 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1839 on 15040 degrees of freedom
## Multiple R-squared:  0.8789,	Adjusted R-squared:  0.8782 
## F-statistic:  1240 on 88 and 15040 DF,  p-value: < 2.2e-16
{% endhighlight %}

# 5. Model Evaluation

{% highlight javascript %}
rmsle <- function(pred, act) {
    if(length(pred) != length(act))
        stop("The length of two vectors are different")
    
    len <- length(pred)
    pred <- log(pred + 1)
    act <- log(act + 1)
    
    msle <- mean((pred - act)^2)
    
    return(sqrt(msle))
}

predict_price <- predict(house_model, test)
predict_price <- exp(predict_price)
predict_price <- ifelse(predict_price < 0, 0, predict_price)

rmsle(predict_price, test$price)
{% endhighlight %}



{% highlight javascript %}
## [1] 0.1800942
{% endhighlight %}
