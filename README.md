# AtliQ-Mart-Promotions-Analytics
Analyzed FMCG promotional campaigns using MySQL and Power BI to evaluate incremental revenue, sales lift, and promotion effectiveness across Diwali 2024 and Sankranti 2025.
Business Objectives

Compare promotional campaign performance.

* Identify promotion types generating the highest incremental revenue.
* Evaluate incremental sold units across promotion types.
* Analyze product and category performance during promotional campaigns.
* Support business decisions using SQL analysis and Power BI visualizations.

Tools & Technologies

* MySQL
* Power BI
* DAX
* Power Query
* Data Modeling

  SQL Ad-hoc Analysis

 1.List products with a base price greater than 500 that are featured in the BOGOF promotion.
 Query: 
     select 
	      p.product_name,e.base_price,e.promo_type
    from fact_events e
    inner join dim_products p
    using(product_code)
    where e.base_price>500 and e.promo_type='BOGOF'
    group by p.product_name;

2.Calculate the number of stores in each city and rank cities by store.
 Query: 
     select 
        	city,count(store_id) as total_stores
    from dim_stores
    group by city
    order by total_stores desc;

3.Display each campaign's total revenue before and after promotions, with revenue.
Query: 
    select 
      	campaign_id,
         concat(round(sum(base_price * `quantity_sold(before_promo)`)/1000000,2),' M') as `revenue(before_promo)`,
         concat(round(sum(base_price * `quantity_sold(after_promo)`)/1000000,2),' M') as `revenue(after_promo)`
    from fact_events
    group by campaign_id;

 4.Calculate and rank categories by Incremental Sold Quantity Percentage (ISU%) during the Diwali campaign.
 Query: 
     with category_sales as 
(
	select
			p.category,
			 SUM(e.`quantity_sold(after_promo)`) as after_qty,
			SUM(e.`quantity_sold(before_promo)`) as before_qty
	from fact_events e 
	join dim_products p
	using(product_code)
	where campaign_id='CAMP_DIW_01'
	group by p.category),
    
isu_calculation as 
(
	select 
		category,
        round((after_qty-before_qty)*100/before_qty,2) as isu_pct
	 from category_sales
)
select
	category,
    isu_pct,
    dense_rank() over(order by isu_pct) as isu_rank
from isu_calculation
order by isu_rank;

5.Identify the top 5 products across all campaigns by Incremental Revenue Percentage (IR%), including product name, category, and IR%.
query:
    with category_sales as (
	select
			p.product_name,p.category,
			 sum(e.base_price * e.`quantity_sold(before_promo)`) as before_revenue,
			SUM(e.base_price * e.`quantity_sold(after_promo)`) as after_revenue
	from fact_events e 
	join dim_products p
	using(product_code)
	group by p.category
),
is_calculation as (
	select
		product_name,category,
        ROUND((after_revenue-before_revenue)*100/before_revenue,2) as IR_pct
	from category_sales
)
select 
	* from is_calculation
    order by IR_pct;


Power BI Dashboard

* Executive Overview
Provides a high-level view of campaign performance through key KPIs, revenue comparisons, and overall promotional impact.

* Promotion & Product Analysis
Analyzes promotion effectiveness, product and category performance, incremental revenue, and incremental sold units to identify successful promotional strategies.

* Store Performance Analysis
Evaluates store-level promotional performance, ranks top- and bottom-performing stores, and compares results across cities to identify performance trends.


Key Findings
* Revenue Growth: Revenue increased from ₹140.7 million before promotions to ₹347.9 million after promotions.
* Incremental Revenue: Promotional campaigns generated an incremental revenue increase of 147.2% compared with pre-promotion revenue.
* Category Performance: The Home Appliances category recorded a 264.4% increase in sold units after promotions, indicating a strong sales lift.
* City Performance: Bengaluru emerged as a leading city for promotional sales, generating ₹51 million in revenue.

