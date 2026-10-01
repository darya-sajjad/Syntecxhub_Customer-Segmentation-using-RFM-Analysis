# All info

### Why RFM Segmentation?
Businesses generate tons of raw sales data, we need to drive growth by segmentation of customers and identifying which of the customers are the most valuable and which ones the business might be losing.
This is where RFM analysis comes in, RFM stands for *Recency*, *Frequency*, and *Monetory*.
- Recency: How recently a customer purchased?
More Recent = More Engaged
- Frequency: How often they buy?
More Purchase = More Loyal
- Monetary: How much they spend?
Higher Spend = Higher Value
Together these three numbers lets us rank customers (e.g. recent, frequent, high spend customers are champions, while low, absent, low spend customers are at risk).
`For RFM analysis the most important fields are:`
- Customer ID and Customer Name
- Order ID and Order Date
- Sales
While the raw data is at the transaction level, we will need to aggregate at the customer level. That means for each customer we need to find:
- Most recent purchase date
- Total no. of purchases (order that have placed)
- Total sales (amount they have contributed)

## Steps:
- 1. Import the dataset in to Power Bi
- 2. Identify the main columns
- 3. Create a new measure called **Max Order Date** to calculate recency:
MaxOrderDate = MAX('Superstore 2025'[Order Date])
- 4. Create a dynamic table *RFM_Calculation* which will be used to aggregate sales data at customer level
- 5. Assign scores for RFM and for that i have to create a few more columns in the RFM_Calculation table (Recency_Score, Frequency_Score, and Monetary_Score)
- 6. Now combine the scores logically in to customer segments (e.g. customers with high ranking across all three are champions and the ones with low ranking across all three are lost). We also have loyal customers, big spenders, and at risk segments depending on which dimension they perform well or poor in.
- 7. Set up clear segmentation scores for each type pf customers, this egmentation usually depends on ones business rules but I am going to use these for this segmentation:
**Champions:** R <= 2, F <= 2, M <= 2
**Loyal Customers:** R <= 3, F <= 2
**Big Spenders:** R >= 3, M <= 2
**At Risk:** R >= 3, F <= 3
**Lost:** R = 5, F = 5, M = 5
- 8. Create a new column, add these segmentation rules, and classify the customer in to meaningful segments (using RFM analysis)
This gives us powerful insights in to customer behaviour and helps us decide where to focus our marketing efforts.
- 9. Create a Date table to filter out all the values - of both RFM calculation and the main table. 
- 10. Modeling: join the tables:
-- DateTable & Main: Date - OrderDate
-- RFM_Calculation & Main: Customer ID - Customer ID
- 11. Create a few more measure (for this in a seperate table):
AvgRecency, AvgFrequency, AvgMonetary, Total Customers, Total Sales
- 12. Starting Visualization

