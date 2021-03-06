---
title: "Google Data Analytics Capstone Project"
author: "Edeh Emeka N"
output:
  html_notebook: default
  pdf_document: default
highlight: tango
Date: '2021-08-26'
---

### Project Goal: 
I am mandated to determine how annual members and casual riders use Cyclistic bikes differently with the aim of converting casual riders into annual member.  

**Key stakeholders**:      
Lily Moreno: The director of marketing,    
Cyclistic executive team and  
Cyclistic marketing analytics team.    
                          
### Ask Phase
By the end of this project, i should be able to determine how annual members and casual riders use Cyclistic bikes differently, Why casual riders would want to buy Cyclistic annual memberships and how Cyclistic can use digital media to influence casual riders to become members.

#### Tools Used for this project.

Excel, SQL and R-Language

### Prepare Phase
The datasets being used for the project are 12 months of trip data owned by Motivate International Inc who have granted me a non-exclusive, royalty-free, limited, perpetual license to access, reproduce, analyze, copy, modify, distribute in my product or service and use the Data for any lawful purpose (“License”).  

There are no issues of bias or credibility with these datasets. They are Reliable, Original, Comprehensive, Current and Cited.  

Upon initial inspection of each of the 12 csv datasets using ISBLANK function in excel, i noticed that some of the observations are missing. This i will address subsequently using RStudio.
                     
### Loading Datasets

Loading the needed 12 csv datasets, i will first load the tidyverse package which contains most of the dataframe manipulation packages needed 
for loading and cleaning the datasets for this project.
```{r}
library("tidyverse")
```

### DATA COMBINATION
I have just the needed 12 csv files for this project in my current working directory with each file representing trip data for a month. read.csv function from readr package will import all the files and the rbind function will bind all the rows of these separate files into one dataframe. 

```{r results="hide"}
files <- dir(pattern = "*.csv")
files

trip_data <- files %>%
  map(read_csv) %>%   
  reduce(rbind)        
trip_data

```
lets take a glimpse of my full data by inspecting the data names and types
```{r}
glimpse(trip_data)
```
### Process Phase

For thise phase, i am using RStudio for my data cleaning.
### DATA CLEANING

from the glimpse, i can see that i now have 4,731,081 rows and it 
also reveals that some variables need renaming for consistency
and clarity sake.  
variables like "started_at" will be renamed "start_time", "ended_at" will
be renamed "end_time" and "member_casual" will be renamed "user_status"

```{r}
renamed_col <- trip_data %>% 
  rename(start_time=started_at, end_time = ended_at,user_status= member_casual
         , bike_type = rideable_type)
glimpse(renamed_col)
head(renamed_col)
```
I will add trip_duration column by using the mutate function
in combination with some functions from the lubridate package.
```{r}
library("lubridate")
with_trip_duration <-mutate(renamed_col, trip_duration= as.duration
                            (interval(ymd_hms(renamed_col$start_time),
                                      ymd_hms(renamed_col$end_time))))


```

```{r}
with_trip_duration
```
I will add another column with ride weekdays in order to determine what day of
the week majority of the users prefer riding.
```{r}
with_week_day <- with_trip_duration %>% 
  mutate(weekday=weekdays(as.Date(with_trip_duration$start_time)))

with_week_day
```

checking for consistency in some of the variable names with unique()

```{r}
unique(with_week_day$user_status)
unique(with_week_day$bike_type)
```
checking and removing null and na data for ease of data analysis
```{r}
is.null(with_week_day)
cleaned_trip <-na.omit(with_week_day) 
cleaned_trip
```
the na.omit function removed about 600,000 values which i am assuming are negligible

### Analyze and Share Phases.

l will first determine the count of the two user_types
```{r}
user_count <-table(cleaned_trip$user_status)
user_count
plot(user_count)
pie(user_count)
```

From the above analysis, it shows that we have more registered users than casual
users. 1828502 for casual users and 2338897 for registered users.  

Now, lets determine the average duration of each of the categories of riders

```{r}
avg_trip_duration <- cleaned_trip %>% 
  group_by(user_status) %>% 
  summarize(avg = mean(trip_duration)/60)

avg_trip_duration 
```

From the above analysis, it is evident that the casual riders tend to ride
longer at an average of of 36.6 minutes/ride than the member/registered riders who ride for an average of 11.2 minutes/ride.  


Next, i will determine when majority of the riders prefer to ride by extracting
the aggregate count of days for each of the user_status
```{r}
day_of_ride<- cleaned_trip %>% 
  group_by(weekday, user_status) %>% 
  select(user_status, weekday) %>% 
  summarize(number = table(weekday))

day_of_ride
```
in order to extract the casual user with their corresponding weekday and number, i exported the outcome above to SQL.  I obtained the desired result by performing the command below;

SELECT * FROM day_of_ride WHERE user_status= "member"

SELECT * FROM day_of_ride WHERE user_status= "casual"



Then i imported the query result into Rstudio for futher analysis.

```{r}
casual_ride <- read.csv("C:\\Users\\EDEH EMEKA NWEKE\\Desktop\\casual_ride_data.csv")
```
```{r}
member_ride <-read.csv("C:\\Users\\EDEH EMEKA NWEKE\\Desktop\\member_ride_data.csv")
```

```{r}
casual_ride
```

```{r}
member_ride
```
Then i sorted the above outcome in a descending order for clearer picture.


```{r}
sort_casual_ride <- casual_ride %>% 
  arrange(number)

sort_casual_ride
```

```{r}
library('ggplot2')
```

```{r}
ggplot(sort_casual_ride, aes(x=weekday, y=number, color =weekday, fill= weekday)) + 
  labs(title = 'Number of Casual Riders vs Days of the week', 
       caption = 'Data analyzed by Edeh Emeka')+ 
  geom_bar(stat = "identity")+
  theme(legend.position="none")

 


```
```{r}
sort_member_ride <- member_ride %>% 
  arrange(number)

sort_member_ride
```
```{r}
ggplot(sort_member_ride, aes(x=weekday, y=number, color =weekday, fill = weekday)) + 
  labs(title = 'Number of member Riders vs Days of the week', 
       caption = 'Data analyzed by Edeh Emeka')+ 
  geom_bar(stat = "identity")+
  theme(legend.position="none")
```
From the above graph, it is evident that majority of the casual riders, close to 60%, prefer riding over the weekend, Fridays, Saturdays and Sundays as against the member/registered riders that are almost evenly distributed across the weeks.
about 60% of casual trips take place over the weekend.  



In trying to extract the months of these rides, i ran the following codes,
exported the outcome to SQL in order to aggregate the user counts with their
respective months.
```{r}
as_month<- cleaned_trip %>% 
  mutate(month=format(cleaned_trip$end_time, "%m"))
as_month
```

Since the month(number) is in character format, i have to convert it to a numeric format by running the code below.

```{r}
month_as_num <- as_month %>% 
  mutate(month_num = as.numeric(as_month$month))

str(month_as_num)
```
I will convert the numeric month to month names by running the code below.
```{r}
month_letter <- month_as_num %>% 
  mutate(month_l = month.name[month_as_num$month_num])
month_letter
```

```{r}
month_of_ride<- month_letter %>% 
  group_by(month_l, user_status) %>% 
  summarize(number = table(month_l))

month_of_ride
```
I will export the above outcome to a csv file for further analysis in SQL in order to aggregate the user counts with their corresponding months.
```{r}
write.csv(month_of_ride, "month_of_ride.csv")
```

I will extract the data for the two different user types, casual and member, using the sql command below.

SELECT * FROM month_of_ride WHERE user_status = "member"

SELECT * FROM month_of_ride WHERE user_status = "casual"


Next, i will import and sort the results of the above commands in a descending order.

```{r}
member_month <-read.csv("C:\\Users\\EDEH EMEKA NWEKE\\Desktop\\member_month_of_ride.csv")
```
```{r}
casual_month <-read.csv("C:\\Users\\EDEH EMEKA NWEKE\\Desktop\\casual_month_of_ride.csv")
```
```{r}


sort_member_month <- member_month %>% 
  arrange(number)

sort_member_month
```

```{r}
sort_casual_month <- casual_month %>% 
  arrange(number)

sort_casual_month
```

I will further present the above results graphically for clearer picture.

```{r}
ggplot(sort_member_month, aes(x=month_l, y=number, color =month_l, fill = month_l)) + 
  labs(title = 'Number of member Rides vs Months of the year', 
       caption = 'Data analyzed by Edeh Emeka')+ 
  geom_bar(stat = "identity")+
  theme(legend.position="none")
```

```{r}
ggplot(sort_casual_month, aes(x=month_l, y=number, color =month_l, fill = month_l)) + 
  labs(title = 'Number of casual Rides vs Months of the year', 
       caption = 'Data analyzed by Edeh Emeka')+ 
  geom_bar(stat = "identity")+
  theme(legend.position="none")
```

From the above graphs, it is evident that casual riders are mostly active in the summer months of June, July and August.

### ACT

Here are my three observations and recommendations in order to convert casual riders to annual riders.  

**Observations**  
1. Casual users are mostly active on weekends (Fridays, Saturdays and Sundays)as against annual/member users that are almost evenly active all through the week.  
2. on the average, Casual users tend to ride for about 25 minutes longer than the member/annual users.  
3. Casual users are mostly active during the summer months (June, July, August) while annual/member users are active in all the months bar winter months.  

**Recommendations**  
1. Offer membership exclusive incentives/packages for weekend rides.  
2. Offer discounted special summer packages to members.  
3. Offer special discounts for rides that last longer than 15 minutes.  



Edeh Emeka N.   
2021-08-26
  
    






