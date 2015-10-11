# ReprodResearch1
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Loading and preprocessing the data
The data is loaded and tidied, converting the *interval* field from a varying 1 to 4 digit number representing a mixture hours/minutes into a simpler numeric factor which can be combined with the *date* to create a proper *time* column.


```r
    #load and tidy data
    myData <- read.csv("activity.csv")
    myData$interval <- as.numeric(as.factor(myData$interval))
    myData$time <- strptime(myData$date,"%Y-%m-%d") + 300 * myData$interval
```

Some snapshots of the data illustrate it's current form, including a number of missing values:


```r
    head(myData, 5)
```

```
##   steps       date interval                time
## 1    NA 2012-10-01        1 2012-10-01 00:05:00
## 2    NA 2012-10-01        2 2012-10-01 00:10:00
## 3    NA 2012-10-01        3 2012-10-01 00:15:00
## 4    NA 2012-10-01        4 2012-10-01 00:20:00
## 5    NA 2012-10-01        5 2012-10-01 00:25:00
```

```r
    tail(myData, 5)
```

```
##       steps       date interval                time
## 17564    NA 2012-11-30      284 2012-11-30 23:40:00
## 17565    NA 2012-11-30      285 2012-11-30 23:45:00
## 17566    NA 2012-11-30      286 2012-11-30 23:50:00
## 17567    NA 2012-11-30      287 2012-11-30 23:55:00
## 17568    NA 2012-11-30      288 2012-12-01 00:00:00
```

In preparation for graphing the lattice library is loaded:

```r
    #Ensure required libraries are loaded.
    require(lattice)
```

```
## Loading required package: lattice
```

## What is mean total number of steps taken per day?
The histogram below is prepared from the tidy data set using the following code.  The mean and median number of steps per day is included in the graph: 


```r
    #summarise data by day
    mySteps<-aggregate(steps ~ date, myData, FUN = mean, rm.na = TRUE)
    hist(mySteps$steps, main = "Histogram of Steps per day", xlab = "")  
    text(70, 18, paste("Mean = ", round(mean(mySteps$steps), 2), "\n Median = ", 
                     round(median(mySteps$steps), 2)))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

## What is the average daily activity pattern?
The data set can also be used to provide a graph of average daily activity. The interval which represents the maximum number of steps is included in the graph.

```r
    #summarise data by day
    myIntervals<-aggregate(steps ~ interval, myData, FUN = mean, rm.na = TRUE)
    plot(myIntervals, type = "l")  
    text(240, 200, paste("Maximum interval = ", 
                         which(myIntervals$steps == max(myIntervals$steps))))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

## Imputing missing values
As noted above there are a number of missing values which are coded as NA. The exact number is calculated by the code:

```r
    sum(is.na(myData$steps))
```

```
## [1] 2304
```
A good way to replace these so that they do not distort the data is by replacing them with the mean response for their interval:

```r
    # Revise data to replace missing values
    myNAs <- which(is.na(myData$steps))
    newData <- myData
    newData[myNAs, "steps"] <- myIntervals[newData[myNAs, "interval"], "steps"]
```

The following snapshots illustrate how this affects the data:


```r
    tail(myData, 5)
```

```
##       steps       date interval                time
## 17564    NA 2012-11-30      284 2012-11-30 23:40:00
## 17565    NA 2012-11-30      285 2012-11-30 23:45:00
## 17566    NA 2012-11-30      286 2012-11-30 23:50:00
## 17567    NA 2012-11-30      287 2012-11-30 23:55:00
## 17568    NA 2012-11-30      288 2012-12-01 00:00:00
```

```r
    tail(newData, 5)
```

```
##           steps       date interval                time
## 17564 4.6981132 2012-11-30      284 2012-11-30 23:40:00
## 17565 3.3018868 2012-11-30      285 2012-11-30 23:45:00
## 17566 0.6415094 2012-11-30      286 2012-11-30 23:50:00
## 17567 0.2264151 2012-11-30      287 2012-11-30 23:55:00
## 17568 1.0754717 2012-11-30      288 2012-12-01 00:00:00
```

As a result of these changes, the histogram prepared from this new data set has changed slightly.  Again, the mean and median number of steps have been included in the graph: 

```r
    # create histogram of average steps per day with revised data
    newSteps <- aggregate(steps ~ date, newData, FUN = mean)
    hist(newSteps$steps, main = "Histogram of Steps per day", xlab = "")
    text(70, 18, paste("Mean = ", round(mean(newSteps$steps), 2), "\n Median = ", 
                     round(median(newSteps$steps), 2)))
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

It appears that replacing the NA's with the mean for their 5-minute interval has had only a minor effect upon the data since both the mean and median have not changed.  

## Are there differences in activity patterns between weekdays and weekends?
There are clear differences in activity patterns between weekdays and weekends.  This can be seen from the following plot:

```r
    #Add column to identify weekday/weekend
    newData$day<-"Weekday"
    newData$day[weekdays(newData$time) %in% c("Saturday", "Sunday")] <- "Weekend"

    #Summarise and plot data
    newIntervals <- aggregate(steps ~ day * interval, data = newData, FUN = mean)
    xyplot(steps ~ interval | day, newIntervals, layout = c(1, 2), type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 
    
