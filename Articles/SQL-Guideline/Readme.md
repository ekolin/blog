# 半小时学会SQL

本篇文章是 **SQL 必知必会** 的读书笔记，SQL必知必会的英文名叫做 *Sams Teach Yourself in 10 Minutes*。但是，我肯定是不能够在10分钟就能学会本书所有涉及到的sql，所以就起个名字叫30分钟学会SQL语句（其实半个小时也没有学会...）。

目前手边的数据库是 mysql，所以以下示例均是由 mysql 演示。由于现在大部分工具都支持语法高亮，所以以下关键字都使用小写。

## 准备

### 工具

[mycli](https://github.com/dbcli/mycli)，一个终端工具，支持语法高亮，自动补全，多行模式，并且如果你熟悉vi的话，可以使用vi-mode快速移动，编辑。总之，vi + mycli 简直是神器！

同样，`postgreSQL` 可以使用[pgcli](https://github.com/dbcli/pgcli)。

``` shell
pip install -U pgcli, pgcli     # 默认你已经安装了pip
```

### 样例表

示例中有两个表，分为 student 学生表与 class 班级表。student 表中有 class_id 关联 class 表。以下是两个表数据的 sql。另外，**最后有三道小练习题会用到样例表**。

``` sql
create table class (
  id int(11) not null auto_increment,
  name varchar(50) not null,
  primary key (id)
);

create table student (
  id int(11) not null auto_increment,
  name varchar(50) not null,
  age smallint default 20,
  sex enum('male', 'famale'),
  score tinyint comment '入学成绩',
  class_id int(11),
  createTime timestamp default current_timestamp,
  primary key (id),
  foreign key (class_id) references class (id)
);

insert into class (name) values ('软件工程'), ('市场营销');

insert into student (name, age, sex, score, class_id) values ('张三', 21, 'male', 100, 1);
insert into student (name, age, sex, score, class_id) values ('李四', 22, 'male', 98, 1);
insert into student (name, age, sex, score, class_id) values ('王五', 22, 'male', 99, 1);
insert into student (name, age, sex, score, class_id) values ('燕七', 21, 'famale', 34, 2);
insert into student (name, age, sex, score, class_id) values ('林仙儿', 23, 'famale', 78, 2);
```

## SQL 基础

### 术语

+ Database

  数据库值一系列有关联数据的集合，而操作和管理这些数据的是DBMS，包括MySQL，PostgreSQL，MongoDB，Oracle，SQLite等等。

+ Table

  具有特定属性的结构化文件。比如学生表，学生属性有学号，年龄，性别等。schema (模式) 用来描述这些信息。

+ Column

  表中的特定属性，如学生的学号，年龄。每一列都具有数据类型。

+ Data Type

  每一列都具有数据类型，如 varchar，int，text，datetime，timestamp。

+ Row

  数据表的每一行记录。如学生张三。

### 检索数据

``` sql
# 检索单列
select name from student;

# 检索多列
select name, age, class from student;

# 检索所有列
select * from student;

# 对某列去重
select distinct class from student;

# 检索列-选择区间
# offset 基数为0，所以 `offset 1` 代表从第2行开始
select * from student limit 1, 10;
select * from student limit 10 offset 1;
```

### 排序

默认排序是 `ASC`，所以一般升序的时候不需指定，降序的关键字是 `DESC`。

``` sql
# 根据学号降序排列
select * from student order by number desc;
```

### 数据过滤


``` sql
# 找到学号为1的学生
select * from student where number = 1;

# 找到学号为在 (1, 10) 的学生
select * from student where number between 1 and 10;

# 找到未设置电子邮箱的学生
select * from student where email is null;

# 找到一班中大于23岁的学生
select * from student where class_id = 1 and age > 23;

# 找到一班或者大于23岁的学生
select * from student where class_id = 1 or age > 22;

# 找到一班与二班的学生
select * from student where class_id in (1, 2);

# 找到不是一班二班的学生
select * from student where class_id not in (1, 2);
```

### 计算字段

+ CONCAT

  ``` sql
  select concat(name, '(', age, ')') as nameWithAge from student;

  select concat('hello', 'world') as helloworld;
  ```

+ Math

  ``` sql
  select age - 18 as relativeAge from student;

  select 3 * 4 as n;
  ```

### 数据汇总

聚集函数，一些对数据进行汇总的函数，常见有 `COUNT`，`MIN`，`MAX`，`AVG`，`SUM` 五种。

``` sql
# 统计1班人数
select count(*) from student where class_id = 1;
```

### 数据分组

``` sql
# 按照班级进行分组
select class_id, count(*) from student group by class_id;

# 列出每个班级的学生 (mysql)
select class_id, group_concat(name) from student group by class_id;
```

### 子查询

``` sql
# 列出软件工程班级中的学生
select * from student where class_id in (
  select id from class where class_id = '软件工程'
);
```

### 联接

虽然两个表拥有公共字段便可以创建联接，但是使用外键可以更好地保证数据完整性。比如当对一个学生插入一条不存在的班级的时候，便会插入失败。

``` sql
# 列出软件工程班级中的学生
select * from student, class where student.class_id = class.id and class.name = '软件工程';
```

+ 内联接

  ``` sql
  # 列出软件工程班级中的学生
  select * from student inner join class on student.class_id = class.id where class.name = '软件工程';
  ```

+ 自联接

  ``` sql
  # 列出与张三同一班级的学生
  select * from student inner join class on student.class_id = class.id where class.name = '软件工程';
  ```

+ 外联接

  ``` sql
  # 列出与张三同一班级的学生
  select * from student left join class on student.class_id = class.id where class.name = '软件工程';
  ```

### 插入数据

可以采用以下方法插入一条数据，不过严重依赖表中列的顺序关系，推荐指定列名插入数据，并且可以插入部分列。

``` sql
# 插入一条数据
insert into student values(8, '陆小凤', 24, 1, 3);

insert into student(name, age, sex, class_id) values(9, '花无缺', 25, 1, 3);
```

### 修改数据

+ 更新

  ``` sql
  # 修改张三的班级
  update student set class_id = 2 where name = '张三';
  ```

+ 删除

  ``` sql
  # 删除张三的数据
  delete from student where name = '张三';

  # 删除表中所有数据
  delete from student;

  # 更快地删除表中所有数据
  truncate table student;
  ```

### 创建表与更新表

``` sql
# 创建学生表
create table student (
  id int(11) auto_increment,
  name varchar(50) not_null unique key,
  age smallint,
  sex tinyint,
  class_id int(11) foreign key refereces class(id)
)

# 根据旧表创建新表
create table student_copy as select * from student;

# 删除 age 列
alter table student drop column age;

# 添加 age 列
alter table student add column age smallint;

# 删除学生表
drop table student;
```

### 视图

视图是一种虚拟的表，便于更好地在多个表中检索数据，视图也可以作写操作，不过最好作为只读。在需要多个表联接的时候可以使用视图。

``` sql
create view v_student_with_classname as
select student.name name, class.name class_name from student where student.class_id = class.id;

select * from v_student_with_classname; 
```

### 约束

+ primiry key

  任意两行绝对没有相同的主键，且任一行不会有两个主键且主键绝不为空。使用主键可以加快索引。

  ``` sql
  alter table student add constraint primary key (id);
  ```

+ foreign key

  外键可以保证数据的完整性。有以下两种情况。
  + 插入5班的张三丰到student表中会失败，因为5班不存在。
  + 删除3班会失败，因为陆小凤和楚留香还在3班。

  ``` sql
  alter table student add constraint foreign key (class_id) references class (id);
  ```

+ unique key

  唯一索引保证该列值是唯一的，但可以允许有null。

  ``` sql
  alter table student add constraint unique key (name);
  ```

+ check

  检查约束可以使列满足特定的条件，如果学生表中所有的人的年龄都应该大于0。

  > 不过很可惜mysql不支持，可以使用**触发器**代替

  ``` sql
  alter table student add constraint check (age > 0);
  ```

+ index

  索引可以更快地检索数据，但是降低了更新操作的性能。

  ``` sql
  create index index_on_student_name on student (name);
  ```

### 触发器

可以在插入，更新，删除行的时候触发事件。

``` sql
# 创建触发器
# 比如mysql中没有check约束，可以使用创建触发器，当插入数据小于0时，置为0。
create trigger reset_age before insert on student for each row
begin
  if NEW.age < 0 then
    set NEW.age = 0;
  end if;
end;

# 打印触发器列表
show triggers;
```

### 存储过程

存储过程可以视为一个函数，根据输入执行一系列的 sql 语句。存储过程也可以看做对一系列数据库操作的封装，一定程度上可以提高数据库的安全性。

``` sql
# 创建存储过程
create procedure create_student(name varchar(50))
begin
  insert into students(name) values (name);
end;

# 调用存储过程
call create_student('shanyue');
```

## SQL 练习

更多练习可以查看 [leetcode](https://leetcode.com/problemset/database/)

### 1. 根据班级学生的分数进行排名，如果分数相等则为同一名次

``` sql
select id, name, score, (select count(distinct score) from student s2 where s2.score >= s1.score) as rank
from student s1 order by s1.score desc;
```

> 在where以及排序中经常用到的字段需要添加Btree索引，因此 score 上可以添加索引。

*Result:*

| id | name   | score | rank |
|----|--------|-------|------|
| 1  | 张三   | 100   | 1    |
| 3  | 王五   | 99    | 2    |
| 2  | 李四   | 98    | 3    |
| 5  | 林仙儿 | 78    | 4    |
| 4  | 燕七   | 34    | 5    |

[leetcode: rank-scores](https://leetcode.com/problems/rank-scores/description/)

### 2. 写一个函数，获取第 N 高的分数

``` sql
create function getNthHighestScore(N int) return int
begin
  declare M int default N-1;
  return (
    select distinct score from student order by score desc limit M, 1;
  )
end;

select getNthHighestScore(2);
```

*Result:*

| getNthHighestScore(2) |
| --------------------- |
| 99                    |

[leetcode: nth highset salary](https://leetcode.com/problems/nth-highest-salary/description/)

###　3. 检索每个班级的前两名学生

``` sql
select class.name as class_name, s.name as student_name, score, rank
from (
  select *, (
    select count(distinct score) from student s2 where s2.score >= s1.score and s2.class_id = s1.class_id
  ) as rank from student s1
) as s left join class on s.class_id = class.id where rank <= 2;
```

*Result:*

| class_name | student_name | score | rank |
|------------|--------------|-------|------|
| 软件工程   | 张三         | 100   | 1    |
| 软件工程   | 王五         | 99    | 2    |
| 市场营销   | 燕七         | 34    | 2    |
| 市场营销   | 林仙儿       | 78    | 1    |

## FAQ

### 1. `inner join` 与 `outer join` 的区别是什么

见上

### 2. 如何根据一个表的数据更新另一个表

比如以上 `student` 表保存着成绩，另有一表 `score_correct` 内存因失误而需修改的学生成绩。

在mysql中，可以使用如下语法

``` sql
update student, score_correct set student.score = score_correct.score where student.id = score_correct.uid;
```

### 3. 索引是如何工作的

简单来说，索引分为 `hash` 和　`btree` 两种。
`hash` 是一种索引时间复杂度为O(1)的数据结构，`btree` 是一种子平衡多叉搜索数，因此适合范围查找以及排序，不过只能搜索最左前缀，如只能索引以`a`开头的姓名，却无法索引以`a`结尾的姓名。

[how dow database indexing work](https://stackoverflow.com/questions/1108/how-does-database-indexing-work)

### 4. 如何联接多个行的字段

[Concatenate many rows into a single text string](https://stackoverflow.com/questions/194852/concatenate-many-rows-into-a-single-text-string)

在mysql中，可以使用`group_concat`

``` sql
select group_concat(name) from student;
```

### 5. 如何在一个sql语句中插入多行数据

[Inserting multiple rows in a single SQL query](https://stackoverflow.com/questions/452859/inserting-multiple-rows-in-a-single-sql-query)

values 使用逗号相隔，可以插入多行数据

``` sql
insert into student(id, name) values (), (), ()
```

### 6. 如何在`select`中使用条件表达式

示例，在student表中，查询所有人成绩，小于60则显示为0

``` sql
select id, name, if(score < 60, 0, score) score from student;
```

### 7. 如何找到重复项

``` sql
select name, sex, count(*) times from student
group by name, sex
having times > 1;
```

### 8. 什么是SQL注入

如有一条查询语句为

``` js
"select * from (" + table + ");"

```

当table取值 `student); drop table student; --` 时，语句变为了，会被删掉表。

``` js
"select * from (student); drop table student; --);"
```
