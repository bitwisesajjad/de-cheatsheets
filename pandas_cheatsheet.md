# Pandas Cheat Sheet for Data Engineering

A practical reference for Pandas operations commonly used in data engineering and ETL workflows.

---

## Setup

```python
import pandas as pd
```

---

## Reading Data

### Read a CSV file
```python
df = pd.read_csv("file.csv")
df = pd.read_csv("file.csv", sep=";", encoding="utf-8")
df = pd.read_csv("file.csv", usecols=["id", "name", "amount"])
df = pd.read_csv("file.csv", dtype={"id": int, "amount": float})
df = pd.read_csv("file.csv", parse_dates=["created_at"])
df = pd.read_csv("file.csv", skiprows=1, nrows=1000)
```

### Read a Parquet file
```python
df = pd.read_parquet("file.parquet")
df = pd.read_parquet("file.parquet", columns=["id", "amount"])
```

### Read an Excel file
```python
df = pd.read_excel("file.xlsx", sheet_name="Sheet1")
```

### Read from a SQL query
```python
import sqlalchemy
engine = sqlalchemy.create_engine("postgresql://user:pass@host/db")
df = pd.read_sql("SELECT * FROM orders WHERE status = 'active'", engine)
```

### Read a JSON file
```python
df = pd.read_json("file.json")
```

---

## Inspecting Data

### Preview the data
```python
df.head(10)
df.tail(10)
df.sample(5)
```

### Shape and structure
```python
df.shape          # (rows, columns)
df.columns        # column names
df.dtypes         # data types of each column
df.info()         # summary including nulls and types
df.describe()     # statistics for numeric columns
```

---

## Selecting Data

### Select a single column
```python
df["name"]
```

### Select multiple columns
```python
df[["id", "name", "amount"]]
```

### Select rows by condition
```python
df[df["amount"] > 100]
df[(df["amount"] > 100) & (df["status"] == "active")]
df[df["status"].isin(["active", "pending"])]
```

### Select by row index — `iloc`
```python
df.iloc[0]        # first row
df.iloc[0:5]      # first 5 rows
df.iloc[:, 0:3]   # first 3 columns
```

### Select by label — `loc`
```python
df.loc[0, "name"]                  # specific cell
df.loc[:, "name":"amount"]         # column range
df.loc[df["status"] == "active"]   # rows by condition
```

---

## Transforming Data

### Rename columns
```python
df = df.rename(columns={"old_name": "new_name", "emp_id": "employee_id"})
```

### Add a new column
```python
df["tax"] = df["amount"] * 0.2
```

### Apply a function to a column
```python
df["name"] = df["name"].str.upper().str.strip()
df["amount"] = df["amount"].apply(lambda x: round(x, 2))
```

### Replace values
```python
df["status"] = df["status"].replace("N/A", None)
df["country"] = df["country"].replace({"FI": "Finland", "DE": "Germany"})
```

### Map values using a dictionary
```python
mapping = {1: "low", 2: "mid", 3: "high"}
df["tier"] = df["tier_code"].map(mapping)
```

### Cast a column to a different type
```python
df["age"] = df["age"].astype(int)
df["created_at"] = pd.to_datetime(df["created_at"])
df["amount"] = df["amount"].astype(float)
```

### Drop a column
```python
df = df.drop(columns=["unnecessary_col"])
```

### Reorder columns
```python
df = df[["id", "name", "amount", "status"]]
```

---

## Cleaning Data

### Check for nulls
```python
df.isnull().sum()
```

### Drop rows with nulls
```python
df = df.dropna()
df = df.dropna(subset=["email", "amount"])
```

### Fill null values
```python
df["amount"] = df["amount"].fillna(0)
df["country"] = df["country"].fillna("unknown")
df = df.fillna({"amount": 0, "country": "unknown"})
```

### Drop duplicate rows
```python
df = df.drop_duplicates()
df = df.drop_duplicates(subset=["email"])
df = df.drop_duplicates(subset=["email"], keep="last")
```

### Strip whitespace from string columns
```python
df["name"] = df["name"].str.strip()
```

### Standardize string columns
```python
df["email"] = df["email"].str.lower().str.strip()
```

---

## Sorting

### Sort by a column
```python
df = df.sort_values("amount", ascending=False)
df = df.sort_values(["region", "amount"], ascending=[True, False])
```

---

## Aggregation & Grouping

### Group by and aggregate
```python
df.groupby("department")["salary"].mean()
df.groupby("department").agg({"salary": "mean", "id": "count"})
```

### Named aggregations
```python
df.groupby("department").agg(
    avg_salary=("salary", "mean"),
    headcount=("id", "count"),
    total_revenue=("revenue", "sum")
)
```

### Pivot table
```python
df.pivot_table(values="revenue", index="region", columns="quarter", aggfunc="sum")
```

---

## Joining & Combining

### Merge two DataFrames (like SQL JOIN)
```python
df = pd.merge(df_orders, df_customers, on="customer_id", how="left")
df = pd.merge(df1, df2, on=["id", "date"], how="inner")
```

### Concatenate DataFrames vertically (stack rows)
```python
df = pd.concat([df1, df2], ignore_index=True)
```

### Concatenate DataFrames horizontally (add columns)
```python
df = pd.concat([df1, df2], axis=1)
```

---

## Date & Time

### Convert to datetime
```python
df["created_at"] = pd.to_datetime(df["created_at"])
```

### Extract parts of a date
```python
df["year"] = df["created_at"].dt.year
df["month"] = df["created_at"].dt.month
df["day"] = df["created_at"].dt.day
df["weekday"] = df["created_at"].dt.day_name()
```

### Filter by date range
```python
df = df[df["created_at"] >= "2024-01-01"]
df = df[(df["created_at"] >= "2024-01-01") & (df["created_at"] < "2025-01-01")]
```

### Truncate to month
```python
df["month"] = df["created_at"].dt.to_period("M")
```

---

## Writing Data

### Write to CSV
```python
df.to_csv("output.csv", index=False)
```

### Write to Parquet
```python
df.to_parquet("output.parquet", index=False)
```

### Write to Excel
```python
df.to_excel("output.xlsx", index=False, sheet_name="Results")
```

### Write to a SQL table
```python
df.to_sql("orders", engine, if_exists="append", index=False)
```

---

## ETL Patterns

### Count value frequency in a column
```python
df["status"].value_counts()
```

### Null audit across all columns
```python
df.isnull().sum().sort_values(ascending=False)
```

### Deduplicate keeping the latest record
```python
df = df.sort_values("updated_at", ascending=False).drop_duplicates(subset=["customer_id"])
```

### Detect changed rows between two snapshots
```python
df_changed = df_new.merge(df_old, on="id", suffixes=("_new", "_old"))
df_changed = df_changed[df_changed["value_new"] != df_changed["value_old"]]
```

### Incremental load — filter rows newer than a watermark
```python
df = df[df["updated_at"] > last_watermark]
```

### Enrich a fact table by joining a lookup table
```python
df = pd.merge(df_orders, df_customers[["id", "name", "country"]], left_on="customer_id", right_on="id", how="left")
```

### Flag bad rows for quarantine
```python
df["is_invalid"] = df["email"].isnull() | (df["amount"] < 0)
df_clean = df[~df["is_invalid"]]
df_quarantine = df[df["is_invalid"]]
```

### Summarize a full table in one call
```python
df.describe(include="all")
```

### Apply a transformation to all string columns at once
```python
str_cols = df.select_dtypes(include="object").columns
df[str_cols] = df[str_cols].apply(lambda col: col.str.strip().str.lower())
```

### Read a large CSV in chunks to avoid memory issues
```python
chunks = []
for chunk in pd.read_csv("large_file.csv", chunksize=100_000):
    chunks.append(chunk[chunk["status"] == "active"])
df = pd.concat(chunks, ignore_index=True)
```

---

*Last updated: 2026*
