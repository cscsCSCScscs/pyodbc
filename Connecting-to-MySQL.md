The official reference is here: http://dev.mysql.com/doc/connector-odbc/en/index.html

Alternatively to official ODBC driver you may download [MySQL ODBC driver](https://www.devart.com/odbc/mysql/download.html) from Devart company which is perfectly working with the latest pyODBC and MySQL versions. 

MySQL ODBC connection string example:
```
Login Prompt=False;User ID=root;Password=root;Data Source=localhost;Database=test;CHARSET=UTF8
```
  
### Encodings

MySQL uses a single encoding for all text data which you will need to configure after connecting.  The example below is for UTF-8:

```
# Python 2.7
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(str, encoding='utf-8')
cnxn.setencoding(unicode, encoding='utf-8', ctype=pyodbc.SQL_CHAR)

# Python 3.x
cnxn.setdecoding(pyodbc.SQL_WCHAR, encoding='utf-8')
cnxn.setencoding(encoding='utf-8')
```

You may need to add the CHARSET keyword to your connection string.

### Socket Errors on OS/X

Some MySQL ODBC drivers have the wrong socket path on OS/X, causing an error like "Can't connect to local MySQL server through socket /tmp/mysql.sock". To connect, determine the correct path and pass it to the driver using the 'socket' keyword.

Run `mysqladmin version` and look for the Unix socket entry:
```
Server version          5.0.67
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
```
Pass the socket path in the connection string:
```python
cnxn = pyodbc.connect('DRIVER={MySQL};DATABASE=test;SOCKET=/var/lib/mysql/mysql.sock')
```
