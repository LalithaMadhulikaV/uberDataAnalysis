## uberDataAnalysis


# Overview

This repository contains R scripts and data for analyzing Uber ride data through visualizations. The purpose of this project is to explore and gain insights from Uber trip data using various graphical representations. The analysis covers aspects such as time trends, geographical patterns, and other interesting patterns in the dataset.


# Necessary libraries

gplot2: ggplot2 is the most popular data visualization library that is most widely used for creating aesthetic visualization plots.
lubridate: Use time-frames in the dataset
dplyr: Data Manipulation
tidyr: Tidy the data
DT: Datatables in JS
scales: With the help of graphical scales, we can automatically map the data to the correct scales with well-placed axes and legends.
library(ggplot2)
library(ggthemes)
library(lubridate)
library(dplyr)
library(tidyr)
library(tidyverse) # metapackage of all tidyverse packages
library(DT)
library(scales)
Attaching package: ‘lubridate’


The following objects are masked from ‘package:base’:

    date, intersect, setdiff, union



Attaching package: ‘dplyr’


The following objects are masked from ‘package:stats’:
    filter, lag


The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union

# Creating vector of colors for the plots
colors = c("#CC1011", "#665555", "#05a399", "#cfcaca", "#f5e840", "#0683c9", "#e075b0")
colors
Read the data from each time-frame
# Read the data for each month separately 
apr <- read.csv("../input/uberdataset/uber-raw-data-apr14.csv")
may <- read.csv("../input/uberdataset/uber-raw-data-may14.csv")
june <- read.csv("../input/uberdataset/uber-raw-data-jun14.csv")
july <- read.csv("../input/uberdataset/uber-raw-data-jul14.csv")
aug <- read.csv("../input/uberdataset/uber-raw-data-aug14.csv")
sept <- read.csv("../input/uberdataset/uber-raw-data-sep14.csv")

# Combine the data together 
data <- rbind(apr, may, june, july, aug, sept)
cat("The dimensions of the data are:", dim(data))
The dimensions of the data are: 4534327 4

data$Date.Time <- as.POSIXct(data$Date.Time, format="%m/%d/%Y %H:%M:%S")
data$Time <- format(as.POSIXct(data$Date.Time, format = "%m/%d/%Y %H:%M:%S"), format="%H:%M:%S")
data$Date.Time <- ymd_hms(data$Date.Time)
# Create individual columns for month day and year
data$day <- factor(day(data$Date.Time))
data$month <- factor(month(data$Date.Time, label=TRUE))
data$year <- factor(year(data$Date.Time))
data$dayofweek <- factor(wday(data$Date.Time, label=TRUE))
# Add Time variables as well 
data$second = factor(second(hms(data$Time)))
data$minute = factor(minute(hms(data$Time)))
data$hour = factor(hour(hms(data$Time)))

Plotting the trips by hours in a day
hourly_data <- data %>% 
                    group_by(hour) %>% 
                            dplyr::summarize(Total = n())

# Shos data in a searchable js table
datatable(hourly_data)

# Plot the data by hour
ggplot(hourly_data, aes(hour, Total)) + 
geom_bar(stat="identity", 
         fill="steelblue", 
         color="red") + 
ggtitle("Trips Every Hour", subtitle = "aggregated today") + 
theme(legend.position = "none", 
      plot.title = element_text(hjust = 0.5), 
      plot.subtitle = element_text(hjust = 0.5)) + 
scale_y_continuous(labels=comma)

Plotting trips by hour and month
# Aggregate the data by month and hour
month_hour_data <- data %>% group_by(month, hour) %>%  dplyr::summarize(Total = n())

ggplot(month_hour_data, aes(hour, Total, fill=month)) + 
geom_bar(stat = "identity") + 
ggtitle("Trips by Hour and Month") + 
scale_y_continuous(labels = comma)

# Plotting data by trips during every day of the month
# Aggregate data by day of the month 
day_data <- data %>% group_by(day) %>% dplyr::summarize(Trips = n())
day_data
# Plot the data for the day
ggplot(day_data, aes(day, Trips)) + 
geom_bar(stat = "identity", fill = "steelblue") +
ggtitle("Trips by day of the month") + 
theme(legend.position = "none") + 
scale_y_continuous(labels = comma)

# Collect data by day of the week and month

day_month_data <- data %>% group_by(dayofweek, month) %>% dplyr::summarize(Trips = n())
day_month_data

# Plot the above data
ggplot(day_month_data, aes(dayofweek, Trips, fill = month)) + 
geom_bar(stat = "identity", aes(fill = month), position = "dodge") + 
ggtitle("Trias by Day and Month") + 
scale_y_continuous(labels = comma) + 
scale_fill_manual(values = colors)

# Number of Trips place during months in a year
month_data <- data %>% group_by(month) %>% dplyr::summarize(Total = n())

month_data

# Heatmap by Hour and Day
day_hour_data <- data %>% group_by(day, hour) %>% dplyr::summarize(Total = n())
datatable(day_hour_data)

# Plot a heatmap 

ggplot(day_hour_data, aes(day, hour, fill = Total)) + 
geom_tile(color = "white") + 
ggtitle("Heat Map by Hour and Day")

Plot Heatmap by day and month
# Collect data by month and day

month_day_data <- data %>% group_by(month, day) %>% dplyr::summarize(Trips = n())
month_day_data

# Plot a heatmap 

ggplot(month_day_data, aes(day, month, fill = Trips)) + 
geom_tile(color = "white") + 
ggtitle("Heat Map by Month and Day")

# Plot a heatmap by day of the week and month

ggplot(day_month_data, aes(dayofweek, month, fill = Trips)) + 
geom_tile(color = "white") + 
ggtitle("Heat Map by Month and Day")

Creating a map visualization of rides in NYC
# Set Map Constants
min_lat <- 40 
max_lat <- 40.91
min_long <- -74.15
max_long <- -73.7004

ggplot(data, aes(x=Lon, y=Lat)) +
  geom_point(size=1, color = "blue") +
     scale_x_continuous(limits=c(min_long, max_long)) +
      scale_y_continuous(limits=c(min_lat, max_lat)) +
        theme_map() +
           ggtitle("NYC MAP BASED ON UBER RIDES DURING 2014 (APR-SEP)")

           


