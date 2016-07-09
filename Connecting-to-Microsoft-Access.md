Opening a connection to a Microsoft Access database (e.g. \*.mdb) is achieved using the Microsoft Access driver "Microsoft Access Driver (\*.mdb, \*.accdb)".  Unfortunately, this driver is available only on Windows, not Unix.  Check if you have this driver on your PC by navigating to Control Panel -> Administrative Tools -> Data Sources (ODBC), and then click on the "Drivers" tab.  The Access driver will be listed there, if it is installed.

Here is an example of how to open an MS Access database:

```python
cnxn = pyodbc.connect(r'DRIVER={Microsoft Access Driver (*.mdb, *.accdb)};DBQ=C:\path\to\db\mydb.mdb;UID=myusername;PWD=mypassword;')
crsr = conn.cursor()
for table_name in crsr.tables(tableType='TABLE'):
    print(table_name)
```

This code will succeed under 32-bit Python if it finds a 32-bit version of the MS Access driver installed, and likewise for 64 bit Python and 64-bit Access. A Microsoft Access driver is typically installed with Microsoft Office, and matches the 32/64 bitness of that Office installation. It may or may not be possible to install additionally the other 32 or 64 bit MS Access driver (that doesn't match the Office installation). Search for details.

 that matches the 32/64 bitness of the Python version being used. The MS Access driver is typically installed with Microsoft Office, and corresponds to the 32/64 bitness of the Office version installed. If 