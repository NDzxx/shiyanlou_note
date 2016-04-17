# mysql存储过程中用变量做表名
     
1.用变量做表名：   
    简单的用set或者declare语句定义变量，然后直接作为sql的表名是不行的，mysql会把变量名当作表名。在其他的sql数据库中也是如此，mssql的解决方法是将整条sql语句作为变量，其中穿插变量作为表名，然后用sp_executesql调用该语句。   
    这在mysql5.0之前是不行的，5.0之后引入了一个全新的语句，可以达到类似sp_executesql的功能（仅对procedure有效，function不支持动态查询）：      
```
    PREPARE stmt_name FROM preparable_stmt;   
    EXECUTE stmt_name [USING @var_name [, @var_name] ...];   
    {DEALLOCATE | DROP} PREPARE stmt_name;   
```
为了有一个感性的认识,下面先给几个小例子：  
```mysql
PREPARE stmt1 FROM 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';   
SET @a = 3;   
SET @b = 4;   
EXECUTE stmt1 USING @a, @b; 
```


    