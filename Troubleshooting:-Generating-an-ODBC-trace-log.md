Users reporting issues with pyodbc will often be asked to provide an ODBC trace log for diagnostic purposes. Fortunately, generating the log file is easy.

### Windows

Launch the 64-bit or 32-bit ODBC Administrator depending on whether you are using 64-bit or 32-bit Python:

If you are running 64-bit Python, launch

```
C:\Windows\System32\odbcad32.exe
```

If you are running 32-bit Python, launch

```
C:\Windows\SysWOW64\odbcad32.exe
```

On the "Tracing" tab, click the "Start Tracing Now" button. Details of subsequent ODBC activity will be appended to the file specified in "Log File Path".

![Windows ODBC Administrator](https://i.stack.imgur.com/29vY0.png)

### Linux

To enable ODBC tracing under Linux (unixODBC), add the following two lines to the `[ODBC]` section of `odbcinst.ini` (usually found in the `/etc` directory).

```
[ODBC]
Trace = yes
TraceFile = /home/gord/odbctrace.txt
```

### Notes

Remember to turn off ODBC tracing after generating your log file. Logging ODBC activity slows down ODBC operations and the log file will quickly become very large.
