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

TBC

```r
dateinterval<-strptime(sprintf("%04d", dailypattern$interval), format="%H%M")
```

With the time sequence generated and properly formatted, `mutate()` is used
to replace the current numeric _interval_ to the time based _interval_.

```r
dailypattern<- cbind(dailypattern,dateinterval)
```

Lastly, time series plot is achieved by plotting the time interval against 
steps with proper labels for the plot.

```r
plot(dailypattern$dateinterval,
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

The 5-minute interval with maximum number of steps is 835

## Imputing missing values

### 1) Total number of missing values in the dataset


```r
num.na<-sum(is.na(activity$steps))
```

The total number of missing values in the dataset is 2304

### 2) Strategy for filling in all missing values in dataset

1) Obtain the median of steps for each interval per day

```r
dailymedian<-activity %>% 
    group_by(interval) %>% 
    summarize(mediansteps = median(steps,na.rm=TRUE))
```

2 & 3. Create new dataset _imputdata_ using `left_join()` to retain all rows
in the original dataset and to join with the daily median of steps across
the entire dataset. Lastly, replace the rows with _NA_ with daily median steps
for the corresponding interval.

```r
imputdata<- activity %>%
    left_join(dailymedian, by = "interval") %>%
    mutate(steps=ifelse(is.na(steps), mediansteps, steps))
```

4) Histogram of the total number of steps taken each day and the mean 
and median total number of steps taken per day.

The total number of steps taken each day for the imputed data is calculated
using `group_by()` on _date_ then `sum()` on _steps_

```r
imputtotalsteps <- imputdata %>% 
    group_by(date) %>% 
    summarize(steps = sum(steps,na.rm=TRUE))
```

The histogram shows the distribution of the total number of steps per day on 
the imputed data.

```r
hist(imputtotalsteps$steps,
     breaks=10,
     col="orange",
     main="Total number of steps taken per day (imputed)",
     xlab="Total steps per day",
     ylab="Percentage of total steps taken per day")
```

![](PA1_template_files/figure-html/imputhistogram-1.png) 

The mean and median total steps per day is calculated with its corresponding 
functions and presented in the table.

```r
meanimputsteps<-mean(imputtotalsteps$steps)

medianimputsteps<-median(imputtotalsteps$steps)
```

| mean total steps per day | median total steps per day | 
| ---- | ------ | 
| 9503.87 | 10395 | 

With the NA values imputed with the average median steps of the dataset for a
particular interval, only the mean of the imputed data is different from the
estimates from the first part of the assignment.

The mean is now 9503.87 as compared to 9354.23 previously. 
The imputing of data has caused the mean of the dataset to shift higher as 
median of the average of steps in a particular interval across the dataset is
added to the mean calculation while keeping the median the same.

## Are there differences in activity patterns between weekdays and weekends?


```r
imputweekdays<- imputdata %>%
    mutate(day=weekdays(as.Date(imputdata$date))) %>%
    mutate(day=ifelse(day=="Saturday",1,ifelse(day=="Sunday",1,0))) %>%
    mutate(day=factor(day,labels = c("Weekday", "Weekend"))) %>%
    select(date,interval,day,steps)
```
