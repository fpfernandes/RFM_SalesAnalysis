# SQL | POWER BI
Dashboard designed to provide a comprehensive overview of sales and customer data from a scale model vehicle factory using SQL to conduct RFM Analysis for customer segmentation and Power BI to craft an interactive and informative dashboard.<br/>

**Key Features:**<br/>
- Sales Revenue Analysis.
- Customer Data Segmentation using RFM (Recency, Frequency, Monetary) analysis.
- Insightful Visualizations.
- Customizable Reports.<br/>

**RFM Analysis in SQL**<br/>
 It is an indexing technique that uses past purchase behaviour to segment customers. An RFM report is a way of segmenting customers using three key metrics:
- Recency: How long ago their last purchase was?
- Frequency: How often they purchase?
- Monetary value: How much they spent?<br/>
  
Data points used in RFM Analysis:<br/>
- Recency = last order date
- Frequency = Count of total orders
- Monetary value = Total spent<br/>
  
It was created a three separate dimensions (R, F, M) using the scale from 1 to 4 and then a single RFM score by concatenating the three scores (e.g., RFM = 321, where 3 represents high recency, 2 represents medium frequency, and 1 represents low monetary value).<br/>

The segment was defined was follows:<br/>
- High-Value Customers: High R, High F, High M
- New Customers: High R, Low F, Low M
- Loyal Customers: High F, High M
- Churning Customers: Low R, Low F, Low M<br/>
  
Below is my code in SQL to define RFM segments:<br/>

```sql
with RFM as
(
select
	CUSTOMERNAME,
	sum(SALES) as monetary_value,
	avg(SALES) as avg_monetary_value,
	count(distinct ORDERNUMBER) as frequency,
	max(ORDERDATE) as last_order_date,
	(
		select max(ORDERDATE)
		from [Projects].[dbo].[sales_data_samp]
	) as max_order_date,
	DATEDIFF(DD, max(ORDERDATE), (select max(ORDERDATE) from [Projects].[dbo].[sales_data_samp])) as recency
from [Projects].[dbo].[sales_data_samp]
group by CUSTOMERNAME
),
calc_rfm as
(
	select R.*,
		NTILE(4) OVER (order by recency desc) as rfm_recency,
		NTILE(4) OVER (order by frequency) as rfm_frequency,
		NTILE(4) OVER (order by monetary_value) as rfm_monetary
	from RFM as R
)
select C.*,
	rfm_recency + rfm_frequency + rfm_monetary as RFM_cell,
	cast(rfm_recency as varchar) + cast(rfm_frequency as varchar) + cast(rfm_monetary as varchar) as RFM_column
into #RFM
from calc_rfm as C

select 
	CUSTOMERNAME,
	rfm_recency , 
	rfm_frequency,
	rfm_monetary,
	RFM_column,
	case
		when RFM_column in (111, 112, 121, 122, 123, 132, 211, 212, 221, 222) then 'Churning Customers'
		when RFM_column in (133, 144, 223, 232, 233, 234, 244) then 'Loyal Customers'
		when RFM_column in (311, 322, 332, 412, 421, 422) then 'New Customers'
		when RFM_column in (333, 343, 344, 423, 433, 434, 443, 444) then 'High-Value Customers'
	end as RFM_segment
from #RFM
```
<br/>
**I used the following skills to manipulate the data and create the dashboard in Power BI:**<br/>
- Slicers
- DAX functions
- Charting
- Formatting
- Power Query
- Data manipulation

Below you can find a picture of the final dashboard, but if you want to access the live dashboard, please click on the following link:<br/>

<iframe title="RFM_Dash" width="600" height="373.5" src="https://app.powerbi.com/view?r=eyJrIjoiNGE2OTNhYjQtY2IwYy00NGZlLWE4ODAtNThjYTNhYjM0M2YzIiwidCI6ImNmYjlhNzBkLTMyY2UtNDM1NS05ZGRmLWMwOTFlOTZiZGIxYyJ9" frameborder="0" allowFullScreen="true"></iframe>
