# Sales Analysis
This exercise is in line with Brainwave Matrix Internship Program.

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

You are a data analyst working on a sales team in a pharmaceutical company. The business owner wants to understand how the retail outlets are performing compared to hospital sales, especially across different cities. 

## Bussiness Task

My Boss has assigned me to analyze 
1. Which Products are driving the most revenue in our pharmacy sub-channel.
2. Are there specific cities or customer types where our pharmacy sales are underperforming?
3. Which sales reps are most effective in driving pharmacy sales, can we replicate their strategies across regions?
4. Are we Pricing our top-selling products competitively, or are we losing out to hospital channel?
5. Can we identify seasonal patterns in the pharmacy sales that can help with inventory and marketing planning?

## Prepare

I downloaded my data from kaggle.  It can be found [here](https://www.kaggle.com/datasets/akanksha995579/pharma-data-analysis). I extracted the data from its zip file. The data is a pharmacy data for 2017 - 2020. All personally identifiable information was removed. The data consist of 14 columns: Distributor, Customer_Name, City, Country, Latitude, Longitude, Channel, Sub_channel, Product_Name, Product_Class, Quantity, Price, Sales, Month, Year, Name_of_Sales_Rep, Manager, Sales_Team. I created a folder for the data I need and named it appropriately.

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

I began my analysis process by finding the best selling products in the pharmacy channel based on their class.

```{sql}
select Top 10 Product_Name, Product_Class, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Product_Name, Product_class
order by Total_Sales desc;
```

### Top and Low Performing Customers and Cities

I checked Top and low performing Customers and Cities with the following codes

```{sql}
select Top 10 Customer_Name, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Customer_Name
order by Total_Sales desc;

select Top 10 Customer_Name, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Customer_Name
order by Total_Sales asc;
```
```{sql}
select Top 10 City, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by "City"
order by Total_Sales desc;

select Top 10 City, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by "City"
order by Total_Sales asc;
```

### Most Effective Sales Reps

I went further to check for Sales reps who are effective in driving the pharmacy sales with the following codes

```{sql}
select Top 5 Name_of_Sales_Rep, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Name_of_Sales_Rep
order by Total_Sales desc;
```

### Price and Quantity

To answer the question of "are we pricing our top selling products competitively, or are we loosing out to hospital channels?", I ran the following codes

```{sql}
select Top 10 Product_Name, Price, Quantity, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Product_Name,Price, Quantity
order by Total_Sales desc, Quantity desc;

select Top 10 Product_Name, Price,Quantity, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Hospital'
group by Product_Name,Price,Quantity
order by Total_Sales desc, Quantity desc;
```

### Sales by Month and Year

I checked the total sales of the Pharmacy channel in months and years to identify seasonal patterns using this code

```{sql}
select Month, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Month
order by Total_Sales desc;

select Year, sum(Sales) as Total_Sales
from dbo.pharma_data
where Channel = 'Pharmacy'
group by Year
order by Total_Sales desc;
```

## Share
I used Tableau Public to carry out my visualizations. Click [here](https://public.tableau.com/app/profile/anurika.chukwunenye/viz/Top_sellingProduct/Pharma_SalesAnalysis) to view my interactive dashboard.

## Findings
- Butzbach leads in sales, signifucantly ahead of other cities. Baesweiler and Cuhaven also performed well but are behind Butzbach. The pharmacy is underperforming in Nowy Targ.
- Mraz-KUtch Pharma Plc and Parker,Green and Emmerich are the top customers. Few customers drive a large portion of the pharmacy sales which indicates customer concentration risk.
- Ionotide is the top-selling product. It accounted for 16.5% of the total sales, followed by Tetranryl and Symdoect at 10.8% and 9.9% respectively. The product mix shows the pharmacy sales are concentrated in few products.
- Jimmy Grey and Abigail Thompsom are the top performing sales reps. The gap between the top and lower performing Sales reps is narrow, suggesting consistent performance overall.
- August has the highest sales, followed by November and March. January and April are the lowest.
- Peak sales occured in 2018 followed by a steady decline which was most significant in 2020 - could be due to market disruption like Covid-19.

## Act

### Recommendation 
Based on my analysis, I recommend the business owner to
- Focus marketing and distribution efforts on sustaining performance in Butzbach and replicating its strategy in cities like Nowy Targ and other similar markets.
- Diversify the customer base and offer incentives to mid-tier customers to grow their sales.
- Ensure stock availability for top selling products and consider bundling with less popular products.
- Identify what top sales reps do differently and use it to train others.
- Explore seasonality factors like health trends to explain and optimize low performing months.
- Investigate the cause of the decline over time and strategize recovery.




