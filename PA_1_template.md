---
title: 'Reproducible Research: Peer Assessment 1'
author: 'Farina Fayyz'
date: '10th Febraury, 2024'
output:
  html_document:
    keep_md: true
  pdf_document: default
---

**Repository:** Explore the GitHub repository for the complete specialization: [Data Science Coursera](https://github.com/FarinaFayyaz/datasciencecoursera)
## Introduction  
The advent of activity monitoring devices, such as Fitbit, Nike Fuelband, or Jawbone Up, has enabled the collection of extensive personal movement data. These devices are integral to the \"quantified self\" movement, where individuals regularly measure themselves to enhance health, identify behavioral patterns, or simply indulge their tech enthusiasm. Despite the potential insights, the raw data's complexity and a dearth of statistical methods and software for analysis hinder the full utilization of this data.  
This project utilizes data from a personal activity monitoring device, capturing information at 5-minute intervals throughout the day. The dataset spans two months (October and November 2012) from an anonymous individual, presenting the count of steps taken during these intervals.  
The dataset is available for download from the course website:\n",
## Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)   
"The dataset includes the following variables:
    "- `steps`: Number of steps taken in a 5-minute interval (missing values denoted as 𝙽𝙰)  
    "- `date`: Date of measurement in YYYY-MM-DD format  
    "- `interval`: Identifier for the 5-minute interval of the measurement  
    "The dataset is stored in a comma-separated-value (CSV) file, comprising 17,568 observations.  
 ## Loading and Preprocessing Data  
 Unzip the data to obtain a CSV file.  
```{r}
if (!dir.exists("Data")) {
dir.create("Data")
}
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",destfile = "Data/archive.zip")
unzip("Data/archive.zip", exdir = "Data")
```
Read the CSV data into a Data.Table.
```{r}
activityDT <- data.table::fread(input = "Data/activity.csv")
```
## Daily Steps Analysis
1. Calculate the total number of steps taken per day.
```{r}
Total_Steps <- activityDT[, c(lapply(.SD, sum, na.rm = FALSE)), .SDcols = c("steps"), by = .(date)] 

head(Total_Steps, 10)
```
2. Create a histogram of the total number of steps taken each day.
```{r}
library(ggplot2)
ggplot(Total_Steps, aes(x = steps))+
        geom_histogram(fill = "blue", binwidth = 1000) +
        labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```
3. Calculate and report the mean and median of the total number of steps taken per day."
```{r}
Total_Steps[, .(Mean_Steps = mean(steps, na.rm = TRUE), Median_Steps = median(steps, na.rm = TRUE))]
```
## Average Daily Activity Pattern
1. Generate a time series plot of the 5-minute interval (x-axis) and the average number of steps taken across all days (y-axis)
```{r}
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)]
ggplot(IntervalDT, aes(x = interval , y = steps)) + 
geom_line(color="blue", size=1) + 
labs(title = "Avg. Daily Steps", x = "Interval", y = "Avg. Steps per day")
```
2. Identify the 5-minute interval, on average across all days, with the maximum number of steps.
```{r}
IntervalDT[steps == max(steps), .(max_interval = interval)]
```
## Imputing Missing Values
1. Calculate and report the total number of missing values in the dataset.
```{r}
activityDT[is.na(steps), .N ]
```
2. Devise a strategy for filling in missing values. For simplicity, fill in missing values with the median of the dataset."
```{r}
activityDT[is.na(steps), "steps"] <- activityDT[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]
```
3. Create a new dataset with missing data filled in.
```{r}
data.table::fwrite(x = activityDT, file = "data/tidyData.csv", quote = FALSE)
```
4. Compare the histogram of total steps taken each day and report the mean and median. Assess the impact of imputing missing data on estimates.
```{r}
Total_Steps <- activityDT[, c(lapply(.SD, sum)), .SDcols = c("steps"), by = .(date)] 
Total_Steps[, .(Mean_Steps = mean(steps), Median_Steps = median(steps))]
ggplot(Total_Steps, aes(x = steps)) + 
geom_histogram(fill = "blue", binwidth = 1000) + 
 labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```

## Differences in Activity Patterns: Weekdays vs. Weekends
1. Create a new factor variable indicating whether a given date is a weekday or weekend.
```{r}
activityDT <- data.table::fread(input = "data/activity.csv")
activityDT[, date := as.POSIXct(date, format = "%Y-%m-%d")]
activityDT[, `Day of Week`:= weekdays(x = date)]
activityDT[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `Day of Week`), "weekday or weekend"] <- "weekday"
activityDT[grepl(pattern = "Saturday|Sunday", x = `Day of Week`), "weekday or weekend"] <- "weekend"
activityDT[, `weekday or weekend` := as.factor(`weekday or weekend`)]
head(activityDT, 10)
```
2. Generate a panel plot of the time series of the 5-minute interval (x-axis) and the average number of steps, grouped by weekdays and weekends 
```{r}
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval, `weekday or weekend`)]
ggplot(IntervalDT , aes(x = interval , y = steps, color=`weekday or weekend`)) +     
geom_line() +
labs(title = "Avg. Daily Steps by Weektype", x = "Interval", y = "No. of Steps") +
facet_wrap(~`weekday or weekend` , ncol = 1, nrow=2)
