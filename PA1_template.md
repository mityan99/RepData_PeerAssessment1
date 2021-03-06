# Activity Monitoring Data
Michelle Fukunaga  



## Load  data

```r
url = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(url, "activity.zip")
unzip(zipfile="activity.zip")

activity_data = read.csv("activity.csv")

str(activity_data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

The dataset contains 3 variable:
- steps: Number of steps taking in a 5-minute interval (missing values are coded as 𝙽𝙰)
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

## What is mean total number of steps taken per day?

1. A histogram of the total number of steps taken each day (ignore NA values)


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
activity_day = group_by(activity_data, date) 
activity_day_sum = summarize(activity_day, total_steps = sum(steps, na.rm=TRUE))
hist(activity_day_sum$total_steps, xlab="Total Steps Taken Each Day", main="Histogram", breaks = 50)
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

2. The mean total number of steps taken per day was 

```r
mean(activity_day_sum$total_steps)
```

```
## [1] 9354.23
```

And the median total number of steps taken per day was 

```r
median(activity_day_sum$total_steps)
```

```
## [1] 10395
```
##What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
activity_interval = group_by(activity_data, interval)
activity_ts = summarize(activity_interval, avg_steps = mean(steps, na.rm=TRUE))
with(activity_ts, 
     plot(interval, avg_steps, type="l", xlab="5-minute interval", ylab="averaged steps across all days", main="Average Daily Activities Pattern")
)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
activity_ts[which.max(activity_ts$avg_steps),"interval"]
```

```
## # A tibble: 1 × 1
##   interval
##      <int>
## 1      835
```

##Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)

```r
sum(is.na(activity_data))
```

```
## [1] 2304
```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

One of the ways to fill in the missing values in the dataset is to use the mice() R function. In this case, I created an imputed dataset using the predictive mean matching. pmm finds a set of observed values with the closest predicted mean as the missing one and imputes the missing values by a random draw from that set. pmm is restricted to the observed values.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
library(mice)
imp = mice(activity_data, method = "pmm")
```

```
## 
##  iter imp variable
##   1   1  steps
##   1   2  steps
##   1   3  steps
##   1   4  steps
##   1   5  steps
##   2   1  steps
##   2   2  steps
##   2   3  steps
##   2   4  steps
##   2   5  steps
##   3   1  steps
##   3   2  steps
##   3   3  steps
##   3   4  steps
##   3   5  steps
##   4   1  steps
##   4   2  steps
##   4   3  steps
##   4   4  steps
##   4   5  steps
##   5   1  steps
##   5   2  steps
##   5   3  steps
##   5   4  steps
##   5   5  steps
```

```r
impData = complete(imp, 1)

sum(is.na(impData))
```

```
## [1] 0
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

A comparison of the total number of steps taken each day between the imputed data and original data shows much higher concentration of total steps around the original mean or 10,000s range. This is expected because of the selected method to impute the missing data was based on the predicted mean of the observed data.  Therefore, the mean and median of the imputed data are higher than that from the original data.  


```r
imp_day = group_by(impData, date) 
imp_day_sum = summarize(imp_day, total_steps = sum(steps))

mean(imp_day_sum$total_steps)
```

```
## [1] 11145.97
```

```r
median(imp_day_sum$total_steps)
```

```
## [1] 11458
```

```r
par(mfrow=c(1,2))
hist(imp_day_sum$total_steps, xlab="Total Steps Taken Each Day", main="Imputed Data", breaks = 50)
hist(activity_day_sum$total_steps, xlab="Total Steps Taken Each Day", main="Original Data", breaks = 50)
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

##Are there differences in activity patterns between weekdays and weekends?

For this part the 𝚠𝚎𝚎𝚔𝚍𝚊𝚢𝚜() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
impData$date = as.Date(impData$date)

newData = mutate(impData, weekdaylab = weekdays(date), weekendflag = "weekday")
newData[grepl("S(at|un)", newData$weekdaylab),"weekendflag"] = "weekend"
newData$weekendflag = as.factor(newData$weekendflag)
```

Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
library(ggplot2)
newData_interval = group_by(newData, interval, weekendflag)
newData_ts = summarize(newData_interval, avg_steps = mean(steps, na.rm=TRUE))
     
ggplot(data=newData_ts, aes(interval, avg_steps, color = weekendflag)) + 
        geom_line() +
        facet_grid(weekendflag~.) +
        labs(y="averaged steps across all days") +
        labs(title="Average Daily Activities Pattern by Weekday and Weekend")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
