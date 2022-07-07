## Bellabeats Case Study R Markdown Data Documentation

### Markdown

---
title: "Bellabeats Case Study - R Markdown"
author: "Estelle Martin"
date: '2022-07-03'
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

#### To complete the Phase 3 process of the analysis I chose to use R to learn more about the dataset.

### **Installing various packages:**


```{r}
install.packages("tidyverse")

install.packages("ggplot2")

install.packages("skimr")

install.packages("janitor")

install.packages("here")

install.packages("dplyr")
```


```{r}
library("tidyverse")
library("ggplot2")
library("skimr")
library("janitor")
library("here")
library("dplyr")
library("lubridate")
library("readr")
```


### **Importing the Bellabeats merged data set and creating data frames:**
#### [The daily_activity file has already been cleaned in Google Sheets.]


```{r}
daily_activity <- read_csv("dailyActivity_Edited.csv")

sleep_day <- read_csv("sleepDay_Edited.csv")

daily_intensities <- read_csv("dailyIntensities_Edited.csv")

weight_log <- read_csv("weightLogInfo_Edited.csv")
```

### **Learning more about the data:**

```{r}
str(daily_activity)
str(sleep_day)
str(daily_intensities)
str(weight_log)
```


## **Comparing data sets:**
### Daily activity entries VS daily sleep VS daily intensities VS weight log entries:


```{r}
daily_activity_nrow <- nrow(daily_activity)
sleep_day_nrow <- nrow(sleep_day)
daily_intensities_nrow <- nrow(daily_intensities)
weight_log_nrow <- nrow(weight_log)

data.frame(daily_activity_nrow, sleep_day_nrow, daily_intensities_nrow, weight_log_nrow)
```

##### *This is also achievable by looking at the data in the environment.*


## **Extra data cleaning steps in R for daily_activity dataset:**

### Renaming 'ActivityDate' column to 'Date'

```{r}
colnames(daily_activity)[colnames(daily_activity) == "ActivityDate"] <- "Date"
```

### Changing 'Date' data type from character to date type

```{r}
daily_activity$Date <-  as.Date(daily_activity$Date,format='%m/%d/%Y')
```

### Changing 'SedentaryMinutes' data type from character to numeric

```{r}
daily_activity$SedentaryMinutes <- suppressWarnings(as.numeric(daily_activity$SedentaryMinutes))
```

### Adding a 'Weekdays' column for future analysis

```{r}
daily_activity$Weekdays<- weekdays(daily_activity$Date, abbr = FALSE)
```



## **Extra data cleaning steps in R for sleep_day dataset:**

### Renaming 'SleepDay' column to 'Date'

```{r}
colnames(sleep_day)[colnames(sleep_day) == "SleepDay"] <- "Date"
```

### Changing 'Date' data type from character to date type

```{r}
sleep_day$Date <-  as.Date(sleep_day$Date,format='%m/%d/%Y')
```

### Adding a 'Weekdays' column for future analysis

```{r}
sleep_day$Weekdays<- weekdays(sleep_day$Date, abbr = FALSE)
```



## **Extra data cleaning steps in R for weight_log dataset:**

### Changing 'Date' data type from character to date type

```{r}
weight_log$Date <-  as.Date(weight_log$Date,format='%m/%d/%Y')
```

### Adding a 'Weekdays' column for future analysis

```{r}
weight_log$Weekdays<- weekdays(weight_log$Date, abbr = FALSE)
```


## **Extra data cleaning steps in R for daily_intensities dataset:**

### Renaming 'ActivityDay' column to 'Date'

```{r}
colnames(daily_intensities)[colnames(daily_intensities) == "ActivityDay"] <- "Date"
```

### Changing 'Date' data type from character to date type

```{r}
daily_intensities$Date <-  as.Date(daily_intensities$Date,format='%m/%d/%Y')
```

### Adding a 'Weekdays' column for future analysis

```{r}
daily_intensities$Weekdays<- weekdays(daily_intensities$Date, abbr = FALSE)
```


## **Insights into our cleaned tables:**


```{r}
head(daily_activity)
head(sleep_day)
head(daily_intensities)
head(weight_log)
```


## **Creating summaries of our data for analysis:**

### Summary of daily_activity dataset:


```{r}
daily_activity %>%  
  select(TotalSteps,
         TotalDistance,
         Calories,
         VeryActiveMinutes,
         FairlyActiveMinutes,
         LightlyActiveMinutes) %>%
  summary()
```

### Summary of sleep_day dataset:


```{r}
sleep_day %>%
  select(TotalSleepRecords,
         TotalMinutesAsleep,
         TotalTimeInBed) %>%
  summary()
```

### Summary of daily_intensities dataset:


```{r}
daily_intensities %>%
  select(Date,
         SedentaryMinutes,
         LightlyActiveMinutes,
         FairlyActiveMinutes,
         VeryActiveMinutes,
         SedentaryActiveDistance,
         LightActiveDistance,
         ModeratelyActiveDistance,
         VeryActiveDistance) %>%
  summary()
```

### Summary of weight_log dataset:


```{r}
weight_log %>%
  select(Date,
         WeightKg,
         WeightPounds,
         Fat,
         BMI,
         IsManualReport) %>%
  summary()
```

## **Establishing relationships in the datasets:**

### Relationship between entries and data types:

```{r}
record_type <- c("Daily Activities","Daily Intensities","Daily Sleep","Daily Weight")
record_unique_IDs <- c(n_distinct(daily_activity$Id),
                       n_distinct(daily_intensities$Id),
                       n_distinct(sleep_day$Id),
                       n_distinct(weight_log$Id))
record_number <- data.frame(record_type,record_unique_IDs)
```


```{r}
ggplot(record_number, aes(record_type, record_unique_IDs)) +
  geom_bar(fill="pink", stat="identity") +
  geom_text(aes(label=record_unique_IDs), vjust=3) +
  labs(title="Total Number of Records by Data Type", 
       subtitle="For a total of 33 participants", 
       x="Record Types", y="Number of Records") +
  theme(plot.title = element_text(face="bold"))
```

### Relationship between total steps and calories burned:


```{r}
ggplot(daily_activity, aes(x=TotalSteps, y=Calories, color = "orange")) +
  geom_point(size=1.25) +
  geom_smooth(method=lm, color="purple") +
  labs(title="Calories Burned VS Total Steps Per Day", 
       subtitle="Data representing 33 Participants", 
       x="Total Steps", y="Calories Burned")+
  theme(legend.position="none") +
  theme(plot.title = element_text(face="bold"))
```


### Relationship between manually entered weight entries VS not:


```{r}
report_type <- c("Manual Report","Automatic Report")
report_method <- c(length(which(weight_log$IsManualReport =="TRUE")),
                   length(which(weight_log$IsManualReport =="FALSE")))
report_preference <- data.frame(report_type,report_method)
```


```{r}
ggplot(report_preference, aes(report_type,report_method)) +
  geom_bar(fill="darkslategray2", stat="identity", width = 0.55) + coord_flip() +
  geom_text(aes(label=report_method), vjust=0.55, hjust=2) +
  labs(title="Weight Log Entries Based on Reporting Preferences", 
       subtitle="For a total of 8 participants", 
       x="Report Type", y="Number of Records") +
  theme(plot.title = element_text(face="bold"))
```


## **Calculating people's sleeping patterns during the week:**

### Creating a tibble to only see weekdays and average minutes asleep:


```{r}
sleep_schedule <- sleep_day %>% 
  group_by(Weekdays) %>% 
  summarize(mean_TotalMinutesAsleep = mean(TotalMinutesAsleep))
head(sleep_schedule)
```

### Ordering the weekdays in chronological order for better visibility:


```{r}
sleep_schedule$Weekdays <- factor(sleep_schedule$Weekdays,
                                  levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))
```

### Creating a visualization off the tibble:     


```{r}
ggplot(sleep_schedule, aes(x=Weekdays,y=mean_TotalMinutesAsleep, group=1))+
  geom_line(color = "springgreen2")+
  labs(title="Average Daily Sleep Schedule per Weekday", 
       subtitle="For a total of 24 participants",
       x= "Weekdays", y="Average Minutes Asleep") +
  theme(plot.title = element_text(face="bold"))
```


## **Determining people's daily average intensities:**

### Creating a tibble showing the average daily intensities for each participant:


```{r}
activity_user_type <- daily_intensities %>% 
  group_by(Id) %>% 
  summarize(mean_SedentaryMinutes = mean(SedentaryMinutes),
            mean_LightlyActiveMinutes = mean(LightlyActiveMinutes),
            mean_FairlyActiveMinutes = mean(FairlyActiveMinutes),
            mean_VeryActiveMinutes = mean(VeryActiveMinutes))
head(activity_user_type)
```

### Averaging daily intensities per category type:

```{r}
user_type <- c("Sedentary","Lightly Active","Fairly Active","Very Active")
average_intensity <- c((mean(activity_user_type$mean_SedentaryMinutes, na.rm = TRUE)),
                       mean(activity_user_type$mean_LightlyActiveMinutes, na.rm = TRUE),
                       mean(activity_user_type$mean_FairlyActiveMinutes, na.rm = TRUE),
                       mean(activity_user_type$mean_VeryActiveMinutes, na.rm = TRUE))
intensity_user_type <- data.frame(user_type,average_intensity)
```


### Visualizing Results:
 
```{r}
ggplot(intensity_user_type, aes(x=user_type,y=average_intensity, fill=user_type, group=1))+
  geom_bar(stat="identity")+
  labs(fill="User Types", title="Average Daily Intensity by User Types", 
       subtitle="For a total of 33 participants",
       x= "User Types", y="Average Intensity (mins)") +
  theme(plot.title = element_text(face="bold"))
```


## **Calculating the difference between minutes asleep VS total time in bed:**

### Creating a tibble to visualize the mean of each variable:

```{r}
sleep_user_pref <- sleep_day %>% 
  group_by(Id) %>% 
  summarize(mean_TotalMinutesAsleep = mean(TotalMinutesAsleep),
            mean_TotalTimeInBed = mean(TotalTimeInBed))
head(sleep_user_pref)
```

### Visualizing the relationship between total minutes asleep VS in bed:

```{r}
ggplot(sleep_day, aes(x=TotalMinutesAsleep, y=TotalTimeInBed)) +
  geom_point(size=1.25, color="gold2") +
  geom_smooth(method=lm, color="purple") +
  labs(title="Total Time Asleep VS Total Time In Bed", 
       subtitle="Data representing 24 Participants", 
       x="Total Minutes Asleep", y="Total Time In Bed")+
  theme(legend.position="none") +
  theme(plot.title = element_text(face="bold"))
```

