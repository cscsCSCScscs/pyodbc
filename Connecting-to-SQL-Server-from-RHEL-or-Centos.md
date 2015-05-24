Microsoft provide a database driver specifically for Red Hat Enterprise (https://msdn.microsoft.com/en-us/library/hh568451.aspx) to connect to SQL Server.  CentOS is derived from Red Hat so the driver works for CentOS as well.  This driver uses unixODBC as its driver manager, not FreeTDS or iODBC.  The driver and driver manager must be installed globally on your server (don't try installing in a Python virtual environment), as follows:

#### Install unixODBC

See http://msdn.microsoft.com/en-us/library/hh568449.aspx for reference.

Microsoft specifies that unixODBC version 2.3.0 should be installed with their driver.  You should be aware that version 2.3.0 has some significant bugs in it, particularly when making multiple connections to the same database. That specific issue (see "Driver version was not being held when a second connection was made to the driver" on http://www.unixodbc.org/), fixed in 2.3.1, can cause segmentation faults in your Python code. Hence, if you are using multiple concurrent database connections, you may want to consider installing the latest version of unixODBC instead, (version 2.3.2 as of April 2015). The rest of these instructions assume that is the case.
```bash
# download and unzip the unixODBC driver
curl -O 'ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.2.tar.gz'
tar -xz -f unixODBC-2.3.2.tar.gz
cd unixODBC-2.3.2

# remove any existing unixODBC drivers - be careful with 'sudo rm'!
sudo rm /usr/lib64/libodbc*

# install the unixODBC driver
# note, adding "--enable-stats=no" here is not specified by Microsoft
export CPPFLAGS="-DSIZEOF_LONG_INT=8"
./configure --prefix=/usr --libdir=/usr/lib64 --sysconfdir=/etc --enable-gui=no --enable-drivers=no --enable-iconv --with-iconv-char-enc=UTF8 --with-iconv-ucode-enc=UTF16LE --enable-stats=no 1> configure_std.log 2> configure_err.log
make 1> make_std.log 2> make_err.log
sudo make install 1> makeinstall_std.log 2> makeinstall_err.log

# the Microsoft driver expects unixODBC to be here /usr/lib64/libodbc.so.1, so add soft links to the '.so.2' files
cd /usr/lib64
sudo ln -s libodbccr.so.2   libodbccr.so.1
sudo ln -s libodbcinst.so.2 libodbcinst.so.1
sudo ln -s libodbc.so.2     libodbc.so.1
```
Check the unixODBC installation with the following:
```bash
ls -l /usr/lib64/libodbc*
odbc_config --version --longodbcversion --cflags --ulen --libs --odbcinstini --odbcini
odbcinst -j
isql --version
```

#### Install the Microsoft ODBC Driver for Linux

See http://msdn.microsoft.com/en-us/library/hh568454.aspx for reference.

First of all, go to https://www.microsoft.com/en-us/download/details.aspx?id=36437 and download the appropriate zip file to your Linux server.  Then install it as follows:

```bash
cd /path/to/your/driver/file/directory
tar -xz -f msodbcsql-11.0.2270.0.tar.gz
cd msodbcsql-11.0.2270.0
sudo ./install.sh install --accept-license --force 1> install_std.log 2> install_err.log
```

Check the msodbc installation with the following:

```bash
ls -l /opt/microsoft/msodbcsql/lib64/
dltest /opt/microsoft/msodbcsql/lib64/libmsodbcsql-11.0.so.2270.0 SQLGetInstalledDrivers
cat /etc/odbcinst.ini   # should contain a section called [ODBC Driver 11 for SQL Server]
```

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

#### Test the Connection

Try the connection to your database with something like this:

```bash
python -c "import pyodbc; print(pyodbc.connect('DSN=MySQLServerDatabase;UID=myuid;PWD=mypwd'))"
```