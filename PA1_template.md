# PA1_template
Joe Larson  
November 20, 2016  


```r
knitr::opts_chunk$set(echo = TRUE, include = TRUE, message = TRUE, warning = FALSE, fig.keep = TRUE, results = TRUE)
```

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:


### If the data set has not been download, download now


```r
if(!file.exists("activity.csv"))  {           
        temp <- tempfile()
        download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", temp)
        unzip(zipfile = temp, "activity.csv")
        datedownload <- Sys.time() ## record the download date of the zip file
}

datedownload <- file.mtime("activity.csv")
data <- read.csv(file="activity.csv",header=TRUE)
```

```
## [1] "The count of NAs in steps is 2304, the count of NAs in date is 0, the count of NAs in interval is 0 for a total of 2304."
```

```
## [1] "The mean step value with NAs is 9354 and the meadian step value with NAs is 10395"
```
### Histogram Plot

```r
histogramsteps <- ggplot(datasum, aes(x=substr(datasum$Date, 6, 12), y=Steps))+ geom_bar(stat = "identity")+theme(axis.text.x=element_text(angle=-90, vjust=0.4,hjust=1))+ labs(x="Date")+ ggtitle("        Total number of steps taken each day with NA's")  
```
![](PA1_template_files/figure-html/Histogram-1.png)<!-- -->

```
## [1] "Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? The answer is interval : 835 which has an avgerage step value 206."
```
### Time Series Plot   

```r
timeseriesinterval <- ggplot(datasumnoNA, aes(x=datasumnoNA$Interval, y=Steps))+                     geom_line()+ theme(axis.text.x=element_text(angle=-90))+                             labs(x="Interval")+                                                                  ggtitle("                  Time series plot of steps taken each day with NA's")
plot(timeseriesinterval)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
### With NA replaced with avg step per interval
no_na_data <- data
na_index <- which(is.na(no_na_data$steps))
data_no_na <- melt(data = data,id.vars="interval", measure.vars="steps", na.rm= TRUE)
interval_data <- dcast(data_no_na, interval ~ variable, mean)

for (counter in na_index) {
        step_temp <- no_na_data$interval[counter]
        index <- which(interval_data$interval == step_temp)
        no_na_data$steps[counter] <- interval_data$steps[index]
}

cat("Test total number of missing values is equal to =",sum(is.na(no_na_data$steps)))
```

```
## Test total number of missing values is equal to = 0
```
### Histogram Plot without NA

```r
histogramsteps <- ggplot(no_na_data, aes(x=substr(no_na_data$date, 6, 12), y=steps))+geom_bar(stat = "identity")+theme(axis.text.x=element_text(angle=-90,vjust=0.4, hjust=1))+labs(x="Date")+ggtitle("        Total number of steps taken each day without NA's")              
```
![](PA1_template_files/figure-html/Historgram without NA-1.png)<!-- -->

```
## [1] "The count of NAs in steps is 0, the count of NAs in date is 0, the count of NAs in interval is 0 for a total of 0. Zeros are  the desired answer."
```

```
## [1] "The mean step value with NAs is 10766 and the meadian step value with NAs is 10766"
```
### Interval comparsion Weekday vs Weekend

![](PA1_template_files/figure-html/Weekend vs Weekday-1.png)<!-- -->
