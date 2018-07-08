---
layout: post
title: "An analysis of household travel diary survey for exploring spatial segregation across different places"
author: "Young Ho Lee"
date: "2018.06.02"
categories: Contest
cover: "/assets/contest/2017-06-02-Segregation-Place/place.jpg"
---

이번 포스팅은 2018 교통분야 빅데이터 및 마이크로 데이터를 활용한 정책논문 및 아이디어 공모전에 제출했던 '수도권 가구통행실태조사를 활용한 장소별 소득계층의 분리 양상 탐색'에 관한 글로 데이터 가공과 데이터 분석 부분에 해당하는 코드를 포함하고 있다. 사용한 데이터는 2016 서울시 가구통행실태조사이며 이 데이터를 활용하여 거주지, 직장, 학교에서의 소득계층별 인구 수를 추출하고자 하였다. 분리를 측정하기 위한 지수로는 노출 지수와 상이 지수를 사용하였고, 데이터 가공 과정과 분석의 진행 과정은 다음과 같다.

{% highlight r %}
# basic pacakges
library(seg)
library(dplyr)
library(reshape2)
library(openxlsx)
library(rgdal)
library(classInt)
library(RColorBrewer)
library(seg)
{% endhighlight %}

# 1. Data Import

{% highlight r %}
# htd
htd <- read.csv("D:/Data/Public_data/HTD/htd.csv")

# dong code
dong.df <- read.xlsx("D:/Study/2018/independentdeep/medical/data/dong_code.xlsx")
{% endhighlight %}

# 2. Data Handling
## 1) Necessary Columns

{% highlight r %}
# columns
htd.re <- htd[, c(4, 14, 13, 15, 12, 17, 46, 51)]

# name
names(htd.re)[1] <- "si_name"
names(htd.re)[2] <- "gu_name"
names(htd.re)[3] <- "gu_code"
names(htd.re)[4] <- "dong_name"
names(htd.re)[5] <- "dong_code"
names(htd.re)[6] <- "income"
names(htd.re)[7] <- "school_code"
names(htd.re)[8] <- "work_code"

# income
htd.re <- htd.re[htd.re$income != 9, ]
{% endhighlight %}

## 2) Gu Data Frame

{% highlight r %}
# columns
gu.df <- htd.re[, 2:3]

# gu code
gu.df$gu_code <- substr(gu.df$gu_code, 1, 5)

# data frame
gu.df <- gu.df[!duplicated(gu.df$gu_code), ]

# arrange
gu.order <- order(gu.df$gu_code)
gu.df <- gu.df[gu.order, ]
{% endhighlight %}

## 3) Dong Data Frame

{% highlight r %}
# filtering
dong.df <- dong.df[!duplicated(dong.df[, 3]), ]
dong.df <- dong.df[, c(3, 6)]

# names
names(dong.df)[1] <- "dong_name"
names(dong.df)[2] <- "dong_code"

# gu code
dong.df <- dong.df %>%
  mutate(gu_code = substr(dong.df$dong_code, 1, 5))

# gu name (join)
dong.df <- merge(dong.df, gu.df, by.x = c("gu_code"), by.y = c("gu_code"),
                 all.x = TRUE)

# arrange
dong.df <- dong.df[, c(4, 1:3)]
dong.order <- order(dong.df$dong_code)
dong.df <- dong.df[dong.order, ]
{% endhighlight %}

## 4) Residence

{% highlight r %}
# columns
residence <- htd.re[, 5:6]

# count
residence <- residence %>%
  group_by(dong_code, income) %>%
  summarise(count = n())

# cast
residence$dong_code <- as.factor(residence$dong_code)
residence$income <- as.factor(residence$income)
residence.df <- dcast(residence, dong_code ~ income)

# name
for (i in 1:6) {
  names(residence.df)[i+1] <- paste0("income", i)
}

# NA
for (i in 1:6) {
  residence.df[is.na(residence.df[, i+1]) == TRUE, i+1] <- 0
}

# join
residence.df <- cbind(dong.df[, 1:3], residence.df)

# group
residence.df1 <- residence.df %>%
  mutate(low = income1 + income2, middle = income3 + income4, 
         high = income5 + income6) %>%
  select(-c(income1, income2, income3, income4, income5, income6))

# ratio
sum.residence <- apply(residence.df1[, 5:7], 1, sum)
residence.ratio <- residence.df1
residence.ratio[, 5:7] <- residence.ratio[, 5:7] / sum.residence
{% endhighlight %}

## 5) School

{% highlight r %}
# columns
school <- htd.re[, c(7, 6)]

# NA
school <- school[is.na(school$school_code) == FALSE, ]

# seoul
school <- school[substr(as.character(school$school_code), 1, 2) == "11", ]

# count
school <- school %>%
  group_by(school_code, income) %>%
  summarise(count = n())

# cast
school$school_code <- as.factor(school$school_code)
school$income <- as.factor(school$income)
school.df <- dcast(school, school_code ~ income)

# name
for (i in 1:6) {
  names(school.df)[i+1] <- paste0("income", i)
}

# join
school.df <- merge(dong.df, school.df, by.x = c("dong_code"), 
                   by.y = c("school_code"), all.x = TRUE)

# arrange
school.df <- school.df[, c(2:4, 1, 5:10)]

# NA
for (i in 1:6) {
  school.df[is.na(school.df[, i+4]) == TRUE, i+4] <- 0
}

# group
school.df1 <- school.df %>%
  mutate(low = income1 + income2, middle = income3 + income4, 
         high = income5 + income6) %>%
  select(-c(income1, income2, income3, income4, income5, income6))

# ratio
sum.school <- apply(school.df1[, 5:7], 1, sum)
school.ratio <- school.df1
school.ratio[, 5:7] <- school.ratio[, 5:7] / sum.school

# NaN
for (i in 1:3) {
  school.ratio[is.nan(school.ratio[, i+4]) == TRUE, i+4] <- 0
}
{% endhighlight %}

## 6) Work

{% highlight r %}
# columns
work <- htd.re[, c(8, 6)]

# NA
work <- work[is.na(work$work_code) == FALSE, ]

# seoul
work <- work[substr(as.character(work$work_code), 1, 2) == "11", ]

# count
work <- work %>%
  group_by(work_code, income) %>%
  summarise(count = n())

# cast
work$work_code <- as.factor(work$work_code)
work$income <- as.factor(work$income)
work.df <- dcast(work, work_code ~ income)

# name
for (i in 1:6) {
  names(work.df)[i+1] <- paste0("income", i)
}

# NA
for (i in 1:6) {
  work.df[is.na(work.df[, i+1]) == TRUE, i+1] <- 0
}

# join
work.df <- cbind(dong.df[, 1:3], work.df)

# group
work.df1 <- work.df %>%
  mutate(low = income1 + income2, middle = income3 + income4, 
         high = income5 + income6) %>%
  select(-c(income1, income2, income3, income4, income5, income6))

# ratio
sum.work <- apply(work.df1[, 5:7], 1, sum)
work.ratio <- work.df1
work.ratio[, 5:7] <-work.ratio[, 5:7] / sum.work
{% endhighlight %}

# 3. Spatial Data
## 1) Data Import

{% highlight r %}
# korea
korea.sp <- readOGR("D:/Data/map/shp/nsdi/kostat/dong/Z_SOP_BND_ADM_DONG_PG.shp",
                    p4s = "+proj=tmerc +lat_0=38 +lon_0=127 +k=1 +x_0=200000 
                    +y_0=500000 +ellps=bessel +units=m +no_defs")
{% endhighlight %}



{% highlight javascript %}
## OGR data source with driver: ESRI Shapefile 
## Source: "D:/Data/map/shp/nsdi/kostat/dong/Z_SOP_BND_ADM_DONG_PG.shp", layer: "Z_SOP_BND_ADM_DONG_PG"
## with 3502 features
## It has 6 fields
{% endhighlight %}



{% highlight r %}
korea.df <- data.frame(korea.sp)

# seoul
seoul.sp <- korea.sp[substr(as.character(korea.sp$ADM_DR_CD), 1, 2) == "11", ]
seoul.wgs <- spTransform(seoul.sp, CRS("+proj=longlat +datum=WGS84 
                                       +no_defs +ellps=WGS84 +towgs84=0,0,0"))
seoul.df <- data.frame(seoul.wgs)
{% endhighlight %}

## 2) Visualizaiton
### 2-1) Arrange

{% highlight r %}
# shp
name.order1 <- order(seoul.df$ADM_DR_NM)
seoul.wgs <- seoul.wgs[name.order1, ]
seoul.df <- data.frame(seoul.wgs)

# data
name.order1 <- order(dong.df$dong_name)
residence.ratio <- residence.ratio[name.order1, ]
school.ratio <- school.ratio[name.order1, ]
work.ratio <- work.ratio[name.order1, ]
{% endhighlight %}

### 2-2) Residence

{% highlight r %}
# x, y axis
x <- bbox(seoul.wgs)

# color
purples <- brewer.pal(5, "Purples")

# class
resi.class1 <- classIntervals(residence.ratio$low, n = 5, style = "jenks")
resi.class2 <- classIntervals(residence.ratio$middle, n = 5, style = "jenks")
resi.class3 <- classIntervals(residence.ratio$high, n = 5, style = "jenks")

# legend break
a1 <- round(resi.class1[[2]], 2) * 100
a2 <- round(resi.class2[[2]], 2) * 100
a3 <- round(resi.class3[[2]], 2) * 100
{% endhighlight %}


{% highlight r %}
# plot(low)
plot(seoul.wgs, col = findColours(resi.class1, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", a1[2]), paste(a1[2], "-", a1[3]), 
                  paste(a1[3], "-", a1[4]), paste(a1[4], "-", a1[5]),
                  paste("Higher than", a1[5])),
       title = "Low Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-12-1.png)


{% highlight r %}
# plot(middle)
plot(seoul.wgs, col = findColours(resi.class2, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", a2[2]), paste(a2[2], "-", a2[3]), 
                  paste(a2[3], "-", a2[4]), paste(a2[4], "-", a2[5]),
                  paste("Higher than", a2[5])),
       title = "Middle Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-13](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-13-1.png)


{% highlight r %}
# plot(high)
plot(seoul.wgs, col = findColours(resi.class3, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", a3[2]), paste(a3[2], "-", a3[3]), 
                  paste(a3[3], "-", a3[4]), paste(a3[4], "-", a3[5]),
                  paste("Higher than", a3[5])),
       title = "High Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-14](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-14-1.png)

### 2-3) Work

{% highlight r %}
# class
work.class1 <- classIntervals(work.ratio$low, n = 5, style = "jenks")
work.class2 <- classIntervals(work.ratio$middle, n = 5, style = "jenks")
work.class3 <- classIntervals(work.ratio$high, n = 5, style = "jenks")

# legend break
c1 <- round(work.class1[[2]], 2) * 100
c2 <- round(work.class2[[2]], 2) * 100
c3 <- round(work.class3[[2]], 2) * 100
{% endhighlight %}


{% highlight r %}
# plot(low)
plot(seoul.wgs, col = findColours(work.class1, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", c1[2]), paste(c1[2], "-", c1[3]), 
                  paste(c1[3], "-", c1[4]), paste(c1[4], "-", c1[5]),
                  paste("Higher than", c1[5])),
       title = "Low Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-16](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-16-1.png)


{% highlight r %}
# plot(middle)
plot(seoul.wgs, col = findColours(work.class2, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", c2[2]), paste(c2[2], "-", c2[3]), 
                  paste(c2[3], "-", c2[4]), paste(c2[4], "-", c2[5]),
                  paste("Higher than", c2[5])),
       title = "Middle Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-17](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-17-1.png)


{% highlight r %}
# plot(high)
plot(seoul.wgs, col = findColours(work.class3, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", c3[2]), paste(c3[2], "-", c3[3]), 
                  paste(c3[3], "-", c3[4]), paste(c3[4], "-", c3[5]),
                  paste("Higher than", c3[5])),
       title = "High Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-18](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-18-1.png)

### 2-4) School

{% highlight r %}
# class
scho.class1 <- classIntervals(school.ratio$low, n = 5, style = "jenks")
scho.class2 <- classIntervals(school.ratio$middle, n = 5, style = "jenks")
scho.class3 <- classIntervals(school.ratio$high, n = 5, style = "jenks")

# legend break
b1 <- round(scho.class1[[2]], 2) * 100
b2 <- round(scho.class2[[2]], 2) * 100
b3 <- round(scho.class3[[2]], 2) * 100
{% endhighlight %}


{% highlight r %}
# plot(low)
plot(seoul.wgs, col = findColours(scho.class1, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", b1[2]), paste(b1[2], "-", b1[3]), 
                  paste(b1[3], "-", b1[4]), paste(b1[4], "-", b1[5]),
                  paste("Higher than", b1[5])),
       title = "Low Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-20](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-20-1.png)


{% highlight r %}
# plot(middle)
plot(seoul.wgs, col = findColours(scho.class2, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", b2[2]), paste(b2[2], "-", b2[3]), 
                  paste(b2[3], "-", b2[4]), paste(b2[4], "-", b2[5]),
                  paste("Higher than", b2[5])),
       title = "Middle Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-21](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-21-1.png)


{% highlight r %}
# plot(high)
plot(seoul.wgs, col = findColours(scho.class3, purples), border = FALSE)
axis(1, at = seq(x[1], x[3], by = ((x[3] - x[1]) / 4)), 
     labels = parse(text = degreeLabelsEW(seq(x[1], x[3],
                                          by = ((x[3] - x[1]) / 4)))))
axis(2, at = seq(x[2], x[4], by = ((x[4] - x[2]) / 4)), 
     labels = parse(text = degreeLabelsNS(seq(x[2], x[4], 
                                          by = ((x[4] - x[2]) / 4)))))
legend(126.72, 37.71, fill = purples,
       legend = c(paste("Lower than", b3[2]), paste(b3[2], "-", b3[3]), 
                  paste(b3[3], "-", b3[4]), paste(b3[4], "-", b3[5]),
                  paste("Higher than", b3[5])),
       title = "High Income Ratio (%)", cex = 1.4, bty = "n")
{% endhighlight %}

![plot of chunk unnamed-chunk-22](/assets/contest/2018-06-02-Segregation-Place/unnamed-chunk-22-1.png)

# 4. Exposure Index
## 1) Function

{% highlight r %}
exposure <- function(data) {
  data <- data[(data[, 1] + data[, 2]) != 0, ]
  
  a <- data[, 1] / sum(data[, 1])
  b <- data[, 2] / (data[, 1] + data[, 2])
  p <- sum(a * b)
  as.vector(p)
}
{% endhighlight %}

## 2) Result
### 2-1) Residence

{% highlight r %}
# ratio
round(apply(residence.df1[, 5:7], 2, sum) / sum(residence.df1[, 5:7]), 3) * 100
{% endhighlight %}



{% highlight javascript %}
##    low middle   high 
##   17.5   57.9   24.6
{% endhighlight %}


{% highlight r %}
# matrix
residence.mat1 <- matrix(0, ncol = 3, nrow = 3)
rownames(residence.mat1) <- c("low", "middle", "high")
colnames(residence.mat1) <- c("low", "middle", "high")

# low - middle
residence.mat1[1, 2] <- exposure(residence.df1[, 5:6])
residence.mat1[2, 1] <- exposure(residence.df1[, 6:5])

# low - high
residence.mat1[1, 3] <- exposure(residence.df1[, c(5, 7)])
residence.mat1[3, 1] <- exposure(residence.df1[, c(7, 5)])

# middle - high
residence.mat1[2, 3] <- exposure(residence.df1[, 6:7])
residence.mat1[3, 2] <- exposure(residence.df1[, 7:6])

round(residence.mat1, 4)
{% endhighlight %}



{% highlight javascript %}
##           low middle   high
## low    0.0000 0.7008 0.4488
## middle 0.2115 0.0000 0.2610
## high   0.3188 0.6143 0.0000
{% endhighlight %}

{% highlight r %}
# matrix
residence.mat2 <- matrix(0, ncol = 3, nrow = 2)
colnames(residence.mat2) <- c("low", "middle", "high")
rownames(residence.mat2) <- c("ab", "ba")

# low
resi.mh <- apply(residence.df1[, 6:7], 1, sum)
resi.low <- cbind(residence.df1[, 5], resi.mh)
residence.mat2[1, 1] <- exposure(resi.low)
residence.mat2[2, 1] <- exposure(resi.low[, 2:1])

# middle
resi.lh <- apply(residence.df1[, c(5, 7)], 1, sum)
resi.middle <- cbind(residence.df1[, 6], resi.lh)
residence.mat2[1, 2] <- exposure(resi.middle)
residence.mat2[2, 2] <- exposure(resi.middle[, 2:1])

# high
resi.lm <- apply(residence.df1[, 5:6], 1, sum)
resi.high <- cbind(residence.df1[, 7], resi.lm)
residence.mat2[1, 3] <- exposure(resi.high)
residence.mat2[2, 3] <- exposure(resi.high[, 2:1])

round(residence.mat2, 4)
{% endhighlight %}



{% highlight javascript %}
##       low middle   high
## ab 0.7559 0.3894 0.6610
## ba 0.1601 0.5359 0.2157
{% endhighlight %}

### 2-2) Work

{% highlight r %}
# ratio
round(apply(work.df1[, 5:7], 2, sum) / sum(work.df1[, 5:7]), 3) * 100
{% endhighlight %}



{% highlight javascript %}
##    low middle   high 
##   10.6   61.7   27.7
{% endhighlight %}


{% highlight r %}
# matrix
work.mat1 <- matrix(0, ncol = 3, nrow = 3)
rownames(work.mat1) <- c("low", "middle", "high")
colnames(work.mat1) <- c("low", "middle", "high")

# low - middle
work.mat1[1, 2] <- exposure(work.df1[, 5:6])
work.mat1[2, 1] <- exposure(work.df1[, 6:5])

# low - high
work.mat1[1, 3] <- exposure(work.df1[, c(5, 7)])
work.mat1[3, 1] <- exposure(work.df1[, c(7, 5)])

# middle - high
work.mat1[2, 3] <- exposure(work.df1[, 6:7])
work.mat1[3, 2] <- exposure(work.df1[, 7:6])

round(work.mat1, 4)
{% endhighlight %}



{% highlight javascript %}
##           low middle   high
## low    0.0000 0.8267 0.6454
## middle 0.1422 0.0000 0.3000
## high   0.2468 0.6670 0.0000
{% endhighlight %}

{% highlight r %}
# matrix
work.mat2 <- matrix(0, ncol = 3, nrow = 2)
colnames(work.mat2) <- c("low", "middle", "high")
rownames(work.mat2) <- c("ab", "ba")

# low
work.mh <- apply(work.df1[, 6:7], 1, sum)
work.low <- cbind(work.df1[, 5], work.mh)
work.mat2[1, 1] <- exposure(work.low)
work.mat2[2, 1] <- exposure(work.low[, 2:1])

# middle
work.lh <- apply(work.df1[, c(5, 7)], 1, sum)
work.middle <- cbind(work.df1[, 6], work.lh)
work.mat2[1, 2] <- exposure(work.middle)
work.mat2[2, 2] <- exposure(work.middle[, 2:1])

# high
work.lm <- apply(work.df1[, 5:6], 1, sum)
work.high <- cbind(work.df1[, 7], work.lm)
work.mat2[1, 3] <- exposure(work.high)
work.mat2[2, 3] <- exposure(work.high[, 2:1])

round(work.mat2, 4)
{% endhighlight %}



{% highlight javascript %}
##       low middle   high
## ab 0.8671 0.3768 0.6967
## ba 0.1029 0.6059 0.2674
{% endhighlight %}

### 2-3) School

{% highlight r %}
# ratio
round(apply(school.df1[, 5:7], 2, sum) / sum(school.df1[, 5:7]), 3) * 100
{% endhighlight %}



{% highlight javascript %}
##    low middle   high 
##    6.7   61.4   31.9
{% endhighlight %}


{% highlight r %}
# matrix
school.mat <- matrix(0, ncol = 3, nrow = 3)
rownames(school.mat) <- c("low", "middle", "high")
colnames(school.mat) <- c("low", "middle", "high")

# low - middle
school.mat[1, 2] <- exposure(school.df1[, 5:6])
school.mat[2, 1] <- exposure(school.df1[, 6:5])

# low - high
school.mat[1, 3] <- exposure(school.df1[, c(5, 7)])
school.mat[3, 1] <- exposure(school.df1[, c(7, 5)])

# middle - high
school.mat[2, 3] <- exposure(school.df1[, 6:7])
school.mat[3, 2] <- exposure(school.df1[, 7:6])

round(school.mat, 4)
{% endhighlight %}



{% highlight javascript %}
##           low middle   high
## low    0.0000 0.7966 0.6750
## middle 0.0874 0.0000 0.2967
## high   0.1425 0.5711 0.0000
{% endhighlight %}


{% highlight r %}
# matrix
school.mat2 <- matrix(0, ncol = 3, nrow = 2)
colnames(school.mat2) <- c("low", "middle", "high")
rownames(school.mat2) <- c("ab", "ba")

# low
school.mh <- apply(school.df1[, 6:7], 1, sum)
school.low <- cbind(school.df1[, 5], school.mh)
school.mat2[1, 1] <- exposure(school.low)
school.mat2[2, 1] <- exposure(school.low[, 2:1])

# middle
school.lh <- apply(school.df1[, c(5, 7)], 1, sum)
school.middle <- cbind(school.df1[, 6], school.lh)
school.mat2[1, 2] <- exposure(school.middle)
school.mat2[2, 2] <- exposure(school.middle[, 2:1])

# high
school.lm <- apply(school.df1[, 5:6], 1, sum)
school.high <- cbind(school.df1[, 7], school.lm)
school.mat2[1, 3] <- exposure(school.high)
school.mat2[2, 3] <- exposure(school.high[, 2:1])

round(school.mat2, 4)
{% endhighlight %}



{% highlight javascript %}
##       low middle   high
## ab 0.8586 0.3383 0.5972
## ba 0.0620 0.5376 0.2796
{% endhighlight %}

# 5. Dissimilarity
## 1) Residence

{% highlight r %}
residence.order <- residence.df1[name.order1, ]
{% endhighlight %}


{% highlight r %}
# matrix
residence.mat3 <- matrix(0, ncol = 3, nrow = 3)
rownames(residence.mat3) <- c("low", "middle", "high")
colnames(residence.mat3) <- c("low", "middle", "high")

# low - middle
residence.mat3[1, 2] <- dissim(data = residence.order[, 5:6])[[1]]
residence.mat3[2, 1] <- dissim(data = residence.order[, 6:5])[[1]]

# low - high
residence.mat3[1, 3] <- dissim(data = residence.order[, c(5, 7)])[[1]]
residence.mat3[3, 1] <- dissim(data = residence.order[, c(7, 5)])[[1]]

# middle - high
residence.mat3[2, 3] <- dissim(data = residence.order[, 6:7])[[1]]
residence.mat3[3, 2] <- dissim(data = residence.order[, 7:6])[[1]]

round(residence.mat3, 4)
{% endhighlight %}



{% highlight javascript %}
##           low middle   high
## low    0.0000 0.2766 0.4154
## middle 0.2766 0.0000 0.2995
## high   0.4154 0.2995 0.0000
{% endhighlight %}


{% highlight r %}
# matrix
residence.mat4 <- matrix(0, ncol = 3, nrow = 1)
colnames(residence.mat4) <- c("low", "middle", "high")

# low
resi.mh <- apply(residence.order[, 6:7], 1, sum)
resi.low <- cbind(residence.order[, 5], resi.mh)
residence.mat4[1, 1] <- dissim(data = resi.low)[[1]]

# middle
resi.lh <- apply(residence.order[, c(5, 7)], 1, sum)
resi.middle <- cbind(residence.order[, 6], resi.lh)
residence.mat4[1, 2] <- dissim(data = resi.middle)[[1]]

# high
resi.lm <- apply(residence.order[, 5:6], 1, sum)
resi.high <- cbind(residence.order[, 7], resi.lm)
residence.mat4[1, 3] <- dissim(data = resi.high)[[1]]

round(residence.mat4, 4)
{% endhighlight %}



{% highlight javascript %}
##         low middle   high
## [1,] 0.2955   0.21 0.3121
{% endhighlight %}

## 2) Work

{% highlight r %}
work.order <- work.df1[name.order1, ]
{% endhighlight %}


{% highlight r %}
# matrix
work.mat3 <- matrix(0, ncol = 3, nrow = 3)
rownames(work.mat3) <- c("low", "middle", "high")
colnames(work.mat3) <- c("low", "middle", "high")

# low - middle
work.mat3[1, 2] <- dissim(data = work.order[, 5:6])[[1]]
work.mat3[2, 1] <- dissim(data = work.order[, 6:5])[[1]]

# low - high
work.mat3[1, 3] <- dissim(data = work.order[, c(5, 7)])[[1]]
work.mat3[3, 1] <- dissim(data = work.order[, c(7, 5)])[[1]]

# middle - high
work.mat3[2, 3] <- dissim(data = work.order[, 6:7])[[1]]
work.mat3[3, 2] <- dissim(data = work.order[, 7:6])[[1]]

round(work.mat3, 4)
{% endhighlight %}



{% highlight javascript %}
##           low middle   high
## low    0.0000 0.1959 0.2964
## middle 0.1959 0.0000 0.1591
## high   0.2964 0.1591 0.0000
{% endhighlight %}


{% highlight r %}
# matrix
work.mat4 <- matrix(0, ncol = 3, nrow = 1)
colnames(work.mat4) <- c("low", "middle", "high")

# low
work.mh <- apply(work.order[, 6:7], 1, sum)
work.low <- cbind(work.order[, 5], work.mh)
work.mat4[1, 1] <- dissim(data = work.low)[[1]]

# middle
work.lh <- apply(work.order[, c(5, 7)], 1, sum)
work.middle <- cbind(work.order[, 6], work.lh)
work.mat4[1, 2] <- dissim(data = work.middle)[[1]]

# high
work.lm <- apply(work.order[, 5:6], 1, sum)
work.high <- cbind(work.order[, 7], work.lm)
work.mat4[1, 3] <- dissim(data = work.high)[[1]]

round(work.mat4, 4)
{% endhighlight %}



{% highlight javascript %}
##         low middle  high
## [1,] 0.2189 0.1069 0.172
{% endhighlight %}

## 3) School

{% highlight r %}
school.order <-school.df1[name.order1, ]
{% endhighlight %}


{% highlight r %}
# matrix
school.mat3 <- matrix(0, ncol = 3, nrow = 3)
rownames(school.mat3) <- c("low", "middle", "high")
colnames(school.mat3) <- c("low", "middle", "high")

# low - middle
school.mat3[1, 2] <- dissim(data = school.order[, 5:6])[[1]]
school.mat3[2, 1] <- dissim(data = school.order[, 6:5])[[1]]

# low - high
school.mat3[1, 3] <- dissim(data = school.order[, c(5, 7)])[[1]]
school.mat3[3, 1] <- dissim(data = school.order[, c(7, 5)])[[1]]

# middle - high
school.mat3[2, 3] <- dissim(data = school.order[, 6:7])[[1]]
school.mat3[3, 2] <- dissim(data = school.order[, 7:6])[[1]]

round(school.mat3, 4)
{% endhighlight %}



{% highlight javascript %}
##           low middle   high
## low    0.0000 0.4464 0.4329
## middle 0.4464 0.0000 0.3023
## high   0.4329 0.3023 0.0000
{% endhighlight %}


{% highlight r %}
# matrix
school.mat4 <- matrix(0, ncol = 3, nrow = 1)
colnames(school.mat4) <- c("low", "middle", "high")

# low
school.mh <- apply(school.order[, 6:7], 1, sum)
school.low <- cbind(school.order[, 5], school.mh)
school.mat4[1, 1] <- dissim(data = school.low)[[1]]

# middle
school.lh <- apply(school.order[, c(5, 7)], 1, sum)
school.middle <- cbind(school.order[, 6], school.lh)
school.mat4[1, 2] <- dissim(data = school.middle)[[1]]

# high
school.lm <- apply(school.order[, 5:6], 1, sum)
school.high <- cbind(school.order[, 7], school.lm)
school.mat4[1, 3] <- dissim(data = school.high)[[1]]

round(school.mat4, 4)
{% endhighlight %}



{% highlight javascript %}
##         low middle   high
## [1,] 0.4286 0.2936 0.2885
{% endhighlight %}

