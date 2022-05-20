#### SQL初体验

###### SQL注释

- 单行注释 `--`or `#`
- 多行注释 `/* */`

###### 限定结果数量

- Top-N排行榜

  ```mysql
  select emp_name,salary
  from employee
  order by salary desc
  limit 5 offset 0;
  ```

  `offset` 表示跳过多少行记录，`limit`表示在`offset`的基础上获取前多少行记录。

- 数据分页显示

  ```mysql
  select emp_name,salary
  from employee
  order by salary desc
  limit 10 offset 10;
  ```

  `offset` 表示偏移量，`limit`表示单分页行数。

###### 中文的排序方式

mysql8.0默认使用`utf8mb4`字符编码，中文按照偏旁部首排序。我们可以通过转化函数`convert()`实现其他的中文排序方式。

- 实现中文拼音排序

  ```mysql
  select emp_name
  from employee
  where dept_id = 4
  order by convert(emp_name using GBK);
  ```

  `convert()`是mysql的一个系统函数，用于转换数据的字符集编码·，中文`GBK`字符集默认使用拼音进行排序。

###### 排除重复数据

- 单字段去重

  ```mysql
  select distinct sex
  from employee;
  ```

- 多字段去重

  ```mysql
  select distinct dept_id,sex
  from employee;
  ```

  以上，查找在员工表中不同部门编号和性别的所有组合。

###### 文本模糊查找（`LIKE`）

- 百分号（%），表示匹配零个或者多个任意字符。

- 下划线（_），表示匹配一个任意字符。

  ```mysql
  create table t_like(c1 varchar(200));
  insert into t_like(c1) values ('项目进度：25%已完成');
  insert into t_like(c1) values ('记录日期：2021年5月25日');
  
  select c1
  from t_like
  where c1 like '%25#%%' escape '#';
  -- where c1 not like '%25\%%';
  ```

  `escape`关键字为`like`运算符指定了一个`#`符号作为转移字符。

  `mysql`中不区分大小写。

  `not like` 执行的操作与`like`相反。

#### 函数与表达式

###### 数值函数

|  数值函数  |                           函数功能                           |
| :--------: | :----------------------------------------------------------: |
|   abs(x)   |                        计算x的绝对值                         |
|  ceil(x)   |                  返回大于或等于x的最小整数                   |
|  floor(x)  |                  返回小于或等于x的最小整数                   |
|  mod(x,y)  |                       计算x除以y的余数                       |
| round(x,n) |                     将x四舍五入到n位小数                     |
|   rand()   | 返回一个伪随机数，默认为[0,1)，且每次调用都会返回不同的随机数。 |

###### 字符函数

- 字符串的长度

  - `char_length(s)` 计算字符串s中的字符数量

  - `octet_length(s)` 计算字符串s中包含的字节数量

    ```mysql
    select char_length('数据库'),octet_length('数据库');
    => 3|9 -- 包含3个字符，在utf-8编码中占用 9 个字节
    -- UTF-8存储中文时占2~4个字节。utf-8是变长的、不定长的
    ```

- 连接字符串

  - `concat(s1,s2,...)`将多个字符串连接在一起，组成一个新的字符串。

  - `concat_ws(separator,s1,s2,...)` ,可以使用指定分隔符连接字符串

    ```mysql
    select concat('S','Q','L'),concat_ws('-,','S','Q','L');
    => SQL | S-Q-L -- 指定'-'进行分割。
    ```

- 大小写转化

  - `lower(s)`将字符串转化为小写形式

  - `upper(s)`将字符串转化为大写形式

    ```mysql
    select lower('SQL'),upper('sql')
    => sql|SQL
    ```

- 获取子串

  - `substr(s,n,m)`返回字符串s中从位置n开始的m个字符的子串。

  - `left(s,n)`返回字符串开头的n个字符。

  - `right(s,n) `返回字符串结尾的n个字符。

    ```mysql
    select substr('数据库',-2,2),left('数据库',0,1),right('数据库',0,1)
    => 据库|数|库
    ```

- 子串查找和替换

  - `instr(s,s1)`返回字符串s中子串s1第一次出现的位置，如果没有找到就返回0。

    ```mysql
    select instr('liubei@shuguo.com','@')
    => 7 -- '@'是'liubei@shuguo.com'的第7个字符。
    ```

- 截断字符串

  - `trim(s1 from s)`删除字符串s开头和结尾得子串s1。

  - `trim(s)`删除字符串s开头和结尾的空格

  - `ltrim(s)`删除字符串开头的空格

  - `rtrim(s)`删除字符串结尾的空格

    ```mysql
    select trim('-' from '--S-Q-L--'),trim('  S-Q-L  '),ltrim(' S-Q-L '),rtrim(' S-Q-L ')
    => S-Q-L|S-Q-L|S-Q-L | S-Q-L
    ```

###### 日期函数

- 返回当前日期和时间

  - `current_date`返回数据库当前的日期

  - `current_time`返回数据库当前的时间

  - `current_timestamp`返回数据库当前的时间戳（日期和时间）

    ```mysql
    select current_date,current_time,current_timestamp;
    => 2022-05-20|11:05:44|2022-05-20 11:05:44
    ```

- 提取日期中的部分信息

  - `extract(p from dt)`提取日期时间中的部分信息，比如`year、month、day、hour、minute、second`

    ```mysql
    select extract(year from hire_date)
    from employee
    where emp_id = 1;
    ```

- 日期的加减运算

  - `datediff(dt2,dt1)`计算日期`dt2`减去日期`dt1`得到的天数

  - `interval`,可以加上一个时间间隔

    ```mysql
    select datediff(date '2021-03-01',date '2021-02-01'),date '2021-02-01' + interval '-1' month;
    =>28|2021-01-01 00:00:00
    ```

###### 转化函数（cast）

- `cast(expr as type)` 用于将数据转换为其它类型

  ```mysql
  select cast('123' as integer)
  -- 将字符串'123'转化成整形
  -- integer分为有符号整型 `signed integer` 和无符号整型 `unsigned integer`
  ```

- 隐形自动转化

  ```mysql
  select '666'+123,concat('Hire_date:',hire_date)
  from employee
  where emp_id = 1
  -- 上面，字符转化为数字，日期转化为字符串
  ```

###### 条件表达式（case、if）

- `case`表达式

  ```mysql
  ### 简单case表达式
  select emp_name as '员工姓名',dept_id as '部门编号',
  case dept_id
  when 1 then '行政管理部'
  when 2 then '人力资源部'
  when 3 then '财政部'
  else '其它部门'
  end as '部门名称'
  from employee;
  --------------------------------------------------
  ### 一般case表达式
  select emp_name as '员工姓名',dept_id as '部门编号',
  case
  when dept_id = 1 then '行政管理部'
  when dept_id = 2 then '人力资源部'
  when dept_id = 3 then '财政部'
  else '其它部门'
  end as '部门名称'
  from employee;
  ---------------------------------------------------
  ### 判断过滤
  select emp_name as "员工姓名",salary as "月薪",
  case
  when salary < 10000 then '低收入'
  when salary < 20000 then '中收入'
  else '高收入'
  end as "收入级别"
  from employee;
  -------------------------------------------------
  ### 配合where,order by 等子句。
  select emp_name as '员工姓名',
  case
  when bonus is null then 0
  else bonus
  end as '奖金'
  from employee
  where dept_id = 2
  order by
  case -- 将bonus空值设为0，实现自定义排序。
  when bonus is null then 0
  else bonus
  end;
  ```

- `if`函数

  `if(expr1,expr2,expr3)`:如果表达式`expr1`为真（`！= 0 and is not null`）,返回表达式`expr2`的值，否则返回表达式`expr3`的值。

  ```mysql
  select if(1<2,'1<2','1>=2') as result;
  =>1<2
  ```

###### 聚合函数（count、avg等）

- `count`函数

  ```mysql
  select count(sex) as '所有性别',count(distinct sex) as '不同性别'
  from employee;
  =>25|2
  ```

- `avg`函数

  ```mysql
  select avg(case when bonus is null then 0 else bonus end) as '平均奖金'
  ```

- `max`和`min`函数

- `group_concat`函数实现字符串聚合操作。

  ```mysql
  select group_concat(email order by email separator ';') as '收件人'
  from employee
  where dept_id = 1;
  ```

#### 分组与空值

###### 数据分组

- 组内汇总

  可以结合使用`group by`子句和聚合函数，将数据进行分组，并在每个组内进行一次数据汇总。

  ```mysql
  select sex as '性别',count(*) as '员工数量',avg(salary) as '平均月薪'
  from employee
  group by sex;
  ```

- 空值为一组

  如果被分组的字段没有值，为`null`,那么它们都将被分到同一组中。

- 使用`having`过滤分组

  ```mysql
  select dept_id as '部门编号',avg(salary) as '平均月薪'
  from employee
  group by dept_id
  having avg(salary)>10000;
  ```

  >  从性能的角度来说，我们尽量使用`where`子句过滤更多的数据，而不是等到分组之后再通过`having`子句进行过滤。

- `rollup`,小计、合计与总计

  ```mysql
  select dept_id as '部门编号',sex as '性别',count(*) as '员工数量'
  from employee
  group by dept_id,sex with rollup;
  =>
  1|男|3
  1| |3
  2|女|2
  2| |2
  3|男|8
  3|女|1
  3| |9
   | |14
  ```

  > group by 子句的rollup选项表示先按照分组字段进行分组汇总，然后从右至左依次去掉一个分组字段再进行分组汇总，被去掉的字段显示为空。最后，将所有的数据进行一次汇总，所有分组字段都显示为空。

- `grouping` 函数

  > 使用`group by`子句的`rollup`扩展功能时，查询会产生一些空值信息。这些空值意味着对应的记录是针对这个字段所有数据的汇总，我们可以利用`grouping`函数识别这些空值数据。

  ```mysql
  select
  case grouping(sex) when 1 then '所有性别' else sex end as '性别',count(*) as '员工数量'
  from employee
  group by sex with rollup;
  =>
  性别|员工数量
  ----------
  女|3
  男|22
  所有性别|25
  ```

  >其中，`grouping(sex)`函数返回`0`,表示当前记录不是所有性别的汇总数据；
  >
  >返回1，表示当前记录是所有性别的汇总数据。

###### 空值问题

- 三值逻辑

  `SQL`中逻辑运算的结果存在三种情况：真、假或者未知。

- 空值比较

  - 任何数据和空值进行算术比较的结果都是==未知的==。

    ```mysql
    null = 0
    null != 0
    null = ''
    null = null
    ```

  - `<=>`运算符，支持等值比较和空值比较。

    ```mysql
    select 1 <=> 1 , 1 <=> null , null <=> null;
    =>
    1|0|1
    ```

- 空值查询

  - `in`

    ```mysql
    select emp_id
    from employee
    where emp_id in (1,2,3,null); -- where emp_id  = 1 or emp_id = 2 or emp_id = 3 or emp_id = null;
    =>
    emp_id
    ------
    1
    2
    3
    ```

  - `not in`

    ```mysql
    select emp_id
    from employee
    where emp_id not in (1,2,3,null); -- where emp_id  != 1 and emp_id != 2 and emp_id != 3 and emp_id != null;
    =>
    emp_id
    ------
    null
    ```

    > 上面查询是有问题的，结果不会返回任何结果。
    >
    > 问题就在于`emp_id!=null`,它的结果是未知的，也就意味着没有任何数据满足查询条件，也就不会返回任何结果。

- 函数中的空值

  > 当表达式或函数的参数中存在空值时，返回结果也是空值

  ```mysql
  select 100 + null,upper(null)
  =>
  null|null
  ```

- 空值处理函数

  - `coalesce`函数

    `coalesce(expr1,expr2,expr3,...)`函数接收一个输入参数的列表，返回第一个非空的参数，如果都为空，则返回null

    ```mysql
    select emp_name as '员工姓名',
    salary*12 + coalesce(bonus,0) as '全年收入'
    from employee
    ```

  - `nullif`函数

    `nullif(expr1,expr2)`函数接收两个输入参数，当相等时返回`null`,否则返回第一个参数。

    ```mysql
    select nullif(1,2),nullif(2,2)
    =>
    1|null
    ```

    `nullif`函数的一个常见的用处是防止除零错误

    ```mysql
    -- 除0错误
    select *
    from employee
    where 1/0 = 1;
    -- 避免
    select *
    from employee
    where 1/nullif(0,0) = 1; # 1除以空值为空值，不会产生错误
    ```

- 空值与约束

  - 非空约束

    在设计表时，如果不允许该字段为空

    ```mysql
    create table t_notnull (v varchar(10) not null);
    ```

  - 唯一约束（unique）

    唯一约束允许数据存在空值，而且多个空值被看作不同的值。

    ```mysql
    create table t_unique (id int unique);
    insert into t_unique values(1);
    insert into t_unique values(null);
    insert into t_unique values(null);
    ```

  - 检查约束（check）

    检查约束对于空值的处理正好和`where`相反，只要检查结果不为`False`,就可以插入或更新数据。

    ```mysql
    create table t_check(
       c1 int check (c1>=0),
       c2 int check (c2>=0),
       check(c1+c2<=100)
    )
    -----------------------
    insert into t_check values (5,5) # 正常
    insert into t_check values (null,null) # 空值不为false,没有违反检查约束
    insert into t_check values (1000,null) # 与null的和运算，值为null。
    
    select * from t_check;
    =>
    c1|c2
    -----
    1|1
    null|null
    1000|null
    ```

#### 连接与集合运算

###### 内连接

![sql-note_内连接_表](F:\云笔记\cloud-note\picture\sql-note_内连接_表.jpg)

- 等值连接

  ```mysql
  select e.emp_name as '员工姓名',j.job_title "职位名称"
  from 
  employee e join job j 
  on (e.job_id = j.job_id)
  ```

- 非等值连接

  ```mysql
  select e.emp_name as '员工姓名',e.salary '月薪'
  from 
  employee e join job j
  on (e.job_id != j.job_id and e.salary between j.min_salary and j.max_salary)
  ```

###### 外连接

- 左连接

  ![sql-note_左连接_表](F:\云笔记\cloud-note\picture\sql-note_左连接_表.jpg)

  

- 右连接

  ![sql-note_右连接_表](F:\云笔记\cloud-note\picture\sql-note_右连接_表.jpg)

- 全外连接

  ![sql-note_全外连接_表](F:\云笔记\cloud-note\picture\sql-note_全外连接_表.jpg)

###### 其它连接

- 交叉连接

  也称笛卡尔积，如果表的数据量比较大，会导致查询结果的数据量急剧膨胀。

  ```mysql
  select count(*)
  from employee e
  cross join department d;
  ```

- 自然连接

  如果连接条件是等值连接，并且两个表中的连接字段名称相同、类型相同，可以使用`using`代替`on`来简化连接条件。

  ```mysql
  select e.emp_name as '员工姓名',j.job_title as '职位名称'
  from employee e
  join job j using(job_id)
  --------------------------
  select e.emp_name as '员工姓名',j.job_title as '职位名称'
  from employee e
  natural join job j # natural join 表示自然连接，我们同时省略了连接条件，表示使用两个表的所有同名同类型字段进行等值连接。
  ```

- 多张表的连接

  ```mysql
  select e.emp_id as '员工编号',e.emp_name as '员工姓名',d.dept_name as '部门名称',j.job_title as '职位名称'
  from employee e
  join department d on(d.dept_id = e.dept_id)
  join job j on(j.job_id = e.job_id)
  ```

###### 交集求同

- 交集运算可以改写为等价的内连接查询

  ```mysql
  select id,name
  from t_set1
  intersect
  select id,name
  from t_set2;
  ---------------
  select t1.id,t1.name
  from t_set1 t1
  join t_set2 t2
  on (t2.id = t1.id and t2.name = t1.name)
  ```

###### 并集存异

- 可以改写为等价的全外连接查询

  ```mysql
  select id,name
  from t_set1
  union
  select id,name
  from t_set2;
  ---------------
  select t1.id,t1.name
  from t_set1 t1
  full join t_set2 t2
  on (t2.id = t1.id and t2.name = t1.name)
  ```

  

#### 嵌套子查询

###### 查询中的查询 

- 标量子查询，返回单个值（一行一列）的子查询。

  ```mysql
  select emp_name as '员工姓名',salary as '月薪',
  salary - (select avg(salary) from employee) as '差值'
  from employee;
  ```

- 行子查询，返回单个记录（一行多列）的子查询。

  ```mysql
  select emp_name,dept_id,job_id
  from employee
  where (dept_id,job_id) = 
  (
     select dept_id,job_id from employee where emp_name = '张三'
  )
  ```

- 表子查询，返回一个临时表（多行多列）的子查询。

  ```mysql
  select emp_name,
  from employee
  where job_id in
  (
     select job_id from employee where dept_id = '1'
  );
  ```

###### all、any运算符

- `all、any`运算符与比较运算符（=、！=、<、<=、>）的组合分别表示等于、不等于、小于等子查询结果中所有的全部数据。

  ```mysql
  select emp_name as '员工姓名',hire_date as '入职日期'
  from employee
  where job_id >all
  (
     select e.hire_date from employee e where e.dept_name = '研发部'
  );
  ```

###### 关联子查询

- 这类子查询在执行时，需要使用外部查询中的字段值

  ```mysql
  select d.dept_name as '部门名称',
  (
     select avg(salary) as avg_salary
     from employee
     where dept_id = d.dept_id # 使用了外部的d.dept_id
  ) as "平均月薪"
  from department d
  ```

> 对于外部查询返回的每条记录，关联查询都会执行一次（数据库可能会对此进行优化），非关联子查询只会独立执行一次。

###### 横向子查询

- 关联子查询可以使用外部查询中的字段，但不能使用同一级别的其它查询或者表中的字段。

  ```mysql
  select d.dept_name,t.max_salary
  from department d
  join
  (
     select max(e.salary) as max_salary
     from employee e
     where e.dept_id = d.dept_id
  ) t;
  ```

- 横向子查询（`lateral`）,允许派生表使用它所在的`from`子句中的左侧的其它查询或者表

  ```mysql
  select d.dept_name,t.max_salary
  from department d
  lateral join
  (
     select max(e.salary) as max_salary
     from employee e
     where e.dept_id = d.dept_id
  ) t;
  ```

###### exists 运算符

​	用于判断子查询结果的存在性。只要子查询返回了任何结果（包括`null`），就表示满足查询条件

