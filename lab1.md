# Python and SQLite Integration Lab Solution

## Part 1: Environment Setup and Database Creation

1. Install required packages:
   ```bash
   apt-get update
   apt-get -y install sqlite3
   pip install csvkit
   ```

2. Create the SQLite database:
   ```bash
   cd /home/pi/notebooks/myfiles
   sqlite3 phonebook.db
   ```

3. Create tables in the database:
   ```sql
   create table coworkers(workid integer,name varchar(20),title varchar(20),number integer);
   create table department(deptid integer,name varchar(20),number integer);
   .quit
   ```

## Part 2: Data Entry and Import

1. Create a CSV file with coworker data:
   ```bash
   cat > coworkers.csv << EOL
   workid,name,title,number
   101,Han Solo,Bounty Hunter,5556667578
   102,Leia Skywalker,Princess,5556542398
   103,Luke Skywalker,Jedi,5558963267
   104,Obi-Wan Kenobi,Jedi,5558963276
   105,Anakin Skywalker,Sith Lord,5553477621
   106,Jabba The Hutt,Gangster,5556613456
   107,Greedo,Debt Collector,5552360918
   108,R2D2,Astro Droid,5555210125
   109,C3PO,Protocol Droid,5556633345
   EOL
   ```

2. Drop the existing coworkers table and import from CSV:
   ```bash
   sqlite3 phonebook.db
   # Inside SQLite prompt
   drop table coworkers;
   .quit
   
   # Import using csvkit
   csvsql --db sqlite:////home/pi/notebooks/myfiles/phonebook.db --insert /home/pi/notebooks/myfiles/coworkers.csv
   ```

3. Verify the import with queries:
   ```bash
   sqlite3 phonebook.db
   # Inside SQLite prompt
   select * from coworkers;
   select name from coworkers where title='Jedi';
   .quit
   ```

## Part 3: Python Integration

1. Create a Python script to interact with the database:

```python
# Import required libraries
import sqlite3
import pandas as pd
from matplotlib import pyplot as plt
%matplotlib inline

# Establish connection to the database
conn = sqlite3.connect('/home/pi/notebooks/myfiles/phonebook.db')

# Create a cursor object
cur = conn.cursor()

# Query 1: Get all records from coworkers
print("All coworkers:")
query = "SELECT * FROM coworkers;"
cur.execute(query)
for row in cur:
    print(row)

# Challenge Solutions:

# 1. Query the names of all the princesses
print("\nPrincesses:")
query = "SELECT name FROM coworkers WHERE title='Princess';"
cur.execute(query)
for row in cur:
    print(row[0])

# 2. Query the names of all the princesses and debt collectors
print("\nPrincesses and Debt Collectors:")
query = "SELECT name FROM coworkers WHERE title='Princess' OR title='Debt Collector';"
cur.execute(query)
for row in cur:
    print(row[0])

# 3. Query the names and numbers of all the Jedi
print("\nJedi names and numbers:")
query = "SELECT name, number FROM coworkers WHERE title='Jedi';"
cur.execute(query)
for row in cur:
    print(f"Name: {row[0]}, Number: {row[1]}")

# 4. Query the names of the droids
print("\nDroids:")
query = "SELECT name FROM coworkers WHERE title LIKE '%Droid%';"
cur.execute(query)
for row in cur:
    print(row[0])

# Example of using pandas for data visualization
print("\nLoading data into pandas DataFrame:")
df = pd.read_sql_query("SELECT * FROM coworkers", conn)
print(df.head())

# Create a simple visualization of job titles count
print("\nCreating job titles distribution chart:")
title_counts = df['title'].value_counts()
title_counts.plot(kind='bar')
plt.title('Distribution of Job Titles')
plt.xlabel('Job Title')
plt.ylabel('Count')
plt.tight_layout()
plt.show()

# Close the connection when done
conn.close()
```

## Detailed Technical Explanation

### Database Connection
SQLite databases are managed through connection objects that provide a direct interface to the database file. The SQLite3 Python module uses a cursor-based approach for executing SQL queries, where:

1. The `connect()` method creates a connection to the database file
2. The `cursor()` method creates a control structure for navigating and manipulating records
3. The `execute()` method runs SQL queries against the database

### Query Execution Flow
When executing a query in Python with SQLite:
1. The SQL string is passed to `execute()`
2. The cursor loads the query results
3. Results can be fetched with `fetchall()`, `fetchone()`, or by iterating through the cursor

### SQL Syntax for Challenges

For the challenge queries:

1. Princess query:
   ```sql
   SELECT name FROM coworkers WHERE title='Princess';
   ```
   
2. Princesses and debt collectors:
   ```sql
   SELECT name FROM coworkers WHERE title='Princess' OR title='Debt Collector';
   ```
   
3. Jedi names and numbers:
   ```sql
   SELECT name, number FROM coworkers WHERE title='Jedi';
   ```
   
4. Droids names:
   ```sql
   SELECT name FROM coworkers WHERE title LIKE '%Droid%';
   ```
   
The `LIKE` operator with '%' wildcards allows for partial matching, which is needed for finding all droid types.

### Performance Considerations

For larger databases, consider:
1. Using indices on frequently queried fields
2. Using parameterized queries to prevent SQL injection
3. Commit transactions in batches for better performance
4. Close connections when done to release resources

This implementation demonstrates the full workflow from database creation to advanced querying with Python integration, allowing you to complete all the lab objectives efficiently.
