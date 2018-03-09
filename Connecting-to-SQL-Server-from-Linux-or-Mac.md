Microsoft's instructions for installing their latest ODBC drivers onto a variety of UNIX-based platforms are [here](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server).

Connecting to the server is then done like this:

    pyodbc.connect('DSN=MySQLServerDatabase;UID=myuid;PWD=mypwd')

OS-specific notes:

#### CentOS

CentOS is derived from RedHat, and their major version numbers (i.e. 6 and 7) match those of RedHat, so just use the instructions for the relevant version of RedHat.