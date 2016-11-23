# PA1_template
Joe Larson  
November 23, 2016  



```r
knitr::opts_chunk$set(echo = TRUE, include = TRUE, message = TRUE, warning = FALSE, fig.keep = TRUE, results = TRUE)
```

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

This was redone trying to mostly(80%) datatable and ggplots.


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


```r
### Make data a datatable
datatable <- data.table(data)

### Setting labels to caps
names(datatable) <- c("Steps", "Date", "Interval")

### Needed to aid in doing datatable math faster, ordered data by Date
setkey(datatable, Date)

### Using is Weekday function to set a logic answer, TRUE = Weekday, FALSE = Weekend
datatableday <- datatable[, c("Logic") :=isWeekday(datatable$Date, wday=1:5)]# TRUE = Weekday, FALSE= Weekend

### Adding a column of data to "Day" and set based on Logic test
datatableday <- datatableday[Logic==TRUE, Day:= "Weekday"]
datatableday <- datatableday[Logic==FALSE, Day:= "Weekend"]

### Calulation the Mean steps by interval, 0 to 288  within any day
datatableNAinterval <- as.data.frame(datatable[, mean(Steps, na.rm = TRUE),by = Interval])
datatableNAinterval <- data.table(datatableNAinterval)
names(datatableNAinterval) <- c("Interval", "Steps")

### Using SQL short cuts to merge two datatables with Interval as the key field
setkey(datatableday, Interval)
setkey(datatableNAinterval, Interval)

### This keeps the SQL column order 
leftCols <- colnames(datatableday)
rightCols <- colnames(datatableNAinterval)
rightCols <- setdiff(rightCols,key(datatableNAinterval))

### This merged data table that has NA and a mean value for steps - six columns
datatabledaynoNA <- merge(datatableday,datatableNAinterval, all.datatableday=TRUE)
setkey(datatabledaynoNA, Interval)

### This is using the Mean steps for interval data set to select and replace NA values, for the full data set stored in FinStpes.  The order the replacement occurs matters as well as using the >= function.  Getting warning due to data type conversions

datatabledaynoNA <- datatabledaynoNA[Steps.y>=0, FinSteps:=0 ]
datatabledaynoNA <- datatabledaynoNA[Steps.y>0, FinSteps:=Steps.y ]
datatabledaynoNA <- datatabledaynoNA[Steps.x==NA, FinSteps:=0]
datatabledaynoNA <- datatabledaynoNA[Steps.x>=0, FinSteps:=Steps.x ]

### counting NAs
nasteps <- sum(is.na(datatable$Steps))
nadate <- sum(is.na(datatable$Date))
nainterval <- sum(is.na(datatable$Interval))

### counting NAs
nonasteps <- sum(is.na(datatabledaynoNA$FinSteps))
nonadate <- sum(is.na(datatabledaynoNA$Date))
nonainterval <- sum(is.na(datatabledaynoNA$Interval))

### Calulation the Mean steps with NA replaced by mean value by interval, 0 to 288  within any day
datatableNAintervalnoNA <- as.data.frame(datatabledaynoNA[, mean(FinSteps, na.rm = TRUE),by = Interval])
datatableNAintervalnoNA <- data.table(datatableNAinterval)
names(datatableNAintervalnoNA) <- c("Interval", "Steps")

### Print NA results
print(paste0("The count of NAs in 'Steps is ", nasteps, ", the count of NAs in 'Date' is ",nadate, ", the count of NAs in 'Interval' is ",nainterval," for a total of ",sum(nasteps + nadate + nainterval),"." ))
```

```
## [1] "The count of NAs in 'Steps is 2304, the count of NAs in 'Date' is 0, the count of NAs in 'Interval' is 0 for a total of 2304."
```

```r
### Print NA results
print(paste0("The count of NAs with NAs replaced is in 'Steps is ", nonasteps, ", the count of NAs in 'Date' is ", nonadate, ", the count of NAs in 'Interval' is ",nonainterval," for a total of ",sum(nonasteps + nonadate + nonainterval),"." ))
```

```
## [1] "The count of NAs with NAs replaced is in 'Steps is 0, the count of NAs in 'Date' is 0, the count of NAs in 'Interval' is 0 for a total of 0."
```

```r
### This takes the datatable and groups by Date and sums the Step with NA's removed crates a datasum datatable without column labels, NA have to be removed to have 'Mean' and 'Median' calculated
datasum <- as.data.frame(datatable[, sum(Steps, na.rm = TRUE),by = Date])
names(datasum) <- c("Date", "Steps")  # adding cloumn lables

datasumnoNA <- as.data.frame(datatabledaynoNA[, sum(FinSteps, na.rm = TRUE),by = Date])
names(datasumnoNA) <- c("Date", "Steps")  # adding cloumn lables


datamean <- mean(datasum$Steps)
datamedian <- median(datasum$Steps)

datameannoNA <- mean(datasumnoNA$FinSteps)
datamediannoNA <- median(datasumnoNA$FinSteps)

print(paste0("The 'Mean' step value with NAs is ", round(datamean, digits = 0), " and the 'Meadian' step value with NAs is ", datamedian))
```

```
## [1] "The 'Mean' step value with NAs is 9354 and the 'Meadian' step value with NAs is 10395"
```

```r
print(paste0("The 'Mean' step value with NAs is ", round(datameannoNA, digits = 0), " and the 'Meadian' step value with NAs is ", datamediannoNA))
```

```
## [1] "The 'Mean' step value with NAs is NA and the 'Meadian' step value with NAs is "
```
### Daily Steps Plot

```r
### Histogram Plot
graphTitle <- "Frequency plot of daily steps with NAs"
xlabel <- "Daily steps (in interval of 1,000)"
hist(datasum$Steps, breaks = 20, col = "light blue", xlab = xlabel, main =graphTitle)
```

![](PA1_template_files/figure-html/Histogram2-1.png)<!-- -->

```r
dailystepswithna <- ggplot(datasum, aes(x=substr(datasum$Date, 6, 12), y=Steps))+      geom_bar(stat = "identity")+theme(axis.text.x=element_text(angle=-90, vjust=0.4,hjust=1))+ labs(x="Date")+ ggtitle("        Total number of steps taken each day with NA's")
plot(dailystepswithna)
```

![](PA1_template_files/figure-html/Histogram2-2.png)<!-- -->

```r
### Set up datatable with out NAs

datatableNAinterval <- as.data.frame(datatable[, mean(Steps, na.rm = TRUE),by = Interval])
datatableNAinterval <- data.table(datatableNAinterval)
names(datatableNAinterval) <- c("Interval", "Steps")

datamaxinterval <- datatableNAinterval[which(datatableNAinterval$Steps == max(datatableNAinterval$Steps)),]

print(paste0("Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? The answer is interval : ", datamaxinterval$Interval," which has an avgerage step value ", trunc(datamaxinterval$Steps,digits = 0),"." ))
```

```
## [1] "Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? The answer is interval : 835 which has an avgerage step value 206."
```
### Time Series Plot  

```r
timeseriesinterval <- ggplot(datatableNAinterval, aes(x=datatableNAinterval$Interval, y=Steps))+                     geom_line()+ theme(axis.text.x=element_text(angle=-90))+                             labs(x="Interval")+                                                                  ggtitle("                  Time series plot of steps taken each day with NA's")
plot(timeseriesinterval)
```

![](PA1_template_files/figure-html/Time Series-1.png)<!-- -->

```r
### With NA replaced with avg step per interval - Imputing missing values
datatabledaynoNA2 <- melt(data = datatabledaynoNA,id.vars="Interval", measure.vars="FinSteps")

cat("Test total number of missing values is equal to =",sum(is.na(datatabledaynoNA$FinSteps)))
```

```
## Test total number of missing values is equal to = 0
```
### Histogram Plot without NA

```r
histogramsteps <- ggplot(datatabledaynoNA, aes(x=substr(datatabledaynoNA$Date, 6, 12), y=FinSteps))+geom_bar(stat = "identity")+theme(axis.text.x=element_text(angle=-90,vjust=0.4, hjust=1))+labs(x="Date")+ggtitle("        Total number of steps taken each day without NA's")
plot(histogramsteps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
### Histogram Plot
graphTitle <- "Frequency plot of daily steps without NAs"
xlabel <- "Daily steps (in interval of 1,000)"
hist(datasumnoNA$Steps, breaks = 20, col = "red", xlab = xlabel, main =graphTitle)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-2.png)<!-- -->

```
## [1] "The count of NAs in steps is 0, the count of NAs in date is 0, the count of NAs in interval is 0 for a total of 0. Zeros are  the desired answer."
```

```
## [1] "The 'Mean' step value with NAs is 10766 and the 'Meadian' step value with NAs is 10766"
```
### Interval comparsion Weekday vs Weekend


```r
par(mfrow=c(2,1))
par(mar=c(2.1,2.1,2.1,1.1))

daytype <- function(date) {
        if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
                "Weekend"
        } else {
                "Weekday"
        }
}
data$daytype <- as.factor(sapply(data$date, daytype))

for (type in c("Weekend", "Weekday")) {
        steps.type <- aggregate(steps ~ interval, data = data, subset = data$daytype == 
                                        type, FUN = mean)
        plot(steps.type, type = "l", main = type)
}
```

![](PA1_template_files/figure-html/Weekend vs Weekday-1.png)<!-- -->
