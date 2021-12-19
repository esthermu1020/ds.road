## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/esthermu1020/dspath/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/esthermu1020/dspath/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.

---

## SQL 考点总结 （刷题版）



#### 1. pivot



#### 2. window function

##### 1) 排序

###### - rank:

rank排序相同时会重复，总数不变，出现1, 1, 3

###### - dense_rank：

dense_rank排序相同时会重复，总数变，出现1, 1, 2

*因为会重复，通常需要伴随* `DISTINCT()`

###### - row_number:

row_number排序相同时不会重复，根据id顺序排序，总数不变

-----

<img src="/Users/dimu/Library/Application Support/typora-user-images/image-20211116103451203.png" alt="image-20211116103451203" style="zoom:25%;" />

```mysql
select *, row_number() over(order by salary) as `rank` from rownumber;
```

<img src="/Users/dimu/Library/Application Support/typora-user-images/image-20211116103602873.png" alt="image-20211116103602873" style="zoom:33%;" />

```mysql
select *, rank() over(order by salary) as `rank` from rownumber;
```

<img src="/Users/dimu/Library/Application Support/typora-user-images/image-20211116103711067.png" alt="image-20211116103711067" style="zoom:25%;" />



```mysql
select *, dense_rank() over(order by salary) as `rank` from rownumber;
```

<img src="/Users/dimu/Library/Application Support/typora-user-images/image-20211116103734026.png" alt="image-20211116103734026" style="zoom:25%;" />

###### - row_number的延伸问题

###### — 分组排序

```mysql
select *, row_number() over(partition by id order by salary) as `rank` from rownumber
```

`partition`的用法相当于`group by`，但是仅对窗口(`rank`)内的作用



###### — 每组内第二名的信息

```mysql
select * from (
      select *, 
      			row_number() over(partition by id order by salary) `rank` 
  		from rownumber) temp
where `rank` = 2;
```

###### — 排序+别的限制

```mysql
select *, row_number() over (order by salary) as `rank` from rownumber
where age between 13 and 16;
```

与`SELECT * FROM rownumber WHERE age BETWEEN 13 AND 16 ORDER BY salary ASC `

区分，这样的话并没有增加rank这一列



###### - NTH_VALUE

Nth-value返回有序行集中的第N行获取值，每一行的值都是从最开始到现在为止的这个**窗口** 里面的第N名的数据

- 没有分区`PARTITION BY`时候的例子

```sql
SELECT
		# 找到salary第二高的name
		# 这里没有partition by 所以是原来的dimension增加1列，
		# 这一列第一行因为没有第二高的返回null,
		# 其余所有行都返回第二名的name
    employee_name,
    salary,
    NTH_VALUE(employee_name, 2) OVER  (
        ORDER BY salary DESC
    ) second_highest_salary
FROM
    basic_pays;
```

- 有`PARTITION BY`分区的例子

```sql
SELECT
     employee_name,
     department,
     salary,
     NTH_VALUE(employee_name, 2) OVER  (
         PARTITION BY department
         ORDER BY salary DESC
       	 # 这里是说前面无限和后面无限，也就是所有的数据
         RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
     ) second_highest_salary
FROM
 basic_pays; 
```



- RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
- RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING



##### 2) DISINCT



using(id)  on [a.id](http://a.id) = b.id



##### **3) LEAD**

返回下面某个行（following row)

- Default: 如果不specify, 就返回

#### 3. Aggregate Function

##### ⚠️聚类函数在比较时要单独select

>  错：``where salary < max(salary)``
>
>  对：``where salary < (select max(salary) from xxx)``



⚠️聚类函数要用在having里面

#### 4. Do not use subquery!



#### 5. join, union

##### 1) cross join

##### 2) inner join

##### 3) left/right join

##### 4) self join

##### 5) union

- 重复记录是指查询中各个字段完全重复的记录

<img src="/Users/dimu/Library/Application Support/typora-user-images/image-20211122145534766.png" alt="image-20211122145534766" style="zoom:33%;" /> 

```mysql
SELECT aid,title FROM article UNION SELECT bid,title FROM blog
```

<img src="/Users/dimu/Library/Application Support/typora-user-images/image-20211122145613850.png" alt="image-20211122145613850" style="zoom:33%;" /> 

- 若 title 一样但 id 号不一样算作不同记录。

##### 6) union all

- 查询两张表中的文章 id 号及标题，并返回所有记录
- 显然，使用 UNION ALL 的时候，只是单纯的把各个查询<u>**组合**</u>到一起而不会去判断数据是否重复。因此，当确定查询结果中不会有重复数据或者不需要去掉重复数据的时候，应当使用 UNION ALL 以提高查询效率。

#### 6.各种summary stats

##### - rate:

- `COALESCE` 可以handle null的情况
- ROUND( *, 2) 管小数点
- 如果想要rate可以直接 (() as a) /(() as b) as rate

```sql
SELECT(
    COALESCE(ROUND(
            (SELECT COUNT(*) FROM (SELECT DISTINCT requester_id, accepter_id  FROM
                                   requestaccepted) a)
            /
            (SELECT COUNT(*) FROM (SELECT DISTINCT sender_id, send_to_id FROM 
                                   friendrequest) b)
            , 2), 0)
    ) accept_rate
```

##### - **cum-rate**:

- union all先合起来
- 加一个flag的列
- 使用sum(if(<condition>, <if_true>, <if_false>)) over(order by)
  - Sum()over(order by date)可以实现 cum sum by date的效果
  - 如果加了partition by <feature>就是先根据某个<feature>划分，然后再这个feature里cum sum by date

```mysql
select day, 
       sum(if(status='a', num, 0)) over(order by a.day) accept,
       sum(if(status='s', num, 0)) over(order by a.day) send,
       sum(if(status='a', num, 0)) over(order by a.day) /  
       sum(if(status='s', num, 0)) over(order by a.day) p
from
(    
  select count(distinct requester_id, accepter_id) num, 
  			 accept_date day, 
  		   'a' status
  from requestaccepted 
	group by day 
  union all
  (select count(distinct sender_id, send_to_id) num, 
   				request_date day, 
   				's' status
  from friendrequest
  group by day)
) a
```



#### 7. Date相关

##### - 现在：

- `CURDATE() = current_date()`: 现在的日期 `2021-11-28`
- `CURTIME()` 现在的时间 `15:05:26`
- `CURRENT_TIMESTAMP()` 现在的日期+时间  `2021-11-28 15:05:14`

##### - 截取：

- `DATE("2017-06-15 09:34:21")` 返回 `2017-06-15`
- `DAY("2017-06-15 09:34:21")` 返回 `15`
- `DAYNAME("2017-06-15 09:34:21")` 返回 `Thursday`

- `DAYOFMONTH("2017-06-15 09:34:21")` 返回 `15`
- `DAYOFWEEK("2017-06-15")` 返回 `5` ==星期天是1==
- `DAYOFYEAR('2017-06-15 09:34:21')` 返回 `166`
- `LASTDAY('2017-06-15')` 返回 `2017-06-30` 给定日期对应月份的最后一天

##### - 计算

- `DATEDIFF`, 前面的时间-后面的时间，返回日期差

  `SELECT DATEDIFF("2017-06-25 09:34:21", "2017-06-15 15:25:35")` -> `10`

- `DATE_ADD(<date>, INTERVAL <value> <add-unit>)` 对输入日期加某一个add-unit单位的value

  ````mysql
  # Add 15 minutes to a date and return the date:
  SELECT DATE_ADD("2017-06-15 09:34:21", INTERVAL 15 MINUTE);
  # Subtract 3 hours to a date and return the date:
  SELECT DATE_ADD("2017-06-15 09:34:21", INTERVAL -3 HOUR);
  # Subtract 2 months to a date and return the date:
  SELECT DATE_ADD("2017-06-15", INTERVAL -2 MONTH);
  ````

- `DATE_SUB(<date>, INTERVAL <value> <sub-unit>)` 对输入日期减一个sub-unit单位的value

  ```mysql
  # Subtract 10 days from a date and return the date:
  SELECT DATE_SUB("2017-06-15", INTERVAL 10 DAY);
  ```

  - calculate the average number of sessions/user per day for the last 30 days

    ```sql
    SELECT date, avg(ct)
    FROM
        (
        SELECT date, user_id, count(session_id) as ct
        FROM t
        # date在过去30天之中
        WHRE date BETWEEN subdate(current_date(),1) AND subdate(current_date(),30)
        GROUP BY date, user_id
        ) as temp
    GROUP BY date
    ORDER BY date
    ```

    

#### 8. update and deleting, replace, insert

- Update

  - 基本语法

  ```sql
  UPDATE table_name
  SET column1 = value1, column2 = value2...., columnN = valueN
  WHERE [condition];
  ```

  - 加上join

  ```sql
  UPDATE T1
  JOIN T2 ON T1.user_id = T2.user_id
  SET (
      CASE
      WHEN THEN
      END
      )
  ```

  

- Delete

  ```sql
  DELETE FROM table_name
  WHERE [condition];
  ```

- Replace

  ```mysql
  REPLACE [INTO] table_name(column_list)
  VALUES(value_list);
  
  REPLACE INTO table
  SET column1 = value1,
      column2 = value2;
  ```
  - Replace select from

  ```sql
  REPLACE INTO summary_table(name,song,tot_count)
  SELECT d_name, d_song, d_cnt + IFNULL(tot_count,0)
  FROM
      (
      SELECT 
          name AS d_name,
          song AS d_song,
          count(*) AS d_cnt 
      FROM today_table AS d 
      WHERE date ='12-26-2017' 
      GROUP BY name,song
      )
  LEFT JOIN
      (
      SELECT 
          name AS s_name,
          song AS s_song,
          tot_count 
      FROM summary_table AS s
      )
  ON s_name=d_name AND s_song=d_song;
  ```

  

- Insert

  ```mysql
  INSERT INTO table_name(column_list)
  VALUES(value_list);
  ```

  



#### Reference:

1. Leetcode SQL 刷完之后的考点汇总: https://www.1point3acres.com/bbs/thread-706788-1-1.html

2. Windows function, cumulative sum/rate https://blog.jooq.org/how-to-calculate-a-cumulative-percentage-in-sql/ 
3. SQL面经 https://lei-d.gitbook.io/sql/facebook-mian-jing-ti/spam-filter
