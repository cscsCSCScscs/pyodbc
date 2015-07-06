### Connect to a Database

Make a direct connection to a database and create a cursor.
```python
cnxn = pyodbc.connect('DRIVER={SQL Server};SERVER=localhost;DATABASE=testdb;UID=me;PWD=pass')
cursor = cnxn.cursor()
```
Make a connection using a DSN. Since DSNs usually don't store passwords, you'll probably need to provide the PWD keyword.
```python
cnxn = pyodbc.connect('DSN=test;PWD=password')
cursor = cnxn.cursor()
```
There are lots of options when connecting, so see the [pyodbc.connect()](module#connectconnectionstring-kwargs) function and the [Connecting to databases](Connecting-to-databases) Wiki section for more details.

### Selecting Some Data
#### Select Basics
All SQL statements are executed using the cursor.execute() function. If the statement returns rows, such as a select statement, you can retrieve them using the Cursor fetch functions - fetchone(), fetchall(), fetchmany(). If there are no rows, fetchone() will return None; fetchall() and fetchmany() will both return empty lists.
```python
cursor.execute("select user_id, user_name from users")
row = cursor.fetchone()
if row:
    print(row)
```
Row objects are similar to tuples, but they also allow access to columns by name:
```python
cursor.execute("select user_id, user_name from users")
row = cursor.fetchone()
print('name:', row[1])          # access by column index
print('name:', row.user_name)   # or access by name
```
The fetchone function returns None when all rows have been retrieved.
```python
while True:
    row = cursor.fetchone()
    if not row:
        break
    print('id:', row.user_id)
```
The fetchall function returns all remaining rows in a list. If there are no rows, an empty list is returned. (If there are a lot of rows, this will use a lot of memory. Unread rows are stored by the database driver in a compact format and are often sent in batches from the database server. Reading in only the rows you need at one time will save a lot of memory.)
```python
cursor.execute("select user_id, user_name from users")
rows = cursor.fetchall()
for row in rows:
    print(row.user_id, row.user_name)
```
If you are going to process the rows one at a time, you can use the cursor itself as an iterator:
```python
cursor.execute("select user_id, user_name from users"):
for row in cursor:
    print(row.user_id, row.user_name)
```
Since cursor.execute() always returns the cursor, you can simplify this even more:
```python
for row in cursor.execute("select user_id, user_name from users"):
    print(row.user_id, row.user_name)
```
A lot of SQL statements don't fit on one line very easily, so you can always use triple quoted strings:
```python
cursor.execute("""
               select user_id, user_name
                 from users
                where last_logon < '2001-01-01'
                  and bill_overdue = 'y'
               """)
```
or use line continuations (but don't forget a space at the end of each string):
```python
cursor.execute("select user_id, user_name " \
               "from users " \
               "where last_logon < '2001-01-01' " \
                 "and bill_overdue = 'y'")
```
#### Parameters
ODBC supports parameters using a question mark as a place holder in the SQL. You provide the values for the question marks by passing them after the SQL:
```python
cursor.execute("""
               select user_id, user_name
                 from users
                where last_logon < ?
                  and bill_overdue = ?
               """, '2001-01-01', 'y')
```
This is safer than putting the values into the string because the parameters are passed to the database separately, protecting against SQL injection attacks. It is also be more efficient if you execute the same SQL repeatedly with different parameters. The SQL will be prepared only once. (pyodbc only keeps the last statement prepared, so if you switch between statements, each will be prepared multiple times.)

The Python DB API specifies that parameters should be passed in a sequence, so this is also supported by pyodbc:
```python
cursor.execute("""
               select user_id, user_name
                 from users
                where last_logon < ?
                  and bill_overdue = ?
               """, ['2001-01-01', 'y'])
```
### Inserting Data
To insert data, pass the insert SQL to Cursor.execute(), along with any parameters necessary:
```python
cursor.execute("insert into products(id, name) values ('pyodbc', 'awesome library')")
cnxn.commit()
```
```python
cursor.execute("insert into products(id, name) values (?, ?)", 'pyodbc', 'awesome library')
cnxn.commit()
```
Note the calls to cnxn.commit(). You must call commit or your changes will be lost! When the connection is closed, any pending changes will be rolled back. This makes error recovery very easy, but you must remember to call commit.

### Updating and Deleting
Updating and deleting work the same way, pass the SQL to execute. However, you often want to know how many records were affected when updating and deleting, in which case you can use the cursor.rowcount value:
```python
cursor.execute("delete from products where id <> ?", 'pyodbc')
print(cursor.rowcount, 'products deleted')
cnxn.commit()
```
Since execute() always returns the cursor, you will sometimes see code like this. (Notice rowcount on the end.)
```python
deleted = cursor.execute("delete from products where id <> 'pyodbc'").rowcount
cnxn.commit()
```
Note the calls to cnxn.commit(). You must call commit or your changes will be lost! When the connection is closed, any pending changes will be rolled back. This makes error recovery very easy, but you must remember to call commit.

### Tips and Tricks
Since single quotes are valid in SQL, use double quotes to surround your SQL:
```python
deleted = cursor.execute("delete from products where id <> 'pyodbc'").rowcount
```
It's also worthwhile considering using 'raw' strings for your SQL to avoid any inadvertent escaping (unless you really do want to specify control characters):
```python
cursor.execute("delete from products where name like '%bad\name%'") # Python will convert \n to 'new line'!
cursor.execute(r"delete from products where name like '%bad\name%'") # no escaping
```
Some databases (e.g. SQL Server) do not generate column names for calculations, in which case you need to access the columns by index. You can also use the 'as' keyword to name columns (the "as user_count" in the SQL below).
```python
row = cursor.execute("select count(*) as user_count from users").fetchone()
print('%s users' % row.user_count)
```
If there is only 1 value you need, you can put the fetch of the row and the extraction of the first column all on one line:
```python
count = cursor.execute("select count(*) from users").fetchone()[0]
print('{} users'.format(count))
```
This will not work if the first column can be NULL! In that case, fetchone() will return None and you'll get a cryptic error about NoneType not supporting indexing. If there is a default value, often you can use ISNULL or COALESCE to convert NULLs to default values directly in the SQL:
```python
maxid = cursor.execute("select isnull(max(id), 0) from users").fetchone()[0]
```
In this example, isnull(max(id), 0) causes the selected value to be 0 if max(id) returns NULL.