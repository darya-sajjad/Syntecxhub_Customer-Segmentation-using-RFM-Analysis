# Dax 
All the Dax code used for doing the customer segmentation RFM analysis:

## Table: Superstore 2025 (Main)
### Measure: Max Order Date
MaxOrderDate = MAX('Superstore 2025'[Order Date])

## Table: RFM_Calculation 
RFM_Calculation = 
SUMMARIZE(
    'Superstore 2025',
    'Superstore 2025'[Customer ID],
    'Superstore 2025'[Customer Name],
    "Recency", ABS(DATEDIFF(
        CALCULATE(MAX('Superstore 2025'[Order Date])),
        CALCULATE(MAX('Superstore 2025'[Order Date]), ALL('Superstore 2025')), DAY)),
    "Frequency", DISTINCTCOUNT('Superstore 2025'[Order ID]),
    "Monetary", SUM('Superstore 2025'[Sales]))

### Customer Segmentation (Assigning Scores)
Customer_Segment = 
SWITCH(
        TRUE(),
        -- Champions
        RFM_Calculation[Recency_Score] <= 2 &&
        RFM_Calculation[Frequency_Score] <= 2 &&
        RFM_Calculation[Monetary_Score] <=2, "Champions",

        -- Loyal
        RFM_Calculation[Recency_Score] <= 2 &&
        RFM_Calculation[Frequency_Score] <= 3, "Loyal",

        -- Big Spenders
        RFM_Calculation[Recency_Score] <= 2 &&
        RFM_Calculation[Monetary_Score] >= 3, "Big Spenders",

        -- At Risk
        RFM_Calculation[Recency_Score] >= 3 &&
        RFM_Calculation[Frequency_Score] <= 3, "At Risk",

        -- Lost
        RFM_Calculation[Recency_Score] = 5 &&
        RFM_Calculation[Frequency_Score] = 5 &&
        RFM_Calculation[Monetary_Score] = 5, "Lost",

        -- Default
        "Others"
)

### Recency Score
Recency_Score = 
VAR RankRecency =
    RANKX(
        ALL(RFM_Calculation),
        RFM_Calculation[Recency],,ASC)

VAR TotalCust = COUNTROWS(ALL(RFM_Calculation))
RETURN
CEILING(DIVIDE(RankRecency * 5, TotalCust), 1)

### Frequency Score
Frequency_Score = 
VAR RankFreq =
    RANKX(
        ALL(RFM_Calculation),
        RFM_Calculation[Frequency],,DESC)

VAR TotalCust = COUNTROWS(ALL(RFM_Calculation))
RETURN
CEILING(DIVIDE(RankFreq * 5, TotalCust), 1)

### Monetary Score
Monetary_Score = 
VAR RankMon =
    RANKX(
        ALL(RFM_Calculation),
        RFM_Calculation[Monetary],,DESC)

VAR TotalCust = COUNTROWS(ALL(RFM_Calculation))
RETURN
CEILING(DIVIDE(RankMon * 5, TotalCust), 1)

## Table: DateTable
DateTable = 
ADDCOLUMNS (
    CALENDAR (DATE(2021,1,1), DATE(2025,12,31)),  -- adjust start and end dates
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM"),
    "Year-Month", FORMAT([Date], "YYYY-MM"),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Day", DAY([Date]),
    "Day of Week", WEEKDAY([Date], 2),   -- 1 = Sunday start, 2 = Monday start
    "Day Name", FORMAT([Date], "dddd"),
    "Week Number", WEEKNUM([Date], 2)
)

## Table: Tbl_Measures
Tbl_Measures = Row("Dummy", BLANK())
*Has a dummy column to help create the table.*

- This table includes all the **Measures** needed for the RFM Analysis.

### 1. Total Customer Count
Total Customers = DISTINCTCOUNT('Superstore 2025'[Customer ID])

### 2. Total Sales Amount
Total Sales = SUM('Superstore 2025'[Sales])

### 3. Average RFM
- Average Recency
AvgRecency = AVERAGE(RFM_Calculation[Recency])

- Average Frequency
AvgFrequency = AVERAGE(RFM_Calculation[Frequency])

- Average Monetary
AvgMonetary = AVERAGE(RFM_Calculation[Monetary])

### 4. Year on year RFM Variance
- Year on year Recency Variance
YoY RecencyVar = 
 VAR LY = CALCULATE([AvgRecency], SAMEPERIODLASTYEAR(DateTable[Date]))
 VAR LYFormatted = FORMAT(LY, "#,0")
 var yoyCalc = DIVIDE(([AvgRecency] - LY) , LY)
 var yoyCalcFormatted = FORMAT(yoyCalc,"0.0%;(0.0%)")
 
RETURN
 SWITCH(
    True(),
    yoyCalc >0, UNICHAR(9650) & " " & yoyCalcFormatted & " | LY " & LYFormatted,
    ISBLANK(LY), UNICHAR(8211) & " | No Data for Last Year ",
    

    UNICHAR(9660) & " " & yoyCalcFormatted  & " | LY " & LYFormatted
 )

- Year on year Frequency Variance
YoY FrequencyVar = 
 VAR LY = CALCULATE([AvgFrequency], SAMEPERIODLASTYEAR(DateTable[Date]))
 VAR LYFormatted = FORMAT(LY, "#,0")
 var yoyCalc = DIVIDE(([AvgFrequency] - LY) , LY)
 var yoyCalcFormatted = FORMAT(yoyCalc,"0.0%;(0.0%)")
 
RETURN
 SWITCH(
    True(),
    yoyCalc >0, UNICHAR(9650) & " " & yoyCalcFormatted & " | LY " & LYFormatted,
    ISBLANK(LY), UNICHAR(8211) & " | No Data for Last Year ",
    

    UNICHAR(9660) & " " & yoyCalcFormatted  & " | LY " & LYFormatted
 )

- Year on year Monetary Variance
YoY MonetaryVar = 
 VAR LY = CALCULATE([AvgMonetary], SAMEPERIODLASTYEAR(DateTable[Date]))
 VAR LYFormatted = FORMAT(LY/1000, "$#,0.00") & "K"
 var yoyCalc = DIVIDE(([AvgMonetary] - LY) , LY)
 var yoyCalcFormatted = FORMAT(yoyCalc,"0.0%;(0.0%)")
 
RETURN
 SWITCH(
    True(),
    yoyCalc >0, UNICHAR(9650) & " " & yoyCalcFormatted & " | LY " & LYFormatted,
    ISBLANK(LY), UNICHAR(8211) & " | No Data for Last Year ",
    

    UNICHAR(9660) & " " & yoyCalcFormatted  & " | LY " & LYFormatted
 )

###  5. Year on year Sales Variance
YoY SalesVarience = 
VAR LY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DateTable[Date]))
VAR LYFormatted = FORMAT(LY/1000, "$#,0") & "K"
VAR yoyCalc = DIVIDE([Total Sales] - LY, LY)
VAR yoyCalcFormatted = FORMAT(yoyCalc, "0.0%; (0.0%)")

RETURN
    SWITCH(
        TRUE(),
        yoyCalc > 0, UNICHAR(9650) & " " & yoyCalcFormatted & " | LY " & LYFormatted,
        ISBLANK(LY), UNICHAR(8211) & " | No Data for Last Year",
        yoyCalc = 0, UNICHAR(8211) & " | No Change from Last Year",
        UNICHAR(9660) & " " & yoyCalcFormatted & " | LY " & LYFormatted
        )

### 6. Year on year RFM Color
- YoY Recency Color
recencyYoYColor = 
 VAR LY = CALCULATE([AvgRecency],SAMEPERIODLASTYEAR(DateTable[Date]))
 VAR YOY = DIVIDE([AvgRecency] - LY ,LY)
RETURN
 SWITCH(TRUE(),
    YOY > 0, "#00B200", 
    OR(ISBLANK(LY), LY = 0),"Orange", "#FF0000")

- YoY Frequency Color
FrequencyYoYColor = 
 VAR LY = CALCULATE([AvgFrequency],SAMEPERIODLASTYEAR(DateTable[Date]))
 VAR YOY = DIVIDE([AvgFrequency] - LY ,LY)
RETURN
 SWITCH(TRUE(),
    YOY > 0, "#00B200", 
    OR(ISBLANK(LY), LY=0),"Orange", "#FF0000")

- YoY Monetary Color
monetaryYoYColor = 
 VAR LY = CALCULATE([AvgMonetary],SAMEPERIODLASTYEAR(DateTable[Date]))
 VAR YOY = DIVIDE([AvgMonetary] - LY ,LY)
RETURN
 SWITCH(TRUE(),
    YOY > 0, "#00B200", 
    OR(ISBLANK(LY),LY=0),"Orange", "#FF0000")

### 7. Last Year Sales Check
Check_LY_Sales = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DateTable[Date]))

