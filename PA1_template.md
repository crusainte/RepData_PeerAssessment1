# Reproducible Research: Peer Assessment 1
- By Yang Yuzhong

The activity.csv contains a total of 17,568 observation of the variables:

- **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- **date**: The date on which the measurement was taken in YYYY-MM-DD format

- **interval**: Identifier for the 5-minute interval in which measurement was taken

## Loading required R packages


```r
#library(dplyr)
```

## Loading and preprocessing the data

activity.csv is loaded using read.csv() function. No preprocessing is performed.


```r
#activity<-read.csv("activity.csv")
```

The format of the loaded table is then changed to the following with select 
function from dply:

| date | interval | steps |
| ---- | -------- | ----- |
| 2012-10-01 | 0 | NA |
| ... | ... | ... |


```r
#activity<-activity %>% select(date,interval,steps)
```

## What is mean total number of steps taken per day?

To obtain the mean total number of steps taken per day, activity data.frame is 
first grouped by date, then the mean of interval for each date is calculated
and put into interval_mean column. Lastly, interval_mean column


```r
#activity.mean <- activity %>% 
#    group_by(date) %>% 
#    summarize(interval_mean = mean(steps, na.rm=TRUE)) %>%
#    mutate(interval_mean = ifelse(is.nan(interval_mean),0,interval_mean))
```

## What is the average daily activity pattern?



## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
