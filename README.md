# cyclistic-bike-data

In this case study, I will be analyzing bike trip data from the Chicago-based bike-share company Cyclistic. Our main goal is to identify how annual subscription holders and casual riders differ in ridership behavior. I will using Python and Tableau for this case study.

Please note this case study is sourced from the Coursera Google Data Analytics Capstone: Complete a Case Study course.


# Background

Cyclistic is a Chicago-based bike share company that has identified annual subscriptions as more profitable than their more casual offerings. Therefore, Cyclistic wants to maximize annual subscriptions. We will identify how annual subscribers differ in ridership behavior from casual customers (those who use single-ride or day passes). We will then use these insights to make key recommendations on how to maximize subscriptions. The key stakeholders here would be Cyclistic executives, and the secondary stakeholders would be the marketing analytics team. 

# The Data
Cyclistic Trip Data: https://divvy-tripdata.s3.amazonaws.com/index.html

I am using Cyclistic trip data from 2019. The datasets are available in quarterly chunks, which I downloaded separately and later ended up merging into one larger dataset for the entire year of 2019. 

# Data Analysis, Cleaning, etc. - Python

The tools I am using are largely Python for analyzing and cleaning the data, and Tableau for data visualization. In Python, I am using pandas, datetime, calendar, and numpy.
```
import pandas as pd
import datetime as datetime
import calendar as calendar
import numpy as np
```


There are four datasets total, which I will load in using Pandas' read_csv function.

```
q1_trips = pd.read_csv(r'C:\Users\garde\.vscode\Projects\bikeshare\divvy_trips_2019\Divvy_Trips_2019_Q1.csv')
q2_trips = pd.read_csv(r'C:\Users\garde\.vscode\Projects\bikeshare\divvy_trips_2019\Divvy_Trips_2019_Q2.csv')
q3_trips = pd.read_csv(r'C:\Users\garde\.vscode\Projects\bikeshare\divvy_trips_2019\Divvy_Trips_2019_Q3.csv')
q4_trips = pd.read_csv(r'C:\Users\garde\.vscode\Projects\bikeshare\divvy_trips_2019\Divvy_Trips_2019_Q4.csv')
q1_trips.head()
```

![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/6a1017b8-7395-48c7-b7c5-4b900241c019)

The dataset is organized into the following columns:

```
trip_id: A unique identifier for each bike-share trip.
start_time: The date and time the particular trip started.
end_time: The date and time the particular trip ended.
bikeid: The unique ID each bicycle has been assigned.
tripduration: The duration of each trip measured in seconds.
from_station_id: The unique ID assigned to each departing bike station.
from_station_name: The name of the particular departing bike station.
to_station_id: The unique ID assigned to each arriving bike station.
to_station_name: The name of the particular arriving bike station.
usertype: Two values, 'Subscriber' and 'Customer', based on whether the customer holds an annual pass.
gender: Two values, 'Male' and 'Female', based on the gender of the customer.
birthyear: The year the customer was born.
```


`q2_trips.head()`

![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/b1f96010-b080-44a3-a980-2fd6658b12bc)

I noticed that the formatting of q2_trips does not match q1_trips, and updated it as such:

```
q2_trips = q2_trips.rename(columns={"01 - Rental Details Rental ID":"trip_id",
                         "01 - Rental Details Local Start Time":"start_time",
                         "01 - Rental Details Local End Time":"end_time",
                         "01 - Rental Details Bike ID":"bikeid",
                         "01 - Rental Details Duration In Seconds Uncapped":"tripduration",
                         "03 - Rental Start Station ID":"from_station_id",
                         "03 - Rental Start Station Name":"from_station_name",
                         "02 - Rental End Station ID":"to_station_id",
                         "02 - Rental End Station Name":"to_station_name",
                         "User Type":"usertype",
                         "Member Gender":"gender",
                         "05 - Member Details Member Birthday Year":"birthyear"})
```

trips_2019 is the DataFrame we will be working with for the rest of this case study. For the purposes of this case study, I am only interested in start_time, tripduration, usertype, gender, and birthyear. 

```
frames = [q1_trips, q2_trips, q3_trips, q4_trips]
trips_2019 = pd.DataFrame(pd.concat(frames))
trips_2019 = trips_2019[['start_time', 'tripduration', 'usertype', 'gender', 'birthyear']]
```
![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/ec63bf7a-2e3a-4972-b177-db7f4c5624b0)

I noticed that the tripduration column is formatted as a string, so I converted it to an integer. I also divided it by 60 so we can measure trip duration in terms of minutes.

`trips_2019['tripduration'] = round((trips_2019['tripduration'].str.replace(',', '').astype(float).astype(int))/60, 1)`

There are a good amount of nulls in the gender (538,751) and birthyear (559,206) columns. I considered dropping these columns or imputing the values, but decided against it. In this case, I believe the existence of these nulls can shed insights on our casual customers. 

```
birthyear_nulls = trips_2019['birthyear'].isnull().sum()
trip_nulls = trips_2019['tripduration'].isnull().sum()
gender_nulls = trips_2019['gender'].isnull().sum()
```

From the start_time column, I am creating 6 more columns:

```
trips_2019['day_of_week'] = pd.to_datetime(trips_2019['start_time']).dt.dayofweek
trips_2019['month'] = pd.to_datetime(trips_2019['start_time']).dt.month
trips_2019['day_of_month'] = pd.to_datetime(trips_2019['start_time']).dt.day
trips_2019['hour'] = pd.to_datetime(trips_2019['start_time']).dt.hour
trips_2019['season'] = pd.cut(trips_2019['month'],
                       bins = [0, 3, 6, 9, 12],
                       labels = ['Winter', 'Spring', 'Summer', 'Autumn'])
trips_2019['time_of_day'] = pd.cut(trips_2019['hour'],
                       bins = [-1, 3, 7, 11, 15, 19, 23],
                       labels = ['Late Night', 'Early Morning', 'Morning',
                                 'Afternoon', 'Evening', 'Night'])
```

```
day_of_week: The day of week expressed as an integer (0 = Sunday, 1 = Monday, etc.)
month: The month expressed as an integer (1 = January, 2 = February, etc.)
day_of_month: The date of the month expressed as an integer (1, 2, etc.)
hour: The hour of the day, starting at 0 (12:00 AM) and ending at 23 (11:00 PM)
season: The 4 seasons of the year. Each is a bin of 3 months. Winter: January, February, March. Spring: April, May, June. Etc.
time_of_day: The hour of the day split into 6 bins of 4 hours each. 'Late Night', 'Early Morning', 'Morning', 'Afternoon, 'Evening', 'Night'.

```

Afterwards, I created two new columns from birthyear.

First, age: This is expressed as 2019 - birthyear. 2019 was chosen as that was the year this dataset was published.

After examining the age column, I noticed there were some outliers. For example, the max is 260.

```
trips_2019['age'] = 2019 - trips_2019.birthyear
percentiles = [0, .25, .5, .75, 1]
column_percentiles = trips_2019['age'].quantile(percentiles)
```

![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/1996d45c-9727-4b0e-9553-aae643475e32)


We will remove these outliers from age, and after that we may as well examine and remove outliers from tripduration.

```
Q1 = trips_2019['age'].quantile(0.25)
Q3 = trips_2019['age'].quantile(0.75)
IQR = Q3 - Q1

# identify outliers
threshold = 3
outliers = trips_2019[(trips_2019['age'] < Q1 - threshold * IQR) | (trips_2019['age'] > Q3 + threshold * IQR)]
trips_2019 = trips_2019[~trips_2019.index.isin(outliers.index)]

percentiles = [0, .25, .50, .75, 1]
column_percentiles = trips_2019['tripduration'].quantile(percentiles)

Q1 = trips_2019['tripduration'].quantile(0.25)
Q3 = trips_2019['tripduration'].quantile(0.75)
IQR = Q3 - Q1

# identify outliers
threshold = 3
outliers = trips_2019[(trips_2019['tripduration'] < Q1 - threshold * IQR) | (trips_2019['tripduration'] > Q3 + threshold * IQR)]
trips_2019 = trips_2019[~trips_2019.index.isin(outliers.index)]
```

Looking at quantiles of age and tripduration again: Our outliers have been removed! I think we are good to bin the ages.

```
percentiles = [0, .25, .50, .75, 1]
column_percentiles = trips_2019['age'].quantile(percentiles)
```
![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/61596775-aa53-40bd-888b-af5e5fb512cd)

```
percentiles = [0, .25, .50, .75, 1]
column_percentiles = trips_2019['tripduration'].quantile(percentiles)
```
![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/f3c3f0aa-1152-44c3-8b1d-2691d4191787)


I have binned age into octiles: 0-25, 26-27, 28-29, 30-32, 33-35, 36-40, 41-50, 51+.

```
trips_2019['generation'] = pd.cut(trips_2019['age'],
                           bins = [0, 25, 27, 29, 32, 35, 40, 50, 79],
                           labels = ['25 and younger', '26-27', '28-29', '30-32', '33-35', '36-40', '41-50', '51+'])

trips_2019.head()
```

One more idea that we can derive from start_time: Is the trip taking place on a holiday or not?

I chose a selection of holidays using Google Trends, and kept the days that scored at least a '1' during their peak in 2019.

```
trips_2019['holiday'] = pd.DataFrame(0, index=trips_2019.index, columns=["trips_2019['holiday']"])

trips_2019['holiday'].loc[(trips_2019['month'] == 12) & (trips_2019['day_of_month'] == 25)] = 1 #christmas, 100 
trips_2019['holiday'].loc[(trips_2019['month'] == 1) & (trips_2019['day_of_month'] == 1)] = 1 #new years, 1
trips_2019['holiday'].loc[(trips_2019['month'] == 12) & (trips_2019['day_of_month'] == 24)] = 1#christmas eve, 10
trips_2019['holiday'].loc[(trips_2019['month'] == 11) & (trips_2019['day_of_month'] == 28)] = 1#thanksgiving, 54
trips_2019['holiday'].loc[(trips_2019['month'] == 10) & (trips_2019['day_of_month'] == 31)] = 1#halloween, 38
trips_2019['holiday'].loc[(trips_2019['month'] == 6) & (trips_2019['day_of_month'] == 19)] = 1#juneteenth, 2
trips_2019['holiday'].loc[(trips_2019['month'] == 2) & (trips_2019['day_of_month'] == 14)] = 1#valentines day, 33
trips_2019['holiday'].loc[(trips_2019['month'] == 4) & (trips_2019['day_of_month'] == 1)] = 1#april fools day, 1
trips_2019['holiday'].loc[(trips_2019['month'] == 4) & (trips_2019['day_of_month'] == 22)] = 1#earth day, 1
trips_2019['holiday'].loc[(trips_2019['month'] == 5) & (trips_2019['day_of_month'] == 5)] = 1#cinco de mayo, 3
trips_2019['holiday'].loc[(trips_2019['month'] == 5) & (trips_2019['day_of_month'] == 12)] = 1#mothers day, 11
trips_2019['holiday'].loc[(trips_2019['month'] == 7) & (trips_2019['day_of_month'] == 14)] = 1#flag day, 1
trips_2019['holiday'].loc[(trips_2019['month'] == 6) & (trips_2019['day_of_month'] == 16)] = 1#fathers day, 7
trips_2019['holiday'].loc[(trips_2019['month'] == 11) & (trips_2019['day_of_month'] == 29)] = 1#black friday, 91
trips_2019['holiday'].loc[(trips_2019['month'] == 12) & (trips_2019['day_of_month'] == 2)] = 1#cyber monday, 18
trips_2019['holiday'].loc[(trips_2019['month'] == 12) & (trips_2019['day_of_month'] == 31)] = 1#new years eve, 10
trips_2019['holiday'].loc[(trips_2019['month'] == 5) & (trips_2019['day_of_month'] == 27)] = 1#memorial day, 12
trips_2019['holiday'].loc[(trips_2019['month'] == 7) & (trips_2019['day_of_month'] == 4)] = 1#july 4th, 24
trips_2019['holiday'].loc[(trips_2019['month'] == 9) & (trips_2019['day_of_month'] == 2)] = 1#labor day, 9
trips_2019['holiday'].loc[(trips_2019['month'] == 11) & (trips_2019['day_of_month'] == 11)] =1 #veterans day, 18
trips_2019['holiday'].loc[(trips_2019['month'] == 10) & (trips_2019['day_of_month'] == 14)] = 1#columbus day, 4
trips_2019['holiday'].loc[(trips_2019['month'] == 2) & (trips_2019['day_of_month'] == 18)] = 1#presidents day, 4
trips_2019['holiday'].loc[(trips_2019['month'] == 4) & (trips_2019['day_of_month'] == 21)] = 1#easter, 22
```

I did do SOME visualization in Python, but decided to do the visualization again in Tableau. 1. Because Tableau is more flexible and user-friendly, but also 2. It is much more visually appealing in my opinion.

```
trips_2019 = trips_2019[['tripduration', 'usertype', 'gender', 'birthyear', 'day_of_week', 'month', 'season', 'hour', 'time_of_day', 'age', 'generation', 'holiday']]
trips_2019.to_csv('bike_data_csv.csv', index=False)
```

#Visualization - Tableau
![image](https://github.com/heatherdogwood/cyclistic-bike-data/assets/65338495/b6def87d-54c6-481c-abd9-49ee275d6526)

