# Internet Meter SQL Lab Solution

## Part 1: Set up your Database

### Environment Setup
```bash
# Install required packages
apt-get update
apt-get -y install sqlite3
pip install csvkit
```

### Database Creation
```bash
# Create the InternetSpeed database
sqlite3 InternetSpeed.db ".databases"

# Remove database if it already exists
test -e InternetSpeed.db && rm InternetSpeed.db

# Import the LA_wifi_speed_UK.csv file into a SQL database
csvsql --db sqlite:///InternetSpeed.db --insert ./Data/LA_wifi_speed_UK.csv
```

## Part 2: Connect to the Database

### Import Required Libraries
```python
# Import libraries for database interaction and visualization
import sqlite3
import pandas as pd
from matplotlib import pyplot as plt
%matplotlib inline
```

### Connect and Execute a Query
```python
# Connect to the database
conn = sqlite3.connect('InternetSpeed.db')

# Create a cursor
cur = conn.cursor()

# Execute a query to get first 10 rows of DateTime
query = 'SELECT DateTime FROM LA_wifi_speed_UK ORDER BY DateTime LIMIT 10;'
cur.execute(query)

# Print results
for row in cur:
    print(row)
```

### Focus on Specific Data Portions
```python
# Get column names
query = 'SELECT * FROM LA_wifi_speed_UK LIMIT 1'
cur.execute(query)

# Extract column names from cursor description
columns = [member[0] for member in cur.description]

# Ignore index column 
columns = columns[1:]

# Remove suffixes to get area names
columns = [c.replace('_p', '') for c in columns]
columns = [c.replace('_d', '') for c in columns]
columns = [c.replace('_u', '') for c in columns]

# Remove duplicates
columns = list(set(columns))

# Create suffix dictionary for measurements
suffix = {'_p':'ping', '_d':'download', '_u':'upload'}

# Plot measurements for a specific area
area = columns[0]
plt.figure(figsize=(10, 8))

for s in suffix.keys():
    query = 'SELECT {}{} FROM LA_wifi_speed_UK ORDER BY DateTime'.format(area, s)
    cur.execute(query)
    plt.plot(cur.fetchall(), label=suffix[s])
    
plt.legend()
plt.title(area)
```

## Part 3: Manipulate the Data with Pandas

### Creating a DataFrame for Averages
```python
# Create empty dataframe with specific columns
new_columns = ['Area', 'Average_p', 'Average_d', 'Average_u']
df = pd.DataFrame(columns=new_columns)

# Calculate average speeds for each area
for i in range(len(columns)-1):
    tmp_list = []
    tmp_list.append(columns[i])
    for s in suffix.keys():
        query = 'SELECT AVG({}{}) FROM LA_wifi_speed_UK'.format(columns[i], s)
        cur.execute(query)
        mean = cur.fetchone()
        tmp_list.append(mean[0])
    df = df.append(pd.Series(tmp_list, index=new_columns), ignore_index=True)

# Visualize averages
plt.figure(figsize=(20,10))
plt.plot(df.index, df[['Average_d','Average_u','Average_p']], 'o')
plt.legend(['Average Download', 'Average Upload', 'Average ping'])

# Save dataframe to database
try:
    cur.execute('DROP TABLE average_speed')
except:
    pass

df.to_sql('average_speed', conn)

# Verify table creation
query_2 = 'SELECT * FROM average_speed'
cur.execute(query_2)
print(cur.fetchone())
print(cur.fetchone())
```

## Part 4: Join Tables

### Adding Population Data
```python
# Close current connection
conn.close()

# Import population data
csvsql --db sqlite:///InternetSpeed.db --insert ./Data/LA_population.csv

# Reopen connection
conn = sqlite3.connect('InternetSpeed.db')
cur = conn.cursor()

# Test the population table
query = 'SELECT * FROM LA_population LIMIT 10'
cur.execute(query)
for row in cur:
    print(row)

# Join the tables
query = 'SELECT * FROM average_speed JOIN LA_population ON LA_population."LA_code"=average_speed.Area'
cur.execute(query)

# Print first 10 rows of the joined table
k = 0
for row in cur:
    if k > 10:
        break
    print(row)
    k += 1
```

## Complete Code for Each Missing Section

### Code Cell 2 (Create Database)
```python
!sqlite3 InternetSpeed.db ".databases"
```

### Code Cell 6 (Connect to Database)
```python
conn = sqlite3.connect('InternetSpeed.db')
cur = conn.cursor()
```

### Code Cell 7 (Execute Query)
```python
query = 'SELECT DateTime FROM LA_wifi_speed_UK ORDER BY DateTime LIMIT 10;'
cur.execute(query)
```

### Code Cell 13 (Remove Suffix)
```python
# remove suffix '_p'
columns = [c.replace('_p', '') for c in columns]
# remove suffix '_d'
columns = [c.replace('_d', '') for c in columns]
# remove suffix '_u'
columns = [c.replace('_u', '') for c in columns]
```

### Code Cell 17 (Plot Query)
```python
area = columns[0]
plt.figure(figsize=(10, 8))

for s in suffix.keys():
    query = 'SELECT {}{} FROM LA_wifi_speed_UK ORDER BY DateTime'.format(area, s)
    cur.execute(query)
    plt.plot(cur.fetchall(), label=suffix[s])
plt.legend()
plt.title(area)
```

### Code Cell 18 (Create DataFrame)
```python
new_columns = ['Area', 'Average_p', 'Average_d', 'Average_u']
df = pd.DataFrame(columns=new_columns)
```

### Code Cell 19 (Compute Averages)
```python
for i in range(len(columns)-1):
    tmp_list = []
    tmp_list.append(columns[i])
    for s in suffix.keys():
        query = 'SELECT AVG({}{}) FROM LA_wifi_speed_UK'.format(columns[i], s)
        cur.execute(query)
        mean = cur.fetchone()
        tmp_list.append(mean[0])
    df = df.append(pd.Series(tmp_list, index=new_columns), ignore_index=True)
df.head()
```

### Code Cell 21 (Save to SQL)
```python
try:
    cur.execute('DROP TABLE average_speed')
except:
    pass

df.to_sql('average_speed', conn)
```

### Code Cell 24 (Import Population Data)
```python
!csvsql --db sqlite:///InternetSpeed.db --insert ./Data/LA_population.csv
```

### Code Cell 25 (Reopen Connection)
```python
conn = sqlite3.connect('InternetSpeed.db')
cur = conn.cursor()
```

### Code Cell 26 (Test Population Table)
```python
query = 'SELECT * FROM LA_population LIMIT 10'
cur.execute(query)

for row in cur:
    print(row)
```

### Code Cell 27 (Join Tables)
```python
query = 'SELECT * FROM average_speed JOIN LA_population ON LA_population."LA_code"=average_speed.Area'
cur.execute(query)
k = 0
for row in cur:
    if k > 10:
        break
    print(row)
    k += 1
```

## Technical Notes

1. The database schema is automatically created by `csvsql` based on the CSV file structure
2. The measurements are organized with suffixes (_p, _d, _u) for ping, download, and upload
3. String formatting is used extensively to build dynamic SQL queries
4. Pandas provides a simple interface to manipulate the data and convert between SQL and dataframes
5. The JOIN operation combines data from two tables based on a common field (Area/LA_code)
