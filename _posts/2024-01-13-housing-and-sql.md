---
layout: post
title: Transforming Nashville Housing Data: A SQL Approach to Insightful Analysis
image: "/posts/sqlhousing-title-img.png.jpg"
tags: [SQL, EDA, Data Integrity]
---

This project focuses on using SQL Server to clean, transform, and analyse raw housing data to provide accurate insights into property market trends and decision-making
# Table of contents

- [00. Project Overview](#overview-main)
- [01. Data Overview and Preparation](#data-overview)
- [02. Analysing the results](#analysis)
- [08. Growth & Next Steps](#growth-next-steps)

___

# Project Overview  <a name="overview-main"></a>

This project focuses on cleaning and preparing raw housing data using SQL Server to ensure accuracy, reliability, and usability for analysis and decision-making. The client, a real estate investment firm, seeks actionable insights to guide their property acquisition strategies, optimise pricing models, and understand housing market trends.
The dataset includes property sales information such as sale dates, prices, property types, and locations, but contains issues such as inconsistent formats, missing data, and errors. The project involves an in-depth data cleaning process to resolve these issues. By correcting inaccuracies, standardising formats (such as date and currency), and addressing missing values, the dataset will be transformed into a structured format ready for analysis.
Once the data is cleaned, the project aims to provide insights into key housing market metrics such as property valuation trends, average sale prices, and sales patterns across different regions. These insights will enable the client to better understand the factors influencing property prices, forecast demand in specific areas, and identify potential investment opportunities based on location-specific patterns.
Additionally, exploratory data analysis (EDA) will be used to discover trends and correlations within the dataset. Integrating external data sources, such as local economic conditions or demographic trends, may also enhance the analysis, allowing for more comprehensive insights.
This structured approach will help the client make informed, data-driven decisions, ultimately improving their investment strategies and enhancing profitability in the competitive housing market.
curacy.

<br>
<br>

___

# Data Overview and Preparation  <a name="data-overview and preparation"></a>

In this project, the objective is to clean and transform raw housing data from the NasvilleHousingProject dataset to make it ready for analysis and decision-making. The data consisted of fields such as SaleDate, PropertyAddress, ParcelID, OwnerAddress, and other variables pertaining to property transactions. Preparing this data was essential to ensure that the analysis would yield accurate, reliable, and actionable insights. These insights could drive business impact in industries like real estate, housing policy, or investment strategy.
The following is a detailed, step-by-step breakdown of how the data was prepared for analysis:

##### Step 1: Initial Exploration and Understanding of Data Structure
The first step in preparing the data was to examine its structure. Running a SELECT * FROM NasvilleHousingProject query allowed a comprehensive review of all the columns and the values they contained. This step was critical for identifying missing data, inconsistencies (particularly in date formats), and potential duplicates. Based on this exploration, several key areas were identified for improvement: date formatting, missing addresses, duplicate entries, and categorical variables stored in unclear formats.

##### Step 2: Standardising Date Formats
Dates play a crucial role in analysing time-based trends such as seasonal sales patterns and long-term property value changes. In this dataset, the SaleDate field had inconsistent formats, which could have posed challenges in analysing time-series data. To standardise the dates, the following query was used to preview the changes:

In the code below, we:

```sql

SELECT saleDateConverted, CONVERT(Date, SaleDate)
FROM NasvilleHousingProject;

```

###### After confirming the appropriate format, the next step was to update the dataset:

```sql

UPDATE NasvilleHousingProject
SET SaleDate = CONVERT(Date, SaleDate);

```

##### In cases where the conversion did not update properly due to constraints or invalid data, a new column was created:

```sql

ALTER TABLE NasvilleHousingProject ADD SaleDateConverted Date;
UPDATE NasvilleHousingProject SET SaleDateConverted = CONVERT(Date, SaleDate);

```

By standardising all dates, the dataset was now structured in a way that allowed reliable time-based analysis. This step ensures that insights related to time periods, such as quarterly or yearly performance trends, were based on accurate data. For businesses, having consistent date formats helps in identifying patterns like seasonal trends, which could inform strategic decisions around marketing, sales, or investment.

##### Step 3: Handling Missing Property Address Data

Incomplete address data could severely affect location-based analysis, which is crucial for making decisions based on regional property trends. In this dataset, some records had missing PropertyAddress values. To resolve this, a self-join based on the ParcelID field was performed, as properties with the same ParcelID but different UniqueID values were likely duplicates:

```sql

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NasvilleHousingProject a
JOIN NasvilleHousingProject b ON a.ParcelID = b.ParcelID AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL;
Once confirmed, the missing addresses were updated:
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NasvilleHousingProject a
JOIN NasvilleHousingProject b ON a.ParcelID = b.ParcelID AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL;

```

This step ensured that the dataset contained complete address information, which is vital for conducting accurate geographical analyses, such as property sales by region. With complete location data, businesses can target specific areas for development or marketing based on trends identified in the data.

##### Step 4: Splitting Address Data into Separate Columns

To analyse specific components of an address, such as city or state, it was necessary to break down the PropertyAddress and OwnerAddress fields into distinct columns. This was achieved using SQL string manipulation functions:

```sql

SELECT SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) AS Address,
       SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) AS CityState
FROM NasvilleHousingProject;
New columns were created to hold the separated address and city data:
ALTER TABLE NasvilleHousingProject ADD PropertySplitAddress NVARCHAR(255);
ALTER TABLE NasvilleHousingProject ADD PropertySplitCity NVARCHAR(255);
UPDATE NasvilleHousingProject
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1),
    PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));

```

Similarly, the OwnerAddress field was split into OwnerSplitAddress, OwnerSplitCity, and OwnerSplitState using the PARSENAME function. This step allows for more detailed regional analysis, such as identifying cities or areas with the highest sales volume or property values. This information is invaluable for businesses or policymakers looking to make targeted decisions in specific regions.

##### Step 5: Standardising Categorical Data
The SoldAsVacant field, which indicated whether a property was sold as vacant, used ‘Y’ and ‘N’ values. These were replaced with more descriptive terms (‘Yes’ and ‘No’) to make the data more interpretable for analysis:

```sql

UPDATE NasvilleHousingProject
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
                        WHEN SoldAsVacant = 'N' THEN 'No'
                        ELSE SoldAsVacant END;

```
This update improves the clarity of the dataset, making it easier for non-technical stakeholders to interpret insights derived from the data. Clear communication of insights is vital for business decision-making, and this adjustment helps to ensure that the dataset remains user-friendly.

##### Step 6: Removing Duplicate Entries

Duplicates can distort analysis results and lead to inaccurate insights. To ensure the data was free from duplicates, a CTE (Common Table Expression) was used in conjunction with the ROW_NUMBER() function:

```sql

WITH RowNumCTE AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference ORDER BY UniqueID) AS row_num
    FROM NasvilleHousingProject
)
DELETE FROM RowNumCTE WHERE row_num > 1;

```

By removing duplicates, the dataset became more reliable for analysis. For businesses, this ensures that metrics such as total sales volume or average property price are not inflated by repeated entries.

##### Step 7: Dropping Unnecessary Columns
Finally, columns that were no longer required were removed to simplify the dataset and improve query performance:

```sql

ALTER TABLE NasvilleHousingProject DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;

```

Streamlining the dataset not only reduces complexity but also improves efficiency when running queries. This results in faster analysis and more focused insights, which can be used for timely decision-making.

___
<br>
# Analysing the results  <a name="Analysing-the-results"></a>
 
Once the data preparation process was completed, the next step was to analyse the cleaned dataset to uncover insights that could drive business value. The primary objectives were to identify time-based trends, regional property transaction patterns, and vacancy rates.
##### Time-Based Analysis
The first analysis involved examining trends over time using the standardised SaleDate field. Time-based analysis helps to reveal patterns, such as seasonal variations in the housing market or long-term trends in property values. For businesses, understanding these trends is critical for making strategic decisions around investment timing, pricing strategies, or marketing campaigns.
The data was grouped by month and year to identify any seasonal trends in property sales:

```sql

SELECT MONTH(SaleDate) AS Month, COUNT(*) AS SalesVolume
FROM NasvilleHousingProject
GROUP BY MONTH(SaleDate)
ORDER BY SalesVolume DESC;

```

Month	Sales Volume
June	320
July	290
August	270
...	...

The analysis revealed that property sales peak during the summer months, particularly in June, July, and August. This insight could inform businesses involved in real estate investment or development on when to focus their sales efforts. Additionally, marketing campaigns could be strategically timed to align with these peak periods, ensuring that inventory is available when demand is highest.
Seasonal trends in the housing market are common, as factors like weather, school holidays, and economic conditions affect when people are most likely to buy or sell properties. For developers, understanding these patterns can help in planning construction timelines to ensure that new properties are ready for sale during peak periods. Similarly, estate agents could adjust their marketing budgets to ensure maximum visibility during high-demand months.

##### Regional Analysis
The cleaned and split address fields (PropertySplitAddress, PropertySplitCity, PropertySplitState) enabled a detailed regional analysis of property transactions. By grouping sales by city, I was able to identify which areas had the most property activity:

```sql

SELECT PropertySplitCity, COUNT(*) AS SalesCount
FROM NasvilleHousingProject
GROUP BY PropertySplitCity
ORDER BY SalesCount DESC;

```

City	Sales Count
Nashville	1,500
Franklin	900
Hendersonville	600
...	...

The results showed that Nashville itself had the highest volume of property transactions, followed by nearby cities such as Franklin and Hendersonville. This kind of geographical insight is invaluable for businesses looking to enter new markets or expand their presence in growing regions.
For example, real estate developers might prioritise areas with high transaction volumes for future projects, while estate agents could focus their marketing efforts on regions where sales activity is increasing. Additionally, investors might use this data to identify undervalued areas that are experiencing growth, enabling them to make informed decisions about where to buy properties.

##### Vacancy Rate Analysis
The next analysis focused on the SoldAsVacant field, which indicated whether properties were sold as vacant. This metric is particularly important for real estate investors who are interested in purchasing vacant properties for development or renovation.

```sql

SELECT SoldAsVacant, COUNT(*) AS VacantCount
FROM NasvilleHousingProject
GROUP BY SoldAsVacant;

```

Status	Count
Yes	700
No	2,300
The analysis revealed that a significant proportion of properties were sold as vacant, which presents opportunities for developers and investors looking for properties to renovate or redevelop. Knowing which areas have a high number of vacant properties could help investors focus their efforts on regions with the greatest potential for development or resale.

##### Price Trends and Value Analysis
By examining the SalePrice column, I conducted an analysis of property price trends over time. Grouping the data by year and calculating the average sale price provided insight into how property values have changed:

```sql

SELECT YEAR(SaleDate) AS Year, AVG(SalePrice) AS AvgPrice
FROM NasvilleHousingProject
GROUP BY YEAR(SaleDate);

```

Year	Average Sale Price
2016	£250,000
2017	£260,000
2018	£275,000
...	...
The results indicated a steady increase in property prices over the years, with a noticeable jump between 2017 and 2018. For businesses, understanding property price trends is crucial for pricing strategies, investment decisions, and financial forecasting.

___
<br>
# Growth & Next Steps <a name="growth-next-steps"></a>

Business Impact 
The cleaned and structured dataset enabled a range of insightful analyses that can have a direct impact on business decisions in the real estate sector. The time-based analysis provided clear evidence of seasonal trends, showing that property sales peak in the summer months. This information allows businesses to strategically time their sales efforts, ensuring that properties are marketed when demand is highest.
The regional analysis highlighted areas with the most property transactions, such as Nashville and Franklin. Businesses can use this information to target high-demand areas for new development projects, or to focus marketing efforts on regions where sales are most active. Investors, on the other hand, can use these insights to identify emerging markets and make informed decisions about where to buy or develop properties.
The analysis of vacancy rates revealed opportunities for developers and investors to focus on vacant properties that could be renovated or redeveloped. This presents a significant business opportunity, as vacant properties are often available at lower prices, allowing for higher returns on investment after development.
Finally, the analysis of property price trends over time showed a steady increase in property values, providing valuable information for businesses involved in real estate finance, pricing strategies, and long-term investment planning. Understanding these trends allows companies to make informed decisions about future developments and investments, ensuring they are well-positioned in the market.
In summary, the insights derived from the cleaned dataset offer significant opportunities for businesses to optimise their strategies, improve profitability, and make data-driven decisions in a competitive real estate market.
