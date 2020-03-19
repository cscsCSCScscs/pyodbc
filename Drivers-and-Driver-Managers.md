## How All the Pieces Fit Together

The following diagram describes how your Python code communicates with the database in a typical scenario:

```
--- App Server ---------------
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
When your Python code wants to communicate with the database, it calls functions in the pyodbc Python module, which in turn communicates with a "driver manager".
The driver manager provides the API that conforms to the [ODBC standard](https://docs.microsoft.com/en-us/sql/odbc), which is common to all ODBC-compliant RDBMS's.
For example, to open a connection to a database, pyodbc calls the [SQLDriverConnect](https://docs.microsoft.com/en-us/sql/odbc/reference/syntax/sqldriverconnect-function) ODBC function in the driver manager. To execute a SQL statement, pyodbc calls [SQLExecDirect](https://docs.microsoft.com/en-us/sql/odbc/reference/syntax/sqlexecdirect-function).
The driver manager then calls the database "driver" which in turn interacts directly with the database, typically across a network, marshaling instructions and data back and forth between the app server and the database server.

Hence, if the driver manager, driver, and database all conform to the ODBC standard, it should be possible to use pyodbc with any ODBC-compliant database vendor.
However, in reality, all RDBMS's have their own particular quirks and idiosyncrasies, and often these are not compliant with ODBC.  Pyodbc attempts to work around these quirks as much as it can.

On both Unix and Windows systems, the driver manager and driver are often rolled into the same software package and referred to simply as "the driver", especially when they are created by the manufacturer of the RDBMS.

Just to be clear, as the name suggests, pyodbc is specific to ODBC databases.  It does not attempt to be compliant with other database standards like JDBC, OLE DB, ADO.NET, etc.

## Driver Managers

The driver manager that pyodbc uses is determined when pyodbc is installed (through the [setup.py](https://github.com/mkleehammer/pyodbc/blob/master/setup.py) script).  If you need to change the driver manager, you have to re-install pyodbc.

Common third-party driver managers on Unix are as follows:

### unixODBC

unixODBC tries to be the definitive standard for ODBC on non-Windows platforms.
It is available on all Unix OS's and Mac OSX (`brew install unixodbc`). It also provides the command line tools `odbcinst` and `isql` which are useful for manipulating the ODBC configuration files (see below) and making SQL calls without pyodbc. For documentation on these utilities, run `man odbcinst` or `man isql`.

The website for unixODBC is http://www.unixodbc.org/. It does not have extensive documentation but the front page is very useful for the release notes about unixODBC itself. The unixODBC codebase is available on [SourceForge](https://sourceforge.net/projects/unixodbc/).

On Unix platforms and Mac OSX, pyodbc has assumed unixODBC is being used rather than iODBC since version 3.0.8 (April 2015).

### iODBC

[iODBC](http://www.iodbc.org/) is another driver manager for Unix platforms. It used to be pre-installed as the default driver manager on Mac OSX, but since around early 2015, this is no longer the case.


## Drivers

Drivers are so specific to each RDBMS that they are typically written by the manufacturer themselves.  They are almost always provided separately (and free of charge) for installation on the app server.
Sometimes, third-party drivers are available for certain RDBMS's.  For example, [FreeTDS](http://www.freetds.org/) works with SQL Server and Sybase.


## ODBC Configuration Files (Unix only)

The driver manager has to communicate with the database driver (and database), and before it can do that, it has to be able to find out what drivers and databases are available. This information is stored in two files on the app server - `odbcinst.ini` (for drivers) and `odbc.ini` (for database servers).

You can usually find out where the `odbcinst.ini` and `odbc.ini` files are on a server by running `odbcinst -j`.  Typically they are in the `/etc/` directory, although they may also been stored as hidden files in your home directory, i.e. `~/.odbc.ini`.

### odbcinst.ini

Information about ODBC drivers on the app server is stored in the `odbcinst.ini` file.
It contains a catalog of the drivers which have been installed, including their locations and their familiar names.  Here is an example of its contents:

```
[ODBC Driver 17 for SQL Server]
Description=Microsoft ODBC Driver 17 for SQL Server
Driver=/opt/microsoft/msodbcsql17/lib64/libmsodbcsql-17.3.so.1.1
Threading=1
UsageCount=1

[ODBC Driver for Vertica]
Description=Vertica ODBC Driver
Driver=/opt/vertica/lib64/libverticaodbc.so
UsageCount=1
```

Here, the familiar name for the SQL Server driver is "ODBC Driver 17 for SQL Server", and "ODBC Driver for Vertica" for the Vertica driver.  The driver software module itself is referred to in the "Driver" line.  Typically, when database drivers are installed, the installation process adds a new catalog entry to the `odbcinst.ini` file so it shouldn't normally be necessary to manually edit this file.  Note, the driver is specific to an RDBMS, not a database instance.  Each driver can be used to connect to multiple database instances of the same RDBMS.

### odbc.ini

Information about database connections on the app server is stored in the `odbc.ini` file.
It contains a catalog of DSN's (Data Source Names), typically one for each database instance.  Each DSN includes a driver reference in the "Driver" line, which is the familiar name from the `odbcinst.ini` file, e.g.:

```
[Samson]
Driver=ODBC Driver 17 for SQL Server
Description=The OLTP server
Server=samson.xyz.com

[Delilah]
Driver=ODBC Driver for Vertica
Description=The OLAP server
Server=delilah.xyz.com
Database=cube
```

Typically, catalog entries to `odbc.ini` are added manually, either by using the `odbcinst -i -s` command, or by editing the file directly.  These catalog entries can also include the database instance name, and even login credentials.

### connecting to a database

Once the two ODBC configuration files are set up correctly, connections to a database can then be conveniently made using just the DSN without having to refer to a server name, driver manager, or driver, e.g.:
```python
cnxn = pyodbc.connect('DSN=Samson;UID=mylogin;PWD=mypassword')
```
