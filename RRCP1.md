---
title: "Reproducible Research Course Project1"
author: "Wang Xuezhi"
date: "2019,3,20"
output: 
  html_document: 
    keep_md: yes
---



## Introduction
This assignment makes use of data from a personal activity monitoring device. this device collects data at 5 minute intervals through out the day. the data consists of two months of data from an anonymous individual collected during the months of October and November,2012 and include the number of steps taken in 5 minute intervals each day.

##Data source
The data was downloaded from course website: [Activity Monitoring Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

**steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA\color{red}{\verb|NA|}NA)
    
**date**: The date on which the measurement was taken in YYYY-MM-DD format
    
**interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

##Library and Load the data

```r
library(plyr)
library(ggplot2)
activity<-read.csv("activity.csv")
```

##Preparing the data 
Have a initial look of the dataset's dimension and colume names. Then creat a colume of "day" to mark weekday and weekend. Screen out the missing value, put the valid data in new dataset named"activity1".

```r
dim(activity)
```

```
## [1] 17568     3
```

```r
head(activity,3)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
```

```r
activity_cl<-activity[!is.na(activity$steps),]
```

## Mean steps per day
Calculate the total number of steps perday

```r
activity_mn<-aggregate(activity_cl$steps~activity_cl$date,FUN=sum)
colnames(activity_mn)<-c("date","steps")
```

Plot histogram of daily steps number.


```r
hist(activity_mn$steps,col="green",main="Daily steps number")
```

![](RRCP1_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Calculate the mean and median of total number per day.

```r
dailymean<-mean(activity_mn$steps)
dailymedian<-median(activity_mn$steps)
print(paste0("Daily mean steps each day is ",as.integer(dailymean)))
```

```
## [1] "Daily mean steps each day is 10766"
```

```r
print(paste0("Daily median steps each day is ",dailymedian))
```

```
## [1] "Daily median steps each day is 10765"
```


##The average daily activity pattern

Calculate the average steps of each interval,put the value in variable activity_av. Then mark the interval that has the maximum average steps in a day. Plot the meansteps versus 5minutes interval of activity_av.

```r
activity_av<-aggregate(activity_cl$steps~activity_cl$interval,FUN=mean)
colnames(activity_av)<-c("interval","steps")
activity_av$steps<-as.integer(activity_av$steps)
stepsmax<-max(activity_av$steps)
intervalmax<-activity_av$interval[activity_av$steps==stepsmax]
with(activity_av,plot(interval,steps,type="l",main="Average steps of every 5minutes in a day"))
abline(v=intervalmax,col="red",lwd=3)
```

![](RRCP1_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
print(paste0("The interval with the max steps ave number of a day is No. ",intervalmax,",  as indicated by the red line in figure"))
```

```
## [1] "The interval with the max steps ave number of a day is No. 835,  as indicated by the red line in figure"
```

```r
print(paste0("The max steps ave number  is ",as.integer(stepsmax)))
```

```
## [1] "The max steps ave number  is 206"
```
##Imputing missing values

Calculate the total number of missing value.

```r
nrow(activity[is.na(activity$steps),])
```

```
## [1] 2304
```
The total number of rows with steps value missed is 2304.

Filling in NAs with average steps of 5 minutes interval:
Firstly,creat a subset containing all NA rows, named "activityna"
Secondly,merge "activityna" with "activity_av"Put the new dataset in variable activity_mg.
Thirdly,subset and rename activity_mg, store the result in activity_fil.
At last,rowbind activity_cl and activity_fil to form the new dataset that missing values were filled.


```r
activityna<-activity[is.na(activity$steps),]
activity_mg<-merge(activityna,activity_av,by="interval")
activity_fil<-activity_mg[,c(4,3,1)]
colnames(activity_fil)<-c("steps","date","interval")
activity_new<-rbind(activity_cl,activity_fil)
dim(activity_new)
```

```
## [1] 17568     3
```

calculate the mean of new dataset.
Plot histogram of new dataset.

```r
activity_mn_new<-aggregate(activity_new$steps~activity_new$date,FUN=sum)
colnames(activity_mn_new)<-c("date","steps")
dim(activity_mn_new)
```

```
## [1] 61  2
```

```r
dim(activity_mn)
```

```
## [1] 53  2
```

```r
mean(activity_mn_new$steps)
```

```
## [1] 10749.77
```

```r
mean(activity_mn$steps)
```

```
## [1] 10766.19
```

```r
hist(activity_mn_new$steps,col="green",main="Steps number w/o NA imputting")
hist(activity_mn$steps,col="red",add=T)
legend("topright",c("Imputed Data","NA excluded data"),fill=c("green","red"))
```

![](RRCP1_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

Calculate the mean and median of new dataset.

```r
Newmean<-as.integer(mean(activity_mn_new$steps))
Newmedian<-median(activity_mn_new$steps)
Newmean
```

```
## [1] 10749
```

```r
Newmedian
```

```
## [1] 10641
```
The new mean with NA imputed by interval average is 10749, compared to 10766 which ignored NA. The new median is 10641,compared to 10765 which ignored NA.

##Imputing missing value with a different way
Filling in NAs with average steps of the day:
Firstly,using subset containing all NA rows, named "activityna"
Secondly,merge "activityna" with "activity_mn" by date.Put the new dataset in variable activity_mg1.
Thirdly,subset and rename activity_mg1, store the result in activity_fil1.
At last,rowbind activity_cl and activity_fil1 to form the new dataset that missing values were filled.


```r
activityna<-activity[is.na(activity$steps),]
activity0<-activity
activity0[is.na(activity0)]<-0

activity_mn1<-aggregate(activity0$steps~activity0$date,FUN=mean)
colnames(activity_mn1)<-c("date","steps")
activity_mn1$steps<-as.integer(activity_mn1$steps)
head(activity_mn1)
```

```
##         date steps
## 1 2012-10-01     0
## 2 2012-10-02     0
## 3 2012-10-03    39
## 4 2012-10-04    42
## 5 2012-10-05    46
## 6 2012-10-06    53
```

```r
activity_mg1<-merge(activityna,activity_mn1,by="date")


activity_fil1<-activity_mg1[,c(4,1,3)]
colnames(activity_fil1)<-c("steps","date","interval")
activity_new1<-rbind(activity_cl,activity_fil1)
head(activityna)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

calculate the mean of new dataset.
Plot histogram of new dataset.

```r
activity_mn_new1<-aggregate(activity_new1$steps~activity_new1$date,FUN=sum)
colnames(activity_mn_new1)<-c("date","steps")
dim(activity_mn_new1)
```

```
## [1] 61  2
```

```r
dim(activity_mn)
```

```
## [1] 53  2
```

```r
mean(activity_mn_new1$steps)
```

```
## [1] 9354.23
```

```r
mean(activity_mn$steps)
```

```
## [1] 10766.19
```

```r
hist(activity_mn_new1$steps,col="green",main="Steps number w/o NA imputting")
hist(activity_mn$steps,col="red",add=T)
legend("topright",c("Imputed Data","NA excluded data"),fill=c("green","red"))
```

![](RRCP1_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

Calculate the mean and median of new dataset.

```r
Newmean1<-as.integer(mean(activity_mn_new1$steps))
Newmedian1<-median(activity_mn_new1$steps)
Newmean1
```

```
## [1] 9354
```

```r
Newmedian1
```

```
## [1] 10395
```
The new mean with NA imputed by interval average is 10749, compared to 10766 which ignored NA. The new median is 10641,compared to 10765 which ignored NA.

##The differences between weekdays and weekends

creat new columes "WKD" to mark weekday and weekend.
Then calculate the average number of steps taken respectively. 

```r
library(lattice)
activity_new$day<-weekdays(as.Date(activity_new$date))
activity_new$WKD<-ifelse(activity_new$day %in% c("星期六", "星期日"), "Weekend", "Weekday")
activity_newAV<-ddply(activity_new,.(interval,WKD),summarize,Avg=mean(steps))
```

Plot the line figure using average number versus interval.


```r
xyplot(Avg~interval|WKD,data=activity_newAV,type="l",layout=c(1,2),main="Average Steps per Interval Based on Type of Day",ylab="Average Number of Steps",xlab="Interval")
```

![](RRCP1_files/figure-html/unnamed-chunk-15-1.png)<!-- -->
