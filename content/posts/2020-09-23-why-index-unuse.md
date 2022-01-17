---
layout:     post
title:      "为什么没有使用索引?"
date:       2020-09-23 11:41:43 +0800
author:     "max"
---

# 为什么没有用索引?
已知表 a 数据总量是 40w, 根据时间范围过滤出最近 7 天的数据, 语句如下:
```
explain SELECT
	`question_id`,
	`student_id`,
	`teacher_id`,
	`question_type`,
	`question_step`,
	`question_no`,
	`level`,
	`class_id`,
	`lesson_id`,
	`teacher_get_at`,
	`teacher_set_at`,
	`status`,
	`teacher_from`,
	`wrong_reason_id`,
	`created_at`,
	`submit_time`
FROM
	`answer_records`
WHERE
	`updated_at` BETWEEN '2020-07-14+00:00:00'
	AND '2020-07-21+00:00:00'
ORDER BY
	`created_at` DESC;
```
![-w883](http://img.fulitu.club/15953253234282.jpg)

从 explain 返回的数据来看, 这条语句并没有使用到索引, 而且 mysql 进行了全表的扫描, 这个原因就是我们要查询的数据太多了.

因为 innodb 引擎会在检索索引后进行回表的操作, mysql 觉得你查询的数据这么多, 我一个个回表的这点时间都可以把全表扫一次了, 我就没必要用索引再去回表了, 所以就会导致这个索引没有用到.

那我们就想用索引, 应该怎么办呢? 

```
FROM
    `answer_records` FORCE INDEX(rqx)
```

在业务层面的话, 如果时间范围比较大, 可以分批次查询, 这样就会快一点, 如果是频繁需要此类操作的话, 还是建议将时间戳提早设计进表结构里, 通过 int 类型的时间戳进行范围查找和排序会事半功倍