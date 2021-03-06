## Business-Metrics-With-DAX
> This repository aims to transform the most critical business logic into code with Dax so you can use these measures on Microsoft Power BI and then utilize them in the charts to analyze the results and hopefully find some patterns. 
- COGS
```dax
COGS = 
     SUMX(
         'historical-sales', 
         'historical-sales'[Sales Qty] * RELATED(Items_List[item_cost]) * IF('historical-sales'[type] = "order", 1, -1) 
         * IF('historical-sales'[type]= "return" , -1, 1) 
     )
```
- COGS%
```dax
COGS % = DIVIDE([COGS],SUMX('historical-sales','historical-sales'[Total Sales]))
```
- GROSS PRROFIT
```dax
Gross Profit = 
     SUMX(
         'historical-sales', 'historical-sales'[Total Sales] - CALCULATE([COGS]) * IF('historical-sales'[type] = "order", 1, -1)
        * IF('historical-sales'[type] = "return", -1, 1)
         )
```
- GROSS MARGIN
```dax
Gross Margin = 
VAR _TotalSales = SUMX('historical-sales', [Total Sales])
VAR _Margin =
    DIVIDE(_TotalSales - CALCULATE([COGS]),_TotalSales)
RETURN
_Margin
```
- CALENDAR
```dax
Calendar = ADDCOLUMNS(CALENDARAUTO(),
"Year", YEAR([Date]),
"MonthNo", MONTH([Date]),
"Month", FORMAT([Date],"mmm"),
"Day", DAY([Date]), 
"Quarter", FORMAT([Date],"\QQ"),
"YearMonth", FORMAT([Date],"YYYY-MM"),
"Week", FORMAT([Date], "MMMM") & " -  Week " & ROUNDDOWN(DATEDIFF(DATE(YEAR([Date]), MONTH([Date]), 1), [Date], DAY)/7,0)+1,
"WeekDay", FORMAT([Date],"ddd"))
```
- AVG RTP
```dax
AVG RTP = 
    VAR _Sales = SUMX('historical-sales','historical-sales'[Total Sales])
    VAR _Result = DIVIDE(_Sales,[Net Sold Quantity])
RETURN
_Result 
```
- AVG COGS
```dax
AVG COGS = DIVIDE([COGS],[Net Sold Quantity])
```
- AVG MARKUP
```dax
AVG MARKUP = DIVIDE(_Measures[AVG RTP],_Measures[AVG COGS])
```
- ROI
```dax
ROI = 
VAR _NetIncome = 
   SUMX(
     'historical-sales', 
     'historical-sales'[Total Sales] - CALCULATE( [COGS] ) 
   )
RETURN DIVIDE(_NetIncome, SUMX('historical-sales','historical-sales'[Sales Qty] * RELATED(Items_List[item_cost]))+SUMX(Stocks,Stocks[Stock Cost]))
```
- STOCK COST
```dax
Stock Cost = SUMX('Stocks',[Stock Qty] * RELATED(Items_List[item_cost]))
```
- STOCK VALUE
```dax
Stock Value = SUMX('Stocks',[Stock Qty] * RELATED(Items_List[rtp]))
```
- TURNOVER
```dax
Turnover = 
VAR turnover = DIVIDE([Net Sold Quantity], SUMX(Stocks,Stocks[Stock Qty])+[Net Sold Quantity])
RETURN IF(LEN(turnover), turnover, 0)
```
- TURNOVER QTY
```dax
Turnover Qty = DIVIDE([Net Sold Quantity],SUMX(Stocks,Stocks[Stock Qty]))
```
- LOW MOVING ITEMS
```dax
Low Moving Items = 
IF([Turnover] < AVERAGEX(ALL('historical-sales'), [Turnover]), IF([Turnover] = 0, "Unsold", "Low"), "Not Low")
```
- NET SALES PER SQUARE METER
```dax
Net Sales Per Square Meter = 
DIVIDE(
    SUMX(
        'historical-sales','historical-sales'[Total Sales]),
        AVERAGEX(Mapping,Mapping[Space Area]))
```
- NET SALES YTD
```dax
Net Sales YTD = 
TOTALYTD(
    SUMX('historical-sales','historical-sales'[Total Sales]) ,
    'Calendar'[Date] 
    )
```
- NET SOLD QUANTITY
```dax
Net Sold Quantity = SUMX('historical-sales','historical-sales'[Sales Qty])
```
- RANK NET SALES
```dax
Rank_Net_Sales = 
VAR rowcount = COUNTROWS(Items_Ranking)
VAR ranking = RANK.EQ(Items_Ranking[Net Sales],Items_Ranking[Net Sales], 1)
RETURN 
ranking/rowcount
```
- RANK STOCK
```dax
Rank_Stock = 
VAR rowcount = COUNTROWS(Items_Ranking)
VAR ranking = RANK.EQ(Items_Ranking[Stock Cost],Items_Ranking[Stock Cost], 1)
RETURN 
ranking/rowcount
```
- ABC RANKING SALES
```dax
ABC Ranking_Net_Sales = 
VAR _Rank = 
DIVIDE(Items_Ranking[Rank_Net_Sales],MAX(Items_Ranking[Rank_Net_Sales]))
VAR Ranking = 
    IF(
    _Rank>= 0.9,
    "A",
    IF(
        _Rank >= 0.7,
        "B", 
        "C"
    )
)
RETURN
Ranking
```
- ABC RANKING STOCK
```dax
ABC Ranking_Stock = 
VAR _Rank = 
DIVIDE(Items_Ranking[Rank_Stock],MAX(Items_Ranking[Rank_Stock]))
VAR Ranking = 
    IF(
    _Rank>= 0.9,
    "A",
    IF(
        _Rank >= 0.7,
        "B", 
        "C"
    )
)
RETURN
Ranking
```
---
## KPI Scorecard For Sales

- Sales Amount
 ```dax
 Sales Amount = SUMX('historical-sales','historical-sales'[Total Sales])
 ```
 - Sales Amount Rank
 ```dax
 Sales Amount Rank = RANKX(ALL('historical-sales'[store]),_Measures[Sales Amount])
 ```
 - Sales Store Rank Text
 ```dax
 Sales Store Rank Text = [Sales Amount Rank] & " OF " & [Store Count All]
 ```
 - Sales Monthly Average
 ```dax
 Sales Monthly Average = AVERAGEX(VALUES('Calendar'[Month_Year]), _Measures[Sales Amount])
 ```
 - Max Sales Month
 ```dax
 Max Sales Month = CALCULATE(MAX('Calendar'[Month_Year]), FILTER(ALL('Calendar'),[Sales Amount]))
 ```
 - Latest Sales Amount
 ```dax
 Latest Sales Amount = CALCULATE([Sales Amount], FILTER('Calendar', 'Calendar'[Month_Year] = [Max Sales Month]))
 ```
 - Sales Six Month Trend
 ```dax
 Sales 6M Trend = 
 VAR LastSaleDate = LASTDATE ('historical-sales'[_date])
 RETURN
 AVERAGEX( 
    DATESBETWEEN( 
        'Calendar'[Month_Year],
        DATEADD(STARTOFMONTH( LastSaleDate ), -5, MONTH),
        ENDOFMONTH( LastSaleDate)
    ),
    [Month Over Month For Sales]
)
```
- Sales Trend KPI
```dax
Sales Trend KPI = 
VAR ChartIncrease = UNICHAR(128200)
VAR ChartDecrease = UNICHAR(128201)
VAR SixMonthTrend = [Sales 6M Trend]
RETURN
SWITCH(TRUE(),
SixMonthTrend >= 0, ChartIncrease,
SixMonthTrend <= 0, ChartDecrease
)
```
---
## KPI Scorecard For Return

- Return Amount
```dax
Return Amount = 
    SUMX(
        FILTER(
            'historical-sales','historical-sales'[type] = "Return"),
            'historical-sales'[Sales Qty] * 'historical-sales'[Total Sales])
```
- Return Amount Rank
```dax
Return Amount Rank = RANKX(ALL('historical-sales'[store]),_Measures[Return Amount])
```
- Return Store Rank Text
```dax
Return Store Rank Text = [Return Amount Rank] & " OF " & [Store Count All]
```
- Return Monthly Average
```dax
Return Monthly Average = AVERAGEX(VALUES('Calendar'[Month_Year]), _Measures[Return Amount])
```
- Max Return Month
```dax
Max Return Month = CALCULATE(MAX('Calendar'[Month_Year]), FILTER(ALL('Calendar'),[Return Amount]))
```
- Latest Return Amount
```dax
Latest Return Amount = CALCULATE([Return Amount], FILTER('Calendar', 'Calendar'[Month_Year] = [Max Return Month]))
```
- Return Six Month Trend
```dax
Return 6M Trend = 
VAR LastReturnDate = LASTDATE ('historical-sales'[_date])
RETURN
 AVERAGEX( 
    DATESBETWEEN( 
        'Calendar'[Month_Year],
        DATEADD(STARTOFMONTH(LastReturnDate), -5, MONTH),
        ENDOFMONTH(LastReturnDate)
    ),
    [Month Over Month For Return]
)
```
- Return Trend KPI
```dax
<!-- We need to fix the logic here -->
Return Trend KPI = 
VAR ChartIncrease = UNICHAR(128200)
VAR ChartDecrease = UNICHAR(128201)
VAR SixMonthTrend = [Return 6M Trend]
RETURN
SWITCH(TRUE(),
SixMonthTrend >= 0, ChartIncrease,
SixMonthTrend <= 0, ChartDecrease
)
```
---
