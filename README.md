# Marketing-analytics-Power-BI-interactive-dashboards

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

