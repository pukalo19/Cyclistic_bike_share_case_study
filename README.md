## bellabeat fitness tracker case study for Google Data Annyltics Course Capstone by: Adam Pukalo
**Date: July 13, 2023** 
#
_This case study will follow the six step data analysis procee:_
###  1. [Ask](#step-1-ask)
###  2. [Prepare](#step-2-prepare)
###  3. [Process](#step-3-process)
###  4. [Analyze](#step-4-analyze)
###  5. [Share](#step-5-share)
###  6. [Act](#step-6-act)

![download](https://github.com/pukalo19/bellabeat_case_study_Google_Capstone_R/assets/131198211/9e72c0f6-3bf2-49b1-86a3-0bf6412ce524)

## Introduction 
bellabea is a high-tech company that manufactures health-focused smart products. Collecting data on activity, sleep, stress, and reproductive health that has allowed Bellabeat to empower
women with knowledge about their own health and habits. Since it was founded in 2013, Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women. Founders Urška Sršen and Sando Mur believe that with proper data analysis, bellabeat can make intelligent decisions to push the company to new heights of success.

## Step 1: Ask

## Business Task
Analyze the data of an established competitor, identify growth opportunities, and make recommendations for the bellabeat marketing strategy.

**Stakeholders:** 

* Urška Sršen: Cheif Creative Officer of bellabeat and cofounder
* Sando Mur: Cofounder of bellabeat and mathematician

**Guiding questions for analysis**
1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?

## Step 2: Prepare
The data used in this anlysis can be found here:[FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) CC0: Public Domain, dataset made available through [Mobius](https://www.kaggle.com/arashnic)
The data was uploaded and stored into R Studio Cloud. 
This dataset was generated by respondents to a distributed survey via Amazon Mechanical Turk between 03.12.2016-05.12.2016. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring.

ROCCC Data Review 
* **Reliability - LOW:** The data comes from only 30 fitbit users who consented to the submission of personal tracker data.
* **Original - LOW:** Amazon Mechanical Turk a third party was used to collect the data.
* **Comprehensive - MED:** The dataset contains six different fields
* **Current - LOW:** The dataset is from 2016, quite old now, and users habits could have changed over this time period.
* **Cited - LOW:** Unkown as the data was collected from a third party.

## Loading necessary packages into R
```
library(tidyverse)
library(lubridate) 
library(dplyr)
library(ggplot2)
library(tidyr)
library(janitor)
```
## Step 3: Process 
[Back to top](#introduction)

### Importing the data
```
#Reading the data
activity <- read.csv("dailyActivity_merged.csv",TRUE,",")
calories <- read.csv("dailyCalories_merged.csv",TRUE,",")
intensities <- read.csv("hourlyIntensities_merged.csv", TRUE, ",")
sleep <- read.csv("sleepDay_merged.csv",TRUE, ",")
weight <- read.csv("weightLogInfo_merged.csv",TRUE, ",")
```
Now that all the data we will be using is uploaded we can start the process of cleaning the data. Lets take a look at the data and 
```
          Id ActivityDate TotalSteps TotalDistance TrackerDistance
1 1503960366    4/12/2016      13162          8.50            8.50
2 1503960366    4/13/2016      10735          6.97            6.97
3 1503960366    4/14/2016      10460          6.74            6.74
4 1503960366    4/15/2016       9762          6.28            6.28
5 1503960366    4/16/2016      12669          8.16            8.16
6 1503960366    4/17/2016       9705          6.48            6.48
  LoggedActivitiesDistance VeryActiveDistance ModeratelyActiveDistance
1                        0               1.88                     0.55
2                        0               1.57                     0.69
3                        0               2.44                     0.40
4                        0               2.14                     1.26
5                        0               2.71                     0.41
6                        0               3.19                     0.78
  LightActiveDistance SedentaryActiveDistance VeryActiveMinutes
1                6.06                       0                25
2                4.71                       0                21
3                3.91                       0                30
4                2.83                       0                29
5                5.04                       0                36
6                2.51                       0                38
  FairlyActiveMinutes LightlyActiveMinutes SedentaryMinutes Calories
1                  13                  328              728     1985
2                  19                  217              776     1797
3                  11                  181             1218     1776
4                  34                  209              726     1745
5                  10                  221              773     1863
6                  20                  164              539     1728
```
Checking the rest of the data 
```
head(calories)
head(intensities)
head(sleep)
head(weight)
head(weight)
```
After looking at all the data, I notice inconsistencies in the recording of the timestamp data. I will convert this to a date time format to fix this. 
```
activity$ActivityDate=as.POSIXct(activity$ActivityDate, format="%m/%d/%Y", tz=Sys.timezone())
activity$date <- format(activity$ActivityDate, format = "%m/%d/%y")

intensities$ActivityHour=as.POSIXct(intensities$ActivityHour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
intensities$time <- format(intensities$ActivityHour, format = "%H:%M:%S")
intensities$date <- format(intensities$ActivityHour, format = "%m/%d/%y")

sleep$SleepDay=as.POSIXct(sleep$SleepDay, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
sleep$date <- format(sleep$SleepDay, format = "%m/%d/%y")
```
Now that we have looked over the data and corrected any inconsistencies/errors we can begin to analyze the data.

## Step 4: Analyze 
[Back to top](#introduction)

We will begin with fiding out how many particitpants there were in each category of the study. 
```
> n_distinct(activity$Id)  
[1] 33
> n_distinct(calories$Id)   
[1] 33
> 
> n_distinct(intensities$Id)
[1] 33
> 
> n_distinct(sleep$Id)
[1] 24
> 
> n_distinct(weight$Id)
[1] 8
```
So, there are 33 participants in the Activity, Calories, and Intensities datasets. 24 participants in the Sleep data. The real problem is there is only eight participants in the weight dataset, meaning that we are not likely to be able to make a strong reccomendation or decision with only data from eight participants.

Let's see if there was any major changes in the weight of our eight participants.
```
weight%>%
  group_by(Id)%>%
  summarise(min(WeightKg),max(WeightKg))

Id `min(WeightKg)` `max(WeightKg)`
       <dbl>           <dbl>           <dbl>
1 1503960366            52.6            52.6
2 1927972279           134.            134. 
3 2873212765            56.7            57.3
4 4319703577            72.3            72.4
5 4558609924            69.1            70.3
6 5577150313            90.7            90.7
7 6962181067            61              62.5
8 8877689391            84              85.8
```
Due to the fact there was hardly any change in our participants weight and a very small sample size, I think it is best that we not use any of the weight data going forward.

Now we can grab a summary of the rest of the data we will be using.
```
activity %>%  
  select(TotalSteps,
         TotalDistance,
         SedentaryMinutes, Calories) %>%
  summary()

activity %>%
  select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes) %>%
  summary()

calories %>%
  select(Calories) %>%
  summary()

sleep %>%
  select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>%
  summary()
```
```
TotalSteps    TotalDistance    SedentaryMinutes    Calories   
 Min.   :    0   Min.   : 0.000   Min.   :   0.0   Min.   :   0  
 1st Qu.: 3790   1st Qu.: 2.620   1st Qu.: 729.8   1st Qu.:1828  
 Median : 7406   Median : 5.245   Median :1057.5   Median :2134  
 Mean   : 7638   Mean   : 5.490   Mean   : 991.2   Mean   :2304  
 3rd Qu.:10727   3rd Qu.: 7.713   3rd Qu.:1229.5   3rd Qu.:2793  
 Max.   :36019   Max.   :28.030   Max.   :1440.0   Max.   :4900
VeryActiveMinutes FairlyActiveMinutes LightlyActiveMinutes 
 Min.   :  0.00    Min.   :  0.00      Min.   :  0.0       
 1st Qu.:  0.00    1st Qu.:  0.00      1st Qu.:127.0       
 Median :  4.00    Median :  6.00      Median :199.0       
 Mean   : 21.16    Mean   : 13.56      Mean   :192.8       
 3rd Qu.: 32.00    3rd Qu.: 19.00      3rd Qu.:264.0       
 Max.   :210.00    Max.   :143.00      Max.   :518.0
 Calories   
 Min.   :   0  
 1st Qu.:1828  
 Median :2134  
 Mean   :2304  
 3rd Qu.:2793  
 Max.   :4900
TotalSleepRecords TotalMinutesAsleep TotalTimeInBed 
 Min.   :1.000     Min.   : 58.0      Min.   : 61.0  
 1st Qu.:1.000     1st Qu.:361.0      1st Qu.:403.0  
 Median :1.000     Median :433.0      Median :463.0  
 Mean   :1.119     Mean   :419.5      Mean   :458.6  
 3rd Qu.:1.000     3rd Qu.:490.0      3rd Qu.:526.0  
 Max.   :3.000     Max.   :796.0      Max.   :961.0
```
## Observations:
* Our participants averaged 7638 steps per day.
* The average sedintary time was about 16.5 hours.
* Participants burned on average 2304 calories each day.
* Lightly active is what most of our participants would be categorized as.
* The average sleep time for participants is approximately seven hours.
* Participants average 7.6 hours spent in bed each night.


## Combining Tables 
Before moving on to the analyze step I will use an inner join to merge the activity and sleep tables.
```
merged_data <- merge(sleep,activity)
```

## Step 5: Share 
[Back to top](#introduction)
<br>

```
ggplot(data = activity, aes(x = TotalSteps, y = Calories)) + geom_point() + geom_smooth() + labs(title = " Steps vs. Calories")
```
![Cals_vs_Steps](https://github.com/pukalo19/bellabeat_case_study_Google_Capstone_R/assets/131198211/2579e209-c8db-425b-8f6d-075f1f24b193)

Looking at the chart above, we can confirm something we may have already known. The more steps our participants are taking the more calories they are burning.

```
 ggplot(data = merged_data, mapping = aes(x= Calories, y= TotalMinutesAsleep )) + geom_point() + labs(title = "Calories Burned vs Time Asleep")
```
![Cals_vs_sleep](https://github.com/pukalo19/bellabeat_case_study_Google_Capstone_R/assets/131198211/00900360-9269-4635-83d0-6f584056c594)
I had a feeling that there would be a correlation between the amount of calories burned and how much sleep each participant got. Given the data in the chart there does not appear to be a correlation. 
Lets run a correlation test as well. 
```
cor(merged_data$Calories,merged_data$TotalSleepRecords)
[1] -0.05105987
```
That confirms that there is no significant correlation between the two.

Next, I will check and see if there is a correlation between sedentary time and sleep duration.
```
ggplot(data = merged_data, mapping = aes(x = SedentaryMinutes, y = TotalMinutesAsleep)) + geom_point() + geom_smooth() + labs(title= "Sleep Duration vs Sedentary Time")
```
![sed_sleep](https://github.com/pukalo19/bellabeat_case_study_Google_Capstone_R/assets/131198211/27382911-ea78-4b4e-bc18-27fa679bcf7d)
We can derive from this chart that there is a negative correlation between sleep and sedentary time, meaning that the more time our participants spent sedentary the less they would sleep.

Lastly, lets check and see what the relationship looks like  between the amount of time spent in bed against time asleep.
```
ggplot(data=sleep, mapping = aes(x=TotalTimeInBed, y=TotalMinutesAsleep)) + geom_point() + geom_smooth() + labs(title = "Time Asleep vs Time In Bed")
```
![time_asleep_vs_in_bed](https://github.com/pukalo19/bellabeat_case_study_Google_Capstone_R/assets/131198211/b0c0fd86-1432-469a-bf16-d8619c6163fb)
It looks like we have a positive relationship between the two. So, the more time our participants are spending in bed the more total minutes of sleep they are getting.

## Step 6: Act 
[Back to top](#introduction)
<br>
![Screen Shot 2023-07-19 at 2 47 43 PM](https://github.com/pukalo19/bellabeat_case_study_Google_Capstone_R/assets/131198211/fd1d79b6-9090-424b-9896-8be5a432102b)
