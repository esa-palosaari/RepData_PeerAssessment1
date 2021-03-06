# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. "Load the data"

First I unzip the data file already provided in the git, if it hasn't been unzipped. Then I will read the .csv file to a data frame.


```r
if (!file.exists('activity.csv')){unzip('activity.zip')}
activity <- read.csv('activity.csv')
```

2. "Process/transform the data (if necessary) into a format suitable for your analysis"

I create a potentially more suitable time variable. First I install and load the lubridate package.

```r
if (!'lubridate' %in% installed.packages()) install.packages('lubridate')
library("lubridate")
```

I make separate date and time variables. First, I format the interval variable such that R can read it. Then I paste it together with the time variable. Finally I add the new vector to the data frame.


```r
date <- ymd(activity$date)
times <- as.character(activity$interval)
for (i in 1:length(times)) {
        if (nchar(times[i]) == 1) times[i] <- paste("000", times[i], sep="")
        if (nchar(times[i]) == 2) times[i] <- paste("00", times[i], sep="")
        if (nchar(times[i]) == 3) times[i] <- paste("0", times[i], sep="") 
}
date_time <- paste(date,times)
date_time <- parse_date_time(date_time, "ymd_hm")
hours <- toString(times)
hours <- parse_date_time(times, "hm")
activity <- cbind(activity, date_time)
activity <- cbind(activity, hours)
```


## What is mean total number of steps taken per day?

For this part of the assignment, I ignore the missing values in the dataset.

1. "Make a histogram of the total number of steps taken each day"

I calculate the total number of steps per day using the aggregate function.


```r
dailySum <- aggregate(activity$steps, by=list(activity$date), FUN=sum, na.rm=T)
head(dailySum)
```

```
##      Group.1     x
## 1 2012-10-01     0
## 2 2012-10-02   126
## 3 2012-10-03 11352
## 4 2012-10-04 12116
## 5 2012-10-05 13294
## 6 2012-10-06 15420
```


I make a histogram of the total number of steps taken each day. I increase the number of breaks from the default to ten in order to get a more detailed view of the frequency of days when each number of steps was taken.


```r
hist(dailySum$x, ylab="Number of days", xlab="Number of steps in a day", main="Histogram of steps in a day",  freq=T, breaks=10)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

2. "Calculate and report the mean and median total number of steps taken per day"

I calculate and report the mean and median of the total number of steps taken per day:


```r
stepMean <- mean(dailySum$x)
stepMean <- round(stepMean, 0)
stepMd <- median(dailySum$x)
```

The mean of the total number of steps taken per day is 9354 and the median is 10395.

## What is the average daily activity pattern?

1. "Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)"

I make a time series plot (type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis). I use the aggregate function which I used previously, but this time with 5-minute intervals instead of days and mean instead of sum. I use the hour:minute variable I previously created.


```r
intervalMean <- aggregate(activity$steps, by=list(activity$hours), FUN=mean, na.rm=T)
head(intervalMean)
```

```
##               Group.1         x
## 1 0000-01-01 00:00:00 1.7169811
## 2 0000-01-01 00:05:00 0.3396226
## 3 0000-01-01 00:10:00 0.1320755
## 4 0000-01-01 00:15:00 0.1509434
## 5 0000-01-01 00:20:00 0.0754717
## 6 0000-01-01 00:25:00 2.0943396
```

```r
plot(intervalMean$Group.1, intervalMean$x, type="l", main="Average steps per 5 minute intervals", xlab="Time of the day", ylab="Average number of steps in 5 minutes")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 


2. "Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?""


```r
i <- which.max(intervalMean[,2])
max5 <- intervalMean[i,1]
minute5 <- minute(max5)
hour5 <- hour(max5)
```

The time of the day containing the maximum number of steps is at 8:35.

## Imputing missing values

* "Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data."

1. "Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)"

I calculate the total number of missing values in the dataset using the sapply function:


```r
missing <- sapply(activity, function(x) sum(is.na(x)))
missing
```

```
##     steps      date  interval date_time     hours 
##      2304         0         0         0         0
```

```r
misstep <- missing[1]
```

There are only missing values in the steps variable. Therefore the missing values of that variable include all the rows with NAs and there are 2304 rows with missing values.



2. "Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc."

I use the mean for the 5-minute interval.

3. "Create a new dataset that is equal to the original dataset but with the missing data filled in."

I create the new dataset by replacing the NAs with the interval means. I use a for-loop.


```r
i_activity <- activity
j=1
for (i in 1:length(activity$steps)){
        if (is.na(activity[i,1])) i_activity[i,1] <- intervalMean[j,2]
        j <- j+1
        if (j > length(intervalMean$x)) j <- 1
}
```


4. "Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?"

I use the same code as previously to make the histogram and calculate the mean and median.


```r
i_dailySum <- aggregate(i_activity$steps, by=list(i_activity$date), FUN=sum)
hist(i_dailySum$x, ylab="Number of days", xlab="Number of steps in a day", main="Histogram of steps in a day, imputed data",  freq=T, breaks=10)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 


```r
i_stepMean <- mean(i_dailySum$x)
i_stepMean <- round(i_stepMean, 0)
i_stepMd <- median(i_dailySum$x)
i_stepMd <- round(i_stepMd,0)
s_stepMean <- toString(i_stepMean)
s_stepMd <- toString(i_stepMd)
```

For the imputed data, the mean of the total number of steps taken per day is 10766 and the median is 10766. These are higher than the values calculated for the data without imputation. The histogram also shows that the number of days with little or no steps has decreased and the number of days with about 10 000 steps has increased.

## Are there differences in activity patterns between weekdays and weekends?

"For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part."

1. "Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day."

I create the factor variable and I use the Finnish names for weekend days because the system is Finnish:


```r
wend <- c("Weekend", "Weekday")
i_activity <- (cbind(i_activity,wend))
i_activity$wend <- as.factor(ifelse(weekdays(i_activity$date_time) %in% c("Lauantai","Sunnuntai"), "Weekend", "Weekday")) 
```


2. "Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data."

First aggregate the means over intervals and weekdays vs. weekends. Then I create the plots using the lattice package. I add the as.table=T option to have the graphs on top of each other but it works only now and then.


```r
wendSteps <- aggregate(steps ~ interval + wend, data=i_activity, FUN=mean)
require(lattice)
```

```
## Loading required package: lattice
```

```r
lattice.options(default.args = list(as.table = TRUE))
xyplot(steps ~ interval |wend, data=wendSteps, type="l", as.table=T)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

It seems that there is a spike on weekday mornings which is not observable on weekends. Going to work?
