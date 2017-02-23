The PostgreSQL ODBC driver is called [psqlodbc](https://odbc.postgresql.org).

## Encodings

PostgreSQL uses a single encoding for all text data which you will need to configure after connecting.  The example below is for UTF-8.

```python
# Python 3.x
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(encoding='utf-8')

# Python 2.7
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(str, encoding='utf-8')
cnxn.setencoding(unicode, encoding='utf-8', ctype=pyodbc.SQL_CHAR)
```

See the [Unicode](Unicode) page if you are using something besides UTF-8.

## Performance

The driver defaults to a 255-byte maximum varchar/wvarchar size for writes which causes very slow writes.  There are two easy fixes:

* Set the MaxVarcharSize parameter in your connection string or DSN.
* Set the Connection's `maxwrite`.

In both cases, set it to something very large.  I use:

    cnxn.maxwrite = 1024 * 1024 * 1024

I would consider making the default something very large, but the driver actually reports the value 255 and pyodbc needs to use the values provided.  I sent an email to the PostgreSQL ODBC driver maintainer (Feb 2017) recommending increasing it but was told to use MaxVarcharSize.
