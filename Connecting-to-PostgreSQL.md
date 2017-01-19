The PostgreSQL ODBC driver is called [psqlodbc](https://odbc.postgresql.org).

PostgreSQL uses a single encoding for all text data which you will need to configure after connecting.  The example below is for UTF-8:

```python
# Python 2.7
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(str, encoding='utf-8')
cnxn.setencoding(unicode, encoding='utf-8', ctype=pyodbc.SQL_CHAR)

# Python 3.x
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(encoding='utf-8')
```

See the [Unicode](Unicode) page if you are using something besides UTF-8.
