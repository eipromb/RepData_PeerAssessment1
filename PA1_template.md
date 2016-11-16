# Reproducible Research: Peer Assessment 1



---

## Loading and preprocessing the data

---

Show any code that is needed to

1. Load the data (i.e. read.csv())
1. Process/transform the data (if necessary) into a format suitable for your analysis



```r
library(dplyr)

steps <- read.csv('activity.csv', colClasses=c("integer", "Date", "integer"))
steps_by_date <- group_by(steps, date)
steps_by_interval <- group_by(steps, interval)
```

---

## What is mean total number of steps taken per day?

---

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
1. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
1. Calculate and report the mean and median of the total number of steps taken per day


```r
plot(summarize(steps_by_date, total = sum(steps, na.rm=TRUE)), type="h", xlab="Day", ylab="# Steps", main="Total Number of Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

The mean of the total number of steps taken per day is:

```r
mean(summarize(steps_by_date, total = sum(steps, na.rm=TRUE))$total)
```

```
## [1] 9354.23
```

The median of the total number of steps taken per day is:

```r
median(summarize(steps_by_date, total = sum(steps, na.rm=TRUE))$total)
```

```
## [1] 10395
```

---

## What is the average daily activity pattern?

---

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
1. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
plot(summarize(steps_by_interval, mean = mean(steps, na.rm=TRUE)), type="l", xlab="Interval", ylab="Average # Steps", main="Average Number of Steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

The 5-minute interval which contains the maximum average number of steps is:

```r
mean_steps_by_interval <- summarize(steps_by_interval, mean = mean(steps, na.rm=TRUE))
mean_steps_by_interval_max <- max(mean_steps_by_interval$mean)
mean_steps_by_interval_max_index <- which(mean_steps_by_interval$mean == mean_steps_by_interval_max)

mean_steps_by_interval[mean_steps_by_interval_max_index,]$interval
```

```
## [1] 835
```

---

## Imputing missing values

---

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
1. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
1. Create a new dataset that is equal to the original dataset but with the missing data filled in.
1. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

The total number of missing values is:

```r
nrow(subset(steps, is.na(steps)))
```

```
## [1] 2304
```

Imputing values using the interval means

```r
imputed_steps <- left_join(steps, mean_steps_by_interval, by="interval")
imputed_steps <- mutate(imputed_steps, final_steps = ifelse(is.na(steps), mean, steps))
imputed_steps_by_date <- group_by(imputed_steps, date)

plot(summarize(imputed_steps_by_date, total = sum(final_steps, na.rm=TRUE)), type="h", xlab="Day", ylab="# Steps", main="Total Number of Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

The mean of the total number of (with imputed) steps taken per day is:

```r
mean(summarize(imputed_steps_by_date, total = sum(final_steps, na.rm=TRUE))$total)
```

```
## [1] 10766.19
```

The median of the total number of (with imputed) steps taken per day is:

```r
median(summarize(imputed_steps_by_date, total = sum(final_steps, na.rm=TRUE))$total)
```

```
## [1] 10766.19
```

The mean/median of the total number of steps with imputed values is higher than without.

---

## Are there differences in activity patterns between weekdays and weekends?

---

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
1. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
imputed_steps_weekday <- mutate(imputed_steps, weekday = as.factor(ifelse(weekdays(date) %in% c("Saturday", "Sunday"), "weekend", "weekday")))

par(mfrow=c(2,1))
imputed_steps_weekday_by_interval <- group_by(imputed_steps_weekday, interval)

plot(summarize(subset(imputed_steps_weekday_by_interval, weekday=="weekend"), mean = mean(final_steps, na.rm=TRUE)), type="l", xlab="Interval", ylim=c(0,200), ylab="Average # Steps", main="Average Number of Steps per Interval (Weekend)")
plot(summarize(subset(imputed_steps_weekday_by_interval, weekday=="weekday"), mean = mean(final_steps, na.rm=TRUE)), type="l", xlab="Interval", ylim=c(0,200), ylab="Average # Steps", main="Average Number of Steps per Interval (Weekday)")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
