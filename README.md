## DuckDB ETL

![1761912291555](https://file+.vscode-resource.vscode-cdn.net/c%3A/Users/Usuario/Desktop/DuckDB-ETL/image/README/1761912291555.png)

As seen in the graphic, I extracted data from `csv` files, 6 to be specific, then created tables from each file with a few lines:

```python
tables = ["students", "teachers", "classes", "courses", "enrollments", "grades"]

for table in tables: 
	conn.execute(f"""
	CREATE OR REPLACE TABLE
		{table}
	AS SELECT
		*
	FROM
		read_csv_auto('data/raw_data/{table}.csv');""")
```

For the transformations, I modified the schema of each tabl. By default, Postgres takes `INTEGER` columns (4 bytes) as `BIGINT` (8 bytes). For columns like `age`, `id`, `credits`, this is a waste of storage. **For example:**

```python
conn.execute("""
CREATE OR REPLACE TABLE
    students_clean
AS SELECT
    CAST(student_id AS INTEGER) AS student_id,
    CAST(age AS SMALLINT) AS age,
    * EXCLUDE (student_id, age)
FROM
    students;
""")
```

Once I transformed all the tables, I simply loaded them into Postgres with this simple query

```python
for table in tables:
    clean_table_name = f"{table}_clean" # students_clean, teachers_clean, etc...
  
    conn.execute(f"""
    CREATE OR REPLACE TABLE 
        pg_db.{table} 
    AS SELECT 
        * 
    FROM 
        {clean_table_name};
    """)
  
    print(f"[{table}] table loaded succesfully into Postgres")

conn.close()
```

## How to run the project

1. Clone repository

   ```bash
   git clone https://github.com/fran-cornachione/DuckDB-ETL
   ```
2. Install requirements

   ```
   pip install -r requirements.txt
   ```
3. Replace this line with your Postgres credentials

   ```python
   conn.execute(f"""
   ATTACH 'dbname=your_dbname user=your_user password=your_password host=your_host port=5432' AS pg_db (TYPE POSTGRES);
   """)
   ```
4. On the `ETL.ipynb` file, click the _"Run All"_ button
