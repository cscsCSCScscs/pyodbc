The official reference is here: http://dev.mysql.com/doc/connector-odbc/en/index.html

#### Unicode
If you are using UTF8 in your database and are getting results like "\x0038", you probably need to add "CHARSET=UTF8" to your connection string.

#### Errors on OS/X
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