The following is a collection of tips for working with specific database platforms. For additional tips that apply to pyodbc as a whole, see [Features beyond the DB API](https://github.com/mkleehammer/pyodbc/wiki/Features-beyond-the-DB-API).

[Microsoft Access](#microsoft-access)&nbsp;&nbsp;|&nbsp;&nbsp;[Microsoft SQL Server](#microsoft-sql-server)

### Microsoft Access

#### Databases containing ODBC Linked Tables

pyodbc can work with Access databases that contain ODBC Linked Tables if we disable connection pooling before connecting to the Access database:

```python
import pyodbc
pyodbc.pooling = False
cnxn = pyodbc.connect(r"DRIVER={Microsoft Access Driver (*.mdb, *.accdb)};DBQ= ... ")
```

#### Creating a new database file

Access DDL does not support `CREATE DATABASE` (or similar). If you need to create a new, empty database file you can use the (free) third-party [msaccessdb](https://github.com/gordthompson/msaccessdb) module.

#### Limitations of the Access ODBC driver

The Access ODBC driver is not able to fully support some of the "complex" column types like "(multi-valued) Lookup" fields and "Attachment" fields. In some cases the ODBC driver can read values from such a column, but is unable to perform INSERT/UPDATE/DELETE operations. If you have to perform such tasks then you may need to use [Access DAO](https://msdn.microsoft.com/en-us/library/office/dn124645.aspx) (Data Access Objects), perhaps in conjunction with [IronPython](https://github.com/IronLanguages/main/releases) or [Python for .NET](http://pythonnet.github.io/).

There is a list of more general Microsoft Access specifications and limitations [here](http://office.microsoft.com/en-ca/access-help/access-2010-specifications-HA010341462.aspx).

#### UnicodeDecodeError when calling Cursor.columns

There is an issue with the Access ODBC driver that can cause a UnicodeDecodeError when trying to use the Cursor.columns method if the table definition in Access includes optional column "Description" information:

```
Field Name  Data Type   Description
----------  ----------  --------------------
ID          AutoNumber  identity primary key
LastName    Text        Family name
FirstName   Text        Given name(s)
DOB         Date/Time   Date of Birth
```

For a discussion of the issue and possible workarounds, see [issue #328](https://github.com/mkleehammer/pyodbc/issues/328).

### Microsoft SQL Server

#### Stored Procedures with output parameters and/or return values

See the [Calling Stored Procedures](https://github.com/mkleehammer/pyodbc/wiki/Calling-Stored-Procedures) page for an example of how to use a bit of T-SQL to retrieve these values.

#### Connecting to a named instance of SQL Server from a Linux client

Microsoft's SQL Server ODBC Driver for Linux is unable to resolve SQL Server instance names. However, if the SQL Browser service is running on the target machine we can use the (free) third-party [sqlserverport](https://github.com/gordthompson/sqlserverport) module to look up the TCP port based on the instance name.

#### DATETIMEOFFSET columns (e.g., "ODBC SQL type -155 is not yet supported")

Use an Output Converter function to retrieve such values. See the examples on the [[Using an Output Converter function]] wiki page.

Query parameters for DATETIMEOFFSET columns currently must be sent as strings. Note that SQL Server is rather fussy about the format of the string:

```python
# sample data
dto = datetime(2018, 8, 2, 0, 28, 12, tzinfo=timezone(timedelta(hours=-6)))

dto_string = dto.strftime("%Y-%m-%d %H:%M:%S %z")  # 2018-08-02 00:28:12 -0600
# Trying to use the above will fail with
#   "Conversion failed when converting date and/or time from character string."
# We need to add the colon for SQL Server to accept it
dto_string = dto_string[:23] + ":" + dto_string[23:]  # 2018-08-02 00:28:12 -06:00
```

#### TIME columns

Due to legacy considerations, pyodbc uses the ODBC `TIME_STRUCT` structure for `datetime.time` query parameters. `TIME_STRUCT` does not understand fractional seconds, so `datetime.time` values have their fractional seconds truncated when passed to SQL Server.

```python
crsr.execute("CREATE TABLE #tmp (id INT, t TIME)")
t = datetime.time(hour=12, minute=23, second=34, microsecond=567890)
crsr.execute("INSERT INTO #tmp (id, t) VALUES (1, ?)", t)
rtn = crsr.execute("SELECT CAST(t AS VARCHAR) FROM #tmp WHERE id=1").fetchval()
print(rtn)  # 12:23:34.0000000
```

The workaround is to pass the query parameter as a string

```python
crsr.execute("INSERT INTO #tmp (id, t) VALUES (1, ?)", str(t))
rtn = crsr.execute("SELECT CAST(t AS VARCHAR) FROM #tmp WHERE id=1").fetchval()
print(rtn)  # 12:23:34.5678900
```

Note that TIME columns *retrieved* by pyodbc have their microseconds intact

```python
rtn = crsr.execute("SELECT t FROM #tmp WHERE id=1").fetchval()
print(repr(rtn))  # datetime.time(12, 23, 34, 567890)
```


#### SQL Server Numeric Precision vs. Python Decimal Precision

Python's decimal.Decimal type can represent floating point numbers with greater than 35 digits of precision, which is the maximum supported by SQL server. Binding parameters that exceed this precision will result in an invalid precision error from the driver ("HY104 [Microsoft][...]Invalid precision value"). 

#### Using fast_executemany with a #temporary table

`fast_executemany` can have difficulty identifying the column types of a local #temporary table under some circumstances ([#295](https://github.com/mkleehammer/pyodbc/issues/295)). 


##### -- Workaround 1: ODBC Driver 17 for SQL Server and `ColumnEncryption=Enabled`

Use "ODBC Driver 17 for SQL Server" (or newer) and include `ColumnEncryption=Enabled` in the connection string, e.g.,

```python
cnxn_str = (
    "Driver=ODBC Driver 17 for SQL Server;"
    "Server=192.168.1.144,49242;"
    "UID=sa;PWD=whatever;"
    "Database=myDb;"
    "ColumnEncryption=Enabled;"
)
```

##### -- Workaround 2: pyodbc 4.0.24 and `Cursor.setinputsizes`

Upgrade to pyodbc 4.0.24 (or newer) and use `setinputsizes` to specify the parameter type, etc..

```python
crsr.execute("""\
CREATE TABLE #issue295 (
    id INT IDENTITY PRIMARY KEY, 
    txt NVARCHAR(50), 
    dec DECIMAL(18,4)
    )""")
sql = "INSERT INTO #issue295 (txt, dec) VALUES (?, ?)"
params = [('Ώπα', 3.141)]
# explicitly set parameter type/size/precision
crsr.setinputsizes([(pyodbc.SQL_WVARCHAR, 50, 0), (pyodbc.SQL_DECIMAL, 18, 4)])
crsr.fast_executemany = True
crsr.executemany(sql, params)
```

##### -- Workaround 3: Use a global ##temporary table

If neither of the previous workarounds is feasible, simply use a global ##temporary table instead of a local #temporary table.