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
Objective:
We are given 100GB of CSV files (10 in total), each containing stock price and volume data in the format:

date:: datetime(%Y-%m-%d) id::int price::float trade_volume::int

Each day has one entry per stock (IDs 1–200). The task is to:

1. Efficiently load and transform this data on a resource-constrained server (32GB RAM, 4 CPUs, 100GB disk, 20GB temp).

2. Output two wide tables (one for prices, one for volumes) with the structure:

  date, stk_001, stk_002, ..., stk_200

3. Calculate daily stock returns.

4. Work within constraints: no permanent tables, limited temp space.

5. Error handling, retries, logging, validation included

 
## 🧠 Design Overview
💡 Transform into Wide tables with 1 row per date, and 200 columns ( stk_001 to stk_200)

Save

price.parquet

volume.parquet

returns.parquet ( % change of price )

Constraints

Limited memory ( 32GB)
CPUs (4)
Temp Storage ( 20GB)
Total disk space (100GB)
No permanent Database tables

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

- Dask for distributed and memory-efficient processing

- Pandas used only for light operations (e.g., reindexing)

- NumPy for numerical efficiency

```python
import dask.dataframe as dd
import pandas as pd
import numpy as np
import os
```



---

### 2. Parameter Setup

```python

logging.basicConfig( level=logging.INFO,format="%(asctime)s - %(levelname)s - %(message)s")
PRICE_TMP_DIR = "tmp_price"
VOLUME_TMP_DIR = "tmp_volume"
FINAL_PRICE_PATH = "final_price.parquet"
FINAL_VOLUME_PATH = "final_volume.parquet"
RETURNS_PATH = "returns.parquet"
STOCK_COLS = [f"stk_{i:03d}" for i in range(1, 201)]
MAX_RETRIES = 3

os.makedirs(PRICE_TMP_DIR, exist_ok=True)
os.makedirs(VOLUME_TMP_DIR, exist_ok=True)
```

- File discovery
- Temp output directories for intermediate storage
- Generate 200 expected stock columns

---

### 3. Read + Pivot + Save Intermediate Chunks

-  ** read_csv_with_retry function **
-  ** process_csv **
-  ** validate_parquet_folder **
-  ** combne_and_save **

### 4. Calculate Returns 

- ** Calculate_returns **
  
### 5. Cleanup 

- ** cleanup **

### 6. Create Mock CSV 

- ** create_mock_csv() **

### Main ETL Pipeline

- ** run_etl () **

 

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
