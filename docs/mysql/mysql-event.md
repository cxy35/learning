---
title: MySQL 事件
date: 2018-08-10 21:27:39
categories: MySQL
tags: [MySQL, 事件]
toc: true
---
通过本文学习 MySQL 事件。
<!-- more -->

## 1 基本语法

```sql
CREATE EVENT
[IF NOT EXISTS]
event_name
ON SCHEDULE schedle
[ON COMPLETION [NOT] PRESERVE]
[ENABLE|DESABLE]
[COMMENT 'comment']
DO sql_statement

-- 说明：
-- event_name：事件的名称
-- ON SCHEDULE 设定计划任务的方式，有两种：
    -- 单次计划任务：AT 时戳
    -- 重复的计划任务：EVERY 时间(单位)的数量 时间单位 [STARTS 时戳][ENDS 时戳]
        -- 在两种计划任务中，时戳可以是任意的 TIMESTAMP 和 DATETIME 数据类型，要求提供的是将来的时间（大于 CURRENT_TIMESTAMP），而且小于 Unix 时间的最后时间（等于或小于'2037-12-31 23:59:59'）
        -- 时间单位是关键词：YEAR，MONTH，DAY，HOUR，MINUTE 或者 SECOND
-- [ON COMPLETION [NOT] PRESERVE]：COMPLETION 当单次计划任务执行完毕后或当重复性的计划任务执行到了 ENDS 阶段。而声明 PRESERVE 的作用是使事件在执行完毕后不会被 Drop 掉
-- [ENABLE|DESABLE]：开启/关闭事件
-- [COMMENT 'comment']：注释
-- DO sql_statement：执行的 sql语句

-- 注意：
-- 全局事件调度器启用状态：SHOW VARIABLES LIKE '%event_scheduler%'; -- ON/OFF
```

## 2 模板与实例

### 2.1 模板

```sql
DELIMITER $$

-- SET GLOBAL event_scheduler = ON$$     -- required for event to execute but not create    

CREATE	/*[DEFINER = { user | CURRENT_USER }]*/	EVENT `E_NAME`

ON SCHEDULE
	 /* uncomment the example below you want to use */

	-- scheduleexample 1: run once

	   --  AT 'YYYY-MM-DD HH:MM.SS'/CURRENT_TIMESTAMP { + INTERVAL 1 [HOUR|MONTH|WEEK|DAY|MINUTE|...] }

	-- scheduleexample 2: run at intervals forever after creation

	   -- EVERY 1 [HOUR|MONTH|WEEK|DAY|MINUTE|...]

	-- scheduleexample 3: specified start time, end time and interval for execution
	   /*EVERY 1  [HOUR|MONTH|WEEK|DAY|MINUTE|...]

	   STARTS CURRENT_TIMESTAMP/'YYYY-MM-DD HH:MM.SS' { + INTERVAL 1[HOUR|MONTH|WEEK|DAY|MINUTE|...] }

	   ENDS CURRENT_TIMESTAMP/'YYYY-MM-DD HH:MM.SS' { + INTERVAL 1 [HOUR|MONTH|WEEK|DAY|MINUTE|...] } */

/*[ON COMPLETION [NOT] PRESERVE]
[ENABLE | DISABLE]
[COMMENT 'comment']*/

DO
	BEGIN
	    (sql_statements)
	END$$

DELIMITER ;
```

### 2.2 实例

```sql
DELIMITER $$   

DROP EVENT IF EXISTS `E_TEST`$$

CREATE EVENT `E_TEST`
ON SCHEDULE
EVERY 1 DAY 
STARTS '2018-01-01 01:00:00'
ON COMPLETION NOT PRESERVE
ENABLE

DO
	BEGIN
        -- 执行 sql
        INSERT INTO t_log SET createTime = NOW();

        -- 调用存储过程
		CALL p_test();
	END$$

DELIMITER ;
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)