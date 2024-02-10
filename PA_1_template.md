**Reproducible Research Project 1**

*Author:* Farina Fayyaz
*Date:* 10 Febraury, 2024.

**Introduction**

With the help of activity monitoring devices like Fitbits, Nike Fuelbands, and Jawbone Ups, we can now gather a vast amount of data on our personal movements. These devices are part of the "quantified self" movement, which consists of individuals who regularly track their health metrics to enhance their well-being, discover patterns in their behavior, or simply because they're tech-savvy. However, this data remains underutilized due to the difficulty of obtaining raw data and the lack of statistical methods and software for processing and interpreting it.

This project makes use of data from an anonymous individual's personal activity monitoring device, which collected data at 5-minute intervals throughout two months (October and November 2012). The data consists of the number of steps taken in each 5-minute interval.

**Loading and Preprocessing the Data**

1. Unzip the data to obtain a CSV file.

```R
library(data.table)
library(ggplot2)

fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = paste0(getwd(), '/repdata%2Fdata%2Factivity.zip'), method = "curl")
unzip("repdata%2Fdata%2Factivity.zip",exdir = "data")
```

2. Read the CSV data into a Data.Table.

```R
activityDT <- data.table::fread(input = "data/activity.csv")
```

**What is the Mean Total Number of Steps Taken per Day?**

1. Calculate the total number of steps taken per day.

```R
Total_Steps <- activityDT[, c(lapply(.SD, sum, na.rm = FALSE)), .SDcols = c("steps"), by = .(date)] 

head(Total_Steps, 10)
```

2. Create a histogram to visualize the distribution of daily steps. If you're unfamiliar with the distinction between histograms and barplots, explore the difference before proceeding.

```R
ggplot(Total_Steps, aes(x = steps)) +
    geom_histogram(fill = "blue", binwidth = 1000) +
    labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```

3. Calculate and report the mean and median of the total number of steps taken per day.

```R
Total_Steps[, .(Mean_Steps = mean(steps, na.rm = TRUE), Median_Steps = median(steps, na.rm = TRUE))]
```

**What is the Average Daily Activity Pattern?**

1. Generate a time series plot showing the average number of steps taken at each 5-minute interval across all days.

```R
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)] 

ggplot(IntervalDT, aes(x = interval , y = steps)) + geom_line(color="blue", size=1) + labs(title = "Avg. Daily Steps", x = "Interval", y = "Avg. Steps per day")
```

2. Identify the 5-minute interval with the highest average number of steps taken across all days.

```R
IntervalDT[steps == max(steps), .(max_interval = interval)]
```

**Imputing Missing Values**

1. Determine the total number of missing values (rows with NAs) in the dataset.

```R
activityDT[is.na(steps), .N ]

# alternative solution
nrow(activityDT[is.na(steps),])
```

2. Develop a strategy to fill in the missing values. This could involve using the mean/median for that day, the mean for that 5-minute interval, or another method.

```R
# Filling in missing values with median of dataset. 
activityDT[is.na(steps), "steps"] <- activityDT[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]
```

3. Create a new dataset with the missing values filled in.

```R
data.table::fwrite(x = activityDT, file = "data/tidyData.csv", quote = FALSE)
```

4. Create a histogram of the total number of steps taken per day and calculate/report the mean and median. Compare these values to the estimates from before