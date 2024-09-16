### Tables
+ Filtered Currencies
```
FilteredCurrencies = ADDCOLUMNS(
    DISTINCT(fact_InternetSales[CurrencyKey]),
    "CurrencyName", LOOKUPVALUE(dim_Currency[CurrencyAlternateKey], dim_Currency[CurrencyKey], fact_InternetSales[CurrencyKey])
)
```

+ dim_date
```
dim_date = ADDCOLUMNS(
    CALENDAR(MIN(fact_InternetSales[ShipDate]), MAX(fact_InternetSales[ShipDate])),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "Month_Name", FORMAT([Date], "MMMM"),
    "Day_of_week", FORMAT([Date], "DDDD"),
    "Quarter", "Q"&FORMAT([Date], "Q"))
```

### kPIs

+ Total Sales
```
Total Sales = 
VAR SalesAmount = SUM(fact_InternetSales[SalesAmount])

RETURN
IF(
    ISBLANK(SalesAmount), 
    "👎", SalesAmount
    )
```
+ Overall Profit  
```
Total Profit = 
VAR TotalProfit = SUM(fact_InternetSales[Profit])

RETURN
IF(
    ISBLANK(TotalProfit), 
    "👎", 
   TotalProfit
)
```

+ Average Profit per Order
```
Average Profit Per Order = 
VAR AverageprofitPerOrder = AVERAGEX(
  VALUES(fact_InternetSales[SalesOrderNumber]),
  CALCULATE(SUM(fact_InternetSales[SalesAmount]) - (SUM(fact_InternetSales[TotalProductCost]) + SUM(fact_InternetSales[Freight]) + SUM(fact_InternetSales[TaxAmt]))))

RETURN
IF(
    ISBLANK(AverageprofitPerOrder), 
    "👎", 
    AverageprofitPerOrder
)
```

+ Unique Products
```
Unique Products = COUNTROWS(DISTINCT(dim_Product[ProductKey]))
```

+ Total Orders
```
Total Orders = 
VAR Totalorders = SUM(fact_InternetSales[OrderQuantity])

RETURN
IF(
    ISBLANK(Totalorders), 
    "👎", 
    Totalorders
)
```

+ Average Shipping Days
```
Average Shipping Days = 
VAR AverageShipping = 
AVERAGEX(
    fact_InternetSales,
    DATEDIFF(
        fact_InternetSales[OrderDate], 
        fact_InternetSales[ShipDate], 
        DAY
    )
)

RETURN
IF(
    ISBLANK(AverageShipping), 
    "👎", AverageShipping
    )
```

+ Best Selling Product by Quantity
```
TopSellingProductbyQuantity = 
CALCULATE(
    SELECTEDVALUE(dim_Product[EnglishProductName]),
    TOPN(
        1, 
        SUMMARIZE(
            fact_InternetSales, 
            dim_Product[EnglishProductName], 
            "TotalQty", SUM(fact_InternetSales[OrderQuantity])
        ),
        [TotalQty], 
        DESC
    )
)
```

+ Best Selling Product by Sales Amount
```
TopSellingProductbySalesAmount = 
CALCULATE(
    SELECTEDVALUE(dim_Product[EnglishProductName]),
    TOPN(
        1, 
        SUMMARIZE(
            fact_InternetSales, 
            dim_Product[EnglishProductName], 
            "TotalQty", SUM(fact_InternetSales[SalesAmount])
        ),
        [TotalQty], 
        DESC
    )
)
```
