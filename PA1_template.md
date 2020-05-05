#Loading and preprocessing the data

if(!file.exists("getdata-projectfiles-UCI HAR Dataset.zip")) {
  temp <- tempfile()
  download.file("http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
  unzip(temp)
  unlink(temp)
}
data <- read.csv("activity.csv")

#What is mean total number of steps taken per day? The mean and median of the total number of steps taken per day

steps_by_day <- aggregate(steps ~ date, data, sum)
hist(steps_by_day$steps, main = paste("Total Steps Per Day"), col="orange", xlab="Number of Steps")
rmean <- mean(steps_by_day$steps)
rmedian <- median(steps_by_day$steps)
The Mean = 10766.19 and The medina = 10765

#What is the average daily activity pattern?

Calculate average steps for each interval for all days.
Plot the Average Number Steps per Day by Interval.
Find interval with most average steps.
steps_by_interval <- aggregate(steps ~ interval, data, mean)
plot(steps_by_interval$interval,steps_by_interval$steps, type="l", xlab="Interval", ylab="No. of Steps",main="Avg. No. of Steps per Day by Interval" )
max_interval <- steps_by_interval[which.max(steps_by_interval$steps),1]
5-minute interval, contains the maximum number of steps = 835

#Impute missing values. Compare imputed to non-imputed data.

Missing values were imputed with by inserting the average for each interval. for instance, if interval 10 was missing on 10-02-2012, the average for that interval for all days (0.1320755), replaced the NA.

incomplete <- sum(!complete.cases(data))
imputed_data <- transform(data, steps = ifelse(is.na(data$steps), steps_by_interval$steps[match(data$interval, steps_by_interval$interval)], data$steps))
The first day (10-01-2012) was imputed with 0 as it would will be over 9,000 steps higher than the following day, which had only 126 steps.

imputed_data[as.character(imputed_data$date) == "2012-10-01", 1] <- 0

Recount total steps by day and create Histogram.

steps_by_day_i <- aggregate(steps ~ date, imputed_data, sum)
hist(steps_by_day_i$steps, main = paste("Total Steps Each Day"), col="red", xlab="Number of Steps")

hist(steps_by_day$steps, main = paste("Total Steps Each Day"), col="orange", xlab="Number of Steps", add=T)
legend("topright", c("Imputed", "Non-imputed"), col=c("red", "orange"), lwd=10)

Create Histogram to show difference.

Calculate new mean and median for imputed data.

rmean.i <- mean(steps_by_day_i$steps)
rmedian.i <- median(steps_by_day_i$steps)

Calculate difference between imputed and non-imputed data.

mean_diff <- rmean.i - rmean
med_diff <- rmedian.i - rmedian

Calculate total difference.

total_diff <- sum(steps_by_day_i$steps) - sum(steps_by_day$steps)

*The imputed data mean is 1.059 × 104 *
The imputed data median is 1.0766 × 104 

*The difference between the non-imputed mean and imputed mean is -176.4949 *
The difference between the non-imputed mean and imputed mean is 1.1887 *The difference between total number of steps between imputed and non-imputed data is 7.5363 × 104. Thus, there were 7.5363 × 104 more steps in the imputed data. 

#Are there differences in activity patterns between weekdays and weekends?

Created a plot to compare and contrast number of steps between the week and weekend. There is a higher peak earlier on weekdays, and more overall activity on weekends.

weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday" )
imputed_data$dow = as.factor(ifelse(is.element(weekdays(as.Date(imputed_data$date)),weekdays), "Weekday", "Weekend"))
steps_by_interval_i <- aggregate(steps ~ interval + dow, imputed_data, mean)
library(lattice)
xyplot(steps_by_interval_i$steps ~ steps_by_interval_i$interval|steps_by_interval_i$dow, main="Avg. Steps per Day by Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l") 
