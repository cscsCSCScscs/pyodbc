# pyodbc Module

For reference, the Python DB API for the pyodbc module is [here](https://www.python.org/dev/peps/pep-0249/#module-interface).

## Attributes

### version

The module version string in *major.minor.patch* format such as `"4.0.2"`.

### apilevel

The string constant `"2.0"` indicating this module supports the DB API level 2.0

### lowercase

A Boolean that controls whether column names in result rows are lowercased. This can be changed
any time and affects queries executed after the change. The default is `False`. This can be
useful when database columns have inconsistent capitalization.

### native_uuid

A Boolean that determines whether SQL_GUID columns are returned as text (`False`, the default)
or `uuid.UUID` (True) objects.  The default is False for backwards compatibility, but this may
change in a future release.

### pooling

A Boolean indicating whether connection pooling is enabled. This is a global (HENV) setting, so
it can only be modified before the first connection is made. The default is `True`, which
enables ODBC connection pooling.

### threadsafety

The value `1`, indicating that threads may share the module but not connections. Note that
connections and cursors may be used by different threads, just not at the same time.

### paramstyle

The string constant "qmark" to indicate parameters are identified using question marks.

## connect

    connect(*connectionstring, **kwargs) --> Connection

Creates and returns a new connection to the database.

The connection string and keywords are put together to construct an ODBC connection string.
Python 2 accepts both ANSI and Unicode strings.  (In Python 3 all strings are Unicode.)

    # a string
    cnxn = pyodbc.connect('driver={SQL Server};server=localhost;database=test;uid=me;pwd=me2')
    # keywords
    cnxn = pyodbc.connect(driver='{SQL Server}', server='localhost', database='test', uid='me', pwd='me2')
    # both
    cnxn = pyodbc.connect('driver={SQL Server};server=localhost;database=test', uid='me', pwd='me2'
    cnxn = pyodbc.connect('DSN=my_dsn', autocommit=True)

The DB API recommends some keywords that are not usually used in ODBC connection strings, so
they are converted:

keyword   | converts to
----------|------------
host      | server
user      | uid
password  | pwd


Some keywords are reserved by pyodbc and are not passed to the odbc driver:

keyword | notes | default
------- | ----- | -------
ansi | If True, the driver does not support Unicode and SQLDriverConnectA should be used. | False
attrs_before | A dictionary of connection attributes to set before connecting. |
autocommit | If False, Connection.commit must be called; otherwise each statement is automatically commited | False
encoding | Optional encoding for the connection string. | utf-16le
readonly | If True, the connection is set to readonly | False
timeout | A timeout for the connection, in seconds.  This causes the connection's SQL_ATTR_LOGIN_TIMEOUT to be set before the connection occurs. |

### ansi</h4>

The ansi keyword should only be used to work around driver bugs. pyodbc will determine if the
Unicode connection function (SQLDriverConnectW) exists and always attempt to call it. If the
driver returns IM001 indicating it does not support the Unicode version, the ANSI version is
tried (SQLDriverConnectA). Any other SQLSTATE is turned into an exception. Setting ansi to true
skips the Unicode attempt and only connects using the ANSI version. This is useful for drivers
that return the wrong SQLSTATE (or if pyodbc is out of date and should support other
SQLSTATEs).

### attrs_before</h4>

The `attrs_before` keyword is an optional dictionary of connection attributes.  These will be
set on the connection via SQLSetConnectAttr before a connection is made.

The dictionary keys must be the integer constant defined by ODBC or the driver.  Only integer
values are supported at this time.  Below is an example that sets the SQL_ATTR_PACKET_SIZE
connection attribute to 32K.

    SQL_ATTR_PACKET_SIZE = 112
    cnxn = connect(cstring, attrs_before={ SQL_ATTR_PACKET_SIZE : 1024 * 32 })

## dataSources

    dataSources() -> { DSN : Description }

Returns a dictionary mapping available DSNs to their descriptions.

Note: unixODBC may have a bug that only returns items from the users odbc.ini file without
merging the system one.

On Windows, these will be the ones defined in the [ODBC Data Source Administrator](https://msdn.microsoft.com/en-us/library/ms188691.aspx).  On Unix, these will be the ones defined in the user's odbc.ini file (typically ~/.odbc.ini) and/or the system odbc.ini file (typically /etc/odbc.ini).

## drivers

    drivers() -> [ DriverName1, DriverName2, ... DriverNameN ]

Returns a list of ODBC Drivers that are available to pyodbc. On Windows this list will be specific to the "platform" under which the Python script is running (i.e., 64-bit or 32-bit).

## TimeFromTicks

    t = pyodbc.TimeFromTicks(tics)

Returns a datetime.time object initialized from the given ticks value (number of seconds since
the epoch; see the documentation of the standard Python time module for details).

## DateFromTicks

    d = pyodbc.DateFromTicks(d)
    
Returns a datetime.date object initialized from the given ticks value (number of seconds since
the epoch; see the documentation of the standard Python time module for details).

## TimestampFromTicks

    dt = pyodbc.TimestampFromTicks(d)
    
Returns a datetime.datetime object initialized from the given ticks value (number of seconds since
the epoch; see the documentation of the standard Python time module for details).

## setDecimalSeparator

    pyodbc.setDecimalSeparator('.')

Sets the decimal separator character used when parsing NUMERIC from the database.  The default
is to use locale's "decimal_point" value or '.' if that fails.  This function can be used to
override this.

## getDecimalSeparator

    print(pyodbc.getDecimalSeparator())
    
Returns the decimal separator character used when parsing NUMERIC from the database.
