# Enterprise SQL Dataset

A curated text-to-SQL dataset for enterprise data warehouse queries.

## Contents

- `dataset.json` - 119 text-to-SQL examples with questions, SQL queries, table mappings, and join keys
- `test-train-split.json` - Pre-defined train/test split (62 train, 57 test samples)
- `tables.json` - Schema definitions for all 97 tables (columns, types, primary/foreign keys)
- `join_keys.json` - All valid foreign key relationships between tables
- `schema_preview.txt` - Human-readable schema with sample rows and value ranges

## Database

The MySQL database dump is available in the [Releases](../../releases) section.

**Download:** `new_dw_indexed.sql.zip` (245 MB compressed, 2.33 GB uncompressed)

### Database Domain

This is an institutional data warehouse with tables covering:

- **Facilities**: `FCLT_BUILDING`, `FCLT_ROOMS`, `FCLT_FLOOR`, `FAC_BUILDING`, `FAC_ROOMS`
- **Organization**: `FCLT_ORGANIZATION`, `FAC_ORGANIZATION`, `SIS_DEPARTMENT`, `MASTER_DEPT_HIERARCHY`
- **Academic**: `ACADEMIC_TERMS`, `SIS_COURSE_DESCRIPTION`, `IAP_SUBJECT_*`
- **Personnel**: `HR_FACULTY_ROSTER`, `EMPLOYEE_DIRECTORY`
- **Resources**: `SPACE_UNIT`, `LIBRARY_*`, `TIP_*`
- **Time**: `TIME_DAY` for temporal joins

### Setting up MySQL

#### 1. Install MySQL (macOS)

```bash
brew install mysql
brew services start mysql
```

#### 2. Create a Database User

Connect as root (no password by default):

```bash
mysql -u root
```

Create your user and grant privileges:

```sql
CREATE USER '<username>'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON *.* TO '<username>'@'localhost';
FLUSH PRIVILEGES;
exit
```

#### 3. Download and Import the Database

Download `new_dw_indexed.sql.zip` from the [Releases](../../releases) page, then:

```bash
# Unzip
unzip new_dw_indexed.sql.zip

# Import (will prompt for password)
mysql -u <username> -p < new_dw_indexed.sql
```

Importing takes a few minutes due to the database size.

#### 4. Verify the Import

```bash
mysql -u <username> -p -e "USE dw; SHOW TABLES;" | head -20
```

### Connecting from Python

```python
import mysql.connector

conn = mysql.connector.connect(
    host="localhost",
    user="<username>",
    password="<password>",
    database="dw"
)

cursor = conn.cursor()
cursor.execute("SELECT COUNT(*) FROM BUILDINGS")
print(cursor.fetchone())
```

Or use environment variables:

```bash
export MYSQL_USER=<username>
export MYSQL_PASSWORD=<password>
export MYSQL_HOST=localhost
export MYSQL_DATABASE=dw
```

## Schema Files

### tables.json

Contains detailed schema for all 97 tables. Each entry includes:

```json
{
  "dw#sep#BUILDINGS": {
    "db_id": "dw",
    "table_name_original": "BUILDINGS",
    "column_names_original": ["BUILDING_KEY", "BUILDING_NAME", ...],
    "column_types": ["varchar NOT NULL", "varchar DEFAULT NULL", ...],
    "primary_key": ["BUILDING_KEY"],
    "foreign_key": []
  }
}
```

### join_keys.json

List of all valid join relationships between tables:

```json
[
  ["FCLT_ROOMS.FCLT_BUILDING_KEY", "FCLT_BUILDING_ADDRESS.FCLT_BUILDING_KEY"],
  ["MASTER_DEPT_HIERARCHY.DLC_KEY", "FCLT_ORG_DLC_KEY.DLC_KEY"],
  ...
]
```

### schema_preview.txt

Human-readable overview of each table with:
- Join relationships
- Column names and types
- Sample rows (3 per table)

```
============================================================
TABLE: ACADEMIC_TERMS
============================================================

JOIN RELATIONSHIPS:
  TERM_CODE -> ACADEMIC_TERMS_ALL.TERM_CODE
  TERM_CODE -> SUBJECT_OFFERED.TERM_CODE
  ...

COLUMNS:
  ACADEMIC_TERMS_KEY: varchar (nullable)
  TERM_CODE: varchar (nullable)
  ...

SAMPLE ROWS (3):
  ACADEMIC_TERMS_KEY | TERM_CODE | TERM_DESCRIPTION       | ...
  ----------------------------------------------------------------
  2010JA             | 2010JA    | January Term 2009-2010 | ...
```

## Dataset Format

Each entry in `dataset.json` contains:

| Field | Description |
|-------|-------------|
| `question` | Natural language question |
| `db_id` | Database identifier |
| `sql` | Gold SQL query (MySQL syntax) |
| `gold_tables` | Tables used in the query |
| `mapping` | Entity-to-column mappings |
| `join_keys` | Foreign key relationships used |
| `sample_id` | Unique sample identifier |

### Example entry

```json
{
  "question": "What is the current building key, building street address, city, state, and postal code of the history department?",
  "db_id": "dw",
  "sql": "SELECT DISTINCT d.FCLT_BUILDING_KEY, e.BUILDING_STREET_ADDRESS, d.CITY, d.STATE, d.POSTAL_CODE FROM FCLT_BUILDING_ADDRESS d JOIN FCLT_ROOMS a ON a.FCLT_BUILDING_KEY = d.FCLT_BUILDING_KEY ...",
  "gold_tables": ["dw#sep#FCLT_BUILDING_ADDRESS", "dw#sep#FCLT_ROOMS", ...],
  "mapping": {
    "building key": ["FCLT_BUILDING_ADDRESS.FCLT_BUILDING_KEY"],
    "city": ["FCLT_BUILDING_ADDRESS.CITY"],
    ...
  },
  "join_keys": [
    ["FCLT_BUILDING_ADDRESS.FCLT_BUILDING_KEY", "FCLT_ROOMS.FCLT_BUILDING_KEY"],
    ...
  ],
  "sample_id": 0
}
```

## Usage

```python
import json

# Load dataset
with open('dataset.json') as f:
    data = json.load(f)

# Load train/test split
with open('test-train-split.json') as f:
    split = json.load(f)

train_samples = [data[i] for i in split['train']]
test_samples = [data[i] for i in split['test']]

print(f"Train: {len(train_samples)}, Test: {len(test_samples)}")
```
