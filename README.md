# Retail Transaction Data Preparation and Sales Insight Project

This project prepares a raw retail transaction dataset for reliable business analysis. It starts with an Excel workbook of invoice-level sales records, explores the structure and quality of the data, enriches each transaction with analytical fields, and exports a cleaner workbook that can be used for sales, customer, product, country, and cancellation analysis.

The work is implemented in [`ProceedData.ipynb`](ProceedData.ipynb), using Python, pandas, NumPy, matplotlib, and seaborn.

## Problem Statement

Retail transaction data is often messy even when it looks structured. Missing customer identifiers, incomplete product descriptions, repeated transaction lines, returns, negative quantities, and unusually high prices can all distort business decisions if they are ignored or handled carelessly.

The main problem in this project is to transform raw transaction records into an analysis-ready dataset while preserving useful business information. The goal is not only to clean the data, but to make careful decisions about what should be corrected, what should be retained, and what should be treated separately during analysis.

## Dataset Overview

The input dataset is stored in [`Data.xlsx`](Data.xlsx).

| Property | Value |
| --- | --- |
| Rows | 541,909 |
| Original columns | 8 |
| Date range | 2010-12-01 to 2011-12-09 |
| Unique invoices | 25,900 |
| Unique products | 4,070 |
| Unique customers with IDs | 4,372 |
| Countries | 38 |

Original columns:

| Column | Meaning |
| --- | --- |
| `InvoiceNo` | Invoice identifier for each transaction |
| `StockCode` | Product code |
| `Description` | Product description |
| `Quantity` | Number of items purchased or returned |
| `InvoiceDate` | Transaction date and time |
| `UnitPrice` | Price per unit |
| `CustomerID` | Customer identifier, when available |
| `Country` | Customer or transaction country |

## Methodology

### 1. Data Loading

The notebook loads the raw Excel file with pandas and keeps a working copy named `rawData`. This protects the original source file while allowing transformations to be applied step by step.

```python
data = pd.read_excel("./Data.xlsx")
rawData = data.copy()
```

### 2. Data Understanding

The project begins with exploratory checks:

- Dataset shape and column names
- Data types
- Summary statistics
- Missing-value counts
- Unique invoice, product, customer, and country counts
- Minimum and maximum invoice dates

This step matters because cleaning decisions should be based on the actual behavior of the data, not assumptions.

### 3. Missing Value Handling

The dataset contains missing values in two important fields:

| Column | Missing values | Missing percentage | Decision |
| --- | ---: | ---: | --- |
| `Description` | 1,454 | 0.27% | Recover where possible using `StockCode` |
| `CustomerID` | 135,080 | 24.93% | Keep as missing |

For missing product descriptions, the notebook builds a mapping between `StockCode` and known `Description` values from other rows. This recovers 1,342 missing descriptions. The remaining 112 descriptions are left blank because guessing would introduce false product information.

For missing customer IDs, the values are kept as `NaN`. This is intentional. A customer ID is an identity field, and filling it with a fake or guessed value would create incorrect customer-level analysis. These records are still useful for total revenue, product, country, and time-based analysis, but they are excluded from customer-specific aggregations.

### 4. Duplicate Review

The notebook checks for exact duplicate rows and finds that duplicates represent about 0.97% of the dataset. It also separates the idea of repeated values from true duplicate records.

Repeated values are normal in retail data:

- One invoice can contain many products.
- One product can appear in many invoices.
- One customer can place many orders.
- One country can have many transactions.

The current processed export preserves the transaction rows, while documenting duplicate presence as a data-quality consideration.

### 5. Data Type Validation

The notebook validates all column data types:

- `InvoiceDate` is already stored as a datetime value.
- `Quantity` is numeric.
- `UnitPrice` is numeric.
- `CustomerID` remains numeric because it contains missing values.

No unnecessary conversion is applied. This avoids changing the meaning of the raw data.

### 6. Standardization Checks

The project checks text consistency, especially product descriptions. It identifies 113,681 descriptions with extra spacing. This highlights formatting noise that may affect grouping or display quality.

Country names and core numeric/date fields are also reviewed to understand whether standardization is needed before analysis.

### 7. Feature Engineering

The notebook creates new analytical columns:

| New column | Formula or logic | Purpose |
| --- | --- | --- |
| `Revenue` | `Quantity * UnitPrice` | Measures transaction-level sales value |
| `CancellationFlag` | Invoice number starts with `C` | Identifies returns or cancellations |
| `Month` | Extracted from `InvoiceDate` | Enables monthly revenue analysis |
| `Year` | Extracted from `InvoiceDate` | Enables yearly grouping |
| `SalesType` | Sale or Cancellation/Return | Separates normal sales from returns |

These fields make the dataset easier to analyze without repeatedly recalculating common business metrics.

### 8. Return and Cancellation Handling

The dataset contains negative quantities and invoice numbers beginning with `C`, which indicate cancellations or returns. Instead of removing these rows, the notebook classifies them with `CancellationFlag` and `SalesType`.

This is important because returns are part of real retail behavior. Removing them would overstate revenue and hide operational patterns.

### 9. Outlier Review

The notebook applies the IQR method to `UnitPrice` and identifies 39,627 price outliers. The review shows that some high-price rows are special cases such as manual adjustments, fees, postage, or bad-debt adjustments.

These rows are investigated rather than blindly removed. In retail analysis, an outlier is not always an error; sometimes it represents a meaningful business event.

### 10. Insight Generation

After preparation, the notebook generates summary insights:

- Top months by revenue
- Top products by revenue
- Top customers by revenue
- Top countries by revenue
- Sales versus cancellation/return summary

Example findings from the notebook:

| Analysis | Top result |
| --- | --- |
| Highest revenue month | 2011-11 |
| Highest revenue product/description | `DOTCOM POSTAGE` |
| Highest revenue customer | `14646.0` |
| Highest revenue country | United Kingdom |
| Sale transactions | 532,621 |
| Cancellation/return transactions | 9,288 |

## Solution

The solution is an analysis-ready Excel file named [`processedData.xlsx`](processedData.xlsx).

It contains the original transaction fields plus five additional columns:

| Column | Description |
| --- | --- |
| `Revenue` | Transaction value calculated from quantity and unit price |
| `CancellationFlag` | Boolean marker for cancellation invoices |
| `Month` | Monthly period derived from invoice date |
| `Year` | Year derived from invoice date |
| `SalesType` | Human-readable sale category |

The final workbook has 541,909 data rows and 13 columns.

## Why This Approach

This project uses a conservative data-preparation strategy. It fixes values when there is reliable evidence, keeps missing values when the truth cannot be known, and labels special business events instead of deleting them.

That approach is useful because it protects the integrity of later analysis:

- Revenue remains realistic because cancellations are included.
- Customer analysis remains trustworthy because unknown customers are not invented.
- Product analysis improves because missing descriptions are recovered from matching product codes.
- Outliers remain visible for review instead of being silently discarded.
- The processed dataset is easier to reuse in dashboards, reports, or machine-learning workflows.

## Project Files

| File | Description |
| --- | --- |
| [`Data.xlsx`](Data.xlsx) | Raw retail transaction dataset |
| [`ProceedData.ipynb`](ProceedData.ipynb) | Notebook containing the full cleaning, enrichment, and analysis flow |
| [`processedData.xlsx`](processedData.xlsx) | Final enriched output dataset |
| [`README.md`](README.md) | Project documentation |

## How to Run

1. Open the project folder.
2. Install the required Python libraries:

```bash
pip install pandas numpy seaborn matplotlib openpyxl
```

3. Open [`ProceedData.ipynb`](ProceedData.ipynb) in Jupyter Notebook, JupyterLab, VS Code, or another notebook environment.
4. Run the notebook cells from top to bottom.
5. The processed workbook will be written to:

```text
processedData.xlsx
```

## Requirements

- Python 3.x
- pandas
- NumPy
- matplotlib
- seaborn
- openpyxl
- Jupyter Notebook or compatible editor

## Final Outcome

The project turns raw retail records into a documented, enriched, and reusable transaction dataset. It provides a clear methodology for handling missing values, validating structure, identifying returns, reviewing outliers, and creating business-ready fields for revenue and time-based analysis.

The result is a stronger foundation for answering retail questions such as:

- Which months generated the most revenue?
- Which products contributed most to sales?
- Which customers are most valuable?
- Which countries drive revenue?
- How much business impact comes from cancellations and returns?

