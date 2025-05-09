# Sales Analysis
This exercise is from Brainwave Matrix Internship Program.

## Table Of Contents 
- [Scenario ](Scenario)
- [Bussiness Task](Bussiness-Task)
- [Prepare](Prepare)
- [Process](Process)
- [Analyze](Analyze)
- [Share](Share)
- [Findings](Findings)
- [Act](Act)


## Scenario 

You are a data analyst working on a sales team in a pharmaceutical company.The business owner wants to understand how the retail outlets are performing compared to hospital sales, especially across different cities. 

## Bussiness Task

My Boss has assigned me to analyze 
1. Which Products are driving the most revenuein our pharmacy sub-channel. annual members and casual riders use Cyclist bikes differently.
2. Are there specific cities or customer typeswhere our pharmacy sales are unerperforming?
3. Which sales reps are most effective in driving pharmacy sales, can we replicate their strategies across regions?
4. Are we Pricing our top-selling products competitively, or are we losing out to hospital channel?
5. Can we identify seasonal patterns in the pharmacy sales that can help with inventory and marketing planning?

## Prepare

I downloaded my data from kaggle.  it can be found [here](https://www.kaggle.com/datasets/akanksha995579/pharma-data-analysis). I extracted the data from its zip file. All personally identifiable information was removed. The data consist of 14 columns: Distributor, Customer_Name, City, Country, Latitude, Longitude, Channel, Sub_channel, Product_Name, Product_Class, Quantity, Price, Sales, Month, Year, Name_of_Sales_Rep, Manager, Sales_Team. I created a folder for the data I need and named it appropriately.

## Process

I examined the data using excel but the data is large, so i chose to do the analysis with Query language in Microsoft SQL Server Management (MSSM). I created a database and activated it. I imported the data into MSSM. Prior to importing the data I changed the quantity column into the float format. I then used the Select statement to view my data.

```{sql}
Create Database Rika_Pharma;

use Rika_Pharma;

SELECT * FROM dbo.pharma_data;
```

I then used DISTINCT to start my cleaning process.

```{sql}
select distinct * from dbo.pharma_data;
```

I found out that my table have 254078 distinct rows out of 254082 row. I checked the duplicated records and deleted them.

```{sql}
With DuplicateRecords As (
SELECT *,
count(*) Over (Partition by Distributor,Customer_Name, City, Country, Latitude, 
Longitude, Channel, Sub_channel, Product_Name,
Product_class, Quantity, Price, Sales, Month, Year, Name_of_Sales_Rep, Manager, Sales_Team) as duplicate_count
FROM dbo.pharma_data
)
select *
from DuplicateRecords
where duplicate_count > 1
```
Added a RowID 
```{sql}
Alter Table dbo.pharma_data
Add RowID INT Identity(1,1);
```
Deleted Duplicates
```{sql}
With CTE As (
SELECT *,
Row_Number() Over (Partition by Distributor,Customer_Name, City, Country, Latitude, 
Longitude, Channel, Sub_channel, Product_Name,
Product_class, Quantity, Price, Sales, Month, Year, Name_of_Sales_Rep, Manager, Sales_Team
Order By RowID
) as rn
FROM dbo.pharma-data
)
DElete from CTE
where rn > 1;
```
Then removed temp column RowID
```{sql}
Alter Table dbo.pharma_data
Drop Column RowID;
```

I checked for null values and deleted them.

```{sql}
select count(*) - count(Customer_Name) as Missing_count
from dbo.pharma_data;
```

## Analyze

### Best Selling Product (Pharmay channel)

I began my analysis process by finding the best selling products in the pharmacy channel and sub_channel.

```{sql}
select Top 10 Product_Name,Sub_channel, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Product_Name, Sub_channel
order by Total_Sales desc;
```
I checked Customers and Cities that are underperforming with the following codes

```{sql}
select Top 10 Customer_Name, sum(Sales) as Total_Sales
from dbo.pharma_data
group by Customer_Name
order by Total_Sales asc;
```
```{sql}
select Top 10 City, sum(Sales) as Total_Sales
from dbo.pharma_data
group by "City"
order by Total_Sales asc;
```

I went further to check for Sales reps who are effective in driving the pharmacy sales with the following codes

```{sql}
select Top 5 Name_of_Sales_Rep, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Name_of_Sales_Rep
order by Total_Sales desc;
```


### Rideable Type

I decided to check if there are differences in the types of vehicles used among member and casual riders.
 
 ```{sql}
select rideable_type, count(rideable_type) as no_of_bike
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration <1440
and member_casual='member'
group by rideable_type;
```

```{sql}
select rideable_type, count(rideable_type) as no_of_bike
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration <1440
and member_casual='casual'
group by rideable_type;
```

### Most Popular Stations

Looking out for the most frequently used stations by member and casual riders, I decided gto first find the most popular start stations for members and casual riders.

```{sql}
select start_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'member' 
and trip_duration >0 
and trip_duration <1440
group by start_station_name
order by rides_taken desc;
```

```{sql}
select start_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'casual' 
and trip_duration >0 
and trip_duration <1440
group by start_station_name
order by rides_taken desc;
```

Finding the most popular end stations for members and casuals

```{sql]
select end_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'member' 
and trip_duration >0 
and trip_duration <1440
group by end_station_name
order by rides_taken desc;
```

```{sql}
select end_station_name, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'casual' 
and trip_duration >0 
and trip_duration <1440
group by end_station_name
order by rides_taken desc;
```

### Number Of Rides

To know how many rides were taken by members and how many by casuals

```{sql}
select member_casual, count(*) as rides_taken
from bikeshare_trip_bkp
where trip_duration >0 
and trip_duration <1440
group by member_casual;
```

How many rides were taken on each day of the week by member and casual riders?

```[sql}
select dayname(started_at) as days, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'member' 
and trip_duration >0 
and trip_duration <1440
group by days
order by days;

```{sql}
select dayname(started_at) as days, count(*) as rides_taken
from bikeshare_trip_bkp
where member_casual = 'casual' 
and trip_duration >0 
and trip_duration <1440
group by days
order by days;
```

Finding how many rides was taken by member and casual riders in each month

 ```{sql}
select  Month, count(*) as ride_count
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration<1440
and member_casual = 'member'
group by Month
order by Month;
```

```{sql}
select  Month, count(*) as ride_count
from bikeshare_trip_bkp
where trip_duration >0 and trip_duration<1440
and member_casual = 'casual'
group by Month
order by Month;
```

Checking how many rides were taken at each hour of the day for each day by member and casual riders

```{sql}
select  days, extract(hour from started_at) as hours,
count(*) as rides_taken
from bikeshare_trip_bkp
where trip_duration >0 
and trip_duration <1440
and member_casual = 'member'
group by days, hours
order by days, hours desc;
```

```{sql}
select  days, extract(hour from started_at) as hours,
count(*) as rides_taken
from bikeshare_trip_bkp
where trip_duration >0 
and trip_duration <1440
and member_casual = 'casual'
group by days, hours
order by days, hours desc;
```

## Share
I used Tableau Public to carry out my visualizations. Click [here](https://public.tableau.com/app/profile/anurika.chukwunenye/viz/Bike_ridecapstone/Dashboard1) to view my interactive dashboard.

## Findings
- Members have the highest average ride duration of 655.3 mins on Sundays while Casual riders have the highest average ride duration of 666.98 Mins on Wednesdays.
- July have the highest average ride duration by members and Casual riders with 743.2 Mins and 790.5 Mins respectively.
- Electric bikes have the highest users, followed by Classic bikes and docked bikes.
- The total number of rides by Members and Casual riders are 59439 and 21390 respectively.
- Highest ride count by month is in January with 19044 rides by Members and 6090 rides by Casual riders.
- Highest ride count by day is on Tuesday by Members and Wednesday by Casual riders.
- The Members and Casual riders' peak ride hour is on the 17th hour.

## Act

### Recommendation 
Based on my analysis, I recommend the following actions
- Cyclist should target casual riders that have a longer ride duration and give them a time-limited membership offer.
- There should be a promo program targeted for Casual riders that have the characteristics of Members.
- Casual riders should be offered various membership option like monthly, quarterly, annual to cater different user preferences.


PS: This is my first Data Analytics Project and it is prone to flaws. Constructive criticism on this project is welcomed.

