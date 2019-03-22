Missing database name in the odbc.ini

just add: `Database    = TestDB`

`[MySQLServerDatabase]`
`Driver      = ODBC Driver 11 for SQL Server`
`Description = My MS SQL Server`
`Trace       = No`
`Server      = mydbserver.mycompany.com`
`Database    = TestDB`