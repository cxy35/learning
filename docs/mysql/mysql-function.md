---
title: MySQL 常用函数
date: 2018-08-06 21:26:24
categories: MySQL
tags: [MySQL, 函数]
toc: true
---
通过本文学习 MySQL 常用函数。
<!-- more -->

## 1 运算符

### 1.1 算术运算符

```sql
-- +, -, *, /
-- 可以在 select 语句中使用
```

### 1.2 比较运算符

```sql
-- >, >=, =, !=, <, <=, like, between, is null, in
```

### 1.3 逻辑运算符 

```sql
-- not, and, or
```

### 1.4 集合运算符

```sql
-- intersect, union, union all, minus 

-- 要求：
-- 1.对应集合的列数和数据类型相同。
-- 2.查询中不能包含 long 列。
-- 3.列的标签是第一个集合的标签。

select * from emp intersect select * from emp where deptno=10;

select * from emp minus select * from emp where deptno=10;

-- 不包括重复行
select * from emp where deptno=10 union select * from emp where deptno in (10,20); 

-- 包括重复行
select * from emp where deptno=10 union all select * from emp where deptno in (10,20);
```

## 2 字符串函数

### 2.1 字符串截取

```sql
-- left/right/substr/substring

-- 截取长度为负数的话从后面开始计算
select left('abcdef',2); -- ab
select right('abcdef',2); -- ef
select substr('abcdef',1,3); -- abc
select substring('abcdef',1,3); -- abc
select substring_index('aa.bb.cc', '.', 2); -- aa.bb
```

### 2.2 查找子串位置

```sql
-- instr

-- 首次出现，从1开始
select instr('abcfdgfdhd','fd'); -- 4
```

### 2.3 字符串连接

```sql
-- concat

select concat('hello ','world'); -- hello world
```

### 2.4 去掉字符串中的空格

```sql
-- ltrim、rtrim、trim

select ltrim(' abc') s1, 
        rtrim('zhang ') s2, 
        trim(' zhang ') s3;

-- 去掉前导和后缀
select trim(leading 9 from 9998767999) s1,
        trim(trailing 9 from 9998767999) s2,
        trim(9 from 9998767999) s3; -- 8767999 9998767 8767
```

### 2.5 返回字符串首字母的Ascii值

```sql
select ascii('abc'); -- 97
```

### 2.6 返回Ascii值对应的字母

```sql
select char(97); -- a
```

### 2.7 计算字符串长度

```sql
-- length

select length('abcdef'); -- 6
```

### 2.8 大小写转换

```sql
-- lower/upper

select lower('ABC') s1, 
        upper('def') s2; -- abc DEF
```

### 2.9 替换

```sql
-- replace

select replace('abcbc','b','xy'); -- axycxyc
```

### 2.10 左右填充（用于控制输出格式）

```sql
-- lpad/rpad

select lpad('func',6,'='),
        lpad('func',3,'='),
        rpad('func',6,'='),
        rpad('func',3,'='); -- ==func fun  func== fun
```

### 2.11 逐字符比较两字串大小

```sql
-- strcmp

select strcmp('a1','a1'); -- 0
select strcmp('a1','a0'); -- 1
select strcmp('a1','a2'); -- -1
```

### 2.12 生成空格

```sql
-- space

select concat('a',space(2),'b'); -- a  b
```

### 2.13 条件判断 if

```sql
select if(1+2=3,'A','B'); -- A
```

### 2.14 条件判断 case

```sql
-- 实现类似 switch case 逻辑

select case 1+2 
	when 3 then 'A'
	when 4 then 'B'
	else 'C'
	end; -- A
```

## 3 日期时间函数

### 3.1 日期转字符串

```sql
-- date_format -- 同 oracle 中的 to_char()

select date_format(now(),'%Y'); -- 2018
select date_format(now(),'%Y-%m-%d'); -- 2018-08-06
select date_format(now(),'%Y-%m-%d %H:%i:%s'); -- 2018-08-06 10:54:46
```

### 3.2 字符串转日期

```sql
-- str_to_date -- 同 oracle 中的 to_date()

select str_to_date('2018-08-06','%Y-%m-%d'); -- 2018-08-06
select str_to_date('2018-08-06 08:30:14','%Y-%m-%d'); -- 2018-08-06
```

### 3.3 秒（毫秒）值与日期转换

```sql
-- from_unixtime()/unix_timestamp()

-- 将秒（毫秒）值转换为日期
select from_unixtime(1454313212000 div 1000,'%Y-%m-%d %H:%i:%s'); 
select from_unixtime(1454313212000/1000,'%Y-%m-%d %H:%i:%s'); 
-- 2016-02-01 15:53:32

-- 将日期转换为日期（毫秒）值
select unix_timestamp('2016-02-01 15:53:32'); 
-- 1454313212(秒)，毫秒需要乘1000
```

### 3.4 日期相关的函数

```sql
-- ADDTIME(date2 ,time_interval) -- 将 time_interval 加到 date2 
-- CONVERT_TZ(datetime2 ,fromTZ ,toTZ) -- 转换时区 
-- CURRENT_DATE() -- 当前日期 
-- CURRENT_TIME() -- 当前时间 
-- CURRENT_TIMESTAMP() -- 当前时间戳 
-- DATE(datetime) -- 返回datetime的日期部分 
-- DATE_ADD(date2 , INTERVAL d_value d_type) -- 在 date2 中加上日期或时间 
-- DATE_FORMAT(datetime ,FormatCodes) -- 使用 formatcodes 格式显示datetime 
-- DATE_SUB(date2 , INTERVAL d_value d_type) -- 在 date2上 减去一个时间 
-- DATEDIFF(date1 ,date2) -- 两个日期差 
-- DAY(date) -- 返回日期的天 
-- DAYNAME(date) -- 英文星期 
-- DAYOFWEEK(date) -- 星期(1-7) ,1为星期天 
-- DAYOFYEAR(date) -- 一年中的第几天 
-- EXTRACT(interval_name FROM date) -- 从 date 中提取日期的指定部分 
-- MAKEDATE(year ,day) -- 给出年及年中的第几天,生成日期串 
-- MAKETIME(hour ,minute ,second) -- 生成时间串 
-- MONTHNAME(date) -- 英文月份名 
-- NOW() -- 当前时间 
-- SEC_TO_TIME(seconds) -- 秒数转成时间 
-- STR_TO_DATE(string ,format) -- 字串转成时间,以 format 格式显示 
-- TIMEDIFF(datetime1 ,datetime2) -- 两个时间差 
-- TIME_TO_SEC(time) -- 时间转秒数] 
-- WEEK(date_time [,start_of_week ]) -- 第几周 
-- YEAR(datetime) -- 年份 
-- DAYOFMONTH(datetime) -- 月的第几天 
-- HOUR(datetime) -- 小时 
-- LAST_DAY(date) -- date 的月的最后日期 
-- MICROSECOND(datetime) -- 微秒 
-- MONTH(datetime) -- 月 
-- MINUTE(datetime) -- 分返回符号,正负或0 
-- SQRT(number2) -- 开平方
```

### 3.5 日期相关的参数说明

```sql
-- 根据 format 字符串格式化 date 值，下列修饰符可以被用在 format 字符串中：
-- %M 月名字(January……December)  
-- %W 星期名字(Sunday……Saturday)  
-- %D 有英语前缀的月份的日期(1st, 2nd, 3rd, 等等。）  
-- %Y 年, 数字, 4 位  
-- %y 年, 数字, 2 位  
-- %a 缩写的星期名字(Sun……Sat)  
-- %d 月份中的天数, 数字(00……31)  
-- %e 月份中的天数, 数字(0……31)  
-- %m 月, 数字(01……12)  
-- %c 月, 数字(1……12)  
-- %b 缩写的月份名字(Jan……Dec)  
-- %j 一年中的天数(001……366)  
-- %H 小时(00……23)  
-- %k 小时(0……23)  
-- %h 小时(01……12)  
-- %I 小时(01……12)  
-- %l 小时(1……12)  
-- %i 分钟, 数字(00……59)  
-- %r 时间,12 小时(hh:mm:ss [AP]M)  
-- %T 时间,24 小时(hh:mm:ss)  
-- %S 秒(00……59)  
-- %s 秒(00……59)  
-- %p AM或PM  
-- %w 一个星期中的天数(0=Sunday ……6=Saturday ）  
-- %U 星期(0……52), 这里星期天是星期的第一天  
-- %u 星期(0……52), 这里星期一是星期的第一天  
-- %% 一个文字“%”
```

## 4 数字函数

### 4.1 向上/向下取整

```sql
-- ceil/floor

select ceil(66.6) N1,floor(66.6) N2; -- 67  66
```

### 4.2 取幂/求平方根

```sql
-- power/sqrt

select power(3,2) N1,sqrt(9) N2; -- 9 3
```

### 4.3 求余

```sql
-- mod

select mod(9,5); -- 4
```

### 4.4 返回固定小数位数（四舍五入）

```sql
-- round

select round(66.667,2); -- 66.67
```

### 4.5 返回值的符号（正数为1, 负数为-1）

```sql
-- sign

select sign(-32),sign(293); -- -1 1
```

### 4.6 求最小值

```sql
-- least

select least(1,2,2,4,3); -- 1
```

### 4.7 随机数

```sql
-- rand

select rand(); -- 0.12119520839415045
```

## 5 转换函数

### 5.1 日期转字符串

见上文。

### 5.2 字符串转日期

见上文。

### 5.3 秒（毫秒）值与日期转换

见上文。

### 5.4 类型转换

```sql
-- cast(xxx AS 类型) 
-- convert(xxx, 类型)

-- 可用的类型：　   
-- 二进制,同带binary前缀的效果 : BINARY    
-- 字符型,可带参数 : CHAR()     
-- 日期 : DATE     
-- 时间: TIME     
-- 日期时间型 : DATETIME     
-- 浮点数 : DECIMAL      
-- 整数 : SIGNED     
-- 无符号整数 : UNSIGNED
```

## 6 分组函数

### 6.1 整个结果集是一个组

```sql
-- max min avg count sum

-- 例：求部门30的最高工资，最低工资,平均工资，总人数，有工作的人数，工种数量及工资总和
select max(ename),max(sal), 
    min(ename),min(sal),
    avg(sal),
    count(*),
    count(job),
    count(distinct(job)),
    sum(sal)
from emp where deptno=30;
```

### 6.2 带 group by 和 having 的分组

```sql
-- 例：按部门分组求最高工资，最低工资，总人数，有工作的人数，工种数量及工资总和
select deptno, 
    max(ename),max(sal),
    min(ename),min(sal),
    avg(sal),
    count(*),
    count(job),
    count(distinct(job)),
    sum(sal)
from emp group by deptno;

-- 例：部门30的最高工资，最低工资，总人数，有工作的人数，工种数量及工资总和
select deptno, 
    max(ename),max(sal),
    min(ename),min(sal),
    avg(sal),
    count(*),
    count(job),
    count(distinct(job)),
    sum(sal)
from emp group by deptno having deptno=30;
```

### 6.3 标准偏差/方差

```sql
-- stddev/variance

select deptno,stddev(sal) from emp group by deptno;
select deptno,variance(sal) from emp group by deptno;
```

### 6.4 带有 rollup 和 cube 操作符的 group By

```sql
-- rollup 按分组的第一个列进行统计和最后的小计
-- cube 按分组的所有列的进行统计和最后的小计

select deptno,job,sum(sal) from emp group by deptno,job;
select deptno,job,sum(sal) from emp group by rollup(deptno,job); 
select deptno,job,sum(sal) from emp group by cube(deptno,job);
```

## 7 其他函数

### 7.1 值是否相等

```sql
-- nullif(ex1,ex2) -- 值相等返回 null ，否则返回第一个值

select nullif(1,1); -- 
select nullif(1,2); -- 1
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)