Microsoft only produces Access ODBC drivers for the Windows platform. Third-party vendors may be able to provide Access ODBC drivers for non-Windows platforms.

There are actually two (2) different Access ODBC drivers from Microsoft:

1. `Microsoft Access Driver (*.mdb)` - This is the older 32-bit "Jet" ODBC driver. It is included as a standard part of a Windows install. It only works with `.mdb` (not `.accdb`) files. It is also officially deprecated.

2. `Microsoft Access Driver (*.mdb, *.accdb)` - This is the newer "ACE" ODBC driver. It is not included with Windows, but it is normally included as part of a Microsoft Office install. It is also available as a free stand-alone "redistributable" install for machines without Microsoft Office. There are separate 64-bit and 32-bit versions of the "ACE" Access Database Engine (and drivers), and normally one has **either** the 64-bit version **or** the 32-bit version installed. (It is possible to force both versions to exist on the same machine but it is not recommended as it can "break" Office installations. Therefore, if you already have Microsoft Office it is *highly recommended* that you use a Python environment that matches the "bitness" of the Office install.)

The easiest way to check if one of the Microsoft Access ODBC drivers is available to your Python environment (on Windows) is to do

```python
>>> import pyodbc
>>> [x for x in pyodbc.drivers() if x.startswith('Microsoft Access Driver')]
```

If you see an empty list then you are running 64-bit Python and you need to install the 64-bit version of the "ACE" driver. If you only see `['Microsoft Access Driver (*.mdb)']` and you need to work with an `.accdb` file then you need to install the 32-bit version of the "ACE" driver.

Here is an example of how to open an MS Access database:

```python
conn_str = (
    r'DRIVER={Microsoft Access Driver (*.mdb, *.accdb)};'
    r'DBQ=C:\path\to\mydb.accdb;'
    )
cnxn = pyodbc.connect(conn_str)
crsr = cnxn.cursor()
for table_name in crsr.tables(tableType='TABLE'):
    print(table_name)
```
### Compatibility issue with Python 3

There is an [outstanding issue](https://github.com/mkleehammer/pyodbc/issues/84) regarding problems working with Access databases via pyodbc under Python 3.x. Until that issue is resolved, consider using Python 2.7 to manipulate Access databases with pyodbc.