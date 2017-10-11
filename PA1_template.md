Loading and preprocessing the data
----------------------------------

Loading and preprocessing the data

Show any code that is needed to

Load the data (i.e. read.csv()) Process/transform the data (if
necessary) into a format suitable for your analysis

    file <- read.csv("activity.csv")
    head(file)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

    str(file)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

What is mean total number of steps taken per day?
-------------------------------------------------

What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in
the dataset.

Calculate the total number of steps taken per day

    totalsteps <- aggregate(steps ~ date, data = file, sum, na.rm = TRUE)
    sum(totalsteps$steps)

    ## [1] 570608

If you do not understand the difference between a histogram and a
barplot, research the difference between them. Make a histogram of the
total number of steps taken each day

    hist(totalsteps$steps, col = "green", xlab = 'Steps', main = 'Histogram of Total Steps by day')

![](PA1_template_files/figure-markdown_strict/Histogram%20of%20Total%20Steps%20By%20Day-1.png)

Calculate and report the mean and median of the total number of steps
taken per day

    mean(totalsteps$steps)

    ## [1] 10766.19

    median(totalsteps$steps)

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

What is the average daily activity pattern?

    time_series <- tapply(file$steps, file$interval, mean, na.rm = TRUE)

Make a time series plot (i.e. type = "l") of the 5-minute interval
(x-axis) and the average number of steps taken, averaged across all days
(y-axis)

    plot(row.names(time_series),time_series, type = "l",xlab = "5 min interval", ylab = "Avg across Days", main = "Average number of steps taken by Day", col = "blue")

![](PA1_template_files/figure-markdown_strict/Plot%20of%20Time%20Series-1.png)

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

    max <- which.max(time_series)
    names(max)

    ## [1] "835"

Imputing missing values
-----------------------

Imputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as NA). The presence of missing days may introduce bias
into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with NAs)

    sum(!complete.cases(file))

    ## [1] 2304

Devise a strategy for filling in all of the missing values in the
dataset. The strategy does not need to be sophisticated. For example,
you could use the mean/median for that day, or the mean for that
5-minute interval, etc.

    StepsAverage <- aggregate(steps ~ interval, data = file, FUN = mean)
    fillNA <- numeric()
    for (i in 1:nrow(file)) {
        obs <- file[i, ]
        if (is.na(obs$steps)) {
            steps <- subset(StepsAverage, interval == obs$interval)$steps
        } else {
            steps <- obs$steps
        }
        fillNA <- c(fillNA, steps)
    }

Create a new dataset that is equal to the original dataset but with the
missing data filled in.

    newdata <- file
    newdata$steps <- fillNA
    summary(newdata)

    ##      steps                date          interval     
    ##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
    ##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
    ##  Median :  0.00   2012-10-03:  288   Median :1177.5  
    ##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
    ##  3rd Qu.: 27.00   2012-10-05:  288   3rd Qu.:1766.2  
    ##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
    ##                   (Other)   :15840

Make a histogram of the total number of steps taken each day and
Calculate and report the mean and median total number of steps taken per
day. Do these values differ from the estimates from the first part of
the assignment? What is the impact of imputing missing data on the
estimates of the total daily number of steps?

    newtotalsteps <- aggregate(steps ~ date, data = file, sum, na.rm = TRUE)
    hist(newtotalsteps$steps, col = "blue", xlab = 'Steps', main = 'New Histogram of Total Steps by day')

![](PA1_template_files/figure-markdown_strict/Hist%20of%20New%20Data-1.png)

    mean(newtotalsteps$steps)

    ## [1] 10766.19

    median(newtotalsteps$steps)

    ## [1] 10765

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Are there differences in activity patterns between weekdays and
weekends?

For this part the weekdays() function may be of some help here. Use the
dataset with the filled-in missing values for this part.

    day <- weekdays(as.Date(file$date))
    daytype <- vector()
    for (i in 1:nrow(file)) {
        if (day[i] == "Saturday") {
            daytype[i] <- "Weekend"
        } else if (day[i] == "Sunday") {
            daytype[i] <- "Weekend"
        } else {
            daytype[i] <- "Weekday"
        }
    }

Create a new factor variable in the dataset with two levels - "weekday"
and "weekend" indicating whether a given date is a weekday or weekend
day.

    file$daytype <- daytype
    file$daytype <- as.factor(file$daytype)

    stepsByDay <- aggregate(steps ~ interval + daytype, data = file, mean)
    names(stepsByDay) <- c("interval", "daytype", "steps")
    head(stepsByDay)

    ##   interval daytype     steps
    ## 1        0 Weekday 2.3333333
    ## 2        5 Weekday 0.4615385
    ## 3       10 Weekday 0.1794872
    ## 4       15 Weekday 0.2051282
    ## 5       20 Weekday 0.1025641
    ## 6       25 Weekday 1.5128205

Make a panel plot containing a time series plot (i.e. type = "l") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis). See the
README file in the GitHub repository to see an example of what this plot
should look like using simulated data.

    library(lattice)
    xyplot(steps ~ interval | daytype, stepsByDay, type = "l", layout = c(1, 2), 
        xlab = "Interval", ylab = "Number of steps")

![](PA1_template_files/figure-markdown_strict/Create%20Plot%20by%20DayType-1.png)
