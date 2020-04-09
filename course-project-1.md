This is the documentation for Course Project 1 for the online course
**Reproducible Research** offered by *Johns Hopkins University* on
Coursera.  
You can find the project details in the Readme file.

### Step 1: Loading and Preprocessing the Data

Let’s load the data from the link and unzip it.

    download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", "activity.zip", method = "curl")
    dataset <- read.csv(unzip("activity.zip"))

Here’s a basic overview of the dataset.

    str(dataset)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

Let’s also process the dates by converting them using the *lubridate*
package.

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    dataset$date <- ymd(dataset$date)

### Step 2: Analysis of the number of steps taken

First, we calculate the total number of steps per day.

    dailysteps <- tapply(dataset$steps, dataset$date, sum, na.rm = TRUE)

We then calculate the mean and median steps taken per day.

    print(paste("Mean steps per day:", (round(mean(dailysteps, na.rm = TRUE), digits = 2))))

    ## [1] "Mean steps per day: 9354.23"

    print(paste("Median steps per day:", (round(median(dailysteps, na.rm = TRUE), digits = 2))))

    ## [1] "Median steps per day: 10395"

Here is a histogram plot of the number of steps per day.

    hist(dailysteps, main = "Histogram of steps per day", xlab = "Total steps per day", ylab = "Number of days", col = "skyblue", ylim = c(0, 40))

![](course-project-1_files/figure-markdown_strict/unnamed-chunk-6-1.png)

### Step 3: Average Daily Activity Pattern

The time interval with the maximum steps per day, on an average is given
by:

    stepstime <- tapply(dataset$steps, dataset$interval, mean, na.rm=TRUE)
    print(paste(names(which(stepstime == max(stepstime))), "with", max(stepstime), "steps on an average"))

    ## [1] "835 with 206.169811320755 steps on an average"

Here is a time series plot of the average daily activity trend.

    plot(names(stepstime), stepstime, type='l', main = "Daily Activity Trend", xlab = "Time", ylab = "Average steps", ylim = c(0, 250))

![](course-project-1_files/figure-markdown_strict/unnamed-chunk-8-1.png)

### Step 4: Imputing Missing Values

    print(paste("Number of missing values:", sum(is.na(dataset))))

    ## [1] "Number of missing values: 2304"

To impute the missing values, we will use the mean steps during that
particular time interval over the entire dataset.

    dataset1 <- dataset
    dataset1 <- cbind(dataset, stepstime)

    ## Warning in data.frame(..., check.names = FALSE): row names were found from a
    ## short variable and have been discarded

    for(i in 1:length(dataset1$steps)){
      if(is.na(dataset1$steps[i])==TRUE){
        dataset1$steps[i] <- dataset1$stepstime[i] 
      }
    }

Let us attempt to analyze this new dataset. First, we calculate the
total number of steps per day.

    dailysteps1 <- tapply(dataset1$steps, dataset1$date, sum, na.rm = TRUE)

We then calculate the mean and median steps taken per day.

    print(paste("Mean steps per day:", (round(mean(dailysteps1, na.rm = TRUE), digits = 2))))

    ## [1] "Mean steps per day: 10766.19"

    print(paste("Median steps per day:", (round(median(dailysteps1, na.rm = TRUE), digits = 2))))

    ## [1] "Median steps per day: 10766.19"

We observe an increase in the mean and median steps per day. This occurs
because the NA values are replaced by average imputed values.  
Here is a histogram plot of the number of steps per day.

    hist(dailysteps1, main = "Histogram of steps per day with imputed data", xlab = "Total steps per day", ylab = "Number of days", col = "skyblue", ylim = c(0, 40))

![](course-project-1_files/figure-markdown_strict/unnamed-chunk-13-1.png)

### Step 5: Weekends vs Weekdays

Let us compare the daily activity trend for weekends and weekdays.

    library(chron)

    ## NOTE: The default cutoff when expanding a 2-digit year
    ## to a 4-digit year will change from 30 to 69 by Aug 2020
    ## (as for Date and POSIXct in base R.)

    ## 
    ## Attaching package: 'chron'

    ## The following objects are masked from 'package:lubridate':
    ## 
    ##     days, hours, minutes, seconds, years

    dataset2 <- dataset1
    isweekend <- is.weekend(dataset2$date)
    dataset2 <- cbind(dataset2, isweekend)
    dataset3 <- dataset2
    for(i in 17568:1){
      if(dataset2$isweekend[i]==TRUE){
        dataset2 <- dataset2[-i, ]
      }
    }
    for(i in 17568:1){
      if(dataset3$isweekend[i]==FALSE){
        dataset3 <- dataset3[-i, ]
      }
    }
    stepstime2 <- tapply(dataset2$steps, dataset2$interval, mean, na.rm=TRUE)
    stepstime3 <- tapply(dataset3$steps, dataset3$interval, mean, na.rm=TRUE)
    plot(names(stepstime2), stepstime2, main = "Weekday Activity", type = "l", xlab = "Time", ylab = "Average steps")

![](course-project-1_files/figure-markdown_strict/unnamed-chunk-14-1.png)

    plot(names(stepstime3), stepstime3, main = "Weekend Activity", type = "l", xlab = "Time", ylab = "Average steps")

![](course-project-1_files/figure-markdown_strict/unnamed-chunk-14-2.png)
