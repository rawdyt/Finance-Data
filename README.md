# Finance Data - Dashboard

### Dashboard Link : https://app.powerbi.com/groups/me/reports/105014dc-f745-4f1e-9fa7-67a299891f54/8ab1d9702fe810cb51c0?experience=power-bi

## Problem Statement

The finance department of a multinational company wants to analyze its overall financial performance across different regions, product categories, and sales representatives. However, the available transactional data contains multiple data quality issues such as inconsistent date formats, missing or invalid values, duplicate transactions, and unstandardized text fields (e.g., payment modes and invoice statuses).

To build reliable financial insights, the company needs to clean, transform, and model the data efficiently using SQL and Power BI.
The goal of this project is to design a Finance Data Analytics Dashboard that highlights key performance indicators (KPIs), while also demonstrating strong data cleaning and transformation skills.


### Steps followed 

- Step 1 : We have laoded data directly to the Power BI to check the column quality and remove errors.

After loading the data using power query editor we made corrections for the errors in the below columns.

1)Duplicate Transaction ID 

2)TransactionDate — mixed date formats (YYYY-MM-DD, DD-MM-YYYY, MM/DD/YYYY, textual dates)

3)CustomerID — customer identifier (some rows NULL / N/A / blank)

4)OutletID — outlet code (some missing)

5)UnitPrice — Currency symbol ₹ in some rows, or be blank for some rows

6)TotalAmount — many rows are mismatched to Quantity*UnitPrice or are 0.0

- Step 2 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.

- Step 3 : Loaded data to SQL Server

- Step 4: Checked of there any in duplicate values Using SQL

- Step 5: Used belwo code deleted all the duplicate entries from  the table.

      DELETE FROM [finance_dataset_with_issues] WHERE 
      TRANSACTIONID  IN(
      SELECT TransactionID FROM
      (SELECT *, ROW_NUMBER() OVER (PARTITION BY  
      TransactionID   
      ORDER  
      BY TransactionID) AS ROW_NUM FROM
      [dbo].[finance_dataset_with_issues]) X
      WHERE X.ROW_NUM>1);

- Step 6: There were some mistaks in the Quantiy table the number 
  were entere negative wrongly. 

   To see those negative amounts SQL code:

       [SELECT * FROM [finance_dataset_with_issues
        WHERE Quantity<0]

  > To make the correction we used below SQL code: 
      UPDATE finance
      SET Quantity = ABS(Quantity)
      WHERE Quantity < 0;


- Step 7: We had Decimal numbers in the Totalamount Column we
  Rounded up those amount SQL code:

    UPDATE finance
    SET TotalAmount = ROUND(Quantity * UnitPrice, 2)
    WHERE ABS(TotalAmount - (Quantity * UnitPrice)) > 0.5;

- Step 8: We had some typos in the PaymentType column We used  
   below SQL code to resolve this issue(Which we      
   alredy seen in Power BI in the filter for PaymentType 
   column):

    Update [finance_dataset_with_issues]
    SET PaymentType = CASE
    WHEN LOWER(TRIM(PaymentType)) Like '%cred%' THEN  
    'Credit    Card'
    WHEN LOWER(TRIM(PaymentType)) Like '%up%' THEN 'UPI'
    WHEN LOWER(TRIM(PaymentType)) Like '%wall%' THEN 'Wallet'
    WHEN LOWER(TRIM(PaymentType)) Like '%cas%' THEN 'CASH'
    ELSE 'Other'
    END;

- Step 9: There were missing and n/a values in the CustomerID
  Column we replaced them with "UNKNOWN" using below code:

      UPDATE [finance_dataset_with_issues] 
      SET CustomerID = 'UNKNOWN' WHERE CustomerID IS NULL    
      OR    
      TRIM(CustomerID) = ' ' 
      OR CustomerID IN ('NULL','N/A');


- Step 10: There Were some NULL values available in the   
  UnitPrice Column but we have the data for the coplete data 
  for the Quantity and TotalAmount so we used below query to  
  replace those null's with there actual value.

      UPDATE [finance_dataset_with_issues]
      SET UNITPRICE = TotalAmount / Quantity
      WHERE UNITPRICE IS NULL;

- Step 11 : As above We also had some null values in the Total amount.
We used below code to find those errors:

    SELECT * FROM [finance_dataset_with_issues]
    WHERE TotalAmount IS NULL OR LTRIM(RTRIM(CAST(UnitPrice   
    AS VARCHAR(50)))) IN ('N/A',' ')

And below code to rectify the errors

    UPDATE [finance_dataset_with_issues]
    SET UNITPRICE = TotalAmount / Quantity
    WHERE UNITPRICE IS NULL;

### Now we have the cleaned data and we can proceed for the     DashBoard creation.  
  
-----------------------Dashboard Creation-----------------------

- Step 1: We have added KPI's 
  1)Total Revenue

   DAX used to create this measure:

      Total Revenue = SUMX('finance_dataset_with_issues (2)', 
      'finance_dataset_with_issues (2)'[UnitPrice] * 
      'finance_dataset_with_issues (2)'[Quantity])

  2)Total Transaction

  DAX used to create this measure:

      Total Transactios = DISTINCTCOUNT  
      ('finance_dataset_with_issues (2)'[TransactionID]) 

  3)Average Order value:

  DAX used to create this measure:

      Average Order Value = [Total Revenue]/[Total Transactios]

  4)% of Paid

  DAX used to create this measure:

      % Paid = DIVIDE([Total Paid], [Total Paid] + [Total    
      Unpaid])

- Step 2: Donut chart to show the revenue by the Statement

- Step 3: Donut chart to show amount and % for the Total amount  
  paid VS Total Amount Unpaid

- Step 4: Line chart for Total Piad and Unpaid by state so that 
  we can compare which is the state where unpaid amount is higher 
  than the paid amount

- Step 5: Column Chart to show revenue as per the Year

- Step 6: Funnel Chart to show the total revenue as per the 
  Payment Type

### Snip of the Dashboad

![DashBoard](https://raw.githubusercontent.com/rawdyt/Finance-Data/main/Finance%20Data%20Dashboard.jpg)

## Insights

- Finance Data Insights Summary

   1)Total Revenue: $18.88M across all transactions.

   2)Total Count of Transactions 9529

   3)The % of total amount paid is higher than the % of Total  
  Unpaid but very close.
   
   4)Gujarat is the state where highest Revenue is generated

   5)Delhi is the state where lowest Revenue is generated

   6)Gujarat and Tamil Nadu these are 2 States where the  
   Unpaid Invoice amount higher than the Piad which shows low 
   recovery rate.

   7)Net Banlking Is payment method by which highest revenue 
   is generated with amount $4.08M

   And Lowest Revenue by UPI with amount $3.59M

