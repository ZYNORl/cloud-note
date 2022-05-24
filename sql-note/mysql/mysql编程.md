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

![sql-note_内连接_表](https://raw.githubusercontent.com/ZYNORl/cloud-note/main/picture/sql-note_内连接_表.jpg)

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

  ![sql-note_左连接_表](https://raw.githubusercontent.com/ZYNORl/cloud-note/main/picture/sql-note_左连接_表.jpg)

  

- 右连接

  ![sql-note_右连接_表](https://raw.githubusercontent.com/ZYNORl/cloud-note/main/picture/sql-note_右连接_表.jpg)

- 全外连接

  ![sql-note_全外连接_表](https://raw.githubusercontent.com/ZYNORl/cloud-note/main/picture/sql-note_全外连接_表.jpg)

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

#### 通用表达式

###### 表即变量（with）

> 变量可以被重复使用；函数可以将代码模块化。与此类似，SQL中的通用表达式（简称CTE）也能够实现查询结果的重复利用，简化复杂的连接查询和子查询。```

- `SQL`通用表达式的基本语法如下：

  ```mysql
  with cte_name(col1,col2,...) as (
     subquery
  )
  select * from cte_name;
  ```

  1. `cte_name`指定了`CTE`的名称，后面是可选的字段名。
  2. `as`关键字后面的子查询是`CTE`的定义语句，定义了它的表结构和数据。
  3. 最后`select`是主查询语句，它可以引用前面定义的`CTE`。
  4. 除了`select`，主查询语句也可以是`insert、update、delete`。

- `CTE`示例如下

  ```mysql
  with dept_cost(dept_name,total) as
  (
     select dept_name,sum(salary*12+coalesce(bonus,0))
     from employee e
     join department d on (e.dept_id = d.dept_id)
     group by dept_name
  ),dept_avg_cost(avg_total) as
  (
     select sum(total)/count(*) avg_total
     from dept_cost
  )
  select dept_cost.dept_name as '部门名称',dept_cost.total as '总成本',dept_avg_cost.avg_total as '平均成本'
  from dept_cost
  join dept_avg_cost
  on (dept_cost.total > dept_avg_cost.avg_total);
  ```

  1. `dept_cost`是一个`CTE`，包含了每个部门的名称和成本。
  2. `dept_avg_cost`也是一个`CTE`,引用前面定义的`dept_cost`
  3. 最后在主查询，引用。

###### 强大的递归（with recursive）

- 递归`CTE`的基本语法如下:

  ```mysql
  with recursive cte_name as
  (
     cte_query_initial -- 初始化部分,用于创建初始结果集
     union [all] -- 最后，用此进行所有递归查询的合并
     cte_query_iterative -- 递归部分，可以对`CTE`进行自我引用。每一次递归查询语句执行的结果都会作为输入，传递给下一次查询。
  ) select * from cte_name
  ```

- 案例一：生成数字序列

  >以下查询通过递归`CTE`生成1~10的数字序列。

  ```mysql
  with recursive t(n) as
  (
     select 1 -- 初始化语句生成1，并作为递归部分的输入。
     union all
     select n+1 from t where n < 10 -- from t: 调用自身；where: 递归结束条件。
  )
  select n from t; -- 调用递归函数。
  ```

- 案例二：遍历层次结构

  > 利用递归`CTE`生成一个组织结构网，显示每个员工从上到下的管理路劲：

  ```mysql
  with recursive employee_path(emp_id,emp_name,path) as
  (
     select emp_id,emp_name,emp_name as path
     from employee
     where manager is null -- 找到树的根
     union all
     select e.emp_id,e.emp_name,concat(ep.path,'->',e.emp_name) -- 连接上级
     from employee_path ep
     join employee e
     on ep.emp_id = e.manager -- 找到下属
  )
  select emp_name as '员工姓名',path as '管理路径'
  form employee_path
  order by emp_id;
  ```

#### 窗口函数

> 窗口函数可以像聚合函数一样对一组数据进行分析并返回结果，二者的不同之处在于，窗口函数不是将一组数据汇总到的单个结果，而是为每一行数据都返回一个结果。聚合函数和窗口函数的区别如下：
>
> ![sql-note_聚合和窗口的区别](https://raw.githubusercontent.com/ZYNORl/cloud-note/main/picture/sql-note_聚合和窗口的区别.jpg)

###### 窗口函数的定义

- 函数初探

  ```mysql
  select sum(salary) as '月薪总和'
  from employee
  =>
  月薪总合
  ------
  245800
  select emp_name as '员工姓名',sum(salary) over() as '月薪总和' -- 关键字`over`表明sum是一个窗口函数。
  from employee
  =>
  员工姓名|月薪总合
  ------|------
  刘备|245800
  关羽|245800
  ...
  ```

- 创建数据分区

  >窗口函数`over`子句中的`partition by`选项用于定义分区，其作用类似于查询语句中的`group by`子句。

  ```mysql
  select emp_name '员工姓名',salary '月薪',dept_id '部门编号',
  sum(salary) over(partition by dept_id) as '部门合计'
  from employee;
  =>
  员工姓名|月薪|部门编号|部门合计
  ------|-----|------|------
  刘备|30000|1|80000
  关羽|26000|1|80000
  ...
  ```

- 分区内的排序

  >窗口函数`over`子句中的`order by`选项用于指定==分区内==数据的排序方式，作用类似于查询语句中的`order by`。

  ```mysql
  select emp_name '姓名',salary '月薪',dept_id '部门编号',
  rank() over(partition by dept_id order by salary desc) as '部门排名'
  from employee;
  =>
  姓名|月薪|部门编号|部门排名
  ---|---|--------|-------
  刘备|30000|1|1
  关羽|26000|1|2
  ...
  ```

###### 指定窗口大小

> 窗口函数`over`子句中的`frame_clause`选项用于指定一个移动的分析窗口，窗口总是位于分区的范围之内，是分区的一个子集。在指定了分析窗口之后，窗口函数不再基于分区进行分析，而是基于窗口大小内的数据进行分析。

- `frame_start`选项用于定义窗口的起始位置
  - `unbounded preceding`: 表示窗口从分区的第一行开始。
  - `n preceding`: 表示窗口从当前行之前的第`N`行开始。
  - `current row`：表示窗口从当前行开始。
- `frame_end`选项用于定义窗口的结束位置
  - `current row`：表示窗口从当前行结束。
  - `m following`: 表示窗口到当前行之后的第M行结束。
  - `unbounded following`: 表示窗口从分区的最后一行结束。

###### 窗口函数分类

- 聚合窗口函数

  >许多常见的聚合函数也可以作为窗口函数使用，包括`avg()、sum()、count()、max()以及min()函数`。

  下面以`avg`函数为案例，用于查询不同产品每个月以及截止当前月最近三个月的平均销售额。

  ```mysql
  select product as '产品',ym '年月',amount '销售额',
  avg(amount) over(
     partition by product
     order by ym
     rows between 2 preceding and current row
  ) as '最近平均销售额'
  from sales_monthly
  order by product,ym;
  ```

- 排名窗口函数

  > 排名窗口函数用于对数据进行分组排名，包括`row_number()、rank()、dense_rank()、precent_rank()`等函数。

  以下查询使用4个不同的排名函数计算每个员工再其部门内的月薪排名：

  ```mysql
  select d.dept_name as '部门名称',e.emp_name as '姓名',e.salary as '月薪',
  row_number()
  over(partition by e.dept_id order by e.salary desc) as 'row_number',
  rank()
  over(partition by e.dept_id order by e.salary desc) as 'rank',
  dense_rank()
  over(partition by e.dept_id order by e.salary desc) as 'dense_rank',
  precent_rank()
  over(partition by e.dept_id order by e.salary desc) as 'precent_rank',
  from employee e
  join department d on (e.dept_id = d.dept_id);
  
  =>
  
  部门名称|姓名|月薪|row_number|rank|dense_rank|precent_rank
  ...
  研发部|哈哈|6500.00|6|6|6|0.65
  研发部|嘿嘿|6500.00|7|6|6|0.65
  研发部|哇哇|6000.00|8|8|7|0.875
  ... 
  ```

- 取值窗口函数

  - `lag`函数可以返回窗口内当前行之前的第`N`行数据。

  - `lead`函数可以返回窗口内当前行之后的第`N`行数据。

  - `first_value`函数可以返回窗口内第一行数据。

  - `last_value`函数可以返回窗口内最后一行数据。

  - `nth_value`函数可以返回窗口内第`N`行数据。

  - 案例分析，统计各个产品每个月相比去年同月份的同比增长率。

    ```mysql
    select product as '产品',ym  as '年月',amount '销售额',
    ((amount-lag(amount,12) over(parition by product order by ym))/lag(amount,12) over(parition by product order by ym))*100
    as '同比增长率'
    from sales_monthly
    order by product,ym;
    ```

#### 数据的增删改

###### 插入数据

- 插入单行数据

  ```mysql
  insert into t(col1,col2,...)
  values (value1,value2,...);
  ```

- 插入多行数据

  > `insert into ...values ...`语句支持一次插入多行记录。

  ```
  insert into t(col1,col2,...)
  values (value1,value2,...),
  (value1,value2,...),
  ,...,
  values (value1,value2,...);
  ```

- 复制数据

  下面，将‘研发部’的员工信息复制到`emp_devp`表中：

  ```mysql
  insert into emp_devp(emp_id,emp_name,sex,dept_id,manager,hire_date,job_id,salary,bonus,email)
  select e.emp_id,e.emp_name,e.sex,e.dept_id,e.manager,e.hire_date,e.job_id,e.salary,e.bonus,e.email
  from employee e
  join department d on (d.dept_id = e.dept_id)
  where dept_name = '研发部';
  ```

###### 更新数据

- `update`语句的基本语法如下：

  ```mysql
  update t
  set col1 = expr1,
  col2 = expr2,
  ...
  [where condition];
  -----------------
  update emp_devp
  set salary = salary + 1000,
  bonus = 8000
  where emp_name = '赵云';
  ```

- 关联（join）更新

  ```mysql
  update emp_devp ed
  join employee e on (e.emp_id = ed.emp_id)
  set ed.salary =  e.salary,
  ed.bonus = e.bonus,
  ed.email = e.email;
  ```

###### 删除数据

- 单表删除

  ```mysql
  delete
  from t
  [where condition];
  ---------------------
  delete
  from employee
  where emp_name in ('张三','李四','王五');
  ```

- 关联删除

  案例分析，删除`emp_devp`表中`emp_id`出现在员工表中的所有数据。

  ```mysql
  delete
  from emp_devp
  where emp_id in (select emp_id from employee);
  ----------------------------------------------
  delete ed
  from emp_devp ed
  join employee e on (ed.emp_id = e.emp_id);
  ```

###### 快速删除全表数据

>如果我们想要删除表中的全部数据，数据量比较少时可以直接使用`delete`语句，但是，这种方式对于数据量很大的表所需的时间比较长。

- 快速删除表中全部数据的`truncate`语句。

  ```mysql
  truncate table emp_devp;
  ```

- `delete`语句通过`where`子句删除指定的数据行。如果不指定过滤条件，将会删除所有的数据。`delete`属于数据操作语言（DML），删除数据后可以选择提交或者回滚。如果删除的数据较多，则其执行速度比较慢。

- `Truncate`语句用于快速删除表中的所有数据，并且释放表的存储空间。`truncate`属于数据定义语言（DDL）,删除数据时默认提交，无法回滚。`truncate`语句相当于删除并重建表，通常其执行速度很快。

###### 外键约束与级联查询

- 违反外键约束

  > 外键约束可以防止由于我们的误操作而导致数据不一致，从而维护数据的完整性。

  ```mysql
  -- 违反外键约束，职位不存在
  update employee
  set job_id = 11  # update语句将员工的职位设置为一个不存在的职位，违反了外键约束。
  where emp_id = 1; 
  -----------------
  -- 违反外键约束，存在子记录
  delete
  from job # delete语句删除了一个职位，同样会到导致员工表中的记录指向一个不存在的职位。
  where job_id = 1;
  ```

- 级联更新和删除

  > - 子表中的外键约束引用的都是父表的主键字段，主键字段通常无须进行更新，或者我们应该避免使用可能被更新的字段作为主键。
  > - 如果需要更新父表中的主键，或者删除父表中的记录，应该同时对子表中的数据进行更新或者删除。为了方便这一操作，数据库提供了级联更新和级联删除的功能。

  ```mysql
  create table t_parent(id integer primary key);
  insert into t_parent(id) values (1);
  --
  create table t_child(
     id integer primary key,
     pid integer not null,
     constraint fk1 foreign key (pid) references t_parent(id)
     on update cascade 
     on delete cascade
  );
  insert into t_child(id,pid) values(1,1);
  insert into t_child(id,pid) values(2,1);
  ------------------------------------------
  update t_parent
  set id  = 3
  where id = 1;
  select * from t_child;
  id|pid
  --|--
  1|3
  2|3
  ------------------------------------------
  delete
  from t_parent
  set id  = 3;
  select * from t_child;
  id|pid
  --|--
  null|null
  ```

  - `no action`=`restrict`-- 如果父表上的`update`或者`delete`语句违反了外键约束，则返回错误，数据库在语句执行时立即检查是否违反约束。
  - `cascade`--  如果父表上执行`update`或者`delete`语句，级联更新或者删除子表上的记录。
  - `set null`--  如果父表上执行`update`或者`delete`语句,将子表的外键字段设置为`null`。

#### 视图

> 视图和子查询和通用表表达式类似，都可以当作查询的数据源。但是，视图时存储在数据库中的对象，可以被重复使用。
>
> 建议不要再视图上使用`order by`,因为在视图上的查询不一定最终结果，会影响性能。

###### 创建视图

```mysql
create view view_name
as
select_statement;
----------------
create view developers
as
select emp_id,emp_name
from employee
where dept_id = 4;
-----------------
select *
from developers -- 创建视图之后，我们可以像普通表一样将其作为查询的数据源。
where sex = '女'；
```

###### 删除视图

```mysql
drop view view_name;
```

