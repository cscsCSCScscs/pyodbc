Connections to databases are made through the use of connection strings in the `pyodbc.connect()` function, but the most important thing to remember is that pyodbc does not even look at the connection string.  It is passed directly to the database driver unmodified (through [SQLDriverConnect](http://msdn.microsoft.com/en-us/library/ms715433.aspx)). Connection strings are therefore driver-specific and all ODBC connection string documentation should be valid. Also, anything ODBC supports, pyodbc supports, including DSN-less connections and file DSNs.

General connection string information for most databases: http://www.connectionstrings.com

Important formatting rules: http://www.connectionstrings.com/formating-rules-for-connection-strings/