
Just installing 'ODBC Driver 17 for SQL Server for linux' and a  proper driver name is enough, don't need to edit /etc/odbc.ini
sql_connection_string = "Driver=ODBC Driver 17 for SQL Server;Server=%s;Database=%s;uid=%s;pwd=%s" % (sql_address, sql_database, sql_login, sql_password)
