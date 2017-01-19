## Python 3

#### Python parameters sent to the database

The following table describes how Python objects passed to Cursor.execute() as parameters are formatted and passed to the driver/database.

| Python Datatype | Description | ODBC Datatype |
| --- | --- | --- |
| None | varies | varies (1) |
| str | UTF-16LE (2) | SQL_VARCHAR or SQL_LONGVARCHAR (2)(3) |
| bytes, bytearray | binary | SQL_VARBINARY or SQL_LONGVARBINARY (3) |
| bool | bit | BIT |
| datetime.date | date | SQL_TYPE_DATE |
| datetime.time | time | SQL_TYPE_TIME |
| datetime.datetime | timestamp | SQL_TIMESTAMP |
| int | integer | SQL_BIGINT |
| float | floating point | SQL_DOUBLE |
| decimal | numeric | SQL_NUMERIC |

1. If the driver supports it, [SQLDescribeParam](https://msdn.microsoft.com/en-us/library/ms710188.aspx) is used to determine the appropriate type. If not supported, SQL_VARCHAR is used.

2. The encoding and ODBC data type can be changed using Connect.setdecoding.  See
   the [Unicode page](Unicode).

3. [SQLGetTypeInfo](https://msdn.microsoft.com/en-us/library/ms714632.aspx) is used to determine when the LONG types are used.  If it is not supported, 1MB is used.

#### SQL values received from the database

The following table describes how database results are converted to Python objects.

| Description | ODBC Datatype | Python Datatype |
|-------------|----------------|------------------|
| NULL | any | None |
| 1-byte text | SQL_CHAR | str via UTF-8 (1) |
| 2-byte text | SQL_WCHAR | str via UTF-16LE (1) |
| GUID | SQL_GUID | str |
| XML | SQL_XML | str via UTF-16LE (1) |
| binary | SQL_BINARY, SQL_VARBINARY | bytes |
| decimal, numeric | SQL_DECIMAL | decimal.Decimal |
| bit | SQL_BIT | bool |
| integers | SQL_TINYINT, SQL_SMALLINT, SQL_INTEGER, SQL_BIGINT | int |
| floating point | SQL_REAL, SQL_FLOAT, SQL_DOUBLE | float |
| time | SQL_TYPE_TIME | datetime.time |
| SQL Server time | SS_TIME2 | datetime.time |
| date | SQL_TYPE_DATE | datetime.date |
| timestamp | SQL_TIMESTAMP | datetime.datetime |

1. The encoding can be changed using Connect.setdecoding.  See the [Unicode page](Unicode).


## Python 2

#### Python parameters sent to the database

| Python Datatype | Description | ODBC Datatype |
|------------------|-------------|----------------|
| None | varies | varies (1) |
| str | UTF-8 (2) | SQL_CHAR (2) |
| unicode | UTF-16LE (2) | SQL_WCHAR (2) |
| bytearray | binary | SQL_VARBINARY or SQL_LONGVARBINARY (3) |
| buffer | binary | SQL_VARBINARY or SQL_LONGVARBINARY (3) |
| bool | bit | BIT |
| datetime.date | date | SQL_TYPE_DATE |
| datetime.time | time | SQL_TYPE_TIME |
| datetime.datetime | timestamp | SQL_TIMESTAMP |
| int | integer | SQL_INTEGER |
| long | bigint | SQL_BIGINT |
| float | double | SQL_DOUBLE |
| decimal | numeric | SQL_NUMERIC |

1. If the driver supports it, [SQLDescribeParam](https://msdn.microsoft.com/en-us/library/ms710188.aspx) is used to determine the appropriate type. If not supported, SQL_VARCHAR is used.

2. The encoding and ODBC data type can be changed using Connect.setencoding.  See
   the [Unicode page](Unicode).


3. [SQLGetTypeInfo](https://msdn.microsoft.com/en-us/library/ms714632.aspx) is used to determine when the LONG types are used.  If it is not supported, 1MB is used.

#### SQL values received from the database

The following table describes how database results are converted to Python objects.

| Description | ODBC Datatype | Python Datatype |
|-------------|----------------|------------------|
| NULL | any | None |
| 1-byte text | SQL_CHAR | unicode via UTF-8 (1) |
| 2-byte text | SQL_WCHAR | unicode via UTF-16LE (1) |
| GUID | SQL_GUID | unicode |
| XML | SQL_XML | unicode |
| binary | SQL_BINARY, SQL_VARBINARY | bytearray |
| decimal, numeric | SQL_DECIMAL, SQL_DECIMAL | decimal.Decimal |
| bit | SQL_BIT | bool |
| integers | SQL_TINYINT, SQL_SMALLINT, SQL_INTEGER, SQL_BIGINT | long |
| floating point | SQL_REAL, SQL_FLOAT, SQL_DOUBLE | float |
| time | SQL_TYPE_TIME | datetime.time |
| SQL Server time | SS_TIME2 | datetime.time |
| date | SQL_TYPE_DATE | datetime.date |
| timestamp | SQL_TIMESTAMP | datetime.datetime |

1. The encoding and the Python type can be changed using Connect.setdecoding.  See the [Unicode page](Unicode).

Note that these are pyodbc 4.x data types.  Earlier versions returned `str` objects for
SQL_CHAR buffers and performed no decoding.  SQL_WCHAR buffers were assumed to be UCS-2.
