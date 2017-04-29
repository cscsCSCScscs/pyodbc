The following instructions assume you already have a SQL Server database running somewhere that your Mac has network access to.

#### Install FreeTDS and unixODBC

The connection to SQL Server will be made using the unixODBC driver manager and the FreeTDS driver. Installing them is most easily done using `homebrew`, the Mac package manager:

```
brew update
brew install unixodbc
brew install freetds --with-unixodbc
```

#### Edit the freetds.conf configuration file

The `freetds.conf` file is usually located in directory `/usr/local/etc/`.  However, Homebrew will create this file as a soft link to the real file located here `/usr/local/Cellar/freetds/<version>/etc/freetds.conf`.  Check this location with `tsql -C`.  The default file already contains a bunch of stuff, but all you need to do is add your server information to the end, as follows:

```
[MYMSSQL]
host = mssqlhost.xyz.com
port = 1433
tds version = 7.3
```
There are other key/value pairs that can be added but this shouldn't usually be necessary, see [here](http://www.freetds.org/userguide/freetdsconf.htm) for details. The `host` parameter should be either the network name (or IP address) of the database server, or "localhost" if SQL Server is running directly on your Mac (e.g. using [Docker](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-docker)).  A TDS version of 7.3 should be OK for SQL Server 2008 and newer, but bear in mind that

* you might need a different value for older versions of SQL Server
* TDS version 8.0 is just an alias for TDS version 7.1, so
  * TDS version "8.0" is **not** newer than TDS versions 7.2, 7.3, or 7.4, and
  * specifying TDS version "8.0" is discouraged because of possible future compatibility issues.

  (For more information on TDS protocol versions see [Choosing a TDS protocol version](http://www.freetds.org/userguide/choosingtdsprotocol.htm).)

Test the connection using the `tsql` utility, e.g. `tsql -S MYMSSQL -U myuser -P mypassword`.  If this works, you should see the following:

```
locale is "en_US.UTF-8"
locale charset is "UTF-8"
using default charset "UTF-8"
1>
```
At this point you can run SQL queries, e.g. "SELECT @@VERSION" but you'll need to enter "GO" on a separate line to actually execute the query.  Type `exit` to get out of the interactive session.

#### Edit the odbcinst.ini and odbc.ini configuration files

Run `odbcinst -j` to get the location of the `odbcinst.ini` and `odbc.ini` files (probably in directory `/usr/local/Cellar/unixodbc/<version>/etc/`). Edit `odbcinst.ini` to include the following:

```
[FreeTDS]
Description=FreeTDS Driver for Linux & MSSQL
Driver=/usr/local/lib/libtdsodbc.so
Setup=/usr/local/lib/libtdsodbc.so
UsageCount=1
```

Edit `odbc.ini` to include the following:

```
[MYMSSQL]
Description         = Test to SQLServer
Driver              = FreeTDS
Servername          = MYMSSQL
```
Note, the "Driver" is the name of the entry in `odbcinst.ini`, and the "Servername" is the name of the entry in `freetds.conf` (not a network name). There are other key/value pairs that can be included, see [here](http://www.freetds.org/userguide/odbcconnattr.htm) for details.

Check that all is OK by running `isql MYMSSQL myuser mypassword`. You should see the following:

```
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
```
You can enter SQL queries at this point if you like.  Type `quit` to exit the interactive session.

#### Connect with pyodbc

It should now be possible to connect to your SQL Server database using pyodbc, for example:
```
import pyodbc
# the DSN value should be the name of the entry in odbc.ini, not freetds.conf
conn = pyodbc.connect('DSN=MYMSSQL;UID=myuser;PWD=mypassword')
crsr = conn.cursor()
rows = crsr.execute("select @@VERSION").fetchall()
print(rows)
crsr.close()
conn.close()
```
