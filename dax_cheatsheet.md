# DAX Cheat Sheet for Power BI

A practical reference for DAX (Data Analysis Expressions) operations commonly used in Power BI for measures, calculated columns, and data modeling.

---

## Core Concepts

| Concept | What it means |
|---------|---------------|
| `Measure` | A calculation evaluated at query time based on filter context — does not store values in the table |
| `Calculated column` | A column computed row-by-row when the model refreshes — stored in the table |
| `Calculated table` | A whole table created from a DAX expression |
| `Filter context` | The set of filters applied to the data when a measure is evaluated (slicers, visuals, rows, columns) |
| `Row context` | The current row being evaluated — exists in calculated columns and inside iterators like `SUMX` |
| `Context transition` | Converting row context to filter context — happens automatically when a measure is referenced inside an iterator |
| `CALCULATE` | The most important DAX function — modifies the filter context for an expression |

---

## Creating Measures & Columns

### Define a measure
```dax
Total Sales = SUM(Sales[Amount])
```

### Define a calculated column
```dax
Full Name = Customers[FirstName] & " " & Customers[LastName]
```

### Define a calculated table
```dax
HighValueCustomers =
FILTER(Customers, Customers[LifetimeValue] > 10000)
```

### Comments
```dax
// single-line comment
-- single-line comment
/* multi-line
   comment */
```

---

## Aggregations

### Basic aggregations
```dax
Total Sales      = SUM(Sales[Amount])
Average Sale     = AVERAGE(Sales[Amount])
Min Sale         = MIN(Sales[Amount])
Max Sale         = MAX(Sales[Amount])
Order Count      = COUNT(Sales[OrderID])
Distinct Customers = DISTINCTCOUNT(Sales[CustomerID])
Row Count        = COUNTROWS(Sales)
Non-blank Count  = COUNTA(Sales[Notes])
```

### Iterator versions — evaluate an expression row by row, then aggregate

```dax
Revenue        = SUMX(Sales, Sales[Quantity] * Sales[Price])
Avg Line Total = AVERAGEX(Sales, Sales[Quantity] * Sales[Price])
Max Line Total = MAXX(Sales, Sales[Quantity] * Sales[Price])
Min Line Total = MINX(Sales, Sales[Quantity] * Sales[Price])
```

> Use the `X` versions when you need to compute something per row before aggregating.

---

## CALCULATE — Modifying Filter Context

`CALCULATE` is the heart of DAX. It evaluates an expression in a modified filter context.

### Basic syntax
```dax
CALCULATE(<expression>, <filter1>, <filter2>, ...)
```

### Apply a single filter
```dax
Sales Online = CALCULATE(SUM(Sales[Amount]), Sales[Channel] = "Online")
```

### Apply multiple filters (combined with AND)
```dax
Sales Online 2024 =
CALCULATE(
    SUM(Sales[Amount]),
    Sales[Channel] = "Online",
    YEAR(Sales[Date]) = 2024
)
```

### Remove all filters from a table
```dax
Total Sales All =
CALCULATE(SUM(Sales[Amount]), ALL(Sales))
```

### Remove filters from a specific column
```dax
Sales All Regions =
CALCULATE(SUM(Sales[Amount]), ALL(Sales[Region]))
```

### Keep only specific filters, remove the rest
```dax
Sales By Region Only =
CALCULATE(SUM(Sales[Amount]), ALLEXCEPT(Sales, Sales[Region]))
```

### Add a filter without removing existing ones
```dax
Sales High Value =
CALCULATE(SUM(Sales[Amount]), KEEPFILTERS(Sales[Amount] > 1000))
```

---

## Filter Functions

### FILTER — return a filtered table
```dax
Big Orders =
CALCULATE(
    COUNTROWS(Sales),
    FILTER(Sales, Sales[Amount] > 1000)
)
```

### ALL — ignore filters from a table or column
```dax
% of Total =
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALL(Sales))
)
```

### ALLSELECTED — respect outer filters but ignore filters from current visual
```dax
% of Visual Total =
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALLSELECTED(Sales))
)
```

### VALUES — distinct values currently in filter context
```dax
Selected Regions = COUNTROWS(VALUES(Sales[Region]))
```

### HASONEVALUE — check if exactly one value is in filter context
```dax
Selected Region =
IF(HASONEVALUE(Sales[Region]), VALUES(Sales[Region]), "Multiple")
```

### SELECTEDVALUE — return value when one is selected, else fallback
```dax
Selected Region = SELECTEDVALUE(Sales[Region], "All Regions")
```

---

## Time Intelligence

> Time intelligence functions require a proper Date table marked as a date table in the model.

### Year-to-date, quarter-to-date, month-to-date
```dax
Sales YTD = TOTALYTD(SUM(Sales[Amount]), 'Date'[Date])
Sales QTD = TOTALQTD(SUM(Sales[Amount]), 'Date'[Date])
Sales MTD = TOTALMTD(SUM(Sales[Amount]), 'Date'[Date])
```

### Same period last year
```dax
Sales LY =
CALCULATE(SUM(Sales[Amount]), SAMEPERIODLASTYEAR('Date'[Date]))
```

### Year-over-year growth
```dax
YoY Growth =
DIVIDE([Total Sales] - [Sales LY], [Sales LY])
```

### Previous month / quarter / year
```dax
Sales Prev Month   = CALCULATE([Total Sales], PREVIOUSMONTH('Date'[Date]))
Sales Prev Quarter = CALCULATE([Total Sales], PREVIOUSQUARTER('Date'[Date]))
Sales Prev Year    = CALCULATE([Total Sales], PREVIOUSYEAR('Date'[Date]))
```

### Next month / quarter / year
```dax
Sales Next Month = CALCULATE([Total Sales], NEXTMONTH('Date'[Date]))
```

### Rolling 12 months
```dax
Sales Rolling 12M =
CALCULATE(
    SUM(Sales[Amount]),
    DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -12, MONTH)
)
```

### Sales for a custom date range
```dax
Sales Last 30 Days =
CALCULATE(
    SUM(Sales[Amount]),
    DATESBETWEEN('Date'[Date], TODAY() - 30, TODAY())
)
```

### Parallel period (shift by N units)
```dax
Sales 3 Months Ago =
CALCULATE(
    SUM(Sales[Amount]),
    PARALLELPERIOD('Date'[Date], -3, MONTH)
)
```

---

## Logical Functions

### IF — conditional
```dax
Order Tier =
IF(Sales[Amount] >= 500, "High",
    IF(Sales[Amount] >= 100, "Mid", "Low"))
```

### SWITCH — cleaner than nested IF
```dax
Order Tier =
SWITCH(
    TRUE(),
    Sales[Amount] >= 500, "High",
    Sales[Amount] >= 100, "Mid",
    "Low"
)
```

### IFERROR — handle errors gracefully
```dax
Safe Ratio = IFERROR(SUM(Sales[Amount]) / SUM(Sales[Quantity]), 0)
```

### DIVIDE — safe division (returns blank or fallback on zero)
```dax
Avg Price = DIVIDE(SUM(Sales[Amount]), SUM(Sales[Quantity]), 0)
```

### AND / OR — combine conditions
```dax
Active High Value =
IF(AND(Sales[Status] = "Active", Sales[Amount] > 1000), "Yes", "No")

// or use && and ||
IF(Sales[Status] = "Active" && Sales[Amount] > 1000, "Yes", "No")
```

### IN — check if a value is in a list
```dax
Is Priority Region = IF(Sales[Region] IN {"North", "West"}, "Yes", "No")
```

### ISBLANK / ISEMPTY / ISERROR
```dax
Has Email = IF(ISBLANK(Customers[Email]), "No", "Yes")
```

---

## Text Functions

### Concatenate text
```dax
Full Name = Customers[FirstName] & " " & Customers[LastName]
Full Name = CONCATENATE(Customers[FirstName], Customers[LastName])
```

### Format numbers and dates as text
```dax
Sales Label = "Total: " & FORMAT(SUM(Sales[Amount]), "#,##0.00")
Date Label  = FORMAT('Date'[Date], "yyyy-mm-dd")
```

### Case conversion
```dax
Upper Name = UPPER(Customers[Name])
Lower Email = LOWER(Customers[Email])
Proper Name = PROPER(Customers[Name])
```

### Trim whitespace
```dax
Clean Name = TRIM(Customers[Name])
```

### Length of a string
```dax
Name Length = LEN(Customers[Name])
```

### Substring
```dax
First Three = LEFT(Customers[Name], 3)
Last Three  = RIGHT(Customers[Name], 3)
Middle      = MID(Customers[Name], 2, 5)    // start at position 2, take 5 chars
```

### Find and replace
```dax
Cleaned Phone = SUBSTITUTE(Customers[Phone], "-", "")
Position      = SEARCH("@", Customers[Email])    // case insensitive, returns position
```

### Concatenate values across rows
```dax
All Regions = CONCATENATEX(VALUES(Sales[Region]), Sales[Region], ", ")
```

---

## Date & Time Functions

### Get parts of a date
```dax
Year     = YEAR('Date'[Date])
Month    = MONTH('Date'[Date])
Day      = DAY('Date'[Date])
Weekday  = WEEKDAY('Date'[Date], 2)        // 2 = Monday is 1
WeekNum  = WEEKNUM('Date'[Date])
Quarter  = "Q" & FORMAT('Date'[Date], "Q")
```

### Today and now
```dax
Today = TODAY()
Now   = NOW()
```

### Build a date
```dax
First Of Month = DATE(YEAR('Date'[Date]), MONTH('Date'[Date]), 1)
```

### Date arithmetic
```dax
Days Ago = DATEDIFF('Date'[Date], TODAY(), DAY)
Plus 7   = 'Date'[Date] + 7
Plus 1Mo = EDATE('Date'[Date], 1)
End of Month = EOMONTH('Date'[Date], 0)
```

### Build a complete Date table
```dax
Date =
ADDCOLUMNS(
    CALENDAR(DATE(2020, 1, 1), DATE(2026, 12, 31)),
    "Year",     YEAR([Date]),
    "Month",    MONTH([Date]),
    "MonthName", FORMAT([Date], "mmmm"),
    "Quarter",  "Q" & FORMAT([Date], "Q"),
    "Weekday",  FORMAT([Date], "dddd"),
    "YearMonth", FORMAT([Date], "yyyy-mm")
)
```

---

## Relationships & Lookups

### RELATED — pull a value from the "one" side of a relationship
```dax
Customer Country = RELATED(Customers[Country])
```

### RELATEDTABLE — get all related rows from the "many" side
```dax
Order Count = COUNTROWS(RELATEDTABLE(Sales))
```

### LOOKUPVALUE — manual lookup (no relationship needed)
```dax
Region Manager =
LOOKUPVALUE(Managers[Name], Managers[Region], Sales[Region])
```

### USERELATIONSHIP — activate an inactive relationship
```dax
Sales By Ship Date =
CALCULATE(
    SUM(Sales[Amount]),
    USERELATIONSHIP(Sales[ShipDate], 'Date'[Date])
)
```

### CROSSFILTER — change relationship direction temporarily
```dax
Customers With Sales =
CALCULATE(
    DISTINCTCOUNT(Customers[CustomerID]),
    CROSSFILTER(Sales[CustomerID], Customers[CustomerID], BOTH)
)
```

---

## Variables (VAR / RETURN)

Variables make complex measures readable and avoid recalculating the same expression.

```dax
YoY Growth =
VAR CurrentSales = SUM(Sales[Amount])
VAR PriorSales   = CALCULATE(SUM(Sales[Amount]), SAMEPERIODLASTYEAR('Date'[Date]))
VAR Growth       = CurrentSales - PriorSales
RETURN
    DIVIDE(Growth, PriorSales)
```

> Variables are evaluated once and cached — always prefer them in non-trivial measures.

---

## Ranking & Top N

### RANKX — rank by a measure
```dax
Region Rank =
RANKX(ALL(Sales[Region]), [Total Sales], , DESC, DENSE)
```

### TOPN — return the top N rows of a table
```dax
Top 5 Sales =
CALCULATE(
    SUM(Sales[Amount]),
    TOPN(5, ALL(Sales[CustomerID]), [Total Sales], DESC)
)
```

---

## Table Functions

### SUMMARIZE — group by and aggregate (like SQL GROUP BY)
```dax
SalesByRegion =
SUMMARIZE(
    Sales,
    Sales[Region],
    "Total", SUM(Sales[Amount]),
    "Orders", COUNTROWS(Sales)
)
```

### SUMMARIZECOLUMNS — preferred modern alternative
```dax
SalesByRegion =
SUMMARIZECOLUMNS(
    Sales[Region],
    "Total", SUM(Sales[Amount]),
    "Orders", COUNTROWS(Sales)
)
```

### ADDCOLUMNS — add columns to an existing table expression
```dax
WithMargin =
ADDCOLUMNS(
    Sales,
    "Margin", Sales[Amount] - Sales[Cost]
)
```

### SELECTCOLUMNS — pick specific columns
```dax
SimpleSales =
SELECTCOLUMNS(Sales, "Region", Sales[Region], "Amount", Sales[Amount])
```

### UNION — stack tables (same columns)
```dax
AllSales = UNION(Sales2023, Sales2024)
```

### EXCEPT — rows in first table not in second
```dax
LostCustomers = EXCEPT(VALUES(Customers[ID]), VALUES(Sales[CustomerID]))
```

### INTERSECT — rows in both tables
```dax
RepeatCustomers = INTERSECT(VALUES(Sales2023[CustomerID]), VALUES(Sales2024[CustomerID]))
```

---

## Common DAX Patterns

### Percentage of total
```dax
% of Grand Total =
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALL(Sales))
)
```

### Percentage of category total
```dax
% of Region Total =
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALLEXCEPT(Sales, Sales[Region]))
)
```

### Year-over-year comparison
```dax
Sales YoY % =
VAR Current = SUM(Sales[Amount])
VAR Prior   = CALCULATE(SUM(Sales[Amount]), SAMEPERIODLASTYEAR('Date'[Date]))
RETURN
    DIVIDE(Current - Prior, Prior)
```

### Running total
```dax
Running Total =
CALCULATE(
    SUM(Sales[Amount]),
    FILTER(ALL('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date]))
)
```

### Moving average — last 7 days
```dax
Moving Avg 7D =
AVERAGEX(
    DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -7, DAY),
    [Total Sales]
)
```

### Count distinct customers who bought a specific product
```dax
Buyers Of Product X =
CALCULATE(
    DISTINCTCOUNT(Sales[CustomerID]),
    Products[Name] = "Product X"
)
```

### New vs returning customers
```dax
New Customers =
VAR CurrentCustomers = VALUES(Sales[CustomerID])
VAR PriorCustomers =
    CALCULATETABLE(
        VALUES(Sales[CustomerID]),
        FILTER(ALL('Date'[Date]), 'Date'[Date] < MIN('Date'[Date]))
    )
RETURN
    COUNTROWS(EXCEPT(CurrentCustomers, PriorCustomers))
```

### Customer lifetime value
```dax
Customer LTV =
CALCULATE(
    SUM(Sales[Amount]),
    ALLEXCEPT(Sales, Sales[CustomerID])
)
```

### Top N with "Other" bucket
```dax
Region Or Other =
VAR TopRegions = TOPN(5, ALL(Sales[Region]), [Total Sales], DESC)
RETURN
    IF(Sales[Region] IN TopRegions, Sales[Region], "Other")
```

### Dynamic title based on selection
```dax
Visual Title =
"Sales for " & SELECTEDVALUE('Date'[Year], "All Years")
    & " — " & SELECTEDVALUE(Sales[Region], "All Regions")
```

### Show measure only when one item is selected
```dax
Detail Measure =
IF(
    HASONEVALUE(Customers[Name]),
    SUM(Sales[Amount]),
    BLANK()
)
```

### Conditional formatting by threshold
```dax
Color =
SWITCH(
    TRUE(),
    [Total Sales] >= 100000, "#2ECC71",
    [Total Sales] >= 50000,  "#F1C40F",
    "#E74C3C"
)
```

### Cumulative count of distinct customers over time
```dax
Cumulative Customers =
CALCULATE(
    DISTINCTCOUNT(Sales[CustomerID]),
    FILTER(ALL('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date]))
)
```

### Flag rows for quarantine
```dax
Is Invalid =
IF(
    ISBLANK(Sales[CustomerID]) || Sales[Amount] < 0,
    "Invalid",
    "OK"
)
```

### Compare to a target/budget
```dax
vs Budget % =
DIVIDE([Total Sales] - [Budget], [Budget])
```

---

## Performance Tips

| Tip | Why it matters |
|-----|----------------|
| Prefer measures over calculated columns | Measures use less memory and respond to filter context |
| Use variables (`VAR`) | Avoid recalculating the same expression multiple times |
| Use `DIVIDE` instead of `/` | Handles divide-by-zero safely without `IFERROR` overhead |
| Use `SUMMARIZECOLUMNS` over `SUMMARIZE` | Newer, faster, designed for client tools |
| Avoid `FILTER` on entire tables when possible | Filter on columns instead — `Sales[Region] = "North"` beats `FILTER(Sales, ...)` |
| Avoid bidirectional relationships | They cause unexpected filter propagation and slow queries |
| Mark your Date table as a date table | Required for time intelligence functions to work correctly |
| Use `KEEPFILTERS` with care | It preserves existing filters instead of overriding them |
| Disable auto date/time | Auto-generated hidden date tables bloat the model — use a real Date table |

---

*Last updated: 2026*
