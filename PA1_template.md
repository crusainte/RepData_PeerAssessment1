# Reproducible Research: Peer Assessment 1
- By Yang Yuzhong

The activity.csv contains a total of 17,568 observation of the variables:

- **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- **date**: The date on which the measurement was taken in YYYY-MM-DD format

- **interval**: Identifier for the 5-minute interval in which measurement was taken

## Loading required R packages
Libraries required to perform the peer assessment is loaded.
Options in R environment are updated so that numbers >= 10^5 will be denoted in 
scientific notation (scipen = 1), and rounded to 2 digits (digits = 2). This
is to beautify subsequent numeric output in the peer assessment.

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(lubridate)
options(scipen = 1, digits = 2)
```

## Loading and preprocessing the data

activity.csv is first loaded using `read.csv()` function.


```r
activity<-read.csv("activity.csv")
```

The format of the loaded table is then changed to the following with select 
function from dply:

| date | interval | steps |
| ---- | -------- | ----- |
| 2012-10-01 | 0 | NA |
| ... | ... | ... |


```r
activity<-activity %>% 
select(date,interval,steps)
```

## What is mean total number of steps taken per day?

To obtain the mean total number of steps taken per day, the total steps per day
is first calculated. This is done through `group_by()` _date_ then `summarize()` 
which performs the `sum()` function on each day.


```r
dailytotalsteps <- activity %>% 
    group_by(date) %>% 
    summarize(steps = sum(steps,na.rm=TRUE))
```

The histogram shows the distribution of the total number of steps per day.


```r
hist(dailytotalsteps$steps,
     breaks=10,
     col="lightskyblue",
     main="Total number of steps taken per day",
     xlab="Total steps per day",
     ylab="Percentage of total steps taken per day")
```

![](PA1_template_files/figure-html/histogram-1.png) 

With the total number of steps taken per day, the mean and median is easily
calculated with the corresponding function.


```r
meansteps<-mean(dailytotalsteps$steps)

mediansteps<-median(dailytotalsteps$steps)
```

| mean total steps per day | median total steps per day | 
| ---- | ------ | 
| 9354.23 | 10395 | 

## What is the average daily activity pattern?

To obtain the average daily activity pattern, `group_by()` is first performed on
_interval_ then `mean()` of _steps_ are taken for each _interval_

```r
dailypattern<-activity %>% 
    group_by(interval) %>% 
    summarize(steps = mean(steps,na.rm=TRUE))
```

In order to plot the time series for the average daily pattern, 
the column _interval_ from the dataset must be converted from numeric data type
to time type. 

Since the _interval_ encompasses 1 day in 5 minutes interval,
`seq.POSIXt()` was used to generate a time sequence by 5 minutes with 
`Sys.Date()` and `Sys.Date()+1` as the limis of the generated time sequence.
After the time sequence is generated, `format()` is used to extract the
hours and minutes that corresponds to the _interval_ column from the dataset.

```r
mininterval<-as.POSIXct(Sys.Date())

maxinterval<-as.POSIXct(Sys.Date()+1)

dateinterval<-seq.POSIXt(mininterval, maxinterval, by = "5 min")

dateinterval<-format( dateinterval,"%H%M", tz="SGT")
```

With the time sequence generated and properly formatted, `mutate()` is used
to replace the current numeric _interval_ to the time based _interval_.

```r
dailypattern<- dailypattern %>%
    mutate(interval=dateinterval[1:288])
```

Lastly, time series plot is achieved by plotting the time interval against 
steps with proper labels for the plot.

```r
plot(dailypattern$interval,
     dailypattern$steps,type="l",
     main="Average Daily Activity Pattern",
     xlab="Daily Interval of 5 minutes",
     ylab="Average steps per interval")
```

![](PA1_template_files/figure-html/timeseriesplot-1.png) 

The 5-minute interval, on average across all the days in the dataset, 
that contains the maximum number of steps is obtained using `which.max()`
and subsetting _interval_.

```r
maxstepinterval<-dailypattern$interval[which.max(dailypattern$steps)]
```

The 5-minute interval with maximum number of steps is 0835

## Imputing missing values

### 1) Total number of missing values in the dataset


```r
num.na<-sum(is.na(activity$steps))
```

The total number of missing values in the dataset is 2304

### 2) Strategy for filling in all missing values in dataset

1. Obtain the median of steps for each interval per day

```r
dailymedian<-activity %>% 
    group_by(interval) %>% 
    summarize(mediansteps = median(steps,na.rm=TRUE))
```

2 & 3. 

```r
imputdata<- activity %>%
    left_join(dailymedian, by = "interval") %>%
    mutate(steps=ifelse(is.na(steps), mediansteps, steps))
```

## Are there differences in activity patterns between weekdays and weekends?
