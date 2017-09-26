Output Converter functions offer a flexible way to work with returned results that pyodbc does not natively support. For example, Microsoft SQL Server returns values from a DATETIMEOFFSET column as SQL type -155, which does not have native support in pyodbc (and many other ODBC libraries). Simply retrieving the value ...

```python
import pyodbc
conn = pyodbc.connect("DSN=myDb")

crsr = conn.cursor()
# create test data
crsr.execute("CREATE TABLE #dto_test (id INT PRIMARY KEY, dto_col DATETIMEOFFSET)")
crsr.execute("INSERT INTO #dto_test (id, dto_col) VALUES (1, '2017-03-16 10:35:18 -06:00')")

value = crsr.execute("SELECT dto_col FROM #dto_test WHERE id=1").fetchval()
print(value)

crsr.close()
conn.close()
```

... results in the error

> pyodbc.ProgrammingError: ('ODBC SQL type -155 is not yet supported.  column-index=0  type=-155', 'HY106')

However, we can define an Output Converter function to decode the bytes returned, [add that function to the Connection object](Connection#add_output_converter), and then use the resulting value, e.g., 

```python
import struct
import pyodbc
conn = pyodbc.connect("DSN=myDb")


def handle_datetimeoffset(dto_value):
    # ref: https://github.com/mkleehammer/pyodbc/issues/134#issuecomment-281739794
    tup = struct.unpack("<6hI2h", dto_value)  # e.g., (2017, 3, 16, 10, 35, 18, 0, -6, 0)
    return "{:04d}-{:02d}-{:02d} {:02d}:{:02d}:{:02d}.{:07d} {:+03d}:{:02d}".format(*tup)


crsr = conn.cursor()
# create test data
crsr.execute("CREATE TABLE #dto_test (id INT PRIMARY KEY, dto_col DATETIMEOFFSET)")
crsr.execute("INSERT INTO #dto_test (id, dto_col) VALUES (1, '2017-03-16 10:35:18 -06:00')")

conn.add_output_converter(-155, handle_datetimeoffset)
value = crsr.execute("SELECT dto_col FROM #dto_test WHERE id=1").fetchval()
print(value)

crsr.close()
conn.close()
```

which prints the string representation of the DATETIMEOFFSET value

```none
2017-03-16 10:35:18.0000000 -06:00
```
