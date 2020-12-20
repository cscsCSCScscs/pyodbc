pyodbc does not currently implement the optional `.callproc` method. (It has been [investigated](https://github.com/mkleehammer/pyodbc/issues/184).) 

However, ODBC defines a [{CALL ...} escape sequence](https://msdn.microsoft.com/en-us/library/ms403294.aspx) that should be supported by well-behaved ODBC drivers.

For example, to call a stored procedure named "usp_NoParameters" that takes no parameters, we can do

```python
crsr.execute("{CALL usp_NoParameters}")
```

To call a stored procedure that takes only input parameters, we can do

```python
params = (14, "Dinsdale")
crsr.execute("{CALL usp_UpdateFirstName (?,?)}", params)
```

### Output Parameters and Return Values

Because pyodbc does not have `.callproc` we need to use a workaround for retrieving the values of output parameters and return values. The specific method will depend on what your particular ODBC driver supports, but for Microsoft's ODBC drivers for SQL Server we can use an "anonymous code block" to EXEC the stored procedure and then SELECT the output parameters and/or return values. For example, for the SQL Server stored procedure

```sql
CREATE PROCEDURE [dbo].[test_for_pyodbc] 
    @param_in nvarchar(max) = N'', 
    @param_out nvarchar(max) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    -- set output parameter
    SELECT @param_out = N'Output parameter value: You said "' + @param_in + N'".';
    
    -- also return a couple of result sets
    SELECT N'SP result set 1, row 1' AS foo
    UNION ALL
    SELECT N'SP result set 1, row 2' AS foo;
    
    SELECT N'SP result set 2, row 1' AS bar
    UNION ALL
    SELECT N'SP result set 2, row 2' AS bar;
END
```

our Python code can do this

```python
sql = """\
DECLARE @out nvarchar(max);
EXEC [dbo].[test_for_pyodbc] @param_in = ?, @param_out = @out OUTPUT;
SELECT @out AS the_output;
"""
params = ("Burma!", )
crsr.execute(sql, params)
rows = crsr.fetchall()
while rows:
    print(rows)
    if crsr.nextset():
        rows = crsr.fetchall()
    else:
        rows = None
```

to produce this:

```none
[('SP result set 1, row 1', ), ('SP result set 1, row 2', )]
[('SP result set 2, row 1', ), ('SP result set 2, row 2', )]
[('Output parameter value: You said "Burma!".', )]
```

Notice that the result set(s) created by the stored procedure are returned first, followed by the result set with the output parameter(s) as returned by the SELECT statement in the anonymous code block passed to the pyodbc `.execute` method.

Similarly, for a SQL Server stored procedure with a RETURN value we can use something like this:


```python
sql = """\
DECLARE @rv int;
EXEC @rv = [dbo].[another_test_sp];
SELECT @rv AS return_value;
"""
crsr.execute(sql)
return_value = crsr.fetchval()

下面这种方式 我觉得更方便 ，解决了我得问题 sqlserver2008测试可行
```python
cursor.execute("""\
	SET NOCOUNT ON; 
	EXEC [dbo].[pos_pr_select_itemstock] @as_item_no=?,@as_item_subno=?,@ai_branchid=?,@as_item_name=?,@as_item_subname=?,@as_clsno=?,@as_itembrand=?;
		
""",('0002483','6901028062008',17,'','','',''))

		
		rows = cursor.fetchall()
		print(rows)
```
### 结果如下：

[('011', '0002483', '苏烟（软五星红杉树）', '苏烟（软五星红杉树）', '6901028062008', Decimal('307.000'), Decimal('0.000'))]
**并不需要做其他处理 仅再调用存储过程上 增加 ：SET NOCOUNT ON;  就行了**