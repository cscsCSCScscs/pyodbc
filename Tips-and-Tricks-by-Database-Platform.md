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

### Microsoft SQL Server

#### Stored Procedures with output parameters and/or return values

See the [Calling Stored Procedures](https://github.com/mkleehammer/pyodbc/wiki/Calling-Stored-Procedures) page for an example of how to use a bit of T-SQL to retrieve these values.

#### Connecting to a named instance of SQL Server from a Linux client

Microsoft's SQL Server ODBC Driver for Linux is unable to resolve SQL Server instance names. However, if the SQL Browser service is running on the target machine we can use the (free) third-party [sqlserverport](https://github.com/gordthompson/sqlserverport) module to look up the TCP port based on the instance name.

#### DATETIMEOFFSET columns ("ODBC SQL type -155 is not yet supported")

Use an Output Converter function to retrieve such values. See the examples on the [[Using an Output Converter function]] wiki page.

#### SQL Server Numeric Precision v. Python Decimal Precision

Python's decimal.Decimal type can represent floating point numbers with greater than 35 digits of precision, which is the maximum supported by SQL server. Binding parameters that exceed this precision will result in an invalid precision error from the driver ("HY104 [Microsoft][...]Invalid precision value"). 

#### Using fast_executemany with a #temporary table

`fast_executemany` can have difficulty identifying the column types of a #temporary table under older versions of "ODBC Driver x for SQL Server" where x < 17. However, it does work with "ODBC Driver 17 for SQL Server" if we include `ColumnEncryption=Enabled` in the connection string, e.g.,

```
cnxn_str = (
    "Driver=ODBC Driver 17 for SQL Server;"
    "Server=192.168.1.144,49242;"
    "UID=sa;PWD=whatever;"
    "Database=myDb;"
    "ColumnEncryption=Enabled;"
)