### Using an ODBC driver

There are actually many SQL Server ODBC drivers written and distributed by Microsoft:

* {SQL Server} - released with SQL Server 2000
* {SQL Native Client} - released with SQL Server 2005 (also known as version 9.0)
* {SQL Server Native Client 10.0} - released with SQL Server 2008
* {SQL Server Native Client 11.0} - released with SQL Server 2012

The connection strings for all these drivers are essentially the same, for example:
```
DRIVER={SQL Server Native Client 11.0};SERVER=test;DATABASE=test;UID=user;PWD=password
```
or, in Python:
```python
conn = pyodbc.connect(r'DRIVER={SQL Server Native Client 11.0};SERVER=test;DATABASE=test;UID=user;PWD=password')
```
Alternative to Microsoft ODBC drivers you can download and use latest [SQL Server ODBC driver](https://www.devart.com/odbc/sqlserver/download.html) from Devart company which is tested and perfectly working with the latest versions of pyODBC and SQL Server. 

This driver connection string will look like:
```
Login Prompt=False;Data Source=DBMSSQL;Initial Catalog=master;User ID=sa
```

You can find out what drivers you have on your Windows PC by navigating to Control Panel -> Administrative Tools -> Data Sources (ODBC).  In the pop-up window, click on the Drivers tab.

It's generally best to use the latest drivers you have, regardless of the version of SQL Server you are connecting to, because the drivers are largely backwards-compatible.  However you may prefer to use the specific driver for your SQL Server instance.


### Using a DSN

DSNs (or Data Source Names) allow you to define the ODBC driver, server, database, login credentials (possibly), and other connection attributes all in one place, so you don't have to provide them in your connection string.  You can set up DSNs on your PC by using your ODBC Data Source Administrator window.

To get to your ODBC Data Source Administrator window, navigate to Control Panel -> Administrative Tools -> Data Sources (ODBC). Under 'User DSN' or 'System DSN' (depending on whether you want just you or everybody to be able to use this DSN) click on the 'Add...' button and follow the wizard instructions.  Choose a driver that is suitable for the version of SQL Server you are connecting to, and add any other connection information that is relevant.  Once you have created your new DSN, use it in the pyodbc.connect() function as follows:
```python
conn = pyodbc.connect(r'DSN=mynewdsn;UID=user;PWD=password')
```
Attributes in the connection string will override any attributes in the DSN.

### Other connection attributes
You can connect to your SQL Server instance using a trusted connection, i.e. without providing a login name and password, by using the Trusted_Connection attribute, e.g.:
```python
conn = pyodbc.connect(r'DRIVER={SQL Server};SERVER=test;DATABASE=test;Trusted_Connection=True;')
```

Adding the APP keyword allows you to provide a descriptive label (of up to 128 characters) for your database connection which is very handy for database administrators, e.g.:
```python
conn = pyodbc.connect(r'DSN=mynewdsn;UID=user;PWD=password;APP=Daily Incremental Backup;')
```