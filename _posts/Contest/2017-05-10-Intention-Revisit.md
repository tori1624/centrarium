---
layout: post
title: "Development prediction model of intention to revisit travel destination using logistic regression analysis"
author: "Young Ho Lee"
date: "2017.05.10"
categories: Contest
cover: "/assets/contest/2017-05-10-Intention-Revisit/revisit/jpg"
---

이번 포스팅은 2017 경기 빅데이터 아이디어 공모전, 2017 시공간 빅데이터 논문 공모전에 제출했던 '로지스틱 회귀분석을 활용한 관광지 재방문 의사 예측모델 구축'에 관한 글이다. 사용한 데이터는 2011-2015년도의 국민여행실태조사의 설문 결과이며, 2011-2014년도의 데이터는 훈련 데이터로 사용하였고, 2015년도의 데이터는 검증 데이터로 사용하여 분석을 진행하였다. 이 분석에서는 전국 단위 모델과 시/도 단위 모델, 두 모델을 구축하였고, 이 두 모델의 결과를 비교하고자 하였다. 분석의 진행과정은 다음과 같다.


{% highlight r %}
# Basic Packages
library(ggplot2)
library(dplyr)
library(readr)
library(gridExtra)
library(caret)
library(e1071)
library(data.table)
{% endhighlight %}

# 1. Data Handling 1
## 1) Data Import

{% highlight r %}
# specify the path
data.path1 <- "D:/Data/Public_data/KNTS/Database/Original/Individual/"

# file name
file.list1 <- list.files(path = data.path1)
file.name1 <- substr(file.list1, 1, nchar(file.list1)-4)

# data import
for(i in 1:length(file.name1)) {
  tmp.csv <- fread(paste0(data.path1, file.list1)[i])
  assign(file.name1[i], tmp.csv)
  message(file.name1[i], "has completed")
}
{% endhighlight %}



{% highlight javascript %}
## 
Read 0.0% of 5796 rows
Read 5796 rows and 13408 (of 13408) columns from 0.145 GB file in 00:00:04
## 
Read 0.0% of 6534 rows
Read 6534 rows and 145237 (of 145237) columns from 1.770 GB file in 00:00:24
{% endhighlight %}

## 2) Extracting Necessary Question

{% highlight r %}
# data2011_1 <- data2011 %>%
#   select(PID_11, type1.1, month.1, q1.1, q3.1, q4_a.1, q5.1, q7_c.1, q10.1, 
#          q12_1.1, q12_2.1, q12_3.1, q12_4.1, q12_5.1, q12_6.1, q12_7.1,
#          q12_8.1, q12_10.1, q12_11.1, q12_12.1, q12_13.1, q6_1.1.1, q6_1_1.1.1,
#          q6_2_a.1.1, q6_3.1.1, q6_6.1.1, q6_7.1.1, q6_8.1.1)

# data2012_1 <- data2012 %>%
#   select(PID_12, type1.1, month.1, q1.1, q3.1, q4_a.1, q5.1, q7_c.1, q10.1, 
#          q12_1.1, q12_2.1, q12_3.1, q12_4.1, q12_5.1, q12_6.1, q12_7.1, 
#          q12_8.1, q12_10.1, q12_11.1, q12_12.1, q12_13.1, q6_1.1.1, q6_1_1.1.1,
#          q6_2_a.1.1, q6_3.1.1, q6_6.1.1, q6_7.1.1, q6_8.1.1)

# data2013_1 <- data2013 %>%
#   select(PID_13, type1.1, month.1, q1.1, q3.1, q4_a.1, q5.1, q7_c.1, q10.1, 
#          q12_1.1, q12_2.1, q12_3.1, q12_4.1, q12_5.1, q12_6.1, q12_7.1, q12_8.1,
#          q12_10.1, q12_11.1, q12_12.1, q12_13.1, q6_1.1.1, q6_1_1.1.1, 
#          q6_2_a.1.1, q6_3.1.1, q6_6.1.1, q6_7.1.1, q6_8.1.1)

# data2014_1 <- data2014 %>%
#   select(PID_14, type1.1, month.1, q1.1, q3.1, q4_a.1, q5.1, q7_c.1, q10.1, 
#          q12_1.1, q12_2.1, q12_3.1, q12_4.1, q12_5.1, q12_6.1, q12_7.1, q12_8.1,
#          q12_9.1, q12_10.1, q12_11.1, q12_12.1, q6_1.1, q6_1_1.1, q6_2_a.1, 
#          q6_3.1, q6_6.1, q6_7.1, q6_8.1)

# data2015_1 <- data2015 %>%
#   select(PID_15, type1.1, month.1, q1.1, q3.1, q4_a.1, q5.1, q7_c.1, q10.1, 
#          q12_1.1, q12_2.1, q12_3.1, q12_4.1, q12_5.1, q12_6.1, q12_7.1, q12_8.1, 
#          q12_9.1, q12_10.1, q12_11.1, q12_12.1, q6_1.1.1, q6_1_1.1.1, q6_2_a.1.1,
#          q6_3.1.1, q6_6.1.1, q6_7.1.1, q6_8.1.1)
{% endhighlight %}

## 3) Saving Data

{% highlight r %}
# file.name_1 <- paste0(file.name1, "_1")
#
# for(i in 1:5) {
#   write.csv(get(file.name_1[i]), paste0(file.name_1[i], ".csv"), 
#             row.names = FALSE)
# }
{% endhighlight %}

# 2. Data Handling 2
## 1) Data Import1

{% highlight r %}
# specify the path
data.path2 <- "D:/Data/Public_data/KNTS/Database/Refined/Individual_1/"

# file name
file.list2 <- list.files(path = data.path2)
file.name2 <- substr(file.list2, 1, nchar(file.list2)-4)

# data import
for(i in 1:length(file.name2)) {
  tmp.csv <- fread(paste0(data.path2, file.list2)[i])
  assign(file.name2[i], tmp.csv)
  message(file.name2[i], "has completed")
}
{% endhighlight %}

## 2) Data Import2

{% highlight r %}
# specify the path
data.path3 <- "D:/Data/Public_data/KNTS/Database/Refined/Individual_Cht/"

# file name
file.list3 <- list.files(path = data.path3)
file.name3 <- substr(file.list3, 1, nchar(file.list3)-4)

# data import
for(i in 1:length(file.name3)) {
  tmp.csv <- fread(paste0(data.path3, file.list3)[i])
  assign(file.name3[i], tmp.csv)
  message(file.name3[i], "has completed")
}
{% endhighlight %}

## 3) Extracting Domestic Tourism

{% highlight r %}
RowExtract <- function(data){
  data %>%
    # Domestic Tourism
    filter(type1.1 == 1)
}

for(i in 1:5) {
  tmp.csv <- get(file.name2[i])
  re.tmp <- RowExtract(tmp.csv)
  assign(paste0(file.name1[i], "_2"), re.tmp)
}
{% endhighlight %}

## 4) Dependent Variable

{% highlight r %}
Depend1 <- function(data){
  data %>%
    mutate(q6_7 = ifelse(q6_7.1.1 > 3, 1,
                         ifelse(q6_7.1.1 <= 3, 0, NA))) %>%
    mutate(q6_8 = ifelse(q6_8.1.1 > 3, 1,
                         ifelse(q6_8.1.1 <= 3, 0, NA))) %>%
    select(-q6_7.1.1, -q6_8.1.1, -q5.1, -q12_4.1)
}

Depend2 <- function(data){
  data %>%
    mutate(q6_7 = ifelse(q6_7.1 > 3, 1,
                         ifelse(q6_7.1 <= 3, 0, NA))) %>%
    mutate(q6_8 = ifelse(q6_8.1 > 3, 1,
                         ifelse(q6_8.1 <= 3, 0, NA))) %>%
    select(-q6_7.1, -q6_8.1, -q5.1, -q12_4.1)
}

data2011_3 <- Depend1(data2011_2)
data2012_3 <- Depend1(data2012_2)
data2013_3 <- Depend1(data2013_2)
data2014_3 <- Depend2(data2014_2)
data2015_3 <- Depend1(data2015_2)
{% endhighlight %}

## 5) Changing Rownames for Rbind

{% highlight r %}
names(data2014_3)[names(data2014_3) == "q6_1_1.1"] <- c("q6_1_1.1.1")
names(data2014_3)[names(data2014_3) == "q6_1.1"] <- c("q6_1.1.1")
names(data2014_3)[names(data2014_3) == "q6_2_a.1"] <- c("q6_2_a.1.1")
names(data2014_3)[names(data2014_3) == "q6_3.1"] <- c("q6_3.1.1")
names(data2014_3)[names(data2014_3) == "q6_6.1"] <- c("q6_6.1.1")

names(data2011_3)[16] <- c("q12_9.1")
names(data2012_3)[16] <- c("q12_9.1")
names(data2013_3)[16] <- c("q12_9.1")

names(data2011_3)[17] <- c("q12_10.1")
names(data2012_3)[17] <- c("q12_10.1")
names(data2013_3)[17] <- c("q12_10.1")

names(data2011_3)[18] <- c("q12_11.1")
names(data2012_3)[18] <- c("q12_11.1")
names(data2013_3)[18] <- c("q12_11.1")

names(data2011_3)[19] <- c("q12_12.1")
names(data2012_3)[19] <- c("q12_12.1")
names(data2013_3)[19] <- c("q12_12.1")
{% endhighlight %}

## 6) Merging Ind & Cht

{% highlight r %}
for(i in 1:5) {
  cha.csv <- get(file.name3[i])
  data3.csv <- get(paste0(file.name1[i], "_3"))
  tmp.csv <- merge(cha.csv, data3.csv, by.x = c("PID"), 
                   by.y = c(paste0("PID_1", i)))
  assign(paste0(file.name1[i], "_4"), tmp.csv)
}
{% endhighlight %}

## 7) Rbind the Data & Saving Data

{% highlight r %}
train <- rbind(data2011_4, data2012_4, data2013_4, data2014_4)
test <- data2015_4

# write.csv(train, "train.csv", row.names = FALSE)
# write.csv(test, "test.csv", row.names = FALSE)
{% endhighlight %}

# 3. Exploration Data Analysis
## 1) Data Import

{% highlight r %}
train <- read.csv("D:/Data/Public_data/KNTS/Database/Refined/individual_2/train.csv")
test <- read.csv("D:/Data/Public_data/KNTS/Database/Refined/individual_2/test.csv")

# feature selection
train <- train %>%
  select(PID, sido, ara_size, sex, age, income2, month.1, q1.1, q3.1, q4_a.1, 
         q7_c.1, q10.1, q6_1.1.1, q6_2_a.1.1, q6_3.1.1, q6_6.1.1, q6_7) %>%
  filter(q6_1.1.1 != 929)
test <- test %>%
  select(PID, sido, ara_size, sex, age, income2, month.1, q1.1, q3.1, q4_a.1, 
         q7_c.1, q10.1, q6_1.1.1, q6_2_a.1.1, q6_3.1.1, q6_6.1.1, q6_7) %>%
  filter(q6_1.1.1 != 929)
{% endhighlight %}

## 2) Visualization
### 1) Sido

{% highlight r %}
train$sido <- as.factor(train$sido)
levels(train$sido) <- c("Seoul", "Busan", "Daegu", "Incheon", "Gwangju", 
                        "Daejeon", "Ulsan", "Gyeonggi", "Gangwon", "Chungbuk", 
                        "Chungnam", "Jeonbuk", "Jeonnam", "Gyeonbuk", 
                        "Gyeongnam", "Jeju")
test$sido <- as.factor(test$sido)
levels(test$sido) <- c("Seoul", "Busan", "Daegu", "Incheon", "Gwangju", 
                       "Daejeon", "Ulsan", "Gyeonggi", "Gangwon", "Chungbuk",
                       "Chungnam", "Jeonbuk", "Jeonnam", "Gyeonbuk", 
                       "Gyeongnam", "Jeju")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(sido) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = sido, y = count, fill = sido)) +
  geom_col() + xlab("Sido") +
  scale_fill_discrete(name = "Sido") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, 
                                   hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-14](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-14-1.png)


{% highlight r %}
train %>%
  group_by(sido, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = reorder(sido, rate), y = rate, fill = rate)) +
  geom_col() + xlab("sido") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, 
                                   hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-15](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-15-1.png)

### 2) Area Size / Sex

{% highlight r %}
train$ara_size <- as.factor(train$ara_size)
levels(train$ara_size) <- c("B_City", "M&S_City", "Village")
test$ara_size <- as.factor(test$ara_size)
levels(test$ara_size) <- c("B_City", "M&S_City", "Village")

train$sex <- as.factor(train$sex)
levels(train$sex) <- c("Male", "Female")
test$sex <- as.factor(test$sex)
levels(test$sex) <- c("Male", "Female")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(ara_size) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = ara_size, y = count, fill = ara_size)) +
  geom_col() + xlab("Area Size") +
  scale_fill_discrete(name = "Area Size") -> g1

train %>%
  group_by(sex) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = sex, y = count, fill = sex)) +
  geom_col() -> g2

grid.arrange(g1, g2, ncol = 2)
{% endhighlight %}

![plot of chunk unnamed-chunk-17](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-17-1.png)


{% highlight r %}
train %>%
  group_by(ara_size, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = factor(ara_size), y = rate, fill = rate)) +
  geom_col() + xlab("area size") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> g3

train %>%
  group_by(sex, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = factor(sex), y = rate, fill = rate)) +
  geom_col() + xlab("Sex") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> g4

grid.arrange(g3, g4, ncol = 2)
{% endhighlight %}

![plot of chunk unnamed-chunk-18](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-18-1.png)

### 3) Age

{% highlight r %}
train %>%
  mutate(age_category = factor(round(age, -1))) %>%
  group_by(age_category, q6_7) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = age_category, y = count, fill = age_category)) +
  geom_col() + xlab("Age") +
  scale_fill_discrete(name = "Age")
{% endhighlight %}

![plot of chunk unnamed-chunk-19](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-19-1.png)


{% highlight r %}
train %>%
  mutate(age_category = factor(round(age, -1))) %>%
  group_by(age_category, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = age_category, y = rate, fill = rate)) +
  geom_col() + xlab("Age") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

![plot of chunk unnamed-chunk-20](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-20-1.png)

### 4) Month

{% highlight r %}
train %>%
  group_by(month.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = factor(month.1), y = count, fill = factor(month.1))) +
  geom_col() + xlab("Month") +
  scale_fill_discrete(name = "Month")
{% endhighlight %}

![plot of chunk unnamed-chunk-21](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-21-1.png)


{% highlight r %}
train %>%
  group_by(month.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = factor(month.1), y = rate, fill = rate)) +
  geom_col() + xlab("Month") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1")
{% endhighlight %}

![plot of chunk unnamed-chunk-22](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-22-1.png)

### 5) A Day Trip or Overnight / Purpose of Travel

{% highlight r %}
train$q1.1 <- as.factor(train$q1.1)
levels(train$q1.1) <- c("A Day", "Overnight")
test$q1.1 <- as.factor(test$q1.1)
levels(test$q1.1) <- c("A Day", "Overnight")

train$q3.1 <- as.factor(train$q3.1)
levels(train$q3.1) <- c("L_R_V", "Treatment", "Religion")
test$q3.1 <- as.factor(test$q3.1)
levels(test$q3.1) <- c("L_R_V", "Treatment", "Religion")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(q1.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q1.1, y = count, fill = q1.1)) +
  geom_col() + xlab("Trip") +
  scale_fill_discrete(name = "Trip") -> a1

train %>%
  group_by(q3.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q3.1, y = count, fill = q3.1)) +
  geom_col() + xlab("Purpose") +
  scale_fill_discrete(name = "Purpose") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, hjust = 1)) -> a2

grid.arrange(a1, a2, ncol = 2)
{% endhighlight %}

![plot of chunk unnamed-chunk-24](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-24-1.png)


{% highlight r %}
train %>%
  group_by(q1.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = q1.1, y = rate, fill = rate)) +
  geom_col() + xlab("Trip") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> a3

train %>%
  group_by(q3.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = q3.1, y = rate, fill = rate)) +
  geom_col() + xlab("Purpose") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> a4

grid.arrange(a3, a4, ncol = 2)
{% endhighlight %}

![plot of chunk unnamed-chunk-25](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-25-1.png)

### 6) Information about Travel

{% highlight r %}
train$q4_a.1 <- as.factor(train$q4_a.1)
levels(train$q4_a.1) <- c("Travel Agency", "Family", "Friend", "Internet", 
                          "Book", "News or TV Program", "Advertising", 
                          "Experience", "App", "the others")
test$q4_a.1 <- as.factor(test$q4_a.1)
levels(test$q4_a.1) <- c("Travel Agency", "Family", "Friend", "Internet", 
                         "Book", "News or TV Program", "Advertising", 
                         "Experience", "App", "the others")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(q4_a.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q4_a.1, y = count, fill = q4_a.1)) +
  geom_col() + xlab("Information") +
  scale_fill_discrete(name = "Information") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, 
                                   hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-27](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-27-1.png)


{% highlight r %}
train %>%
  group_by(q4_a.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = q4_a.1, y = rate, fill = rate)) +
  geom_col() + xlab("Information") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", vjust = 1, 
                                   hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-28](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-28-1.png)

### 7) Package Travel

{% highlight r %}
train$q10.1 <- as.factor(train$q10.1)
levels(train$q10.1) <- c("Yes", "No")
test$q10.1 <- as.factor(test$q10.1)
levels(test$q10.1) <- c("Yes", "No")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(q10.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q10.1, y = count, fill = q10.1)) +
  geom_col() + xlab("Package") +
  scale_fill_discrete(name = "Package") -> b1

train %>%
  group_by(q10.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = q10.1, y = rate, fill = rate)) +
  geom_col() + xlab("Package") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> b2

grid.arrange(b1, b2, ncol = 2)
{% endhighlight %}

![plot of chunk unnamed-chunk-30](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-30-1.png)

### 8) Destination(Sido)

{% highlight r %}
train$q6_1.1.1 <- as.factor(train$q6_1.1.1)
levels(train$q6_1.1.1) <- c("Seoul", "Busan", "Daegu", "Incheon", "Gwangju", 
                            "Daejeon", "Ulsan", "Gyeonggi", "Gangwon", 
                            "Chungbuk", "Chungnam", "Jeonbuk", "Jeonnam", 
                            "Gyeonbuk", "Gyeongnam", "Jeju")
test$q6_1.1.1 <- as.factor(test$q6_1.1.1)
levels(test$q6_1.1.1) <- c("Seoul", "Busan", "Daegu", "Incheon", "Gwangju", 
                           "Daejeon", "Ulsan", "Gyeonggi", "Gangwon", 
                           "Chungbuk", "Chungnam", "Jeonbuk", "Jeonnam", 
                           "Gyeonbuk", "Gyeongnam", "Jeju")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(q6_1.1.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q6_1.1.1, y = count, fill = q6_1.1.1)) +
  geom_col() + xlab("Destination") +
  scale_fill_discrete(name = "Destinaion") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                   vjust = 1, hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-32](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-32-1.png)


{% highlight r %}
train %>%
  group_by(q6_1.1.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = reorder(q6_1.1.1, rate), y = rate, fill = rate)) +
  geom_col() + xlab("Destination") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                   vjust = 1, hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-33](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-33-1.png)

### 9) Reason for Selection

{% highlight r %}
train$q6_2_a.1.1 <- as.factor(train$q6_2_a.1.1)
levels(train$q6_2_a.1.1) <- c("Awareness", "Attraction", "Cheap Cost", 
                              "Distance", "Limited Time", "Accommodation", 
                              "Companion Type", "Shopping", "Food", 
                              "Transportation", "Experience Program", 
                              "Recommendation", "Convenient Facilitiy", 
                              "Education", "the others")
test$q6_2_a.1.1 <- as.factor(test$q6_2_a.1.1)
levels(test$q6_2_a.1.1) <- c("Awareness", "Attraction", "Cheap Cost", 
                             "Distance", "Limited Time", "Accommodation", 
                             "Companion Type", "Shopping", "Food", 
                             "Transportation", "Experience Program", 
                             "Recommendation", "Convenient Facilitiy", 
                             "Education", "the others")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(q6_2_a.1.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q6_2_a.1.1, y = count, fill = q6_2_a.1.1)) +
  geom_col() + xlab("Reason") +
  scale_fill_discrete(name = "Reason") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                   vjust = 1, hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-35](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-35-1.png)


{% highlight r %}
train %>%
  group_by(q6_2_a.1.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = q6_2_a.1.1, y = rate, fill = rate)) +
  geom_col() + xlab("Reason") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                   vjust = 1, hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-36](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-36-1.png)

### 10) Transportaion

{% highlight r %}
train$q6_3.1.1 <- as.factor(train$q6_3.1.1)
levels(train$q6_3.1.1) <- c("Car", "Train", "Flight", "Ship", "Subway", 
                            "Regular Bus", "Irregular Bus", "Rent", "Bicycle", 
                            "the others")
test$q6_3.1.1 <- as.factor(test$q6_3.1.1)
levels(test$q6_3.1.1) <- c("Car", "Train", "Flight", "Ship", "Subway", 
                           "Regular Bus", "Irregular Bus", "Rent", "Bicycle", 
                           "the others")
{% endhighlight %}


{% highlight r %}
train %>%
  group_by(q6_3.1.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = q6_3.1.1, y = count, fill = q6_3.1.1)) +
  geom_col() + xlab("Transportation") +
  scale_fill_discrete(name = "Transportation") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                   vjust = 1, hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-38](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-38-1.png)


{% highlight r %}
train %>%
  group_by(q6_3.1.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = q6_3.1.1, y = rate, fill = rate)) +
  geom_col() + xlab("Transportation") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") +
  theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                   vjust = 1, hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-39](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-39-1.png)

### 11) Satisfaction

{% highlight r %}
train %>%
  group_by(q6_6.1.1) %>%
  summarise(count = n()) %>%
  ggplot(aes(x = factor(q6_6.1.1), y = count, fill = factor(q6_6.1.1))) +
  geom_col() + xlab("Satisfaction") +
  scale_fill_discrete(name = "Satisfaction") -> c1

train %>%
  group_by(q6_6.1.1, q6_7) %>%
  summarise(count = n()) %>%
  mutate(rate = count / sum(count)) %>%
  filter(q6_7 == 1) %>%
  ggplot(aes(x = factor(q6_6.1.1), y = rate, fill = rate)) +
  geom_col() + xlab("Satisfaction") +
  scale_fill_gradient(low = "deepskyblue1", high = "indianred1") -> c2

grid.arrange(c1, c2, ncol = 2)
{% endhighlight %}

![plot of chunk unnamed-chunk-40](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-40-1.png)

# 3 Global Model
## 1) Modeling

{% highlight r %}
# deleting NA rows
train <- na.omit(train)
test <- na.omit(test)

logit.global <- glm(q6_7 ~ age + income2 + month.1 + q1.1 + q3.1 + 
                    q4_a.1 + q7_c.1 + q6_1.1.1 + q6_2_a.1.1 + q6_3.1.1 
                    + q6_6.1.1, data = train, family = "binomial")
summary(logit.global)
{% endhighlight %}



{% highlight javascript %}
## 
## Call:
## glm(formula = q6_7 ~ age + income2 + month.1 + q1.1 + q3.1 + 
##     q4_a.1 + q7_c.1 + q6_1.1.1 + q6_2_a.1.1 + q6_3.1.1 + q6_6.1.1, 
##     family = "binomial", data = train)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -3.2855   0.1118   0.4196   0.5514   4.1819  
## 
## Coefficients:
##                                  Estimate Std. Error z value Pr(>|z|)    
## (Intercept)                    -9.169e+00  3.155e-01 -29.064  < 2e-16 ***
## age                             2.454e-03  1.618e-03   1.517 0.129313    
## income2                         1.035e-04  1.025e-04   1.009 0.312744    
## month.1                         2.000e-02  8.683e-03   2.303 0.021269 *  
## q1.1Overnight                   2.950e-01  6.102e-02   4.834 1.34e-06 ***
## q3.1Treatment                   3.559e-01  2.451e-01   1.452 0.146458    
## q3.1Religion                    6.258e-01  1.997e-01   3.134 0.001726 ** 
## q4_a.1Family                    1.070e-01  1.562e-01   0.685 0.493163    
## q4_a.1Friend                    4.948e-03  1.554e-01   0.032 0.974598    
## q4_a.1Internet                 -6.542e-01  1.715e-01  -3.814 0.000137 ***
## q4_a.1Book                     -1.696e-01  3.139e-01  -0.540 0.589015    
## q4_a.1News or TV Program       -4.284e-01  2.017e-01  -2.124 0.033707 *  
## q4_a.1Advertising              -2.708e-01  2.448e-01  -1.106 0.268744    
## q4_a.1Experience                5.564e-01  1.658e-01   3.355 0.000793 ***
## q4_a.1App                      -4.468e-01  3.809e-01  -1.173 0.240851    
## q4_a.1the others                1.421e-01  2.293e-01   0.620 0.535293    
## q7_c.1                         -3.257e-07  1.538e-07  -2.117 0.034236 *  
## q6_1.1.1Busan                  -3.165e-01  1.822e-01  -1.737 0.082334 .  
## q6_1.1.1Daegu                   1.356e-02  2.789e-01   0.049 0.961231    
## q6_1.1.1Incheon                -8.677e-01  1.869e-01  -4.643 3.44e-06 ***
## q6_1.1.1Gwangju                -2.998e-01  3.359e-01  -0.893 0.372094    
## q6_1.1.1Daejeon                -3.686e-01  2.691e-01  -1.370 0.170769    
## q6_1.1.1Ulsan                  -6.104e-01  2.487e-01  -2.454 0.014128 *  
## q6_1.1.1Gyeonggi               -3.488e-01  1.566e-01  -2.228 0.025876 *  
## q6_1.1.1Gangwon                -4.771e-01  1.595e-01  -2.992 0.002768 ** 
## q6_1.1.1Chungbuk                3.061e-02  1.968e-01   0.156 0.876367    
## q6_1.1.1Chungnam               -5.062e-01  1.660e-01  -3.049 0.002294 ** 
## q6_1.1.1Jeonbuk                -2.695e-01  1.791e-01  -1.505 0.132302    
## q6_1.1.1Jeonnam                -4.229e-01  1.630e-01  -2.595 0.009470 ** 
## q6_1.1.1Gyeonbuk               -1.088e-01  1.684e-01  -0.646 0.518317    
## q6_1.1.1Gyeongnam              -2.050e-01  1.634e-01  -1.255 0.209575    
## q6_1.1.1Jeju                   -3.886e-01  2.562e-01  -1.517 0.129310    
## q6_2_a.1.1Attraction           -3.794e-01  7.203e-02  -5.267 1.39e-07 ***
## q6_2_a.1.1Cheap Cost           -3.601e-01  1.117e-01  -3.224 0.001263 ** 
## q6_2_a.1.1Distance             -3.170e-01  9.776e-02  -3.242 0.001185 ** 
## q6_2_a.1.1Limited Time         -8.683e-01  1.598e-01  -5.434 5.52e-08 ***
## q6_2_a.1.1Accommodation        -1.266e+00  1.576e-01  -8.036 9.25e-16 ***
## q6_2_a.1.1Companion Type       -7.958e-01  8.827e-02  -9.016  < 2e-16 ***
## q6_2_a.1.1Shopping             -2.719e-01  4.315e-01  -0.630 0.528624    
## q6_2_a.1.1Food                 -2.855e-01  1.480e-01  -1.929 0.053753 .  
## q6_2_a.1.1Transportation       -1.031e+00  3.122e-01  -3.302 0.000960 ***
## q6_2_a.1.1Experience Program   -4.790e-01  1.920e-01  -2.495 0.012603 *  
## q6_2_a.1.1Recommendation       -6.638e-01  1.351e-01  -4.913 8.96e-07 ***
## q6_2_a.1.1Convenient Facilitiy  8.517e-01  3.666e-01   2.323 0.020177 *  
## q6_2_a.1.1Education            -4.985e-01  2.874e-01  -1.735 0.082806 .  
## q6_2_a.1.1the others           -9.865e-01  1.964e-01  -5.022 5.12e-07 ***
## q6_3.1.1Train                   4.606e-01  1.957e-01   2.354 0.018579 *  
## q6_3.1.1Flight                  3.227e-01  2.443e-01   1.321 0.186494    
## q6_3.1.1Ship                   -9.034e-01  2.528e-01  -3.573 0.000353 ***
## q6_3.1.1Subway                  1.824e-01  1.980e-01   0.921 0.356817    
## q6_3.1.1Regular Bus            -1.851e-01  1.299e-01  -1.425 0.154185    
## q6_3.1.1Irregular Bus          -3.954e-01  8.859e-02  -4.463 8.07e-06 ***
## q6_3.1.1Rent                   -4.321e-01  1.687e-01  -2.562 0.010421 *  
## q6_3.1.1Bicycle                 1.095e+01  1.678e+02   0.065 0.947978    
## q6_3.1.1the others             -1.759e-01  2.501e-01  -0.703 0.481796    
## q6_6.1.1                        2.805e+00  5.685e-02  49.347  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 15656  on 15658  degrees of freedom
## Residual deviance: 10701  on 15603  degrees of freedom
## AIC: 10813
## 
## Number of Fisher Scoring iterations: 12
{% endhighlight %}

## 2) Global Model Evaluation(Multi Logloss)

{% highlight r %}
MultiLogLoss <- function(act, pred) {
    if(length(pred) != length(act))
        stop("The length of two vectors are different")
    
    eps <- 1e-15
    pred <- pmin(pmax(pred, eps), 1 - eps)
    sum(act * log(pred) + (1 - act) * log(1 - pred)) * -1/NROW(act)
}
{% endhighlight %}


{% highlight r %}
global.pred <- predict(logit.global, test, type = "response")

MultiLogLoss(test$q6_7, global.pred)
{% endhighlight %}



{% highlight javascript %}
## [1] 0.2772779
{% endhighlight %}


{% highlight r %}
MLL.global <- MultiLogLoss(test$q6_7, global.pred)
{% endhighlight %}

## 3) Global Model Evaluation(Accuracy)

{% highlight r %}
global.binary <- ifelse(global.pred > 0.5, 1, 0)

confusionMatrix(global.binary, test$q6_7, positive = "1")
{% endhighlight %}



{% highlight javascript %}
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    0    1
##          0  384   84
##          1  326 4007
##                                           
##                Accuracy : 0.9146          
##                  95% CI : (0.9063, 0.9224)
##     No Information Rate : 0.8521          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.6056          
##  Mcnemar's Test P-Value : < 2.2e-16       
##                                           
##             Sensitivity : 0.9795          
##             Specificity : 0.5408          
##          Pos Pred Value : 0.9248          
##          Neg Pred Value : 0.8205          
##              Prevalence : 0.8521          
##          Detection Rate : 0.8346          
##    Detection Prevalence : 0.9025          
##       Balanced Accuracy : 0.7602          
##                                           
##        'Positive' Class : 1               
## 
{% endhighlight %}


{% highlight r %}
BA.global <- confusionMatrix(global.binary, test$q6_7, positive = "1")
BA.global <- BA.global$byClass[[11]]
{% endhighlight %}

# 4. Local Model
## 1) Feature Engineering

{% highlight r %}
x <- levels(train$q4_a.1)
y <- levels(train$q6_2_a.1.1)
z <- levels(train$q6_3.1.1)

# function
FeatureEngineering <- function(data){
  data %>%
    mutate(q4_a.1 = ifelse(q4_a.1 == x[2] | q4_a.1 == x[3], "Surroundings",
                    ifelse(q4_a.1 == x[8], "Experience",
                    ifelse(q4_a.1 == x[1] | q4_a.1 == x[4] | q4_a.1 == x[5] | 
                           q4_a.1 == x[6] | q4_a.1 == x[7] | q4_a.1 == x[9], 
                           "Media",
                    ifelse(q4_a.1 == x[10], "the others", NA))))) %>%
    mutate(q6_2_a.1.1 = ifelse(q6_2_a.1.1 == y[1], "Awareness",
                        ifelse(q6_2_a.1.1 == y[3] | q6_2_a.1.1 == y[4] | 
                               q6_2_a.1.1 == y[5], "Limited factor",
                        ifelse(q6_2_a.1.1 == y[2] | q6_2_a.1.1 == y[6] | 
                               q6_2_a.1.1 == y[8] | q6_2_a.1.1 == y[9] | 
                               q6_2_a.1.1 == y[10] | q6_2_a.1.1 == y[11] | 
                               q6_2_a.1.1 == y[13], "Facility",
                        ifelse(q6_2_a.1.1 == y[7] | q6_2_a.1.1 == y[12] | 
                               q6_2_a.1.1 == y[14] | q6_2_a.1.1 == y[15], 
                               "the others", NA))))) %>%
    mutate(q6_3.1.1 = ifelse(q6_3.1.1 == z[1] | q6_3.1.1 == z[7] | 
                             q6_3.1.1 == z[8] | q6_3.1.1 == z[9], "Private",
                      ifelse(q6_3.1.1 == z[2] | q6_3.1.1 == z[3] | 
                             q6_3.1.1 == z[4] | q6_3.1.1 == z[5] | 
                             q6_3.1.1 == z[6], "Public",
                      ifelse(q6_3.1.1 == z[10], "the others", NA))))
}

train_2 <- FeatureEngineering(train)
test_2 <- FeatureEngineering(test)

# factoring
train_2$q4_a.1 <- as.factor(train_2$q4_a.1)
train_2$q6_3.1.1 <- as.factor(train_2$q6_3.1.1)
train_2$q6_2_a.1.1 <- as.factor(train_2$q6_2_a.1.1)

test_2$q4_a.1 <- as.factor(test_2$q4_a.1)
test_2$q6_3.1.1 <- as.factor(test_2$q6_3.1.1)
test_2$q6_2_a.1.1 <- as.factor(test_2$q6_2_a.1.1)
{% endhighlight %}

## 2) Modeling

{% highlight r %}
destination <- unique(train$q6_1.1.1)

for(i in 1:length(destination)) {
  # filtering
  tr.csv <- train_2 %>%
    filter(q6_1.1.1 == destination[i])
  te.csv <- test_2 %>%
    filter(q6_1.1.1 == destination[i])
  
  assign(paste0("train.", destination[i]), tr.csv)
  assign(paste0("test.", destination[i]), te.csv)
  
  # modeling
  if(unique(tr.csv$q6_1.1.1) == "Gangwon") {
    logit.local <- glm(q6_7 ~ age + income2 + month.1 + q1.1 + q4_a.1 + 
                       q7_c.1 + q6_2_a.1.1 + q6_3.1.1 + q6_6.1.1, 
                       data = tr.csv, family = "binomial")
  } else {
    logit.local <- glm(q6_7 ~ age + income2 + month.1 + q1.1 + q3.1 + q4_a.1 + 
                     q7_c.1 + q6_2_a.1.1 + q6_3.1.1 + q6_6.1.1, 
                     data = tr.csv, family = "binomial")
  }
  
  assign(paste0("local.", destination[i]), logit.local)
  
  # prediction
  pred <- predict(logit.local, te.csv, type = "response")
  
  pred.df <- data.frame(PID = te.csv$PID, q6_7 = pred)
  
  assign(paste0("pred.", destination[i]), pred)
  assign(paste0("preddf.", destination[i]), pred.df)
  
  # model evaluation(multiLogLoss)
  MLL <- MultiLogLoss(te.csv$q6_7, pred)
  
  assign(paste0("MLL.", destination[i]), MLL)
  
  # model evaluation(accuracy)
  binary <- ifelse(pred > 0.5, 1, 0)
  cfm <- confusionMatrix(binary, te.csv$q6_7, positive = "1")
  BA <- cfm$byClass[[11]]
  
  assign(paste0("BA.", destination[i]), BA)
  
  message(destination[i], " has completed")
}
{% endhighlight %}

## 3) Result Visualization

{% highlight r %}
result.df <- data.frame(Destination = factor(c("Korea ","Seoul", "Busan", 
                                               "Daegu", "Incheon", "Gwangju", 
                                               "Daejeon", "Ulsan", "Gyeonggi", 
                                               "Gangwon", "Chungbuk", 
                                               "Chungnam", "Jeonbuk", "Jeonnam", 
                                               "Gyeonbuk", "Gyeongnam", 
                                               "Jeju")),
                        MultiLogLoss = c(MLL.global, MLL.Seoul, MLL.Busan, 
                                         MLL.Daegu, MLL.Incheon, MLL.Gwangju, 
                                         MLL.Daejeon, MLL.Ulsan, MLL.Gyeonggi, 
                                         MLL.Gangwon, MLL.Chungbuk, 
                                         MLL.Chungnam, MLL.Jeonbuk, MLL.Jeonnam,
                                         MLL.Gyeonbuk, MLL.Gyeongnam, MLL.Jeju),
                        BalancedAccuracy = c(BA.global, BA.Seoul, BA.Busan, 
                                             BA.Daegu, BA.Incheon, BA.Gwangju, 
                                             BA.Daejeon, BA.Ulsan, BA.Gyeonggi, 
                                             BA.Gangwon, BA.Chungbuk, 
                                             BA.Chungnam, BA.Jeonbuk, 
                                             BA.Jeonnam, BA.Gyeonbuk, 
                                             BA.Gyeongnam, BA.Jeju))
{% endhighlight %}


{% highlight r %}
result.df %>%
  ggplot(aes(x = Destination, y = MultiLogLoss, color = Destination)) +
    geom_line(group = 1, color = "black", alpha = 0.5) +
    geom_point(size = 3) +
    geom_vline(xintercept = 12, linetype = "dashed", alpha = 0.5 ) +
    geom_vline(xintercept = 7, linetype = "dashed", alpha = 0.5 ) +
    theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                     vjust = 1 ,hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-50](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-50-1.png)


{% highlight r %}
result.df %>%
  ggplot(aes(x = Destination, y = BalancedAccuracy, color = Destination)) +
    geom_line(group = 1, color = "black", alpha = 0.5) +
    geom_point(size = 3) +
    geom_vline(xintercept = 12, linetype = "dashed", alpha = 0.5 ) +
    geom_vline(xintercept = 7, linetype = "dashed", alpha = 0.5 ) +
    theme(axis.text.x = element_text(angle = 45, face = "italic", 
                                     vjust = 1 ,hjust = 1))
{% endhighlight %}

![plot of chunk unnamed-chunk-51](/assets/contest/2017-05-10-Intention-Revisit/unnamed-chunk-51-1.png)

## 4) Local Model Evaluation(Multi Logloss)

{% highlight r %}
local.pred <- rbind(preddf.Seoul, preddf.Busan, preddf.Daegu, preddf.Incheon,
                    preddf.Gwangju, preddf.Daejeon, preddf.Ulsan, 
                    preddf.Gyeonggi, preddf.Gangwon, preddf.Chungbuk, 
                    preddf.Chungnam, preddf.Jeonbuk, preddf.Jeonnam,
                    preddf.Gyeonbuk, preddf.Gyeongnam, preddf.Jeju)
local.pred <- arrange(local.pred, PID)
{% endhighlight %}


{% highlight r %}
MultiLogLoss(test$q6_7, local.pred$q6_7)
{% endhighlight %}



{% highlight javascript %}
## [1] 0.3377981
{% endhighlight %}

## 5) Local Model Evaluation(Accuracy)

{% highlight r %}
local.binary <- ifelse(local.pred$q6_7 > 0.5, 1, 0)

confusionMatrix(local.binary, test$q6_7, positive = "1")
{% endhighlight %}



{% highlight javascript %}
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    0    1
##          0  376   99
##          1  334 3992
##                                           
##                Accuracy : 0.9098          
##                  95% CI : (0.9014, 0.9178)
##     No Information Rate : 0.8521          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.5855          
##  Mcnemar's Test P-Value : < 2.2e-16       
##                                           
##             Sensitivity : 0.9758          
##             Specificity : 0.5296          
##          Pos Pred Value : 0.9228          
##          Neg Pred Value : 0.7916          
##              Prevalence : 0.8521          
##          Detection Rate : 0.8315          
##    Detection Prevalence : 0.9011          
##       Balanced Accuracy : 0.7527          
##                                           
##        'Positive' Class : 1               
## 
{% endhighlight %}

