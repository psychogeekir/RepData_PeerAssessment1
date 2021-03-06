# Reproducible Research: Peer Assessment 1

Set the global options


```r
opts_chunk$set(fig.width = 12, fig.height = 8, fig.align = "center", fig.path = "figures/", 
    echo = TRUE, warning = FALSE, message = FALSE)
```


## Loading and preprocessing the data


```r
if (!file.exists("activity.csv")) {
    unzip("activity.zip")
}

AMdata = read.csv("activity.csv", na.strings = "NA")
```




## What is mean total number of steps taken per day?


```r

steps_perday = aggregate(AMdata$steps, by = list(AMdata$date), FUN = sum, na.rm = TRUE)

hist(steps_perday[, 2], xlab = "steps", breaks = 10, main = "Histogram of steps", 
    col = "red")
```

<img src="figures/unnamed-chunk-1.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" style="display: block; margin: auto;" />


The **mean** total number of steps taken per day is


```r
mean(steps_perday[, 2])
```

```
## [1] 9354
```


The **median** total number of steps taken per day is


```r
median(steps_perday[, 2])
```

```
## [1] 10395
```



## What is the average daily activity pattern?


```r

steps_interval = aggregate(AMdata$steps, by = list(AMdata$interval), FUN = function(x) mean(x, 
    na.rm = TRUE))

plot(steps_interval, type = "l", xlab = "Interval", ylab = "average steps", 
    main = "average steps per interval", col = "blue")
```

<img src="figures/unnamed-chunk-4.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />



The 835 5-minute interval ,which is the 104th row, contains the maximum number of steps.


```r
steps_interval[steps_interval[, 2] == max(steps_interval[, 2]), ]
```

```
##     Group.1     x
## 104     835 206.2
```




## Imputing missing values


```r
na_row = nrow(AMdata[is.na(AMdata$steps), ])

AMdata_new = AMdata

set.seed(100)

AMdata_new[is.na(AMdata_new$steps), ]$steps = sample(steps_interval[, 2], na_row, 
    replace = TRUE)

steps_perday_new = aggregate(AMdata_new$steps, by = list(AMdata_new$date), FUN = function(x) sum(x, 
    na.rm = TRUE))
```


1. The total number of missing values in the dataset is 2304.

2. The strategy adopted here is sampling from the the mean for that 5-minute interval. 

3. The new dataset without `NA` is `AMdata_new`.

4. The histogram of the new dataset


```r
hist(steps_perday_new[, 2], xlab = "steps", breaks = 10, main = "Histogram of steps", 
    col = "red")
```

<img src="figures/unnamed-chunk-7.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" style="display: block; margin: auto;" />


The **mean** total number of steps taken per day of the new dataset is


```r
mean(steps_perday_new[, 2])
```

```
## [1] 10825
```


The **median** total number of steps taken per day of the new dataset is


```r
median(steps_perday_new[, 2])
```

```
## [1] 10833
```



```r
library(knitr)

d = data.frame(mean_value = c(mean(steps_perday[, 2]), mean(steps_perday_new[, 
    2])), median_value = c(median(steps_perday[, 2]), median(steps_perday_new[, 
    2])), row.names = c("old_data", "new_data"))

kable(d)
```

```
## |id        |  mean_value|  median_value|
## |:---------|-----------:|-------------:|
## |old_data  |        9354|         10395|
## |new_data  |       10825|         10833|
```



## Are there differences in activity patterns between weekdays and weekends?

Set the language


```r
Sys.setlocale("LC_TIME", "C")
```

```
## [1] "C"
```


`dayind` is a new factor variable in the dataset with two levels �C ��weekday�� and ��weekend�� indicating whether a given date is a weekday or weekend day.



```r
date_st = as.Date(AMdata_new$date, format = "%Y-%m-%d")

dayind = {
    weekdays(date_st, abbreviate = TRUE) == "Sun" | weekdays(date_st, abbreviate = TRUE) == 
        "Sat"
}

dayind[dayind == TRUE] = "weekend"
dayind[dayind == FALSE] = "weekday"
dayind = as.factor(dayind)
```


Make a panel plot


```r
steps_interval_weekday = with(AMdata_new[dayind == "weekday", ], aggregate(steps, 
    by = list(interval), FUN = "mean"))

steps_interval_weekday$dayind = "weekday"

steps_interval_weekend = with(AMdata_new[dayind == "weekend", ], aggregate(steps, 
    by = list(interval), FUN = "mean"))

steps_interval_weekend$dayind = "weekend"

newdata = rbind(steps_interval_weekday, steps_interval_weekend)

colnames(newdata) = c("interval", "steps_avg", "dayind")

library(lattice)

xyplot(steps_avg ~ interval | dayind, data = newdata, type = "l", layout = c(1, 
    2), ylab = "Number of steps", xlab = "Interval")
```

<img src="figures/unnamed-chunk-13.png" title="plot of chunk unnamed-chunk-13" alt="plot of chunk unnamed-chunk-13" style="display: block; margin: auto;" />



