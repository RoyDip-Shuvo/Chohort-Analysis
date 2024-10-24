# Cohort Analysis
## Executive Summary            [Live Dashboard Link](https://app.powerbi.com/view?r=eyJrIjoiMjQ1ODA3OTItMjZkNC00ODExLThlZTEtZWM1ZDNhODI3ZTZkIiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)
The Customer Retention Cohort Analysis Dashboard provides a data-driven perspective on customer acquisition, retention, and churn, allowing businesses to track their customer lifecycle performance. This Power BI report uses cohort analysis to deliver actionable insights into customer behavior and retention rates over time, helping businesses understand key metrics that influence their growth and sustainability.

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
- Column Removal: Eliminated redundant or irrelevant columns to streamline the data.
- Data Type Conversion: Adjusted data types (e.g., text to number) for accurate analysis and calculations.


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
