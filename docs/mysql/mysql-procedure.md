---
title: MySQL 存储过程
date: 2018-08-08 21:27:27
categories: MySQL
tags: [MySQL, 存储过程]
toc: true
---
通过本文学习 MySQL 存储过程的语法和使用。
<!-- more -->

## 1 基本语法

```sql
CREATE OR REPLACE PROCEDURE 存储过程名 (
    -- in/out 参数名 参数类型
    in param1 varchar(32)，
    out param2 varchar(32)
)
BEGIN
    -- 变量名 变量类型（取值范围）
    declare var1 varchar(32);
    declare var2 varchar(32) default '';
    
    -- 定义游标遍历时，作为判断是否遍历完全部记录的标记
    declare _done int default 0;
    -- 定义游标
    declare _pqList cursor for select o.`uid`,o.`name` from grid_org_org o where o.type='17';
    -- 定义处理程序，声明当游标遍历完全部记录后将标志变量置成某个值
    declare continue handler for not found set _done=1;
    
    -- 具体实现
END;
```

## 2 给变量赋值

```sql
-- 方法1
set var1='100';
-- 方法2
select col1,col2 into var1,var2 from test_tb;
-- 方法3
set @sqlStr='select count(*) from test_tb';  -- 构造 sql 赋值给一个变量（可以之前没有定义，但要以@开头）
prepare var1 from @sqlStr; -- 预处理需要执行的动态 sql ，其中 var1 是一个变量
execute var1;  -- 执行sql语句
deallocate prepare var1;     -- 释放掉预处理段
```

## 3 游标

### 3.1 cursor 型游标（**不能用于参数传递**） 

- 赋值

```sql
-- 定义变量时赋值
declare var3 cursor for select col1 from temp_tb where id='1';
```

- 遍历

```sql
open _pqList;
loop_label:loop
    fetch _pqList into var1,var2;
    if _done=1 then
        leave loop_label;
    end if;

    -- 具体操作	
    if var1 is not null and length(var1)>0 then

    end if;
end loop;
close _pqList;
```

## 4 实例

```sql
create or replace procedure p_test (
  in var1 varchar(32),-- 输入参数
  out var2 varchar(32)-- 输出普通参数
)
begin
  -- 输出普通参数
  select col1 into var2 from test_tb where id=var1;
  -- set var2='10';
  
  -- 输出结果集，可多个，不用作为参数传进来
  select * from test_tb where id='1';
  select * from test_tb where id='2';
end;
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)