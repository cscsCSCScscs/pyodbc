## Python 3

#### Python parameters sent to the database

The following table describes how Python objects passed to Cursor.execute() as parameters are formatted and passed to the driver/database.

| Python Datatype | Description | ODBC Datatype |
|------------------|-------------|----------------|
| None | varies | varies (1) |
| str | UCS-2 encoded Unicode | SQL_WVARCHAR or SQL_WLONGVARCHAR (2) |
| bytes, bytearray | binary | SQL_VARBINARY or SQL_LONGVARBINARY (2) |
| bool | bit | BIT |
| datetime.date | date | SQL_TYPE_DATE |
| datetime.time | time | SQL_TYPE_TIME |
| datetime.datetime | timestamp | SQL_TIMESTAMP |
| int | integer | SQL_BIGINT |
| float | floating point | SQL_DOUBLE |
| decimal | numeric | SQL_NUMERIC |

1. If the driver supports it, [SQLDescribeParam](https://msdn.microsoft.com/en-us/library/ms710188.aspx) is used to determine the appropriate type. If not supported, SQL_VARCHAR is used.
2. [SQLGetTypeInfo](https://msdn.microsoft.com/en-us/library/ms714632.aspx) is used to determine when the LONG types are used. If not supported by the driver, VARCHAR and WVARCHAR will be 255 and BINARY will be 510.

#### SQL values received from the database

The following table describes how database results are converted to Python objects.

| Description | ODBC Datatype | Python Datatype |
|-------------|----------------|------------------|
| NULL | any | None |
| UCS-2 encoded Unicode | SQL_WCHAR, SQL_WVARCHAR | str |
| ASCII | SQL_CHAR, SQL_VARCHAR | str |
| GUID | SQL_GUID | str |
| XML | SQL_XML | str |
| binary | SQL_BINARY, SQL_VARBINARY | bytes |
| decimal, numeric | SQL_DECIMAL | decimal.Decimal |
| bit | SQL_BIT | bool |
| integers | SQL_TINYINT, SQL_SMALLINT, SQL_INTEGER, SQL_BIGINT | int |
| floating point | SQL_REAL, SQL_FLOAT, SQL_DOUBLE | float |
| time | SQL_TYPE_TIME | datetime.time |
| SQL Server time | SS_TIME2 | datetime.time |
| date | SQL_TYPE_DATE | datetime.date |
| timestamp | SQL_TIMESTAMP | datetime.datetime |

## Python 2

#### Python parameters sent to the database

| Python Datatype | Description | ODBC Datatype |
|------------------|-------------|----------------|
| None | varies | varies (1) |
| unicode | UCS-2 encoded Unicode | SQL_WVARCHAR or SQL_WLONGVARCHAR (2) |
| str | ASCII | SQL_VARCHAR or SQL_LONGVARCHAR (2) |
| bytearray (3) | binary | SQL_VARBINARY or SQL_LONGVARBINARY (2) |
| buffer | binary | SQL_VARBINARY or SQL_LONGVARBINARY (2) |
| bool | bit | BIT |
| datetime.date | date | SQL_TYPE_DATE |
| datetime.time | time | SQL_TYPE_TIME |
| datetime.datetime | timestamp | SQL_TIMESTAMP |
| int | integer | SQL_INTEGER |
| long | bigint | SQL_BIGINT |
| float | double | SQL_DOUBLE |
| decimal | numeric | SQL_NUMERIC |

1. If the driver supports it, [SQLDescribeParam](https://msdn.microsoft.com/en-us/library/ms710188.aspx) is used to determine the appropriate type. If not supported, SQL_VARCHAR is used.
2. [SQLGetTypeInfo](https://msdn.microsoft.com/en-us/library/ms714632.aspx) is used to determine when the LONG types are used. If not supported by the driver, VARCHAR and WVARCHAR will be 255 and BINARY will be 510.
3. Introduced in Python 2.6

#### SQL values received from the database

The following table describes how database results are converted to Python objects.

| Description | ODBC Datatype | Python Datatype |
|-------------|----------------|------------------|
| NULL | any | None |
| UCS-2 encoded Unicode | SQL_WCHAR, SQL_WVARCHAR | unicode |
| ASCII | SQL_CHAR, SQL_VARCHAR | str |
| GUID | SQL_GUID | str |
| XML | SQL_XML | str |
| binary | SQL_BINARY, SQL_VARBINARY | bytearray (Python 2.6+) or buffer(str) (Python 2.4 & 2.5) |
| decimal, numeric | SQL_DECIMAL, SQL_DECIMAL | decimal.Decimal |
| bit | SQL_BIT | bool |
| integers | SQL_TINYINT, SQL_SMALLINT, SQL_INTEGER, SQL_BIGINT | long |
| floating point | SQL_REAL, SQL_FLOAT, SQL_DOUBLE | float |
| time | SQL_TYPE_TIME | datetime.time |
| SQL Server time | SS_TIME2 | datetime.time |
| date | SQL_TYPE_DATE | datetime.date |
| timestamp | SQL_TIMESTAMP | datetime.datetime |