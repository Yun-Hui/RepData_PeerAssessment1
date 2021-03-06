Loading and preprocessing the data
----------------------------------

    if(!file.exists("getdata-projectfiles-UCI HAR Dataset.zip")) {
            temp <- tempfile()
            download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
            unzip(temp)
            unlink(temp)}

    df<-read.csv(file='activity.csv')

Part I: What is mean total number of steps taken per day?
---------------------------------------------------------

#### 1.Calculate the total number of steps taken per day

    df_s_d<-aggregate(df$steps, list(df$date), sum, na.rm=TRUE)
    names(df_s_d)=c("date","total")

#### 2.Make a histogram of the total number of steps taken each day.Calculate and report the mean and median of the total number of steps taken per day

    qplot(df_s_d$total, geom="histogram",binwidth = 1000,xlab="total number of steps taken each day") 

![](figure-markdown_strict/hisgram-1.png)

    mean<-round(mean(df_s_d$total),digits=2)
    median<-median(df_s_d$total)

**The mean and median of the total number of steps taken per day is
9354.23 and 10395 respectively.**

Part II: What is the average daily activity pattern?
----------------------------------------------------

#### 1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

    df_interval<-aggregate(df$steps, list(df$interval), mean, na.rm=TRUE)
    names(df_interval)=c("interval","steps")

    plot(df_interval$interval, df_interval$steps, 
         type="l",
         xlab="Interval",
         ylab="Average steps taken",
         main="Time series - Average steps taken during 5 minute interval")

![](figure-markdown_strict/interval%20-1.png)

#### 2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

    max<-df_interval$interval[which.max(df_interval$steps)]

**The 5 minutes interval that contains the maximum number of steps? is
835.**

Part III: Imputing missing values
---------------------------------

#### 1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA's)

    missing<-sum(rowSums(is.na(df)))

**There are a total of 2304 missing value in the dataset.**

#### 2.Devise a strategy for filling in all of the missing values in the datase: I use mean steps of the whole dataset to fill the missing values.New dataset called d2 was created impute value.

    df2<-df
    df2[is.na(df2)]<- mean(df2$steps,na.rm=TRUE) 
    df2$steps<-as.integer(df2$steps)

#### 3.Create a Histogram with new dataset

    df2_agg<-aggregate(df2$steps, by=list(df2$date), sum, na.rm=TRUE)
    names(df2_agg)=c("date","steps")
    qplot(df2_agg$steps, geom="histogram",binwidth = 1000,xlab="total number of steps taken each day -impute missing steps") 

![](figure-markdown_strict/histogram_Nomissing-1.png)

#### 4.Report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

    mean2<-mean(df2_agg$steps)
    median2<-median(df2_agg$steps)

After the impute missing value, the new average steps is
1.075173810^{4}, which is **higher** than the original average
(9354.23). The new median after imputing missing value is 10656, higher
than the original median (10395). Note in the histogram, the count of
zero steps is much lower than the previoud Histogram because in the
first one, missing value was treated as zero. Filling missing value
would lead to the decreasing of zero steps

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

#### 1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

    df2$weekday <- weekdays(as.Date(df2$date))
    weekday_hc <- c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday')
    df2$wknd <- ifelse(df2$weekday %in% weekday_hc, 'Weekday', 'Weekend')

#### 2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5- minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

    df2_agg<-aggregate(df2$steps, by=list(df2$wknd,df2$weekday,df2$interval), mean, na.rm=TRUE)
    names(df2_agg)<-c("weekend",'weekday',"interval","steps")
    p<-ggplot(df2_agg,aes(interval,steps))
    p+geom_line()+facet_wrap(~weekend, nrow = 2)+labs(x="Interval",y="Number of Steps")

![](figure-markdown_strict/time%20series%20chart-1.png)

