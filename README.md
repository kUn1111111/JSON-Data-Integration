# SSIS Project — JSON Data Integration

## Project Overview

This project demonstrates how to ingest semi-structured JSON data into a relational SQL Server table using **SQL Server Integration Services (SSIS)**.

The goal of the project is to:
- Read JSON files from the file system
- Parse JSON objects into a tabular structure
- Convert data types correctly
- Load the data into a SQL Server table ready for BI and analytics

---

## Technologies Used

- SQL Server
- SSIS (SQL Server Integration Services)
- OPENROWSET & OPENJSON
- Visual Studio (SSDT)

---

##  Input Data

The source data is a JSON file containing order information.

### Example JSON structure (`orders.json`)
```json
[
  {
    "OrderID": 1,
    "CustomerID": 101,
    "ProductKey": 776,
    "OrderDate": "2023-01-15",
    "SalesAmount": 250.75,
    "Quantity": 2
  }
]
```

---

## Solution Design

### 1️ OLE DB Source — Reading JSON

The JSON file is read using a SQL command with OPENROWSET and OPENJSON.

```sql
SELECT
    OrderID,
    CustomerID,
    ProductKey,
    OrderDate,
    SalesAmount,
    Quantity
FROM OPENROWSET(
    BULK 'C:\SSIS\JsonFiles\orders.json',
    SINGLE_CLOB
) AS j
CROSS APPLY OPENJSON(j.BulkColumn)
WITH (
    OrderID INT '$.OrderID',
    CustomerID INT '$.CustomerID',
    ProductKey INT '$.ProductKey',
    OrderDate DATE '$.OrderDate',
    SalesAmount MONEY '$.SalesAmount',
    Quantity INT '$.Quantity'
);
```

This step converts each JSON object into a single row with structured columns.

---

### 2️ Data Conversion

Since SSIS often reads JSON fields as text (DT_WSTR), a Data Conversion transformation is used to convert columns into appropriate data types:

| Column      | Target Data Type |
|-------------|------------------|
| OrderID     | DT_I4 (INT)      |
| CustomerID  | DT_I4 (INT)      |
| ProductKey  | DT_I4 (INT)      |
| Quantity    | DT_I4 (INT)      |
| SalesAmount | DT_CY (Money)    |
| OrderDate   | DT_DBDATE        |

This ensures compatibility with the destination SQL table.

---

### 3️ OLE DB Destination — Load to SQL Server

The cleansed and converted data is loaded into a relational table.

```sql
CREATE TABLE dbo.JSON_Orders (
    OrderID INT,
    CustomerID INT,
    ProductKey INT,
    OrderDate DATE,
    SalesAmount MONEY,
    Quantity INT
);
```

The destination uses **Table or View – Fast Load** for performance.

---

## Final Result

- JSON data successfully converted into rows and columns
- Strongly typed relational table
- Data ready for BI tools such as Power BI
- Clear separation between ingestion, transformation, and loading

---

## Key Learnings

- Handling semi-structured data in SSIS
- Using OPENROWSET and OPENJSON
- Importance of data type conversion in ETL pipelines
- Preparing JSON data for analytical models

---

## Next Improvements

- Load multiple JSON files using ForEach Loop Container
- Add error handling and rejected rows
- Add logging and audit tables
