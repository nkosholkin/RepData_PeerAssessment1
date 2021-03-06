# Reproducible research Assignment 1

## Loading and preprocessing the data

First, set your working directory with the "activity.csv" file then load data




```r
activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

### Make a histogram of the total number of steps taken each day


```r
table <- tapply(activity$steps, activity$date, sum)
hist(table, main = NULL, xlab="Total number of steps per day (not including missing values)")
```

![plot of chunk Total_number_of_steps_per_day_frequency](figure/Total_number_of_steps_per_day_frequency.png) 

### Calculate and report the mean and median total number of steps taken per day


```r
library(plyr)
summaryData <- ddply(activity, .(date), summarize, steps = sum(steps))
meanSteps <- mean(summaryData$steps, na.rm = TRUE)
medianSteps <- median(summaryData$steps, na.rm = TRUE)
```

The mean steps are 10766.19 and median 10765.

## What is the average daily activity pattern?


```r
table2 <- ddply(activity, .(interval), summarize, steps = mean(steps, na.rm=TRUE))
plot(table2, type = 'l', main = "Average Steps by interval all days")
```

![plot of chunk Average_steps_by_interval_per_day](figure/Average_steps_by_interval_per_day.png) 

```r
maxSteps <- max(table2$steps)
maxInterval <- table2[table2$steps == maxSteps, 1]
```

Maximum (206.1698) steps are on 835th interval

## Imputing missing values


```r
totalMissingValues <- sum(is.na(activity))
```

Total missing values: 2304  

Strategy for filling missing data: mean for particular 5-minute interval for each NA

```r
activityNoNA <- activity
n <- 0
for (i in activityNoNA[, 1]) {
        n <- n + 1
        if (is.na(i)) {
                # take particular interval
                interval <- activityNoNA[n, 3]
                # get mean value for this interval
                value <- as.numeric(table2[which(table2$interval == interval), 2])
                # fill NA with this value
                activityNoNA[n, 1] <- value
        }
}

# activityNoNA is a new dataset without NA

tableNoNA <- tapply(activityNoNA$steps, activityNoNA$date, sum)
hist(tableNoNA, main=NULL, xlab="Total number of steps per day (without missing values)")
```

![plot of chunk Total_number_of_steps_per_day_frequency_substitute_missing_values](figure/Total_number_of_steps_per_day_frequency_substitute_missing_values.png) 

Calculating mean and median for data without NA

```r
summaryDataNoNA <- ddply(activityNoNA, .(date), summarize, steps = sum(steps))
meanStepNoNA <- mean(summaryDataNoNA$steps, na.rm = TRUE)
medianStepsNoNA <- median(summaryDataNoNA$steps, na.rm = TRUE)
```

The new value of mean is the same - 10766.19 and the mean a bit bigger 10766.19 compare to 10765

## Are there differences in activity patterns between weekdays and weekends?

```r
activityNoNA$day <- weekdays(as.Date(activityNoNA$date))
# Subset workdays
weekday <- subset(activityNoNA, activityNoNA$day %in% c("Monday","Tuesday", 
                                                     "Wednesday", "Thursda", "Friday"))
# Subset weekends
weekend <- subset(activityNoNA, activityNoNA$day %in% c("Saturday","Sunday"))

# Calculate mean for each interval on workdays (weekdays) and weekends
meanStepsWeekday <- ddply(weekday, .(interval), summarize, steps = mean(steps, na.rm=TRUE))
meanStepsWeekend <- ddply(weekend, .(interval), summarize, steps = mean(steps, na.rm=TRUE))

par(mfrow = c(2,1), mar = c(4, 4, 4, 1))
plot(meanStepsWeekday, type = 'l', main = "Average steps on Weekdays", ylim = c(0, 250))
plot(meanStepsWeekend, type = 'l', main = "Average steps on Weekdend", ylim = c(0, 250))
```

![plot of chunk Compare_weekday_and_weekends](figure/Compare_weekday_and_weekends.png) 

We can see that on weekend activity is spread across all day compare to some descrease in the middle of weekday
