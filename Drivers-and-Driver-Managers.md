## How All the Pieces Fit Together

The following diagram attempts to describe how your Python code communicates with the target database:

```
--- Your Server --------------
|                            |
|      your Python code      |
|             |              |
|           pyodbc           |
|             |              |
|       driver manager       |
|             |              |
|           driver           |
|             |              |
------------------------------
              |
      (network connection)
              |
--- Database Server ----------
|             |              |
|          database          |
|                            |
------------------------------
```

As you can see, there are a series of intermediary interfaces between your Python code and the target database.
Your Python code calls functions in the pyodbc Python package which in turn communicates with a "driver manager".
The driver manager provides the API that (should) conform to the [ODBC standard](https://docs.microsoft.com/en-us/sql/odbc).
For example, to open a connection to a database pyodbc calls the [SQLDriverConnect](https://docs.microsoft.com/en-us/sql/odbc/reference/syntax/sqldriverconnect-function) function in the driver manager, and to execute a SQL statement pyodbc calls [SQLExecDirect](https://docs.microsoft.com/en-us/sql/odbc/reference/syntax/sqlexecdirect-function).
The driver manager then calls the database "driver" which in turn interacts directly with the database, typically across a network, marshalling instructions and data back and forth between your server and the database server.
Hence, if the driver manager, driver, and database all conform to the ODBC standards, it should be possible to use pyodbc with any database vendor.
However, in reality, they all have their own particular quirks and idiosyncracies, and often these are not compliant with ODBC.  Pyodbc attempts to work around these quirks as much as it can.

On both Unix and Windows systems, the driver manager and driver are often rolled into the same software and referred to simply as "the driver", especially when they are created by the manufacturer of the RDBMS.

## Driver Managers

The driver manager that pyodbc uses is determined when pyodbc is installed (through the [setup.py](https://github.com/mkleehammer/pyodbc/blob/master/setup.py) script).  If you need to change the driver manager, you have to re-install pyodbc.

Common third-party driver managers on Unix are as follows:

### unixODBC

unixODBC tries to be the definitive standard for ODBC on non-Windows platforms.
It is available on all Unix OS's and Mac OSX (`brew install unixodbc`). It also provides the command line tools `odbcinst` and `isql` which are useful for manipulating the ODBC configuration files (see below) and making SQL calls without pyodbc. For documentation about these utilities, best run `man odbcinst` or `man isql`.

The website for unixODBC is here http://www.unixodbc.org, but it's worth noting this website is not particularly well maintained so don't look to it for extensive documentation.  However, the front page is very useful for the release notes about unixODBC itself.

Since version 3.0.8 (April 2015), pyodbc has assumed unixODBC is being used rather than iODBC (on Unix platforms).

### iODBC

[iODBC](http://www.iodbc.org/) is another driver manager for Unix platforms. It used to be pre-installed as the default driver manager on Mac OSX, but since around early 2015, this is no longer the case.


## Drivers

Drivers are so specific to a RDBMS they are typically written by the manufacturer of the RDBMS.  Sometimes, however, there are third-party drivers available for certain RDBMS's.  For example, [FreeTDS](http://www.freetds.org/) works with SQL Server and Sybase.


## ODBC Configuration Files _odbc.ini_ and _odbcinst.ini_ (unix only)

The driver manager has to be told what drivers and databases are available on your server. This information is stored in the `odbcinst.ini` and `odbc.ini` files, respectively.

Information about ODBC drivers on a server is stored in the `odbcinst.ini` file.
It contains a catalog of the drivers which have been installed, including the familiar names by which they are referenced, e.g.:

```
[ODBC Driver 11 for SQL Server]
Description=Microsoft ODBC Driver 11 for SQL Server
Driver=/opt/microsoft/msodbcsql/lib64/libmsodbcsql-11.0.so.2270.0
Threading=1
UsageCount=1

[ODBC Driver for Vertica]
Description=Vertica ODBC Driver
Driver=/opt/vertica/lib64/libverticaodbc.so
UsageCount=1
```

Here, the familiar name for the SQL Server driver is "ODBC Driver 11 for SQL Server", and "ODBC Driver for Vertica" for the Vertica driver.  Note the driver software itself is referred to in the "Driver" line.  Typically, when database drivers are installed, the installation process adds a new catalog entry to the `odbcinst.ini` file so it shouldn't normally be necessary to edit this file.

Information about database connections on your server is stored in the `odbc.ini` file.
It contains a catalog of DSN's (Data Source Names), one for each database connection.  Each DSN includes a driver reference in the "Driver" line, which is the driver's familiar name from the `odbcinst.ini` file, e.g.:

```
[Samson]
Driver=ODBC Driver 11 for SQL Server
Description=The OLTP server
Server=samson.xyz.com

[Delilah]
Driver=ODBC Driver for Vertica
Description=The OLAP server
Server=delilah.xyz.com
Database=cube
```

Typically, catalog entries to `odbc.ini` are added manually, either by using the `odbcinst -i -s` command, or by editing the file directly.

Once the two ODBC configuration files are set up correctly, connections to a database can then be conveniently made using just the DSN without having to refer to a server, driver manager, or driver, e.g.:
```python
conn = pyodbc.connect('DSN=Samson;UID=mylogin;PWD=mypassword')
```

You can find out where the `odbc.ini` and `odbcinst.ini` files are on a server by running `odbcinst -j`.  Typically they are in the `/etc/` directory, although they may also been stored as hidden files in your home directory, i.e. `~/.odbc.ini`.
