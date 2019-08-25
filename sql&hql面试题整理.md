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

这前50题是直接在别人的博客中摘抄来的。https://www.jianshu.com/p/d6ca0a611fd2

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



#### 23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比
#### 24、查询学生平均成绩及其名次
#### 25、查询各科成绩前三名的记录
#### 26、查询每门课程被选修的学生数
#### 27、查询出只选修了一门课程的全部学生的学号和姓名
#### 28、查询男生、女生人数
#### 29、查询名字中含有"风"字的学生信息
#### 30、查询同名同性学生名单，并统计同名人数
#### 31、查询1990年出生的学生名单(注：Student表中Sage列的类型是datetime)
#### 32、查询每门课程的平均成绩，结果按平均成绩升序排列，平均成绩相同时，按课程号降序排列
#### 37、查询不及格的课程，并按课程号从大到小排列
#### 38、查询课程编号为"01"且课程成绩在60分以上的学生的学号和姓名；
#### 40、查询选修“张三”老师所授课程的学生中，成绩最高的学生姓名及其成绩
#### 42、查询每门功课成绩最好的前两名
#### 43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
#### 44、检索至少选修两门课程的学生学号
#### 45、查询选修了全部课程的学生信息
#### 46、查询各学生的年龄
#### 47、查询本周过生日的学生
#### 48、查询下周过生日的学生
#### 49、查询本月过生日的学生
#### 50、查询下月过生日的学生