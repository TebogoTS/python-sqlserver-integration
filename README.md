
```markdown
Python and SQL Server Integration Example

Overview
This repository demonstrates a complete example of using Python to connect to a SQL Server database with `pyODBC` and `SQLAlchemy`. It includes:
- Setting up a local SQL Server instance using Docker.
- Creating a sample database and table.
- Inserting and cleaning data from a Pandas DataFrame.
- Logging and handling data insertion errors.

Prerequisites
- Python 3.8 or later
- Docker (for local SQL Server setup)
- SQL Server ODBC Driver (`msodbcsql17`)
- Python Packages: `pandas`, `sqlalchemy`, `pyodbc`

```

Setup Instructions

1. Start a Local SQL Server Instance
```bash
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourPassword123" -p 1433:1433 --name sqlserver -d mcr.microsoft.com/mssql/server:2022-latest
```

2. Install Required Python Packages
```bash
pip install pandas sqlalchemy pyodbc
```

3. Configure ODBC Driver (macOS)
Install and configure the SQL Server ODBC driver and `unixODBC`:
```bash
brew install unixodbc
brew tap microsoft/mssql-release https://github.com/Microsoft/homebrew-mssql-release
brew install msodbcsql17
```

4. Create a Sample Table in SQL Server
```sql
-- Connect to SQL Server and execute:
CREATE DATABASE TestDB;

USE TestDB;

CREATE TABLE JSEMTMDetailed (
    bond_code NVARCHAR(50),
    bp_spread NUMERIC(18,2),
    mtm NUMERIC(18,2),
    clean_price NUMERIC(18,2),
    duration NUMERIC(18,4)
);
```

5. Run the Python Script
```bash
python testdb.py
```

Python Script Example
```python
import pandas as pd
from sqlalchemy import create_engine
import logging

logging.basicConfig(level=logging.DEBUG)

server = "localhost,1433"
database = "TestDB"
username = "sa"
password = "YourPassword123"
driver = "ODBC Driver 17 for SQL Server"
connection_string = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver={driver}"
engine = create_engine(connection_string)

# Sample data
data = {
    "bond_code": ["BND1", "BND2", "BND3", "BND4"],
    "bp_spread": [1.5, "invalid", 3.2, None],
    "mtm": [100.5, 200.7, "bad_data", 400.9],
    "clean_price": [99.0, 100.0, 101.0, "N/A"],
    "duration": [5.25, 10.5, 15.0, None]
}
df = pd.DataFrame(data)

# Data cleaning
for col in ["bp_spread", "mtm", "clean_price", "duration"]:
    df[col] = pd.to_numeric(df[col], errors="coerce")

# Insert data into SQL Server
failed_rows = []
for idx, row in df.iterrows():
    try:
        row_df = pd.DataFrame([row])
        row_df.to_sql("JSEMTMDetailed", engine, index=False, if_exists="append", schema="dbo")
        logging.info(f"Row {idx} inserted successfully.")
    except Exception as e:
        logging.error(f"Error inserting row {idx}: {e}")
        failed_rows.append((idx, row.to_dict(), str(e)))

if failed_rows:
    print(f"\nFailed rows: {failed_rows}")
```
