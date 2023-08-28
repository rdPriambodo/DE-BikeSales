# About Data & Projects
- we are using bicycles sales dataset to make dashboard report of sales growth, customer demographic, etc. 
- The design of data flow for this projects are ![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/75844e22-447e-4bc1-ad7e-9f9034292ac3)
- The tools used for this projects are : SQL Server, SSIS, SSAS, Power BI

# Data Lake (Staging)
- At this stage, data from sources are extracted and transformed with ETL methods such as Lookup or Timestamp Filter
- lets take an example of Currency table for Lookup Methods
![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/47a0b2cc-7c76-45c1-b605-6992795d2ac7)
![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/9372a9b6-ffa9-4831-9202-e6c3b41b6371)
  - for ETL of stgCurrency table, we extract from source using OLE DB Source from Currency Table
  - we also apply transformation with data conversion (to change data type as needed) and derived column (to add columns as needed, usually we at least add a timestamp column)
  - then we use lookup to determine whether there are any changes on the data, data that have changes will be inserted into stgCurrency (for this project we only insert data because we will delete the duplicate/redundant data)
  - Then we will delete any duplicate or redundant data

- Other than lookup methods, we also can use Timestamp Filter. Lets take an example of sales data
  - this method can be applied if the data has timstamp column that can specify whether there are any changes in the data, so we will only retrieve data that has changes
  - it has similar flow iwth lookup method, but since we only process data that has changes, this method is more efficient
![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/77ffb571-5645-41f8-b496-e60166f4d624)
![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/5c9e98f5-e0c9-4afd-8cc0-ba48958289ea)
  - for this data sales we take source from csv files, we will only retrieve data that has OrderDate greater than MAXOrderDate in stgSales, we use conditional split for this filter
  - just like lookup method, we apply transformation as needed, in this case we only use derived column
  - then insert the data into stgSales, after that we delete any duplicate or redundant data

# Data Warehouse (Dim & Fact)
- At this stage the data in the data lake will be extracted and processed to be ready for analyzing
- here, we are using SQL merge Query to insert or update date, here are the examples for dimCurrency
```
MERGE dimCurrency AS target
USING stgCurrency
AS  source
ON (target.CurrencyKey = source.CurrencyKey)
WHEN MATCHED AND 
	ISNULL(target.[CurrencyKey],'NULL')<> ISNULL(source.[CurrencyKey],'NULL') OR
	ISNULL(target.[CurrencyAlternateKey],'NULL')<> ISNULL(source.[CurrencyAlternateKey],'NULL') OR
	ISNULL(target.[CurrencyName],'NULL')<> ISNULL(source.[CurrencyName],'NULL')
THEN
    UPDATE SET 
	[CurrencyKey]= source.[CurrencyKey]
    ,[CurrencyAlternateKey]= source.[CurrencyAlternateKey]
	,[CurrencyName]= source.[CurrencyName]
	, ETLDate = GETDATE()

WHEN NOT MATCHED 
THEN 
	INSERT (
		[CurrencyKey]
		,[CurrencyAlternateKey]
		,[CurrencyName]
		,[ETLDate]
		)
		VALUES (
		source.[CurrencyKey]
		,source.[CurrencyAlternateKey]
		,source.[CurrencyName]
		,GETDATE()
		);
```
- we can also use Timestamp filter method for this stage (similar with the one in the data lake), but here we are also updates data since we can’t insert duplicate data and doesn’t delete redundant at the end
![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/b5ca76e6-f964-4b38-8c35-3b1fa01e79c1)

# OLAP
- at this stage we create/manage each relation for the table we made in the data warehouse
- then process the data and create a variable that can help us in the dashboard
![image](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/aec246ba-2575-44f1-9cb7-62fedee6c698)

# Dashboard/ Result
![Portofolio 3 Currency and ProductSubCategory](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/daa144f8-b018-4762-8881-acaa82b65909)
![Portofolio 2 Product Sales Summary](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/c628646d-8dfc-4d21-82f3-b1bd5aea48c1)
![Portofolio 1 Customer Sales Summary](https://github.com/rdPriambodo/DE-BikeSales/assets/59159771/53c75a07-d350-4423-a3e5-41e5d31e1799)
