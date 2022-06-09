## Business-Metrics-With-Dax
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
