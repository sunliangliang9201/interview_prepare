## sql部分

表结构信息如下

#### 学生信息表student

| sid  | sname | sage       | ssex |
| ---- | ----- | ---------- | ---- |
| 01   | 赵雷  | 1990-01-01 | 男   |
| 02   | 李云  | 1990-08-06 | 男   |

...

#### 课程表course

| cid  | cname | tid          |
| ---- | ----- | ------------ |
| 01   | 语文  | 02（教师id） |
| 02   | 数学  | 01           |

...

#### 教师表teacher

| tid  | tname |
| ---- | ----- |
| 01   | 张三  |
| 02   | 李四  |

...

成绩表sc

| sid  | cid  | score |
| ---- | ---- | ----- |
| 01   | 01   | 80.0  |
| 01   | 03   | 90.0  |
| 02   | 01   | 65.0  |

...

注意前...个题时以这三个表位基础的。

这前50题是直接在别人的博客中摘抄来的。https://www.jianshu.com/p/d6ca0a611fd2，属于非常非常基础的sql题。后续会把真实的面试中的sql题整理进来。

#### 1.查询“01”课程比“02”课程成绩高的所有学生的学号？

```sql
SELECT 
    t1.sid
FROM
    sc t1,
    sc t2
WHERE
    t1.sid = t2.sid AND t1.cid = '01'
        AND t2.cid = '02'
        AND t1.score > t2.score;
```

```sql
SELECT 
    *
FROM
    sc t1
        INNER JOIN
    sc t2 ON t1.sid = t2.sid
WHERE
    t1.cid = '01' AND t2.cid = '02'
        AND t1.score > t2.score;
```

```sql
SELECT 
    t1.sid
FROM
    ((SELECT 
        sid, cid, score
    FROM
        sc
    WHERE
        cid = '01') t1
    LEFT JOIN (SELECT 
        sid, cid, score
    FROM
        sc
    WHERE
        cid = '02') t2 ON t1.sid = t2.sid)
WHERE
    t1.score > t2.score;
```



#### 2.查询平均成绩大于60分的同学的学号和平均成绩?

```sql
SELECT 
    sid, AVG(score)
FROM
    sc
GROUP BY sid
HAVING AVG(score) > 60;
```



#### 3.查询所有同学的学号、姓名、选课数、总成绩?

```sql
SELECT 
    student.sid, sname, COUNT(sc.cid), SUM(sc.score)
FROM
    student
        LEFT JOIN
    sc ON student.sid = sc.sid
GROUP BY student.sid , student.sname;
```



#### 4.查询姓“李”的老师的个数?

```sql
SELECT 
    COUNT(DISTINCT tname)
FROM
    teacher
WHERE
    tname LIKE '李%';
```



#### 5.查询没学过“张三”老师课的同学的学号、姓名?

```sql
SELECT 
    sid, sname
FROM
    student
WHERE
    sid NOT IN (SELECT 
            sid
        FROM
            (SELECT 
                sc.sid, sc.cid, course.tid
            FROM
                sc
            INNER JOIN course ON sc.cid = course.cid) t1
                INNER JOIN
            teacher ON t1.tid = teacher.tid
        WHERE
            teacher.tname = '张三')
```

**注意：**这里有一个优化就是left join左边最好放小表，很明显teacher表很小，其次是课程表，最后是成绩表，所以join的顺序为teacher join course然后再joinsc。



#### 6.查询学过编号“01”并且也学过编号“02”课程的同学的学号、姓名?

```sql
SELECT 
    t1.sid
FROM
    (SELECT 
        sid
    FROM
        sc
    WHERE
        cid = '01') t1
        INNER JOIN
    (SELECT 
        sid
    FROM
        sc
    WHERE
        cid = '02') t2 ON t1.sid = t2.sid;
```

```sql
SELECT 
    sid, sname
FROM
    student
WHERE
    sid IN (SELECT 
            sid
        FROM
            sc
        WHERE
            cid = '01'
        HAVING sid IN (SELECT 
                sid
            FROM
                sc
            WHERE
                cid = '02'));
```



#### 7.查询学过“张三”老师所教的课的同学的学号、姓名?

```sql
SELECT 
    sid, sname
FROM
    student
WHERE
    sid IN (SELECT 
            sc.sid
        FROM
            teacher
                LEFT JOIN
            course ON teacher.tid = course.tid
                LEFT JOIN
            sc ON course.cid = sc.cid
        WHERE
            teacher.tname = '张三');
```



#### 8.查询课程编号“01”的成绩比课程编号“02”课程低的所有同学的学号、姓名?

```sql
SELECT 
    student.sid, student.sname
FROM
    (SELECT 
        *
    FROM
        sc
    WHERE
        cid = '01') t1
        LEFT JOIN
    (SELECT 
        *
    FROM
        sc
    WHERE
        cid = '02') t2 ON t1.sid = t2.sid
        LEFT JOIN
    student ON t1.sid = student.sid
WHERE
    t1.score < t2.score;
```



#### 9.查询所有课程成绩小于60分的同学的学号、姓名?

```sql
SELECT 
    student.sid, student.sname
FROM
    (SELECT 
        sid, AVG(score)
    FROM
        sc
    GROUP BY sid
    HAVING AVG(score) < 60) t1
        LEFT JOIN
    student ON student.sid = t1.sid;
```



#### 10.查询没有学全所有课的同学的学号、姓名?

```sql
SELECT 
    student.sid, student.sname
FROM
    (SELECT 
        sid, COUNT(DISTINCT (cid))
    FROM
        sc
    GROUP BY sid
    HAVING COUNT(DISTINCT (cid)) < (SELECT 
            COUNT(1)
        FROM
            course)) t1
        LEFT JOIN
    student ON student.sid = t1.sid;
```



#### 11.查询至少有一门课与学号为“01”的同学所学相同的同学的学号和姓名?

```sql
select
    distinct sc.sid
from 
    (
        select
            cid
        from sc
        where sid='01'
    )t1
left join sc
    on t1.cid=sc.cid where sc.sid <> '01';
```



#### 12.查询和"01"号的同学学习的课程完全相同的其他同学的学号和姓名?

```sql
SELECT 
    sc.sid
FROM
    (SELECT 
        cid
    FROM
        sc
    WHERE
        sid = '01') t1
        INNER JOIN
    sc ON t1.cid = sc.cid
WHERE
    sc.sid <> '01'
GROUP BY sc.sid
HAVING COUNT(DISTINCT (sc.cid)) = (SELECT 
        COUNT(1)
    FROM
        sc
    WHERE
        sid = '01');
```



#### 13.把“SC”表中“张三”老师教的课的成绩都更改为此课程的平均成绩?

```sql
select course.cid as cid , t2.pingjun as pinjgun from
(select tid from teacher where tname = '张三') t1 
inner join course 
on t1.tid = course.tid
inner join 
(select cid, avg(score) as pingjun from sc group by cid) t2
on course.cid = t2.cid;
```

**注意：**上面这个sql不是更新的sql，而是查的“张三”老师上的所有课的平均成绩，关于如何更新，我觉得可以结合代码来完成。



#### 14.查询没学过"张三"老师讲授的任一门课程的学生姓名?

```sql
SELECT 
    sname
FROM
    student
WHERE
    sid NOT IN (SELECT 
            sc.sid
        FROM
            (SELECT 
                tid
            FROM
                teacher
            WHERE
                tname = '张三') t1
                INNER JOIN
            course ON t1.tid = course.tid
                INNER JOIN
            sc ON sc.cid = course.cid);
```



#### 15.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩?

```sql
SELECT 
    student.sid, student.sname, t1.pingjun
FROM
    (SELECT 
        sid, AVG(score) AS pingjun
    FROM
        sc
    WHERE
        sid IN (SELECT 
                sid
            FROM
                sc
            WHERE
                score < 60
            GROUP BY sid
            HAVING COUNT(score) >= 2)
    GROUP BY sid) t1
        INNER JOIN
    student ON t1.sid = student.sid;
```

**注意：**其实有更好的一个方法就是不通过过滤条件来判断不及格课程数，而是通过函数来判断

```sql
SELECT 
    student.sid, student.sname, t1.avg_score
FROM
    (SELECT 
        sid, AVG(score) AS avg_score
    FROM
        sc
    GROUP BY sid
    HAVING COUNT(IF(score < 60, cid, NULL)) >= 2) t1
        LEFT JOIN
    student ON student.sid = t1.sid;
```



#### 16.检索"01"课程分数小于60，按分数降序排列的学生信息?

```sql
SELECT 
    *
FROM
    sc
WHERE
    cid = '01' AND score < 60
ORDER BY score DESC;
```



#### 17.按平均成绩从高到低显示所有学生的平均成绩?

```sql
SELECT 
    sid, AVG(score) AS avg_score
FROM
    sc
GROUP BY sid
ORDER BY avg_score DESC;
```



#### 18.查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率?

```sql
SELECT 
    cid,
    MIN(score),
    MAX(score),
    AVG(score),
    COUNT(IF(score > 60, score, NULL)) / COUNT(score)
FROM
    sc
GROUP BY cid;
```



#### 19.按各科平均成绩从低到高和及格率的百分数从高到低顺序?

```sql
SELECT 
    cid,
    AVG(score) AS avg_score,
    COUNT(IF(score > 60, cid, NULL)) / COUNT(score) AS rate
FROM
    sc
GROUP BY cid
ORDER BY avg_score , rate DESC;
```



#### 20.查询学生的总成绩并进行排名?

```sql
SELECT 
    student.sid, student.sname, t1.total
FROM
    student
        INNER JOIN
    (SELECT 
        sid, SUM(score) AS total
    FROM
        sc
    GROUP BY sid) t1 ON student.sid = t1.sid
ORDER BY total DESC;
```



#### 21、查询不同老师所教不同课程平均分从高到低显示?

```sql
SELECT 
    t1.cid, course.cname, t1.avg_score, teacher.tname
FROM
    ((SELECT 
        cid, AVG(score) AS avg_score
    FROM
        sc
    GROUP BY cid) t1
    INNER JOIN course ON t1.cid = course.cid
    LEFT JOIN teacher ON course.tid = teacher.tid);
```



#### 22.查询所有课程的成绩第2名到第3名的学生信息及该课程成绩?

**注意：**这个问题看似简单，确实非常考验基础的问题！！！因为group by cid之后只能查询聚合结果了，不能查询某一条或者多条记录的！所以这个问题还是有水平的，至少不常写sql的我第一时间没有意识到这个问题嘛，真正写sql时才发现思路不对。那么应该如何实现组内取出多条记录呢？针对组内按照一定顺序取出某些位置的数据有一个很好的函数rank() over和ro w_number() over，关于他俩啥意思在下面介绍。趁此机会学习一下吧。

```sql
select * from 
(SELECT 
    sid,
    cid,
    score,
    rank() over(partition by cid order by score desc) as rank_num
FROM
    sc) t1 where t1.rank_num in (2,3);
```

- rank() over：使用规则是 rank() over(partition by cid order by score desc) as rank_num，意思就是这个rank_num代表cid分组内根据score倒序排序后的顺序num，从0开始 0 1 2...，说白了就是原来表的基础上给每条记录添加一个rank_num字段！！！

  **注意：**rank() over函数处理数据时，相同数值的组内排序num时一致的，那么就有一个问题，当第一名和第二名成绩相同时，rank over会认为他俩都是第一名，我们取第二名时是没有结果的！！！！！

  ```sql
  select * from
  (SELECT 
      sid,
      cid,
      score,
      row_number() over(partition by cid order by score desc) as row_num
  FROM
      sc) t1 where t1.row_num in (2,3);
  ```

- row_number() over:使用规则与ran k() over一样，唯一的区别就在于组内相同数据的组内排序值不同，所以假如第一名和第二名成绩相同，他俩还是第一名和第二名！！



#### 23.统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比？

```sql
SELECT 
    cid,
    COUNT(IF(score BETWEEN 85 AND 100,score,NULL)) / COUNT(1),
    COUNT(IF(score BETWEEN 70 AND 85, score, NULL)) / COUNT(1),
    COUNT(IF(score BETWEEN 60 AND 70, score, NULL)) / COUNT(1),
    COUNT(IF(score BETWEEN 0 AND 60, score, NULL)) / COUNT(1)
FROM
    sc
GROUP BY cid
```

**注意：**between and 函数要求小的在前，大的在后，如果score bewteen 100 and 85的话，是没有结果的。要注意哦！



#### 24.查询学生平均成绩及其名次？

```sql
select 
	*, 
    row_number() over(order by t1.avg_score desc) 
from 
	(select 
		sid, 
		avg(score) as avg_score 
	from 
		sc 
	group by sid) t1;
```



#### 25.查询各科成绩前三名的记录？

**哎呦喂**想啥来啥——刚才在介绍rank over和rownumber over的时候我本想说一下如果不用这两个函数如何实现topN的结果集，说曹操曹操就到。首先我们用rank或者rownumber来实现每科前三名。

```sql
select * from
(SELECT 
    sid, cid, score, row_number() over(partition by cid order by score desc) as row_num
FROM
    sc) t1 where t1.row_num <= 3;
```

下面我们用第二种方式来实现：

思想：这种思路就是，前三名，对于每一条记录我们要保证比这条记录的>=score的最多有三个，也就是保证了这条记录是前三甲！

```sql
SELECT 
    sid, cid, score
FROM
    sc t1
WHERE
    (SELECT 
            COUNT(1)
        FROM
            sc t2
        WHERE
            t1.cid = t2.cid AND t1.score <= t2.score) <= 3
ORDER BY cid , score DESC;
```



#### 26.查询每门课程被选修的学生数?

```sql
SELECT 
    course.cid, COUNT(sid)
FROM
    course
        LEFT JOIN
    sc ON course.cid = sc.cid
GROUP BY cid;
```



#### 27.查询出只选修了一门课程的全部学生的学号和姓名?

```sql
SELECT 
    sid, sname
FROM
    student
WHERE
    sid IN (SELECT 
            sid
        FROM
            sc
        GROUP BY sid
        HAVING COUNT(cid) = 1);
```



#### 28.查询男生、女生人数?

```sql
SELECT 
    ssex, COUNT(ssex)
FROM
    student
GROUP BY ssex;
```



#### 29.查询名字中含有"风"字的学生信息?

```sql
SELECT 
    *
FROM
    student
WHERE
    sname LIKE '%风%';
```



#### 30.查询同名同性学生名单，并统计同名人数?

```sql
SELECT 
    sname, COUNT(*)
FROM
    student
GROUP BY sname
HAVING COUNT(*) > 1;
```



#### 31.查询1990年出生的学生名单(注：Student表中Sage列的类型是datetime)?

```sql
SELECT 
    *
FROM
    student
WHERE
    sage BETWEEN '1990-01-01 00:00:00' AND '1990-12-31 23:59:59';
```



#### 32.查询每门课程的平均成绩，结果按平均成绩升序排列，平均成绩相同时，按课程号降序排列?

```sql
SELECT 
    cid, AVG(score) AS avg_score
FROM
    sc
GROUP BY cid
ORDER BY avg_score , cid;
```



#### 37.查询不及格的课程，并按课程号从大到小排列?

```sql
SELECT 
    course.cid, course.cname
FROM
    (SELECT 
        cid
    FROM
        sc
    GROUP BY cid
    HAVING MIN(score) < 60) t1
        LEFT JOIN
    course ON t1.cid = course.cid
ORDER BY course.cid desc;
```



#### 38.查询课程编号为"01"且课程成绩在60分以上的学生的学号和姓名?

```sql
SELECT 
    student.sid, student.sname
FROM
    (SELECT 
        sid
    FROM
        sc
    WHERE
        cid = '01' AND score > 60) t1
        LEFT JOIN
    student ON t1.sid = student.sid;
```



#### 40.查询选修“张三”老师所授课程的学生中，成绩最高的学生姓名及其成绩?

```sql
SELECT 
    *
FROM
    teacher
        LEFT JOIN
    course ON teacher.tid = course.tid
        LEFT JOIN
    sc ON sc.cid = course.cid
        LEFT JOIN
    student ON student.sid = sc.sid
WHERE
    teacher.tname = '张三'
ORDER BY sc.score DESC
LIMIT 1;
```



#### 42.查询每门功课成绩最好的前两名?

这个题还用两种方法哈，一般来说直接用窗口即可，但是为了面试官两种方式都写一下吧

```sql
select student.sname, course.cname, t1.score from
(SELECT 
    sid, 
    cid, 
    score, 
    row_number() over(partition by cid order by score desc) as row_num
FROM
    sc) t1 left join course on course.cid = t1.cid left join student on student.sid = t1.sid
    where t1.row_num <=2;
```

```sql
SELECT 
    student.sname, t3.cid, t3.score
FROM
    (SELECT 
        t1.sid, t1.cid, t1.score
    FROM
        sc t1
    WHERE
        (SELECT 
                COUNT(1)
            FROM
                sc t2
            WHERE
                t2.cid = t1.cid AND t2.score >= t1.score) <= 2) t3
        LEFT JOIN
    student ON student.sid = t3.sid;
```



#### 43.统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列?

```sql
SELECT 
    cid, COUNT(sid) AS total
FROM
    sc
GROUP BY cid
HAVING total >= 5
ORDER BY total DESC , cid;
```



#### 44.检索至少选修两门课程的学生学号?

```sql
SELECT 
    sid
FROM
    sc
GROUP BY sid
HAVING COUNT(1) >= 2;
```



#### 45.查询选修了全部课程的学生信息?

```sql
SELECT 
    sid
FROM
    course
        INNER JOIN
    sc ON course.cid = sc.cid
GROUP BY sid
HAVING COUNT(1) = (SELECT 
        COUNT(1)
    FROM
        course);
```



#### 46.查询各学生的年龄?

```sql
SELECT 
    sname, YEAR(CURRENT_DATE()) - YEAR(sage) AS age
FROM
    student;
```



#### 47.查询本周过生日的学生?

```sql
SELECT 
    *
FROM
    student
WHERE
    WEEKOFYEAR(CURRENT_DATE()) = WEEKOFYEAR(sage);
```



#### 48.查询上周过生日的学生?

```sql
SELECT 
    *
FROM
    student
WHERE
    WEEKOFYEAR(DATE_ADD(CURRENT_DATE(),INTERVAL 1 WEEK)) = WEEKOFYEAR(sage);
```



#### 49.查询本月过生日的学生?

```sql
SELECT 
    *
FROM
    student
WHERE
    MONTH(CURRENT_DATE) = MONTH(sage);
```



#### 50.查询上月过生日的学生?

```sql
SELECT 
    *
FROM
    student
WHERE
    MONTH(date_sub(CURRENT_DATE,interval 1 month)) = MONTH(sage);
```



> 总结：从以上50个sql题我写到了两点，第一：就是基础的join以及各种分组聚合操作；第二：我学到了rank（） over 和row_number() over这两个函数。
>
> 再不考虑数量的情况下我认为基础到现在是没有问题的，那么接下来我将记录面试中的复杂sql以及hivesql的整理。如果可以的话我顺便将sql的计算流程或者是hivesql的计算流程、效率等问题阐述一下。当然了如果能够分析sql的计算流程和性能的话，那就可以进行sql调优了，我现在还没有达到那个水平，毕竟sql是计算机专业的基础。





#### 51.一张login表，请求今天登陆过昨天未登录的用户uid(头条)？

| uid  | login_date |
| ---- | ---------- |
| 001  | 2019-08-25 |

```sql
SELECT 
    uid
FROM
    login
WHERE
    login_date BETWEEN '2019-08-26 00:00:00' AND '2019-08-26 23:59:59'
        AND uid NOT IN (SELECT 
            uid
        FROM
            login
        WHERE
            login_date BETWEEN '2019-08-25 00:00:00' AND '2019-08-25 23:59:59');
```

```sql
SELECT 
    t1.uid
FROM
    (SELECT 
        uid
    FROM
        login
    WHERE
        login_date BETWEEN '2019-08-26 00:00:00' AND '2019-08-26 23:59:59') t1
        LEFT JOIN
    (SELECT 
        uid
    FROM
        login
    WHERE
        login_date BETWEEN '2019-08-25 00:00:00' AND '2019-08-25 23:59:59') t2 ON t1.uid = t2.uid
WHERE
    t2.uid IS NULL;
```



#### 52.组内聚合问题？

| rq         | shengfu |
| ---------- | ------- |
| 2005-05-09 | 胜      |
| 2005-05-09 | 负      |
| 2005-05-09 | 胜      |
| 2005-05-10 | 负      |

把上表转换为如下

| rq         | 胜   | 负   |
| ---------- | ---- | ---- |
| 2005-05-09 | 2    | 1    |
| 2005-05-10 | 0    | 1    |

- 第一种方案：

```sql
SELECT 
    rq,
    COUNT(IF(shengfu = '胜', 1, NULL)) AS 胜,
    COUNT(IF(shengfu = '负', 1, NULL)) AS 负
FROM
    tmp
GROUP BY rq;
```

```sql
SELECT 
    rq,
    count(case when shengfu = '胜' then 1 else null end) AS 胜,
    count(case when shengfu = '负' then 1 else null end) AS 负
FROM
    tmp
GROUP BY rq;
```

**注意：**其实我想说的问题是这里，如果我们把count变成sum，可以吗？？？？

答案是可以的但是要注意一点就是sum是加和，count是计数，所以用sum的前提条件是数值，比如前面50个基础题中有count(if(sid = '01', sid, null))这个函数，如果用sum肯定是不行的，sid不能加和的。那么换一句话说如果用sum怎么做？

```sql
SELECT 
    rq,
    sum(case when shengfu = '胜' then 1 else 0 end) AS 胜,
    sum(case when shengfu = '负' then 1 else 0 end) AS 负
FROM
    tmp
GROUP BY rq;
```

```sql
SELECT 
    rq,
    sum(IF(shengfu = '胜', 1, NULL)) AS 胜,
    sum(IF(shengfu = '负', 1, NULL)) AS 负
FROM
    tmp
GROUP BY rq;
```

总结一句话就是，聚合函数的应用需要看类型，一般来说没有问题的。

- 第二种方案

```sql
SELECT 
    ifnull(t1.rq, t2.rq), ifnull(t1.胜, 0), ifnull(t2.负, 0)
FROM
    (SELECT 
        rq, COUNT(1) AS 胜
    FROM
        tmp
    WHERE
        shengfu = '胜'
    GROUP BY rq) t1
        JOIN
    (SELECT 
        rq, COUNT(1) AS 负
    FROM
        tmp
    WHERE
        shengfu = '负'
    GROUP BY rq) t2 ON t1.rq = t2.rq;
```



#### 53.请教一个面试中遇到的SQL语句的查询问题表中有A B C三列,用SQL语句实现：当A列大于B列时选择A列否则选择B列，当B列大于C列时选择B列否则选择C列?

```sql
SELECT 
    IF(A > B, A, B), IF(B > C, B, C)
FROM
    tmp2;
```

```sql
SELECT 
    case when A > B then A  when A =B then B else B end,
    case when B > C then B else C end
FROM
    tmp2;
```

**注意：**其实这个题我想说的是一下三个函数

if(condition, 'true', 'false')：如果condition成立则返回第二个参数，否则返回第三个参数

ifnull(A ,B)：如果A是null的话返回B，否则就是A，这种应用在判断null，比如join的时候

case when condition2 then A when condition2 B else C：它是实现多重逻辑判断的方式！



#### 54.一张表有一个sendtime字段（datetime），请找出当天的所有记录？

假设有字段如下uid,uname,sendtime

```sql
select * from tmp3 where datediff(sendtime, now())=0;
```

想要说的是datediff函数，表示两个日期相减返回相差的天数！



#### 55.有一张表，里面有3个字段：语文，数学，英语。其中有3条记录分别表示语文70分，数学80分，英语58分，请用一条sql语句查询出这三条记录并按以下条件显示出来？  

 大于或等于80表示优秀，大于或等于60表示及格，小于60分表示不及格。

| 语文 | 数学   | 英语 |
| ---- | ------ | ---- |
| 及格 | 不及格 | 及格 |

```sql
select
(case when 语文>=80 then '优秀'
        when 语文>=60 then '及格'
else '不及格') as 语文,
(case when 数学>=80 then '优秀'
        when 数学>=60 then '及格'
else '不及格') as 数学,
(case when 英语>=80 then '优秀'
        when 英语>=60 then '及格'
else '不及格') as 英语,
from table
```



#### 56.请用一个sql语句得出结果从table1中取出如table2所列格式数据?

table1

| monthId | depId | grade |
| ------- | ----- | ----- |
| 01      | 01    | 100   |
| 01      | 02    | 80    |
| 01      | 03    | 110   |
| 02      | 01    | 150   |

Table2

| depId | 一月 | 二月 | 三月 |
| ----- | ---- | ---- | ---- |
| 01    | 100  | 150  | 0    |
| 02    | 80   | 0    | 0    |

这是一个典型的行转列，关于行转列和列转行我会在下面专门整理。

```sql
SELECT 
    depId,
    SUM(CASE WHEN monthId = '01' THEN grade ELSE 0 END) AS '一月',
    SUM(CASE WHEN monthId = '02' THEN grade ELSE 0 END) AS '二月',
    SUM(CASE WHEN monthId = '03' THEN grade ELSE 0 END) AS '三月',
    SUM(CASE WHEN monthId = '04' THEN grade ELSE 0 END) AS '四月',
    SUM(CASE WHEN monthId = '05' THEN grade ELSE 0 END) AS '五月',
    SUM(CASE WHEN monthId = '06' THEN grade ELSE 0 END) AS '六月',
    SUM(CASE WHEN monthId = '07' THEN grade ELSE 0 END) AS '七月',
    SUM(CASE WHEN monthId = '08' THEN grade ELSE 0 END) AS '八月',
    SUM(CASE WHEN monthId = '09' THEN grade ELSE 0 END) AS '九月',
    SUM(CASE WHEN monthId = '10' THEN grade ELSE 0 END) AS '十月',
    SUM(CASE WHEN monthId = '11' THEN grade ELSE 0 END) AS '十一月',
    SUM(CASE WHEN monthId = '12' THEN grade ELSE 0 END) AS '十二月'
FROM
    table1
GROUP BY depId;
```

当然了，这个题还可以把sum直接换成select子查询也可以！实际上函数嘛，感觉就是一个子查询

**因此**：下回再有面试官问这种看似行转列，列转行的问题时，一定要先看看是不是真的行列置换，如果不是那么就用这种方式，分组+聚合函数。

**如果做出来的这个题，面试官可能再问一句，如果从后表转换为之前的表（列转行）————方法就是union！！**

```sql
select * from
(select depId, '一月' as monthId, 一月 as grade from table1)union
(select depId, '二月' as monthId, 二月 as grade from table1)union
(select depId, '三月' as monthId, 三月 as grade from table1)union
....
(select depId, '十二月' as monthId, 十二月 as grade from table1))
order by depId;
```

这就是用union得到的列转行。



#### 57.按照顺序累加的sql题如下？

| T    | Salary |
| ---- | ------ |
| 2000 | 1000   |
| 2001 | 2000   |
| 2002 | 3000   |


| T    | Salary |
| ---- | ------ |
| 2000 | 1000   |
|      | 3000   |
| 2002 | 6000   |

> 我去，有没有点乱！一开始确实思路乱了，但是仔细回想一下，我们不用row_number over函数实现topN的时候是怎么做的？？？？我们用了一个子查询，里面的子查询保证了数据是满足条件的。
>
> 所以应用在这里，我们用子查询，在里面把小于或等于某年之前的进行sum即可。
>
> 区别在于：topN的时候子查询作为where条件，这里的子查询直接就是字段值。

```sql
SELECT 
    t,
    (SELECT 
            SUM(salary)
        FROM
            table2 t2
        WHERE
            t1.t >= t2.t)
FROM
    table2 t1;
```

这种题一般来说都可以用连接来搞定

```sql
SELECT 
    t1.t, SUM(t2.salary)
FROM
    table2 t1,
    table2 t2
WHERE
    t2.t <= t1.t
GROUP BY t1.t;
```

这种table1, table2 这种关联查询说白了就是笛卡尔积。然后加一个条件再分组求和即可。



#### 58.一个叫department的表，里面只有一个字段name,一共有4条纪录，分别是a,b,c,d,对应四个球对，现在四个球对进行比赛，用一条sql语句显示所有可能的比赛组合？

分析：这是一个组合问题，C42，结果是6场比赛，这个题跟上一题题一摸一样！

```sql
select a.name, b.name
from team a, team b
where a.name < b.name;
```

如果是分主客场的情况下就是全链接，把where条件换成a.name <> b.name，一共是12场比赛。



#### 59.从TestDB数据表中查询出所有月份的发生额都比101科目相应月份的发生额高的科目。请注意：TestDB中有很多科目，都有1－12月份的发生额？

这个题用一个连接即可，类似于前面50基础题中查询所有课程都比01号学生分数高的学生。



#### 60.整理一下行列转换的问题？

为什么要单独说一下呢？因为上面遇到了但是没有整理，这里整理一下，以后遇到了就不用再说了。我在网上找了很多mysql的行列转置的问题，千篇一律！！！跟56题是一样的！！

下面就从行转列、列转行来总结一下吧。其实上面说了，行转列用聚合函数、列转行用union。

- 行转列

数据就用上面50基础题的那四个表，想要得到的结果如下

| 姓名 | 数学 | 语文               | 英语 |
| ---- | ---- | ------------------ | ---- |
| 赵雷 | 80   | 90                 | 100  |
| 张三 | 65   | 0（没有则认为是0） | 40   |

```sql
select student.sname, t1.语文,t1.数学,t1.英语 from
(select 
	sc.sid,
    max(case when course.cname = '语文' then sc.score else 0 end) as "语文",
    max(case when course.cname = '数学' then sc.score else 0 end) as "数学",
    max(case when course.cname = '英语' then sc.score else 0 end) as "英语"
from sc
left join course
on course.cid = sc.cid
group by sc.sid) t1 left join student on student.sid = t1.sid;
```

简单说就是分组+聚合函数

- 列转行

如果我们要从上面这个表反推回sc表怎么办？

```sql
select * from
((select sname, '语文' as cname, 语文 as score from sc2) union
(select sname, '数学' as cname, 数学 as score from sc2) union
(select sname, '英语' as cname, 英语 as score from sc2)) t1
where t1.score <> 0
order by t1.sname;
```

说白了就是用union















## hive sql部分

#### 1.





原表（渠道有xyz，广告商有abcd）

| Rood | a    | b    | c    | d    |
| ---- | ---- | ---- | ---- | ---- |
| x    | 100  | 300  | 0    | 20   |
| y    | 50   | 0    | 0    | 40   |
| z    | 200  | 150  | 100  | 30   |

想要变成如下

| Shangjia | x    | y    | z    |
| -------- | ---- | ---- | ---- |
| a        | 100  | 50   | 200  |
| b        | 300  | 0    | 150  |
| c        | 0    | 0    | 100  |
| d        | 20   | 40   | 30   |

**注意：**这里跟之前的行转列不同的是，uid不变，所以分组之后用sum/max/子查询就可以解决行转列。

这里直接转置了！





#### 行转列的复杂一题(360)？

原表（渠道有xyz，广告商有abcd）

|      | tou zi                 |
| ---- | ---------------------- |
| x    | a:100,b:300,d:20       |
| y    | a:50,d:40              |
| z    | a:200,b:150,c:100,d:30 |

变为上一题中的样子





