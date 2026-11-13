# Whale vs Diversity Sales Analysis

SQL star schema + Power BI dashboard measuring how much reseller revenue depends on a small number of very large ("whale") orders, by territory — built on AdventureWorks2025.

## 🎯 Business Question

How dependent is reseller revenue on a handful of very large orders, and which territories carry the most concentration risk? When a few large orders make up most of a territory's revenue, losing them hurts much more than losing several small ones.

## 🔍 Key Findings

- **500 orders produce almost half the revenue.** Orders over $50K are only 13% of the 3,806 reseller orders but bring in 45.5% of revenue ($36.6M of $80.5M).
- **Four territories get more than half their revenue from whale orders:** France (57%), Northwest (52%), United Kingdom (51%) and Southwest (51%).
- **Southwest is both the largest and one of the most concentrated territories:** $18.5M in revenue, with 51% from 127 whale orders.
- **Southeast (32%) and Australia (22%) have the most diversified revenue.**
- **Revenue spikes at quarter-end.** March, June and September are the strongest months, and November is the weakest ($3.7M). These monthly totals combine all years in the dataset, so a month that appears in more years can look stronger.

## 💡 Recommendations

- In France, Northwest, the UK and Southwest, identify which resellers place the whale orders and track them closely. These territories lose the most if large orders stop.
- Grow mid-size orders in those territories to reduce dependence on a few large buyers.
- Plan inventory and staffing around quarter-end ordering peaks.

## Dashboard
🔗 **[View the interactive dashboard](https://app.powerbi.com/view?r=eyJrIjoiNTg2OTcyNWYtMDMxNS00NGI3LTgwMmUtZWY1ZmY3ZTI4ZjRmIiwidCI6ImY1MmYyMTgzLTlmNjctNGFkMi1iNjU2LTZmNzU0ZmUxOTZjYiIsImMiOjZ9)**

![Dashboard](Whale%20Sales%20Dashboard.png)

## Star Schema
![Star Schema](Whale%20Sale%20Star%20schema.png)
## 📐 Architecture

A star schema data mart is built via a single SQL Server stored procedure (`Procedure Scripts.sql`):

| Object | Type | Description |
|---|---|---|
| `DimDate` | Table | Calendar attributes (year, quarter, month) |
| `DimProduct` | Table | Product hierarchy (category → subcategory) |
| `DimSalesTerritory` | Table | Territory, region group, and country |
| `DimSalesPerson` | Table | Salesperson names |
| `FactSales` | Table | Order-line grain — qty, unit price, line total |
| `v_FactSales_WhaleAnalysis` | View | Fact + whale classification + KPIs (Power BI source) |
| `v_DimProduct_Diversity` | View | Product revenue split by order category (Power BI source) |
| `v_DimTerritory_GapAnalysis` | View | Territory lookup for slicer filtering (Power BI source) |

## 📈 KPI Summary

| Metric | Value |
|---|---|
| Total Revenue | 80.5 M |
| Whale Concentration | 45.5 % |
| Revenue Diversity | 54.5 % |
| Average Order Value (AOV) | 21.1 K |
| Orders | 3,806 |

### Metric definitions

| Metric | Definition |
|---|---|
| Scope | Reseller orders only: orders with an assigned salesperson (online sales are excluded) |
| Whale order | An order whose total value is over $50,000. This fixed cutoff captures the largest ~13% of reseller orders (500 of 3,806) |
| Whale Concentration | Revenue from whale orders ÷ total revenue |
| Revenue Diversity | Revenue from all other orders ("Healthy Pipe") ÷ total revenue |
| AOV | Total revenue ÷ distinct orders |

### Data validation

All KPIs were reconciled against the AdventureWorks source tables (`Sales.SalesOrderHeader`, `Sales.SalesOrderDetail`): total revenue, whale revenue, AOV and order totals match the source exactly.

## 🚀 Setup

### Prerequisites
- SQL Server with **AdventureWorks2025** restored
- Power BI Desktop

### 1 — Build the data mart

```sql
USE [AdventureWorks2025];
GO
EXEC dbo.sp_CreateFactSAndDimTables;
GO
```

The procedure drops and recreates all objects — safe to re-run at any time.

### 2 — Connect Power BI
- **Get Data → SQL Server** → point to your AdventureWorks2025 database
- Import the three reporting views:
  - `v_FactSales_WhaleAnalysis`
  - `v_DimProduct_Diversity`
  - `v_DimTerritory_GapAnalysis`
- In the **Model view**, confirm relationships on `TerritoryID` and `ProductID`
- Add DAX measures for AOV, Whale Concentration, Revenue Diversity %, etc.

## 🛠️ Tech Stack
- **SQL Server** — stored procedure, star schema, views
- **T-SQL** — data transformation and mart build
- **Power BI Desktop** — data model, DAX measures, report visuals
