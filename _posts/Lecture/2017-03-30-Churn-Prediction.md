---
layout: post
title: "2017-03-30-Churn_Prediction"
author: "Young Ho Lee"
date: "2017.03.30"
categories: Lecture
cover: "/assets/Lecture/2017-03-30-Churn-Prediction/churn.jpg"
---




{% highlight javascript %}
# Bacis Packages
library(ggplot2)
library(readr)
library(dplyr)
library(xgboost)
library(gridExtra)
{% endhighlight %}

# 1. Data Import

{% highlight javascript %}
train <- read.csv("D:/Data/Machine_Learning/Churn/train.csv")
test <- read.csv("D:/Data/Machine_Learning/Churn/test.csv")

head(train)
{% endhighlight %}



{% highlight javascript %}
##   state account_length area_code international_plan voice_mail_plan
## 1    KS            128       415                 no             yes
## 2    OH            107       415                 no             yes
## 3    NJ            137       415                 no              no
## 4    OH             84       408                yes              no
## 5    OK             75       415                yes              no
## 6    AL            118       510                yes              no
##   number_vmail_messages total_day_minutes total_day_calls total_day_charge
## 1                    25             265.1             110            45.07
## 2                    26             161.6             123            27.47
## 3                     0             243.4             114            41.38
## 4                     0             299.4              71            50.90
## 5                     0             166.7             113            28.34
## 6                     0             223.4              98            37.98
##   total_eve_minutes total_eve_calls total_eve_charge total_night_minutes
## 1             197.4              99            16.78               244.7
## 2             195.5             103            16.62               254.4
## 3             121.2             110            10.30               162.6
## 4              61.9              88             5.26               196.9
## 5             148.3             122            12.61               186.9
## 6             220.6             101            18.75               203.9
##   total_night_calls total_night_charge total_intl_minutes total_intl_calls
## 1                91              11.01               10.0                3
## 2               103              11.45               13.7                3
## 3               104               7.32               12.2                5
## 4                89               8.86                6.6                7
## 5               121               8.41               10.1                3
## 6               118               9.18                6.3                6
##   total_intl_charge number_customer_service_calls churn
## 1              2.70                             1     0
## 2              3.70                             1     0
## 3              3.29                             0     0
## 4              1.78                             2     0
## 5              2.73                             3     0
## 6              1.70                             0     0
{% endhighlight %}



{% highlight javascript %}
str(train)
{% endhighlight %}



{% highlight javascript %}
## 'data.frame':	3333 obs. of  20 variables:
##  $ state                        : Factor w/ 51 levels "AK","AL","AR",..: 17 36 32 36 37 2 20 25 19 50 ...
##  $ account_length               : int  128 107 137 84 75 118 121 147 117 141 ...
##  $ area_code                    : int  415 415 415 408 415 510 510 415 408 415 ...
##  $ international_plan           : Factor w/ 2 levels "no","yes": 1 1 1 2 2 2 1 2 1 2 ...
##  $ voice_mail_plan              : Factor w/ 2 levels "no","yes": 2 2 1 1 1 1 2 1 1 2 ...
##  $ number_vmail_messages        : int  25 26 0 0 0 0 24 0 0 37 ...
##  $ total_day_minutes            : num  265 162 243 299 167 ...
##  $ total_day_calls              : int  110 123 114 71 113 98 88 79 97 84 ...
##  $ total_day_charge             : num  45.1 27.5 41.4 50.9 28.3 ...
##  $ total_eve_minutes            : num  197.4 195.5 121.2 61.9 148.3 ...
##  $ total_eve_calls              : int  99 103 110 88 122 101 108 94 80 111 ...
##  $ total_eve_charge             : num  16.78 16.62 10.3 5.26 12.61 ...
##  $ total_night_minutes          : num  245 254 163 197 187 ...
##  $ total_night_calls            : int  91 103 104 89 121 118 118 96 90 97 ...
##  $ total_night_charge           : num  11.01 11.45 7.32 8.86 8.41 ...
##  $ total_intl_minutes           : num  10 13.7 12.2 6.6 10.1 6.3 7.5 7.1 8.7 11.2 ...
##  $ total_intl_calls             : int  3 3 5 7 3 6 7 6 4 5 ...
##  $ total_intl_charge            : num  2.7 3.7 3.29 1.78 2.73 1.7 2.03 1.92 2.35 3.02 ...
##  $ number_customer_service_calls: int  1 1 0 2 3 0 3 0 1 0 ...
##  $ churn                        : int  0 0 0 0 0 0 0 0 0 0 ...
{% endhighlight %}



{% highlight javascript %}
summary(train)
{% endhighlight %}



{% highlight javascript %}
##      state      account_length    area_code     international_plan
##  WV     : 106   Min.   :  1.0   Min.   :408.0   no :3010          
##  MN     :  84   1st Qu.: 74.0   1st Qu.:408.0   yes: 323          
##  NY     :  83   Median :101.0   Median :415.0                     
##  AL     :  80   Mean   :101.1   Mean   :437.2                     
##  OH     :  78   3rd Qu.:127.0   3rd Qu.:510.0                     
##  OR     :  78   Max.   :243.0   Max.   :510.0                     
##  (Other):2824                                                     
##  voice_mail_plan number_vmail_messages total_day_minutes total_day_calls
##  no :2411        Min.   : 0.000        Min.   :  0.0     Min.   :  0.0  
##  yes: 922        1st Qu.: 0.000        1st Qu.:143.7     1st Qu.: 87.0  
##                  Median : 0.000        Median :179.4     Median :101.0  
##                  Mean   : 8.099        Mean   :179.8     Mean   :100.4  
##                  3rd Qu.:20.000        3rd Qu.:216.4     3rd Qu.:114.0  
##                  Max.   :51.000        Max.   :350.8     Max.   :165.0  
##                                                                         
##  total_day_charge total_eve_minutes total_eve_calls total_eve_charge
##  Min.   : 0.00    Min.   :  0.0     Min.   :  0.0   Min.   : 0.00   
##  1st Qu.:24.43    1st Qu.:166.6     1st Qu.: 87.0   1st Qu.:14.16   
##  Median :30.50    Median :201.4     Median :100.0   Median :17.12   
##  Mean   :30.56    Mean   :201.0     Mean   :100.1   Mean   :17.08   
##  3rd Qu.:36.79    3rd Qu.:235.3     3rd Qu.:114.0   3rd Qu.:20.00   
##  Max.   :59.64    Max.   :363.7     Max.   :170.0   Max.   :30.91   
##                                                                     
##  total_night_minutes total_night_calls total_night_charge
##  Min.   : 23.2       Min.   : 33.0     Min.   : 1.040    
##  1st Qu.:167.0       1st Qu.: 87.0     1st Qu.: 7.520    
##  Median :201.2       Median :100.0     Median : 9.050    
##  Mean   :200.9       Mean   :100.1     Mean   : 9.039    
##  3rd Qu.:235.3       3rd Qu.:113.0     3rd Qu.:10.590    
##  Max.   :395.0       Max.   :175.0     Max.   :17.770    
##                                                          
##  total_intl_minutes total_intl_calls total_intl_charge
##  Min.   : 0.00      Min.   : 0.000   Min.   :0.000    
##  1st Qu.: 8.50      1st Qu.: 3.000   1st Qu.:2.300    
##  Median :10.30      Median : 4.000   Median :2.780    
##  Mean   :10.24      Mean   : 4.479   Mean   :2.765    
##  3rd Qu.:12.10      3rd Qu.: 6.000   3rd Qu.:3.270    
##  Max.   :20.00      Max.   :20.000   Max.   :5.400    
##                                                       
##  number_customer_service_calls     churn       
##  Min.   :0.000                 Min.   :0.0000  
##  1st Qu.:1.000                 1st Qu.:0.0000  
##  Median :1.000                 Median :0.0000  
##  Mean   :1.563                 Mean   :0.1449  
##  3rd Qu.:2.000                 3rd Qu.:0.0000  
##  Max.   :9.000                 Max.   :1.0000  
## 
{% endhighlight %}



{% highlight javascript %}
train$churn <- as.factor(train$churn)
{% endhighlight %}

# 2. Data Exploration
## 2-1) Colum Explanation

{% highlight javascript %}
names(train)
{% endhighlight %}



{% highlight javascript %}
##  [1] "state"                         "account_length"               
##  [3] "area_code"                     "international_plan"           
##  [5] "voice_mail_plan"               "number_vmail_messages"        
##  [7] "total_day_minutes"             "total_day_calls"              
##  [9] "total_day_charge"              "total_eve_minutes"            
## [11] "total_eve_calls"               "total_eve_charge"             
## [13] "total_night_minutes"           "total_night_calls"            
## [15] "total_night_charge"            "total_intl_minutes"           
## [17] "total_intl_calls"              "total_intl_charge"            
## [19] "number_customer_service_calls" "churn"
{% endhighlight %}



{% highlight javascript %}
unique(train$state)
{% endhighlight %}



{% highlight javascript %}
##  [1] KS OH NJ OK AL MA MO LA WV IN RI IA MT NY ID VT VA TX FL CO AZ SC NE
## [24] WY HI IL NH GA AK MD AR WI OR MI DE UT CA MN SD NC WA NM NV DC KY ME
## [47] MS TN PA CT ND
## 51 Levels: AK AL AR AZ CA CO CT DC DE FL GA HI IA ID IL IN KS KY LA ... WY
{% endhighlight %}



{% highlight javascript %}
unique(train$account_length)
{% endhighlight %}



{% highlight javascript %}
##   [1] 128 107 137  84  75 118 121 147 117 141  65  74 168  95  62 161  85
##  [18]  93  76  73  77 130 111 132 174  57  54  20  49 142 172  12  72  36
##  [35]  78 136 149  98 135  34 160  64  59 119  97  52  60  10  96  87  81
##  [52]  68 125 116  38  40  43 113 126 150 138 162  90  50  82 144  46  70
##  [69]  55 106  94 155  80 104  99 120 108 122 157 103  63 112  41 193  61
##  [86]  92 131 163  91 127 110 140  83 145  56 151 139   6 115 146 185 148
## [103]  32  25 179  67  19 170 164  51 208  53 105  66  86  35  88 123  45
## [120] 100 215  22  33 114  24 101 143  48  71 167  89 199 166 158 196 209
## [137]  16  39 173 129  44  79  31 124  37 159 194 154  21 133 224  58  11
## [154] 109 102 165  18  30 176  47 190 152  26  69 186 171  28 153 169  13
## [171]  27   3  42 189 156 134 243  23   1 205 200   5   9 178 181 182 217
## [188] 177 210  29 180   2  17   7 212 232 192 195 197 225 184 191 201  15
## [205] 183 202   8 175   4 188 204 221
{% endhighlight %}



{% highlight javascript %}
unique(train$area_code)
{% endhighlight %}



{% highlight javascript %}
## [1] 415 408 510
{% endhighlight %}



{% highlight javascript %}
unique(train$international_plan)
{% endhighlight %}



{% highlight javascript %}
## [1] no  yes
## Levels: no yes
{% endhighlight %}



{% highlight javascript %}
unique(train$voice_mail_plan)
{% endhighlight %}



{% highlight javascript %}
## [1] yes no 
## Levels: no yes
{% endhighlight %}



{% highlight javascript %}
unique(train$number_vmail_messages)
{% endhighlight %}



{% highlight javascript %}
##  [1] 25 26  0 24 37 27 33 39 30 41 28 34 46 29 35 21 32 42 36 22 23 43 31
## [24] 38 40 48 18 17 45 16 20 14 19 51 15 11 12 47  8 44 49  4 10 13 50  9
{% endhighlight %}



{% highlight javascript %}
unique(train$number_customer_service_calls)
{% endhighlight %}



{% highlight javascript %}
##  [1] 1 0 2 3 4 5 7 9 6 8
{% endhighlight %}

## 2-2) Visulization
### 1) State

{% highlight javascript %}
train %>%
  group_by(state, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = reorder(state, rate), y = rate, fill = rate)) +
  geom_col() +
  scale_fill_gradient(low="beige", high="coral1") +
  theme(axis.text.x = element_text(angle = 90, face = "italic", vjust = 1, hjust = 1)) +
  xlab("state")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-4-1.png" title = "plot1" alt = "plot1" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 2) Account Length

{% highlight javascript %}
train %>%
  ggplot(aes(x = churn, y = account_length, fill = churn)) +
  geom_boxplot()
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-5-1.png" title = "plot2" alt = "plot2" width = "1008" height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(account_length, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = account_length, y = rate, color = account_length)) +
  geom_point(shape = 21) +
  geom_line() +
  scale_color_gradient(low = "deepskyblue", high = "hotpink")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-5-2.png" title = "plot3" alt = "plot3" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 3) Area Code

{% highlight javascript %}
train %>%
  group_by(area_code, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count/sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = factor(area_code), y = rate, fill = rate)) +
  geom_col() +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") +
  xlab("Area Code")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-6-1.png" title = "plot4" alt = "plot4" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 4) International / Voice Mail Plan

{% highlight javascript %}
train %>%
  group_by(international_plan, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = international_plan, y = rate, fill = rate)) +
  geom_col() +
  theme(legend.position="none")+
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> g1

train %>%
  group_by(voice_mail_plan, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = voice_mail_plan, y = rate, fill = rate)) +
  geom_col() +
  theme(legend.position="none")+
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> g2

grid.arrange(g1, g2, ncol = 2)
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-7-1.png" title = "plot5" alt = "plot5" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 5) Number Vmail Message

{% highlight javascript %}
train %>%
  ggplot(aes(x = churn, y = number_vmail_messages, fill = churn)) +
  geom_boxplot()
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-8-1.png" title = "plot6" alt = "plot6" width = "1008" height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  mutate(number_vmail_messages_category = factor(round(number_vmail_messages, -1))) %>%
  group_by(number_vmail_messages_category, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = number_vmail_messages_category, y = rate, fill = rate)) +
  geom_col() +
  scale_fill_gradient(low = "deepskyblue", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-8-2.png" title = "plot7" alt = "plot7" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 6) Number Customer Service Calls

{% highlight javascript %}
train %>%
  ggplot(aes(x = churn, y = number_customer_service_calls, fill = churn)) +
  geom_boxplot()
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-9-1.png" title = "plot8" alt = "plot8" width = "1008" height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(number_customer_service_calls, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = number_customer_service_calls, y = rate, fill = rate)) +
  geom_col() +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-9-2.png" title = "plot9" alt = "plot9" width = "1008" height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  mutate(customer_service_calls = ifelse(number_customer_service_calls > 0, 1, 0)) %>%
  group_by(customer_service_calls, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = factor(customer_service_calls), y = rate, fill = rate)) +
  geom_col() + xlab("customer_service_calls") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-9-3.png" title = "plot10" alt = "plot10" width = "1008" height = "500" style = "display: block; margin: auto;" />
### 7) Total Calls

{% highlight javascript %}
train %>%
  mutate(total_domestic_calls = total_day_calls + total_eve_calls + total_night_calls) %>%
  group_by(total_domestic_calls, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = total_domestic_calls, y = rate, color = rate)) +
  geom_point(shape = 21) + geom_line() +
  scale_color_gradient(low = "deepskyblue1", high = "indianred1") -> g1

train %>%
  group_by(total_intl_calls, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = total_intl_calls, y = rate, color = rate)) +
  geom_point(shape = 21) + geom_line() +
  scale_color_gradient(low = "deepskyblue1", high = "indianred1") -> g2

grid.arrange(g1, g2, nrow = 2)
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-10-1.png" title = "plot11" alt = "plot11" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 8) Total Minutes

{% highlight javascript %}
train %>%
  mutate(total_domestic_minutes = total_day_minutes + total_eve_minutes + total_night_minutes) %>%
  mutate(total_domestic_minutes_category = ifelse(total_domestic_minutes <= 335, "0~335",
                                                  ifelse(total_domestic_minutes > 335 & total_domestic_minutes <= 395, "336~395",
                                                  ifelse(total_domestic_minutes > 395 & total_domestic_minutes <= 455, "396~455",
                                                  ifelse(total_domestic_minutes > 455 & total_domestic_minutes <= 515, "456~515",
                                                  ifelse(total_domestic_minutes > 515 & total_domestic_minutes <= 575, "516~575",
                                                  ifelse(total_domestic_minutes > 575 & total_domestic_minutes <= 635, "576~635",
                                                  ifelse(total_domestic_minutes > 635 & total_domestic_minutes <= 695, "636~695",
                                                  ifelse(total_domestic_minutes > 695 & total_domestic_minutes <= 755, "696~755",
                                                  ifelse(total_domestic_minutes > 755 & total_domestic_minutes <= 815, "756~815",
                                                  ifelse(total_domestic_minutes > 815, "816~", NA))))))))))) %>%
  select(-total_domestic_minutes) %>%
  group_by(total_domestic_minutes_category, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = total_domestic_minutes_category, y = rate, fill = rate)) +
  geom_col() +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-11-1.png" title = "plot12" alt = "plot12" width = "1008" height = "500" style = "display: block; margin: auto;" />
{% highlight javascript %}
train %>%
  group_by(total_intl_minutes, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = total_intl_minutes, y = rate, color = rate)) +
  geom_point(shape = 21) + geom_line() +
  scale_color_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-11-2.png" title = "plot13" alt = "plot13" width = "1008" height = "500" style = "display: block; margin: auto;" />

### 9) Total Charges

{% highlight javascript %}
train %>%
  mutate(total_domestic_charge = total_day_charge + total_eve_charge + total_night_charge) %>%
  mutate(total_domestic_charge_category = factor(round(total_domestic_charge, -1))) %>%
  group_by(total_domestic_charge_category, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = total_domestic_charge_category, y = rate, fill = rate)) +
  geom_col() +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-12-1.png" title = "plot14" alt = "plot14" width = "1008" height = "500" style = "display: block; margin: auto;" />

{% highlight javascript %}
train %>%
  group_by(total_intl_charge, churn) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(churn == 1) %>%
  ggplot(aes(x = total_intl_charge, y = rate, color = rate)) +
  geom_point(shape = 21) + geom_line() +
  scale_color_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

<img src = "/assets/Lecture/2017-03-30-Churn-Prediction/unnamed-chunk-12-2.png" title = "plot1" alt = "plot1" width = "1008" height = "500" style = "display: block; margin: auto;" />

# 3. Feature Engineering

{% highlight javascript %}
FeatureEngineering <- function(data){
  data %>%
    # Domestic
    mutate(total_domestic_calls = total_day_calls + total_eve_calls + total_night_calls) %>%
    mutate(total_domestic_minutes = total_day_minutes + total_eve_minutes + total_night_minutes) %>%
    mutate(total_domestic_charge = total_day_charge + total_eve_charge + total_night_charge) %>%
    
    # Category
    mutate(total_domestic_charge_category = factor(round(total_domestic_charge, -1))) %>%
    mutate(total_domestic_minutes_category = ifelse(total_domestic_minutes <= 335, "0~335",
                                             ifelse(total_domestic_minutes > 335 & total_domestic_minutes <= 395, "336~395",
                                             ifelse(total_domestic_minutes > 395 & total_domestic_minutes <= 455, "396~455",
                                             ifelse(total_domestic_minutes > 455 & total_domestic_minutes <= 515, "456~515",
                                             ifelse(total_domestic_minutes > 515 & total_domestic_minutes <= 575, "516~575",
                                             ifelse(total_domestic_minutes > 575 & total_domestic_minutes <= 635, "576~635",
                                             ifelse(total_domestic_minutes > 635 & total_domestic_minutes <= 695, "636~695",
                                             ifelse(total_domestic_minutes > 695 & total_domestic_minutes <= 755, "696~755",
                                             ifelse(total_domestic_minutes > 755 & total_domestic_minutes <= 815, "756~815",
                                             ifelse(total_domestic_minutes > 815, "816~", NA))))))))))) %>%
    mutate(number_vmail_messages_category = factor(round(number_vmail_messages, -1))) %>%
    
    # Customer Service Calls
    mutate(customer_service_calls = ifelse(number_customer_service_calls > 0, 1, 0)) %>%
    
    # Delete Unnecessory Colums
    dplyr::select(-(total_day_minutes:total_night_charge))
    
    
}

# Apply FeatureEngineering
train <- FeatureEngineering(train)
test <- FeatureEngineering(test)

train$area_code <- as.factor(train$area_code)
test$area_code <- as.factor(test$area_code)

train$total_domestic_minutes_category <- as.factor(train$total_domestic_minutes_category)
test$total_domestic_minutes_category <- as.factor(test$total_domestic_minutes_category)

train$customer_service_calls <- as.factor(train$customer_service_calls)
test$customer_service_calls <- as.factor(test$customer_service_calls)
{% endhighlight %}

# 4. Modeling

{% highlight javascript %}
churn_model <- glm(churn ~ ., data = train, family = "binomial")
summary(churn_model)
{% endhighlight %}



{% highlight javascript %}
## 
## Call:
## glm(formula = churn ~ ., family = "binomial", data = train)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -2.7716  -0.4121  -0.2453  -0.1186   3.4971  
## 
## Coefficients:
##                                          Estimate Std. Error z value
## (Intercept)                            -7.059e+00  1.956e+00  -3.609
## stateAL                                 4.705e-02  8.412e-01   0.056
## stateAR                                 7.298e-01  8.319e-01   0.877
## stateAZ                                 2.500e-01  9.028e-01   0.277
## stateCA                                 2.199e+00  8.308e-01   2.646
## stateCO                                 5.092e-01  8.451e-01   0.602
## stateCT                                 8.143e-01  7.873e-01   1.034
## stateDC                                 2.747e-01  8.860e-01   0.310
## stateDE                                 7.112e-01  8.065e-01   0.882
## stateFL                                 5.293e-01  8.190e-01   0.646
## stateGA                                 3.448e-01  8.730e-01   0.395
## stateHI                                -4.721e-01  1.017e+00  -0.464
## stateIA                                 2.250e-01  9.601e-01   0.234
## stateID                                 9.521e-01  8.134e-01   1.171
## stateIL                                 3.743e-01  8.726e-01   0.429
## stateIN                                 4.685e-01  8.158e-01   0.574
## stateKS                                 1.051e+00  7.922e-01   1.327
## stateKY                                 6.740e-01  8.465e-01   0.796
## stateLA                                 8.101e-01  8.805e-01   0.920
## stateMA                                 1.224e+00  8.072e-01   1.516
## stateMD                                 1.014e+00  7.794e-01   1.301
## stateME                                 1.535e+00  7.865e-01   1.952
## stateMI                                 1.471e+00  7.735e-01   1.902
## stateMN                                 1.217e+00  7.727e-01   1.574
## stateMO                                 1.006e-01  8.746e-01   0.115
## stateMS                                 1.163e+00  7.905e-01   1.471
## stateMT                                 2.038e+00  7.739e-01   2.633
## stateNC                                 9.393e-01  8.007e-01   1.173
## stateND                                 7.481e-01  8.495e-01   0.881
## stateNE                                 4.161e-01  8.656e-01   0.481
## stateNH                                 1.083e+00  8.712e-01   1.243
## stateNJ                                 1.708e+00  7.665e-01   2.228
## stateNM                                 6.772e-01  8.555e-01   0.792
## stateNV                                 1.204e+00  7.774e-01   1.549
## stateNY                                 1.181e+00  7.786e-01   1.516
## stateOH                                 6.819e-01  8.211e-01   0.831
## stateOK                                 7.765e-01  8.310e-01   0.934
## stateOR                                 6.374e-01  8.029e-01   0.794
## statePA                                 1.193e+00  8.331e-01   1.431
## stateRI                                -3.877e-02  8.562e-01  -0.045
## stateSC                                 1.635e+00  7.993e-01   2.045
## stateSD                                 1.025e+00  8.178e-01   1.253
## stateTN                                 6.040e-01  8.772e-01   0.689
## stateTX                                 1.630e+00  7.693e-01   2.119
## stateUT                                 1.092e+00  7.997e-01   1.366
## stateVA                                -2.875e-01  8.886e-01  -0.324
## stateVT                                -7.314e-02  8.639e-01  -0.085
## stateWA                                 1.287e+00  7.902e-01   1.628
## stateWI                                 3.997e-02  8.359e-01   0.048
## stateWV                                 7.006e-01  7.831e-01   0.895
## stateWY                                 4.033e-01  8.232e-01   0.490
## account_length                          1.955e-03  1.618e-03   1.209
## area_code415                           -9.345e-02  1.596e-01  -0.585
## area_code510                           -4.879e-02  1.821e-01  -0.268
## international_planyes                   2.488e+00  1.701e-01  14.625
## voice_mail_planyes                     -1.591e+01  2.400e+03  -0.007
## number_vmail_messages                   1.546e-02  5.813e-02   0.266
## total_intl_minutes                     -3.659e+00  6.159e+00  -0.594
## total_intl_calls                       -1.070e-01  2.912e-02  -3.675
## total_intl_charge                       1.389e+01  2.281e+01   0.609
## number_customer_service_calls           8.038e-01  5.609e-02  14.332
## total_domestic_calls                    2.019e-03  1.902e-03   1.062
## total_domestic_minutes                  1.830e-03  4.088e-03   0.448
## total_domestic_charge                   1.441e-01  2.637e-02   5.466
## total_domestic_charge_category30       -2.021e+00  1.481e+00  -1.365
## total_domestic_charge_category40       -3.109e+00  1.628e+00  -1.910
## total_domestic_charge_category50       -3.875e+00  1.704e+00  -2.275
## total_domestic_charge_category60       -6.170e+00  1.799e+00  -3.429
## total_domestic_charge_category70       -5.993e+00  1.905e+00  -3.147
## total_domestic_charge_category80       -4.509e+00  2.026e+00  -2.225
## total_domestic_charge_category90        8.997e+00  6.412e+02   0.014
## total_domestic_minutes_category336~395 -1.409e+00  1.325e+00  -1.063
## total_domestic_minutes_category396~455 -1.181e+00  1.442e+00  -0.819
## total_domestic_minutes_category456~515 -1.522e+00  1.523e+00  -0.999
## total_domestic_minutes_category516~575 -2.076e+00  1.632e+00  -1.272
## total_domestic_minutes_category576~635 -2.561e+00  1.754e+00  -1.460
## total_domestic_minutes_category636~695 -2.041e+00  1.896e+00  -1.076
## total_domestic_minutes_category696~755 -1.470e+00  2.050e+00  -0.717
## total_domestic_minutes_category756~815 -2.006e+00  2.233e+00  -0.898
## total_domestic_minutes_category816~    -7.195e-01  2.602e+00  -0.277
## number_vmail_messages_category10        1.206e+00  2.437e+03   0.000
## number_vmail_messages_category20        1.336e+01  2.400e+03   0.006
## number_vmail_messages_category30        1.443e+01  2.400e+03   0.006
## number_vmail_messages_category40        1.450e+01  2.400e+03   0.006
## number_vmail_messages_category50        1.456e+01  2.400e+03   0.006
## customer_service_calls1                -1.389e+00  2.106e-01  -6.594
##                                        Pr(>|z|)    
## (Intercept)                            0.000308 ***
## stateAL                                0.955396    
## stateAR                                0.380375    
## stateAZ                                0.781813    
## stateCA                                0.008136 ** 
## stateCO                                0.546842    
## stateCT                                0.300983    
## stateDC                                0.756551    
## stateDE                                0.377902    
## stateFL                                0.518065    
## stateGA                                0.692891    
## stateHI                                0.642435    
## stateIA                                0.814739    
## stateID                                0.241793    
## stateIL                                0.667929    
## stateIN                                0.565744    
## stateKS                                0.184600    
## stateKY                                0.425901    
## stateLA                                0.357602    
## stateMA                                0.129553    
## stateMD                                0.193355    
## stateME                                0.050972 .  
## stateMI                                0.057151 .  
## stateMN                                0.115388    
## stateMO                                0.908467    
## stateMS                                0.141300    
## stateMT                                0.008456 ** 
## stateNC                                0.240748    
## stateND                                0.378523    
## stateNE                                0.630702    
## stateNH                                0.213804    
## stateNJ                                0.025853 *  
## stateNM                                0.428618    
## stateNV                                0.121421    
## stateNY                                0.129454    
## stateOH                                0.406220    
## stateOK                                0.350084    
## stateOR                                0.427246    
## statePA                                0.152291    
## stateRI                                0.963880    
## stateSC                                0.040818 *  
## stateSD                                0.210081    
## stateTN                                0.491129    
## stateTX                                0.034117 *  
## stateUT                                0.172033    
## stateVA                                0.746299    
## stateVT                                0.932529    
## stateWA                                0.103463    
## stateWI                                0.961866    
## stateWV                                0.370932    
## stateWY                                0.624185    
## account_length                         0.226809    
## area_code415                           0.558272    
## area_code510                           0.788727    
## international_planyes                   < 2e-16 ***
## voice_mail_planyes                     0.994711    
## number_vmail_messages                  0.790266    
## total_intl_minutes                     0.552480    
## total_intl_calls                       0.000238 ***
## total_intl_charge                      0.542530    
## number_customer_service_calls           < 2e-16 ***
## total_domestic_calls                   0.288436    
## total_domestic_minutes                 0.654375    
## total_domestic_charge                  4.59e-08 ***
## total_domestic_charge_category30       0.172403    
## total_domestic_charge_category40       0.056157 .  
## total_domestic_charge_category50       0.022926 *  
## total_domestic_charge_category60       0.000606 ***
## total_domestic_charge_category70       0.001652 ** 
## total_domestic_charge_category80       0.026092 *  
## total_domestic_charge_category90       0.988805    
## total_domestic_minutes_category336~395 0.287648    
## total_domestic_minutes_category396~455 0.412873    
## total_domestic_minutes_category456~515 0.317663    
## total_domestic_minutes_category516~575 0.203290    
## total_domestic_minutes_category576~635 0.144426    
## total_domestic_minutes_category636~695 0.281888    
## total_domestic_minutes_category696~755 0.473153    
## total_domestic_minutes_category756~815 0.369029    
## total_domestic_minutes_category816~    0.782110    
## number_vmail_messages_category10       0.999605    
## number_vmail_messages_category20       0.995556    
## number_vmail_messages_category30       0.995203    
## number_vmail_messages_category40       0.995179    
## number_vmail_messages_category50       0.995160    
## customer_service_calls1                4.28e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 2758.3  on 3332  degrees of freedom
## Residual deviance: 1706.7  on 3247  degrees of freedom
## AIC: 1878.7
## 
## Number of Fisher Scoring iterations: 15
{% endhighlight %}

# 5. Model Evaluation

{% highlight javascript %}
MultiLogLoss <- function(act, pred) {
    if(length(pred) != length(act))
        stop("The length of two vectors are different")
    
    eps <- 1e-15
    pred <- pmin(pmax(pred, eps), 1 - eps)
    sum(act * log(pred) + (1 - act) * log(1 - pred)) * -1/NROW(act)
}
churn_pred <- predict(churn_model, test, type = "response")

MultiLogLoss(test$churn, churn_pred)
{% endhighlight %}



{% highlight javascript %}
## [1] 0.2524587
{% endhighlight %}

