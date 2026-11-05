# Marketing analytics Power BI interactive dashboards

## :hammer_and_wrench: Utilized tools
- Microsoft Excel
- Microsoft SQL Server
- Python 
- Power BI

## :memo: Project overview
The project is based on  ShopEasy,an e-commerce retailer which has recently launched a stream of marketing campaigns to draw traction to their platform. However, they are still facing a decline in customer engagement and conversion rates. They require an analysis of their marketing campaigns in order to improve their conversion rate and customer engagement. Furthermore, they want to determine the customer feedback average scores.

## :mag: Key Performance indicators for analysis;
-	Conversion rate – percentage of customers who complete the lifecycle stages due to the marketing campaign
-	Customer engagement rate – Metrics; number of clicks on campaigns, social media impressions and actual engagement on campaigns (e.g. comments)
-	Customer feedback score – reviews on products 

## :clipboard: Data cleaning notes
I imported the data (i.e. fact and dim tables) into Power BI via SQL and their respective scripts. The data was further cleaned to ensure a standardized format which is ready for analysis.
Fact_customer_journey table:
-	VisitDate column from Date/Time to Date
Fact_customer_reviews table:
-	ReviewDate column from Date/Time to Date, ensure to select localized selection to avoid errors
-	Rating column from Decimal to Whole number
Fact_engagement_data table:
-	Views, Clicks and Likes column, from Text to Whole number
-	EngagementDate column Date/Time to Date

 Dim_customers:
-	Age column from Text to Whole number

Categorizing the products according to; Low, Medium and High tiers based on a price range
```sql
SELECT
	ProductID,
	ProductName,
	Price,
CASE
	WHEN Price < 50 THEN 'Low'
	WHEN Price BETWEEN 50 AND 200 THEN 'Medium'
	ELSE 'High'
END AS PriceCategory
FROM 
PortfolioProject_MarketingAnalytics.dbo.products
```

SQL statement to join dim_customers with dim_geography to enrich customer data with geographic information
```sql
SELECT
	C.CustomerID,
	c.CustomerName,
	c.Email,
	c.Gender,
	c.Age,
	g.Country,
	g.City
FROM
	 PortfolioProject_MarketingAnalytics.dbo.customers AS c
LEFT JOIN
PortfolioProject_MarketingAnalytics.dbo.geography AS g
ON
	c.GeographyID = g.GeographyID
```

Cleaning the white spacing in the ReviewText column
```sql
SELECT 
	ReviewID,
	CustomerID,
	ProductID,
	ReviewDate,
	Rating,
	REPLACE(ReviewText, '  ', ' ') AS ReviewText
FROM 
PortfolioProject_MarketingAnalytics.dbo.customer_reviews
```

SQL statement to clean and normalize the engagement data table
```sql
SELECT
	EngagementID,
	ContentID,
	CampaignID,
	ProductID,
	UPPER(REPLACE(ContentType, 'Socialmedia','Social Media')) AS ContentType,
LEFT(ViewsClicksCombined, CHARINDEX('-', ViewsClicksCombined) -1) AS Views,
RIGHT(ViewsClicksCombined, LEN(ViewsClicksCombined) - CHARINDEX('-', ViewsClicksCombined)) AS Clicks,
Likes,
FORMAT(CONVERT(DATE, EngagementDate), 'dd-MM-yyyy') AS EngagementDate
FROM 
PortfolioProject_MarketingAnalytics.dbo.engagement_data
WHERE
	ContentType != 'Newsletter'
```

Common Table Expression – CTE to identify and tag duplicate records which will be subsequently used to clean and standardize the required CustomerJourney table 
```sql
WITH DuplicateRecords AS (
	SELECT
		JourneyID,
		CustomerID,
		ProductID,
		VisitDate,
		Stage,
		Action,
		Duration,
		ROW_NUMBER() OVER (
			PARTITION BY CustomerID, ProductID, VisitDate, Stage, Action
			ORDER BY JourneyID
		) AS row_num
	FROM
		PortfolioProject_MarketingAnalytics.dbo.customer_journey
)

SELECT *
FROM
	DuplicateRecords
WHERE
	row_num > 1
ORDER BY
	JourneyID

SELECT
	JourneyID,
	CustomerID,
	ProductID,
	VisitDate,
	Stage,
	Action,
	COALESCE(Duration, avg_duration) AS Duration
FROM
	(
		SELECT
			JourneyID,
			CustomerID,
			ProductID,
			VisitDate,
			UPPER (Stage) AS Stage,
			Action,
			Duration,
			AVG(Duration) OVER (PARTITION BY VisitDate) AS avg_duration,
			ROW_NUMBER () OVER (
PARTITION BY CustomerID, ProductID, VisitDate, UPPER(Stage), Action
			ORDER BY JourneyID
			) AS row_num
		FROM
			PortfolioProject_MarketingAnalytics.dbo.customer_journey
	) AS subquery
WHERE
	row_num = 1;
```

Wrote a python script to pull data from the SQL dim_customer_review table in order to create a csv file which will have a customer sentiment score based on their reviews.
Python script: ![ipynb](https://github.com/SRMadia-Portfolio/Marketing-analytics-Power-BI-interactive-dashboards-/blob/d8befdc03f3de7a02d505e124a5a9e843b0dcb5f/PortfolioProject-MarketingAnalytics.ipynb)

## :notebook: Statistical analysis on Power BI
## Overview
### Decreased Conversion Rates:
The highest declined is noted between January to February (i.e. 17.3% - 10.2%) and September to October (i.e. 12.2% - 6.1%). A change in seasonal sports periods accounts for the decline, with products such as Ski boots, Surfboards, Tennis rackets and Soccer balls having the highest conversion rate drop. October had the lowest conversion rate which indicates that the marketing campaigns need to be looked into to improve performance for the seasonal products during that period.

### Reduced Customer Engagement:
The highest views were in January (i.e. 982 444) which had a gradual decline throughout the year, with December having the lowest viewership (i.e. 510 022). Despite the high viewership, engagement (i.e. clicks and likes) with the campaigns remain low compared to the viewership. Blog posting accounted for the highest contribution towards viewership throughout the year besides October – December.

### Customer Feedback Analysis:
The customer reviews depict a positive response whereby 431 reviews gave a 4-star rating and 409 reviews gave a 5-star rating. On the opposite end, a small portion of customers gave lower ratings whereby 80 reviews gave a 1-star rating and 153 reviews gave a 2-star rating. Furthermore, a positive response is noted with the sentiment analysis whereby 840 reviews had a positive sentiment. However, 226 reviews gave a negative sentiment with mixed reviews of neutrality or disappointment. 

## :bar_chart: Data Visualization - Power BI 
A detailed break down is given on four interactive dashboards; an overview of the entire project, conversion details, social media details and customer review details. Each of the dashboards has a slicer for a product drill down, year (i.e. 2023 – 2025) drill down and month (i.e. January – December) drill down to give a detailed break down of the data such as conversion rate by product or ratings given by the different customer reviews.
Power BI - interactive dashboards: ![pbix](vhttps://github.com/SRMadia-Portfolio/Marketing-analytics-Power-BI-interactive-dashboards-/blob/d344e4c04f46172be7da3fcc551b3fb035985dd8/Marketing%20analytics.pbix)

## :pushpin: Actions and limitations
### Conversion rate – percentage of customers who complete the lifecycle stages due to the marketing campaign
Increase conversion rate
Do a comparison study to look at campaigns which improved the conversion rates of high performing products such as Ski boots, Soccer ball and yoga mats, and apply similar strategies to the low performing products. Furthermore, determine if there are any active campaigns during periods with low conversion rates such as May and October. If any campaigns are active, determine if the campaigns align with the sports that are currently in season. 

### Customer engagement rate – Metrics; number of clicks on campaigns, social media impressions and actual engagement on campaigns.
Improve the customer engagement
The campaign’s viewership is relatively high with low engagement. Reframe the campaigns to be more end user engaging and get the clientele to participate in the campaigns through their own social media platforms. Collaborate with a variety of influencers to garner different engagement mechanisms for the campaigns. 

### Customer feedback score – reviews on products
There’s sufficient data to interpret mixed and negative feedback of products from 2023 – 2025, determine whether the reviews are reoccurring from the same products. Have a minimum threshold of number of mixed or negative reviews a product can have in a month or quarterly. Look at the factors contributing to the mixed or negative reviews and address them.
