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

The Access ODBC driver is not able to fully support some of the "complex" column types like "(multi-valued) Lookup" fields and "Attachment" fields. In some cases the ODBC driver can read values from such a column, but is unable to perform INSERT/UPDATE/DELETE operations. If you need to perform such tasks then you may need to use .NET (possibly [IronPython](https://github.com/IronLanguages/main/releases)) and Access DAO (Data Access Objects).

### Microsoft SQL Server

#### Stored Procedures with output parameters and/or return values

See the [Calling Stored Procedures](https://github.com/mkleehammer/pyodbc/wiki/Calling-Stored-Procedures) page for an example of how to use a bit of T-SQL to retrieve these values.