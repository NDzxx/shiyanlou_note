# mySQL CAST与CONVERT函数的用法

MySQL 的CAST()和CONVERT()函数可用来获取一个类型的值，并产生另一个类型的值。两者具体的语法如下：

CAST(value as type);
CONVERT(value, type);

就是CAST(xxx AS 类型), CONVERT(xxx,类型)。

可以转换的类型是有限制的。这个类型可以是以下值其中的一个：  
- 二进制，同带binary前缀的效果 : BINARY    
- 字符型，可带参数 : CHAR()     
- 日期 : DATE     
- 时间: TIME     
- 日期时间型 : DATETIME     
- 浮点数 : DECIMAL      
- 整数 : SIGNED     
- 无符号整数 : UNSIGNED 

下面举几个例子：

例一
```
 SELECT CONVERT('23',SIGNED);
```
+----------------------+  
| CONVERT('23',SIGNED) |  
+----------------------+  
|                   23 |  
+----------------------+  
1 row in set  

例二  
```
SELECT CAST('125e342.83' AS signed);  
```
+------------------------------+  
| CAST('125e342.83' AS signed) |  
+------------------------------+  
|                          125 |  
+------------------------------+  
1 row in set  

例三
```
 SELECT CAST('3.35' AS signed);
```
+------------------------+  
| CAST('3.35' AS signed) |  
+------------------------+  
| 3 |  
+------------------------+  
1 row in set  


像上面例子一样，将varchar 转为int 用 cast(a as signed)，其中a为varchar类型的字符串。

例4

在SQL Server中，下面的代码演示了datetime变量中，仅包含单纯的日期和单纯的时间时，日期存储的十六进制存储表示结果。

DECLARE @dt datetime
 
--单纯的日期
SET @dt='1900-1-2'
SELECT CAST(@dt as binary(8))
--结果: 0x0000000100000000
 
--单纯的时间
SET @dt='00:00:01'
SELECT CAST(@dt as binary(8))
--结果: 0x000000000000012C

MySQL的类型转换和SQL Server一样，就是类型参数有点点不同：CAST(xxx AS 类型) ,CONVERT(xxx,类型)。