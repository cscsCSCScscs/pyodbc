The Excel driver does not support transactions, so you will need to use the `autocommit` keyword:
```python
cnxn = pyodbc.connect('DSN=test', autocommit=True)
```

Excel defaults to a read-only connection, so if you want to update the spreadsheet include `ReadOnly=0` in your connection string:
```python
cnxn = pyodbc.connect('Driver={Microsoft Excel Driver (.xls)};Dbq=C:\MyExcel.xls;ReadOnly=0', autocommit=True)
```

Be careful of the data types in your Excel spreadsheet.  The Excel driver uses the most common data type from the first 8 rows of the spreadsheet to determine the data type of each column.  So if you have 5 numbers and 3 text values in the first rows of a column, the column will be considered numeric, and the 3 text values will be returned as NULL!