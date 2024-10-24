# Cohort Analysis
## Executive Summary
The Customer Retention Cohort Analysis Dashboard provides a data-driven perspective on customer acquisition, retention, and churn, allowing businesses to track their customer lifecycle performance. This Power BI report uses cohort analysis to deliver actionable insights into customer behavior and retention rates over time, helping businesses understand key metrics that influence their growth and sustainability.

### [Live Dashboard Link](https://app.powerbi.com/view?r=eyJrIjoiMjQ1ODA3OTItMjZkNC00ODExLThlZTEtZWM1ZDNhODI3ZTZkIiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)


![Navigation Page](https://github.com/RoyDip-Shuvo/Chohort-Analysis/blob/main/Image/Github/_1_Navigation.jpg)

## Data Processing Overview
Before building the Customer Retention Cohort Analysis Dashboard, the data went through several preprocessing and transformation steps using Power Query to ensure data accuracy and consistency. Below is a summary of the key data processing tasks:

### Date Table Creation :
I created a custom Date table in Power Query, which includes additional fields like start_of_the_month. This allows for time-based analysis and ensures all date-related calculations, such as retention and churn, are handled efficiently.

```bash
  let
    Source = {Number.From(StartDate)..Number.From(EndDate)},
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Changed Type" = Table.TransformColumnTypes(#"Converted to Table",{{"Column1", type date}}),
    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"Column1", "Date"}}),
    #"Inserted Start of Month" = Table.AddColumn(#"Renamed Columns", "Start of Month", each Date.StartOfMonth([Date]), type date)
in
    #"Inserted Start of Month"
```


### Merging and Cleaning Data :

**Problem**

- Multiple CSV files: Client provided two CSV files spanning two years.
- Inconsistent data: Unnecessary columns and incorrect data types needed to be addressed.

**Solution**

- Power Query Integration: Leveraged Power Query's capabilities to efficiently merge and clean the data.
- Data Merging: Utilized the ```Invoke Custom Function``` feature to combine the two datasets into a single table.
-  Use this code for data marge and clean.
  ```bash
  (DataSheet as table) => 

let
    #"Promoted Headers" = Table.PromoteHeaders(#"DataSheet", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Invoice", type any}, {"StockCode", type any}, {"Description", type text}, {"Quantity", Int64.Type}, {"InvoiceDate", type datetime}, {"Price", type number}, {"Customer ID", Int64.Type}, {"Country", type text}}),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each [Customer ID] <> null and [Customer ID] <> ""),
    #"Changed Type1" = Table.TransformColumnTypes(#"Filtered Rows",{{"Price", Currency.Type}}),
    #"Filtered Rows1" = Table.SelectRows(#"Changed Type1", each [Price] > 0),
    #"Changed Type2" = Table.TransformColumnTypes(#"Filtered Rows1",{{"Invoice", type text}}),
    #"Filtered Rows2" = Table.SelectRows(#"Changed Type2", each not Text.StartsWith([Invoice], "C")),
    #"Filtered Rows3" = Table.SelectRows(#"Filtered Rows2", each true),
    #"Changed Type3" = Table.TransformColumnTypes(#"Filtered Rows3",{{"InvoiceDate", type date}}),
    #"Reordered Columns" = Table.ReorderColumns(#"Changed Type3",{"InvoiceDate", "Invoice", "Customer ID", "Price", "Quantity", "Country", "StockCode", "Description"}),
    #"Removed Columns" = Table.RemoveColumns(#"Reordered Columns",{"StockCode", "Description"})
in
    #"Removed Columns"
  ```
- Column Removal: Eliminated redundant or irrelevant columns to streamline the data.
- Data Type Conversion: Adjusted data types (e.g., text to number) for accurate analysis and calculations.

** Dim_Customer **
Created a Dim_Customer table that consolidates all customer-related information (e.g., customer ID, acquisition date, demographics). This dimension table was used to build relationships with transaction data for cohort analysis and improve overall model performance.
Code

```dash
let
    Source = Fact_Sales,
    #"Grouped Rows" = Table.Group(Source, {"Customer ID"}, {{"First_Join", each List.Min([InvoiceDate]), type nullable date}}),
    #"Renamed Columns" = Table.RenameColumns(#"Grouped Rows",{{"First_Join", "First_transation_month"}}),
    #"Calculated Start of Month" = Table.TransformColumns(#"Renamed Columns",{{"First_transation_month", Date.StartOfMonth, type date}})
in
    #"Calculated Start of Month"
```

**Calculated Columns and Measures**
- Developed calculated columns in Power Query, such as Customer Acquisition Month and Churn Status, to enable tracking of customer cohorts and determine when customers drop off or return.
- The following measure calculates the number of months that have passed since a customer's first transaction. 
  ```bash
  Month Since First Transaction = 
      DATEDIFF(
        RELATED(Dim_Customer[First_Transation_Month]),
        RELATED(DimDate[Date]),
        MONTH
      )
  ```
  
- Measures in DAX were added for Cohort performance, Retention rates, Total sales, and Other KPIs.

  ```bash
  Active Customer = 
  COUNTROWS(VALUES(Fact_Sales[Customer ID]))


  New Customer = 
  CALCULATE(
    [Active Customer],
    Fact_Sales[Month Since First Transaction]=0)


  Cohort Performance = 

  VAR _minDate = MIN(DimDate[Start of Month])

  VAR _maxDate = MAX(DimDate[Start of Month])


  RETURN
  CALCULATE(
    [Active Customer],
    REMOVEFILTERS(DimDate[Start of Month]),
    RELATEDTABLE(Dim_Customer),
    Dim_Customer[First_transation_month] >= _minDate 
        &&
        Dim_Customer[First_transation_month] <= _maxDate
  )



  Retaintion Rate = 
  DIVIDE([Cohort Performance], [New Customer])


  Recovered Customers = 
  VAR _Customers_This_Month = 
    VALUES(Fact_Sales[Customer ID])

  VAR _Customer_last_month = 
    CALCULATETABLE(
        VALUES(Fact_Sales[Customer ID]),
        PREVIOUSMONTH(DimDate[Start of Month])
    )

  VAR _New_customer = 
    CALCULATETABLE(
        VALUES(Fact_Sales[Customer ID]),
        Fact_Sales[Month Since First Transaction] = 0
    )

  VAR _Recovered_Customers = 
    EXCEPT(EXCEPT(_Customers_This_Month,_Customer_last_month), _New_customer)

  RETURN
    COUNTROWS(_Recovered_Customers)




  Retaind Customers = 
  VAR _ThisMonth=VALUES(Fact_Sales[Customer ID])
  VAR _LastMonth = CALCULATETABLE(
    VALUES(Fact_Sales[Customer ID]),
    PREVIOUSMONTH(DimDate[Start of Month])
  )
  
  VAR _RetaindCustomer = 
    INTERSECT(_ThisMonth, _LastMonth)
  
  RETURN
  COUNTROWS(_RetaindCustomer)




  Lost_customer = 

  VAR _Customers_This_Month = 
    VALUES(Fact_Sales[Customer ID])

  VAR _Customer_last_month = 
    CALCULATETABLE(
        VALUES(Fact_Sales[Customer ID]),
        PREVIOUSMONTH(DimDate[Start of Month])
    )

  VAR _Lost_Customer =
    EXCEPT(_Customer_last_month, _Customers_This_Month)

  RETURN
    COUNTROWS(_Lost_Customer)


  
  ```


### Dashboard Highlights:
Revenue Metrics
- Total Revenue: $17.74M
- Average Revenue per Customer: $3.02K
- Total Orders: 36.97K
- Average Order Value: $479.95

![Overview](https://github.com/RoyDip-Shuvo/Chohort-Analysis/blob/main/Image/Github/_2_Overview_final.jpg)
![Cohort Performance](https://github.com/RoyDip-Shuvo/Chohort-Analysis/blob/main/Image/Github/_3_Details_Cohort.jpg)
![Churned Customer](https://github.com/RoyDip-Shuvo/Chohort-Analysis/blob/main/Image/Github/_4_Churned%20Customer.jpg)
![Churned Rate](https://github.com/RoyDip-Shuvo/Chohort-Analysis/blob/main/Image/Github/_5_Churned%20Rate.jpg)
![Retaintion Rate](https://github.com/RoyDip-Shuvo/Chohort-Analysis/blob/main/Image/Github/_6_Retaintion%20Rate.jpg)
