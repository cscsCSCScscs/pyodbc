For reference, the Python DB API for the pyodbc module is [here](https://www.python.org/dev/peps/pep-0249/#module-interface).

### Pyodbc Attributes
#### version
The module version string in the format *major.minor.revision*

#### apilevel
The string constant "2.0" indicating this module supports DB API level 2.0.

#### lowercase
A boolean value that controls whether column names in result rows are lowercased. This can be changed any time and affects queries executed after the change. The default is False. This can be useful when database columns have inconsistent capitalization.

#### pooling
A boolean value indicating whether connection pooling is enabled. This is a global (HENV) setting, so it can only be modified before the first connection is made. The default is True, which enables ODBC connection pooling.

#### threadsafety
The integer 1, indicating that threads may share the module but not connections. Note that connections and cursors may be used by different threads, just not at the same time.

#### paramstyle
The string constant "qmark" to indicate parameters are identified using question marks.

### Functions
#### connect(\*connectionstring, **kwargs)

Creates and returns a new connection to the database, e.g.:

`cnxn = pyodbc.connect(myconnectionstring, autocommit=True)`

The optional connection string can be passed as a string, as keywords, or both.
```python
# a single semicolon-delimited string
cnxn = connect('driver={SQL Server};server=localhost;database=test;uid=myuid;pwd=mypwd')
# keywords
cnxn = connect(driver='{SQL Server}', server='localhost', database='test', uid='myuid', pwd='mypwd')
# both
cnxn = connect('driver={SQL Server};server=localhost;database=test', uid='myuid', pwd='mypwd')
```

The [Python DB API](https://www.python.org/dev/peps/pep-0249/#footnotes) recommends some keywords that are not usually used in ODBC connection strings, so they are converted:

- host => server
- user => uid
- password => pwd

Some keywords are used by pyodbc and are not passed to the odbc driver:

- **autocommit** (default False): if True, after execution, each SQL statement is automatically committed by the database itself (note, not explicitly by pyodbc)
- **readonly** (default False): if True, the connection is set to read-only
- **unicode_results** (default False): if True, strings returned in result sets are always Unicode (2.1.5+)
- **ansi** (default False): if True, the driver does not support Unicode and SQLDriverConnectA should be used

The ansi keyword should be used only to work around driver bugs. Pyodbc will determine if the Unicode connection function SQLDriverConnectW exists and always attempt to call it. If the driver returns IM001 indicating it does not support the Unicode version, the ANSI version is tried (SQLDriverConnectA). Any other SQLSTATE is turned into an exception. Setting ansi to true skips the Unicode attempt and only connects using the ANSI version. This is useful for drivers that return the wrong SQLSTATE (or if pyodbc is out of date and should support other SQLSTATEs).

#### dataSources()
Returns a dictionary object, mapping available DSN names to their descriptions, e.g.:

`mysources = dataSources()`

On Windows, these will be the ones defined in the [ODBC Data Source Administrator](https://msdn.microsoft.com/en-us/library/ms188691.aspx).  On Unix, these will be the ones defined in the user's odbc.ini file (typically ~/.odbc.ini) and/or the system odbc.ini file (typically /etc/odbc.ini).