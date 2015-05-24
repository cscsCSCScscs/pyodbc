Connection objects manage connections to the database. Each object manages a single ODBC connection (specifically a single HDBC).

Connections are created through the module's `connect()` function, e.g.:

`myconnection = pyodbc.connect(myconnectionstring, autocommit=True)`

For reference, the Python DB API for connections is [here](https://www.python.org/dev/peps/pep-0249/#connection-objects).

### Connection Attributes
#### autocommit
True if the connection is in autocommit mode, False otherwise. As per the [Python DB API](https://www.python.org/dev/peps/pep-0249/), the default value is False (even though the ODBC default value is [True](https://msdn.microsoft.com/en-us/library/ms131281.aspx)). This value can be changed on a connection dynamically, and all subsequent SQL statements will be executed using the new setting.

#### searchescape
The ODBC search pattern escape character, as returned by [SQLGetInfo](https://msdn.microsoft.com/en-us/library/ms711681%28v=vs.85%29.aspx)(SQL\_SEARCH\_PATTERN\_ESCAPE), used to escape special characters such as '%' and ''. These are driver specific.

#### timeout
The timeout value, in seconds, for an individual SQL query. Use zero, the default, to disable.

The timeout is applied to all cursors created by the connection, so it cannot be changed for a specific cursor or SQL statement. If a query timeout occurs, the database should raise an OperationalError exception with SQLSTATE HYT00 or HYT01.

Note, this attribute affects only SQL queries. To set the timeout for the actual connection process, use the `timeout` keyword of the `pyodbc.connect()` function.

### Functions
#### cursor()

Returns a new Cursor object using the connection.

`mycursor = cnxn.cursor()`

pyodbc supports multiple cursors per connection but your database may not.

#### commit()
Commits all SQL statements executed on the connection since the last commit/rollback.

`cnxn.commit()`

Note, this will commit the SQL statements from ALL the cursors created from this connection.

#### rollback()
Rolls back all SQL statements executed on the connection since the last commit/rollback.

`cnxn.rollback()`

You can call this even if no SQL statements have been executed on the connection, allowing it to be used in finally statements, etc.

#### close()
Closes the connection.  Note, any uncommitted effects of SQL statements on the database from this connection will be rolled back and lost forever!

`cnxn.close()`

Connections are automatically closed when they are deleted (typically when they go out of scope) so you should not normally need to call this, but you can explicitly close the connection if you wish.

Trying to use a connection after it has been closed will result in a ProgrammingError exception.

#### getinfo()
This function is not part of the Python DB API.

Returns general information about the driver and data source associated with a connection by calling [SQLGetInfo](https://msdn.microsoft.com/en-us/library/ms711681.aspx), e.g.:

`data_source_name = cnxn.getinfo(pyodbc.SQL_DATA_SOURCE_NAME)`

See Microsoft's SQLGetInfo [documentation](https://msdn.microsoft.com/en-us/library/ms711681.aspx "SQLGetInfo function") for the types of information available.

#### execute()
This function is not part of the Python DB API.

Creates a new Cursor object, calls its execute method, and returns the new cursor.

`num_products = cnxn.execute("SELECT COUNT(*) FROM product")`

See Cursor.execute() for more details. This is a convenience method that is not part of the DB API. Since a new Cursor is allocated by each call, this should not be used if more than one SQL statement needs to be executed on the connection.

#### add_output_converter()

Register an output converter function that will be called whenever a value with the given SQL type is read from the database.

`add_output_converter(sqltype, func)`

* sqltype: the integer SQL type value to convert, which can be one of the defined standard constants (e.g. pyodbc.SQL_VARCHAR) or a database-specific value (e.g. -151 for the SQL Server 2008 geometry data type).
* func: the converter function which will be called with a single parameter, the value, and should return the converted value. If the value is NULL, the parameter will be None. Otherwise it will be a Python string.

#### clear_output_converters()

Removes all output converter functions.

### Database Transaction Management When AutoCommit is False

When using pyodbc with `autocommit=False`, it is important to understand that database transactions are never explicitly opened.  A database transaction is implicitly opened when a Connection object is created, with `pyodbc.connect()`. That database transaction is then either committed or rolled-back by explicitly calling `commit()` or `rollback()` on the connection, at which point a new database transaction is implicitly opened. Bear in mind, each database transaction may include the effects of just one SQL statement or multiple SQL statements, each executed using the `Cursor.execute()` function.  Hence, the equivalent of the following SQL:
```
BEGIN TRANSACTION
  UPDATE T1 SET ...
  DELETE FROM T1 WHERE ...
  INSERT INTO T1 VALUES ...
COMMIT TRANSACTION
```
in Python would be:
```python
cnxn = pyodbc.connect('mydsn', autocommit=False)
crsr = cnxn.cursor()
crsr.execute("UPDATE T1 SET ...")
crsr.execute("DELETE FROM T1 WHERE ...")
crsr.execute("INSERT INTO T1 VALUES ...")
cnxn.commit()
```
As you can see, transactions are managed at the Connection level, not the Cursor level.  Cursors are merely vehicles to execute SQL statements and manage their results.  There is a convenience function `commit()` on the Cursor object but that simply calls `commit()` on the cursor's parent Connection object.  When `commit()` is called on a connection, all executed SQL statements from ALL the cursors on that connection are committed together (similarly for `rollback()`).  In the event that a Connection object goes out of scope before it is closed, it is automatically deleted by Python, and a `rollback()` is issued on the connection as part of the deletion process.

### Connection objects and the Python context manager syntax
The Python context manager syntax can be used with Connection objects, and the following code:
```python
with pyodbc.connect('mydsn') as cnxn:
    do_stuff
```    
is exactly equivalent to:
```python
cnxn = pyodbc.connect('mydsn')
do_stuff
if not cnxn.autocommit:
    cnxn.commit()  
```