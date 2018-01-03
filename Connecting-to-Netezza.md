These unicode connection settings should work with Netezza:

```python
# Python 2.7
cnxn.setdecoding(pyodbc.SQL_CHAR, encoding='utf-8')
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setdecoding(pyodbc.SQL_WMETADATA, encoding='utf-8')
cnxn.setencoding(str, encoding='utf-8')
cnxn.setencoding(unicode, encoding='utf-8')

# Python 3.x
cnxn.setdecoding(pyodbc.SQL_CHAR, encoding='utf-8')
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setdecoding(pyodbc.SQL_WMETADATA, encoding='utf-8')
cnxn.setencoding(encoding='utf-8')
```
