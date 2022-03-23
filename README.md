 #Uber Visualization Uber V/S Lyft

Your project is for visulizing the data of uber and lyft comparision.This data is of UK 

Code :

#import the libary

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime


# Load the data 
cab_df = pd.read_csv("cab_rides.csv")


#It the header of data
#In the we can see the column of data
cab_df.head()


#checking the null value
cab_df.isnull().sum()


# We see that price column has 55095 rows missing data. 
#Total rows we have 693071 so we can remove the rows with no price.
cab_df.dropna(inplace=True)
#Againg seeing that null value
cab_df.isnull().sum()


# First, lets see what columns do we have
cab_df.columns


# We don't need    id    product_id    surge_multiplier. So lets get rid of them
cab_df = cab_df[['distance', 'cab_type', 'time_stamp', 'destination', 'source', 'price', 'name']]
cab_df.head()


# We see that time_stamp is Unix, so we need to convert it to the readable form.
# Using the time_stamp column, lets convert it to date, week day, hour and time of day.
cab_df["rounded_timestamp"] = cab_df["time_stamp"] / 1000
cab_df["rounded_timestamp"] = cab_df["rounded_timestamp"].apply(np.floor)



#we are creating four different column
cab_df["date"] = cab_df["rounded_timestamp"].apply(lambda x : datetime.fromtimestamp(x).date())
cab_df["time"] = cab_df["rounded_timestamp"].apply(lambda x: datetime.fromtimestamp(x).time())
cab_df['weekday'] = cab_df['date'].apply(lambda x: x.weekday())
cab_df["weekday"] = cab_df["weekday"].map({0: 'Monday', 1: 'Tuesday', 2: 'Wednesday', 3: 'Thursday', 4: 'Friday', 5: 'Saturday', 6: 'Sunday'})
cab_df['hour'] = cab_df['time'].apply(lambda time: time.hour)


#see the date
cab_df['date'].head()


#see the time
cab_df['time'].head()


# We calculate time of day into: Morning,Afternoon,Evening and Night
cab_df.loc[(cab_df.hour >= 6) & (cab_df.hour < 12) , 'time_of_day'] = 'Morning'
cab_df.loc[(cab_df.hour >= 12) & (cab_df.hour < 16) , 'time_of_day'] = 'Afternoon'
cab_df.loc[(cab_df.hour >= 16) & (cab_df.hour < 22) , 'time_of_day'] = 'Evening'
cab_df.loc[(cab_df.hour >= 22) | (cab_df.hour < 6) , 'time_of_day'] = 'Night'


#After adding  column 'date', 'time','weekday', 'hour', 'time_of_day' see that
cab_df.columns


#need of column which is required for uber visualization
cab_df = cab_df[['distance', 'cab_type', 'time_stamp', 'price', 'name', 'date', 'time', 'weekday', 'hour', 'time_of_day']]


#if you want this new data set then use with conver of date and time from unix format
cab_df.to_csv("new.csv")
cab_df.head()


cab_df['cab_type'].value_counts()


#see the sift of time 
cab_df['time_of_day'].value_counts()


# So we can see we have two cab types: Uber and Lyft
# So we need to separate the datasets
uber_df = cab_df[cab_df['cab_type'] =="Uber"]
lyft_df = cab_df[cab_df['cab_type'] =="Lyft"]


# From above we see that for Uber we have only one value of price
# So we consider the price for uber only and plot which day it is highest. 
# We only consider the price > 1.
high_price_dataset = uber_df[uber_df["price"]>1]
high_distance_dataset = uber_df[uber_df["distance"]> 0.1]# From above we see that for Uber we have only one value of price


# So we consider the price for uber only and plot which day it is highest. 
# We only consider the price > 1.
high_price_dataset = uber_df[uber_df["price"]>1]
high_distance_dataset = uber_df[uber_df["distance"]> 0.1]


t_high_price = pd.DataFrame(high_price_dataset.groupby(["weekday", "price"]).size().reset_index())
t_high_price.columns = ["Weekday", "price","hour"]
plt.figure(figsize=(15, 6))
sns.barplot(x="Weekday", y="price", data=t_high_price,ci=None,estimator=np.max).set_title("Weekday wise price range");


t_high_price.groupby("Weekday").price.describe()


t_high_price1 = pd.DataFrame(high_price_dataset.groupby(["weekday", "price","name"]).size().reset_index())
t_high_price1.columns = ["Weekday", "price","name","count"]
plt.figure(figsize=(22, 5))
sns.barplot(x="Weekday", y="count", hue="name", data=t_high_price1,ci=None,estimator=np.mean).set_title("Weekday wise Surge");


td_high_day = pd.DataFrame(high_distance_dataset.groupby(["weekday","time_of_day"]).size().reset_index())
td_high_day.columns = ["Weekday", "Time of Day", "Count"]


plt.figure(figsize=(15, 10))
sns.lineplot(x="Time of Day", y="Count", data=td_high_day).set_title("Time of Day wise Distance");


td_high_day = pd.DataFrame(high_distance_dataset.groupby(["weekday","name","time_of_day"]).size().reset_index())
td_high_day.columns = ["Weekday", "name","Time of Day", "Count"]


plt.figure(figsize=(15, 10))
sns.lineplot(x="name", y="Count",hue="Time of Day",data=td_high_day,estimator=np.max).set_title("Time of Day wise Surge");
