Microsoft's instructions for installing their latest ODBC drivers onto a variety of UNIX-based platforms are [here](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server).

Prepare a file for defining the DSN to your database with a temporary file something like this:    

```
[MySQLServerDatabase]
Driver      = ODBC Driver 11 for SQL Server
Description = My MS SQL Server
Trace       = No
Server      = mydbserver.mycompany.com
```
    
In that file, leave the 'Driver' line exactly as specified above, but modify the rest of the file as necessary.  Then run the following commands:

```bash
# register the SQL Server database DSN information in /etc/odbc.ini
sudo odbcinst -i -s -f /path/to/your/temporary/dsn/file -l

# check the DSN installation with:
cat /etc/odbc.ini   # should contain a section called [MySQLServerDatabase]
```

Connecting to the server is then done like this:

    pyodbc.connect('DSN=MySQLServerDatabase;UID=myuid;PWD=mypwd')

### OS-specific notes:

#### CentOS

CentOS is derived from RedHat, and their major version numbers (i.e. 6 and 7) match those of RedHat, so just use the instructions for the relevant version of RedHat.