The PostgreSQL ODBC driver is called [psqlodbc](https://odbc.postgresql.org).

PostgreSQL uses a single encoding for all text data which you will need to configure after connecting.  The example below is for UTF-8.

Also, the driver incorrectly reports that the maximum sizes for VARCHAR and WVARCHAR writes are 255 characters at a time which leads to very poor performance.  Manually overwrite those:

```python
# Python 2.7
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(str, encoding='utf-8')
cnxn.setencoding(unicode, encoding='utf-8', ctype=pyodbc.SQL_CHAR)
cnxn.maxwrite = 1024 * 1024 * 1024

# Python 3.x
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(encoding='utf-8')
cnxn.maxwrite = 1024 * 1024 * 1024
```

See the [Unicode](Unicode) page if you are using something besides UTF-8.
