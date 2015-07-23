Opening a connection to a Microsoft Access database (e.g. \*.mdb) is achieved using the Microsoft Access driver "Microsoft Access Driver (\*.mdb, \*.accdb)".  Unfortunately, this driver is available only on Windows, not Unix.  Check if you have this driver on your PC by navigating to Control Panel -> Administrative Tools -> Data Sources (ODBC), and then click on the "Drivers" tab.  The Access driver will be listed there, if it is installed.

Here is an example of how to open an MS Access database:

```python
cnxn = pyodbc.connect(r'DRIVER={Microsoft Access Driver (*.mdb, *.accdb)};DBQ=C:\path\to\db\mydb.mdb;UID=myusername;PWD=mypassword;')
crsr = conn.cursor()
for table_name in crsr.tables(tableType='TABLE'):
    print(table_name)
```
