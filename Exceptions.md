Python exceptions are raised by pyodbc when ODBC errors are detected. The exception classes specified in the [Python DB API](https://www.python.org/dev/peps/pep-0249/#exceptions "database exceptions") specification are used:

* DatabaseError
  * DataError
  * OperationalError
  * IntegrityError
  * InternalError
  * ProgrammingError
  * NotSupportedError

When an error occurs, the type of exception raised is based on the SQLSTATE value, typically provided by the database.

| SQLSTATE | Exception |
|----------|-----------|
| 22xxx | pyodbc.DataError |
| 23xxx 40002 | pyodbc.IntegrityError |
| 24xxx 25xxx 42xxx | pyodbc.ProgrammingError |
| 0A000 | pyodbc.NotSupportedError |
| All others | pyodbc.DatabaseError |

For example, a primary key error (attempting to insert a value when the key already exists) will raise an IntegrityError.
