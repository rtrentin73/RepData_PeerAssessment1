---
title: "Personal Movement Data Analysis"
author: "Ricardo Trentin"
date: "March 15, 2015"
output: html_document
---

Load activity data and store it in a  variable: 


```r
if (getwd() != "/Users/rtrentin/bigdata/rprogramming"){
        setwd("/Users/rtrentin/bigdata/rprogramming")
}
rawdata <- read.csv("activity.csv")
data <- na.omit(rawdata)
```

Agregrate the data per steps:


```r
data.steps <- aggregate(steps ~ date, data, sum)
```

Plot the total number of steps per day:


```r
hist(data.steps$steps, main="Total Number of Steps per Day", xlab="Total Number of Steps in a Day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

The mean number of steps per day is:


```r
mean(data.steps$steps)
```

```
## [1] 10766.19
```

The median of steps per day is:


```r
median(data.steps$steps)
```

```
## [1] 10765
```

Aggregate the data activity per steps/interval:


```r
data.interval <- aggregate(steps ~ interval, data, mean)
```

Plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis):


```r
plot(data.interval$interval, data.interval$steps, type='l', main="Average N. of Steps averaged over all days", xlab="Interval", ylab="Average Number of steps")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png) 

The 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps is:


```r
data.interval[which.max(data.interval$steps), ]
```

```
##     interval    steps
## 104      835 206.1698
```

The total number of missing values in the dataset is: 


```r
sum(!complete.cases(rawdata))
```

```
## [1] 2304
```

I replace the NA by the mean for that 5-minute interval. I loop across the rows of the data. If the steps value is NA for a row, I find the corresponding value of interval. 


```r
for (i in 1:nrow(rawdata)){
        if (is.na(rawdata$steps[i])){
                interval_val <- rawdata$interval[i]
                row_id <- which(data.interval$interval == interval_val)
                steps_val <- data.interval$steps[row_id]
                rawdata$steps[i] <- steps_val
  }
}
```

Aggregate the activity data with NA replaced by the average:


```r
data.interval.added <- aggregate(steps ~ date, rawdata, sum)
```

Histogram of total number of steps in a day:


```r
hist(data.interval.added$steps, main="Avg. Added Total number of steps per day", xlab="Total number of steps in a day")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 

The mean number of steps per day (NAs replaced by avg) is:


```r
mean(data.interval.added$steps)
```

```
## [1] 10766.19
```

The median of steps per day (NAs replaced by avg) is:


```r
median(data.interval.added$steps)
```

```
## [1] 10766.19
```

There is a very small difference between the the median values calculated replacing the missing values by the average while the mean values are the same.

A new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day is created:


```r
rawdata$date <- as.Date(rawdata$date, "%Y-%m-%d")
rawdata$day <- weekdays(rawdata$date)
rawdata$day_type <- c("weekday")
for (i in 1:nrow(rawdata)){
        if (rawdata$day[i] == "Saturday" || rawdata$day[i] == "Sunday"){
                rawdata$day_type[i] <- "weekend"
  }
}
rawdata$day_type <- as.factor(rawdata$day_type)
```

Aggregate steps as interval to get average number of steps in an interval across all days:


```r
rawdata.day <- aggregate(steps ~ interval + day_type, rawdata, mean)
```

Plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
library(ggplot2)
qplot(interval, steps, data=rawdata.day, geom=c("line"), xlab="Interval", ylab="Number of steps", main="") + facet_wrap(~ day_type, ncol=1)
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-17-1.png) 
