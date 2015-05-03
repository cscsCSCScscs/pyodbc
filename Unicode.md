# Unicode

Unicode and character encodings is a common problem with drivers.

The ODBC specification only allows two byte-sizes:
* 1-byte ANSI which can be passed to the original functions, such as SQLGetInfo.
* 2-byte UCS2 or UTF16 which can be passed to the wide functions, such as SQLGetInfoW.

That is all that is supported.  For whatever reason many drivers are not written to this and cause a lot of problems.

# Specification Notes

http://support.microsoft.com/default.aspx?scid=kb;EN-US;q294169

> For many of these Unicode functions, the ODBC Programmer's Reference provides incorrect or
> ambiguous descriptions for some of the function arguments. Specifically, this problem
> relates to arguments that are used to specify the length of character string input and
> output values.

> Regardless of what the documentation says for each ODBC function, the following paragraph
> from the Unicode section of "Chapter 17: Programming Considerations" in the ODBC
> Programmer's Reference is the ultimate rule to use for length arguments in Unicode
> functions:

>     "Unicode functions that always return or take strings or length arguments are passed as
>     count-of-characters. For functions that return length information for server data, the
>     display size and precision are described in number of characters. When a length
>     (transfer size of the data) could refer to string or nonstring data, the length is
>     described in octet lengths. For example, SQLGetInfoW will still take the length as
>     count-of-bytes, but SQLExecDirectW will use count-of-characters."

> This means that if the argument in question describes the length of another argument that
> is always a string (typically represented as a SQLCHAR), then the length reflects the
> number of characters in the string. If the length argument describes another argument that
> could be a string or some other data type (typically represented as a SQLPOINTER), the
> length is in bytes.

# Library Notes

## Driver Managers

### unixODBC

unixODBC follows the specification and uses UCS2 by default.  

*However*, it seems to define SQLWCHAR as wchar_t even when wchar_t is 4-byte.  It will *not* accept 4-byte Unicode in this case - it will simply crash.  You must still pass 2-byte UCS2 to the wide functions regardless of the size of SQLWCHAR.  I consider this a bug and now have to define my own data type (ODBCCHAR) instead of using on SQLWCHAR.

You *can* compile unixODBC to accept UCS4 but the driver author specifically discourages it on the mailing list.  (Need the references here.)

## Drivers

### PostgreSQL

I believe the PostgreSQL driver correctly expects UCS2: http://archives.postgresql.org/pgsql-odbc/2006-02/msg00112.php

### Microsoft SQL Server Driver

MS SQL Server on Windows & Linux correctly uses UCS-2.  SQL Server 2012 and higher now support UTF16.

### MySQL

I believe the MySQL driver is broken and does no Unicode conversion.  If your data is in UTF8, then it will blindly pass that along instead of UCS2.

http://mysqlworkbench.org/?p=1399

### FreeTDS

http://www.freetds.org/userguide/unicodefreetds.htm
  
Definitely use 0.91 or later.

Have seen reference to a new --wide-unicode flag for 0.92+ (broken in 0.91) which causes
SQL_WCHAR to equal wchar_t instead of UTF16.
