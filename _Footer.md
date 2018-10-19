### Connect with PHP PDO

`$pdo = new \PDO('odbc:MYMSSQL', 'myusername', 'mypass');
$pdostmt = $pdo->prepare('SELECT TOP 10 FROM dbo.MyTableName');
if (!$pdostmt) {
    throw new Exception('There was an error');
}
if (false === $pdostmt->execute()) {
    throw new Exception('The pdo statement did not execute successfully');
}
var_dump($pdostmt->fetchAll(\PDO::FETCH_ASSOC));`

This should work. Note that you do not have to pass any tag to the dsn like in : DSN=MYMSSQL => bad. 
Just pass the dsn name directly after odbc:MYMSSQL.

IMPORTANT: you really have to make sure you pass the correct library name. There are plenty of versions in your system in different places. I used:

In /usr/local/etc/odbcinst.ini : 
`[TDSODBCDriver]
Description=ODBC Driver for SQL Server
Driver=/usr/local/lib/libtdsodbc.so
UsageCount=1
`
In /usr/local/etc/odbc.ini : 
`[MYMSSQL]
Description=Some Descirption
Servername=FMyServername
Driver=TDSODBCDriver
`
In /usr/local/etc/freetds.conf : 
`# A typical Microsoft server
[FMyServername]
    host = 123.456.56.789
    port = 1218 
    tds version = 7.0
    database = mydbname
    dump file = /usr/local/var/log/freetds.dump
    client charset = UTF-8
    debug flags = 0x80
`

If you have errors, play around with the tds version number. And note that I have a 2005 SQL Server, which should use 7.2, but it works with 7.0. So try them all if need be.