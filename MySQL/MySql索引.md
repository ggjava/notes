### 什么是索引?

索引是一种数据结构，可以帮助我们快速的进行数据的查找.

### 如何为表字段添加索引？

1. 添加PRIMARY KEY（主键索引）

```sql
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )
```

2. 添加UNIQUE(唯一索引)

```sql
ALTER TABLE `table_name` ADD UNIQUE ( `column` )
```

3. 添加INDEX(普通索引)

```sql
ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
```

4. 添加FULLTEXT(全文索引)

```sql
ALTER TABLE `table_name` ADD FULLTEXT ( `column`)
```

5. 添加多列索引

```sql
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
```

### 索引的分类?

1. 从存储结构上来划分：BTree索引（B-Tree或B+Tree索引），Hash索引，full-index全文索引，R-Tree索引。这里所描述的是索引存储时保存的形式，
2. 从应用层次来分：普通索引，唯一索引，复合索引
3. 根据中数据的物理顺序与键值的逻辑（索引）顺序关系：聚集索引，非聚集索引。

平时讲的索引类型一般是指在应用层次的划分。

* 普通索引：即一个索引只包含单个列，一个表可以有多个单列索引
* 唯一索引：索引列的值必须唯一，但允许有空值
* 复合索引：多列值组成一个索引，专门用于组合搜索，其效率大于索引合并
* 聚簇索引(聚集索引)：并不是一种单独的索引类型，而是一种数据存储方式。具体细节取决于不同的实现，InnoDB的聚簇索引其实就是在同一个结构中保存了B-Tree索引(技术上来说是B+Tree)和数据行。
* 非聚簇索引：不是聚簇索引，就是非聚簇索引

### 索引是个什么样的数据结构呢?

索引的数据结构和具体存储引擎的实现有关, 在MySQL中使用较多的索引有Hash索引,B+树索引等,而我们经常使用的InnoDB存储引擎的默认索引实现为:B+树索引.

### Hash索引和B+树所有有什么区别或者说优劣呢?

首先要知道Hash索引和B+树索引的底层实现原理:  
hash索引底层就是hash表,进行查找时,调用一次hash函数就可以获取到相应的键值,之后进行回表查询获得实际数据.B+树底层实现是多路平衡查找树.对于每一次的查询都是从根节点出发,查找到叶子节点方可以获得所查键值,然后根据查询判断是否需要回表查询数据.  
那么可以看出他们有以下的不同:  
hash索引进行等值查询更快(一般情况下),但是却无法进行范围查询.  
因为在hash索引中经过hash函数建立索引之后,索引的顺序与原顺序无法保持一致,不能支持范围查询.而B+树的的所有节点皆遵循(左节点小于父节点,右节点大于父节点,多叉树也类似),天然支持范围.  
hash索引不支持使用索引进行排序,原理同上.  
hash索引不支持模糊查询以及多列索引的最左前缀匹配.原理也是因为hash函数的不可预测.AAAA和AAAAB的索引没有相关性.  
hash索引任何时候都避免不了回表查询数据,而B+树在符合某些条件(聚簇索引,覆盖索引等)的时候可以只通过索引完成查询.  
hash索引虽然在等值查询上较快,但是不稳定.性能不可预测,当某个键值存在大量重复的时候,发生hash碰撞,此时效率可能极差.而B+树的查询效率比较稳定,对于所有的查询都是从根节点到叶子节点,且树的高度较低.

因此,在大多数情况下,直接选择B+树索引可以获得稳定且较好的查询速度.而不需要使用hash索引.

### 上面提到了B+树在满足聚簇索引和覆盖索引的时候不需要回表查询数据,什么是聚簇索引?

在B+树的索引中,叶子节点可能存储了当前的key值,也可能存储了当前的key值以及整行的数据,这就是聚簇索引和非聚簇索引. 在InnoDB中,只有主键索引是聚簇索引,如果没有主键,则挑选一个唯一键建立聚簇索引.如果没有唯一键,则隐式的生成一个键来建立聚簇索引.  
当查询使用聚簇索引时,在对应的叶子节点,可以获取到整行数据,因此不用再次进行回表查询.

### 非聚簇索引一定会回表查询吗?

不一定,这涉及到查询语句所要求的字段是否全部命中了索引,如果全部命中了索引,那么就不必再进行回表查询.  
举个简单的例子,假设我们在员工表的年龄上建立了索引,那么当进行select age from employee where age < 20的查询时,在索引的叶子节点上,已经包含了age信息,不会再次进行回表查询.

### 在建立索引的时候,都有哪些需要考虑的因素呢?

建立索引的时候一般要考虑到字段的使用频率,经常作为条件进行查询的字段比较适合.如果需要建立联合索引的话,还需要考虑联合索引中的顺序.此外也要考虑其他方面,比如防止过多的所有对表造成太大的压力.这些都和实际的表结构以及查询方式有关.

### 联合索引是什么?为什么需要注意联合索引中的顺序?

MySQL可以使用多个字段同时建立一个索引,叫做联合索引.在联合索引中,如果想要命中索引,需要按照建立索引时的字段顺序挨个使用,否则无法命中索引.
具体原因为:MySQL索引的最左匹配原则

### 什么是最左前缀原则？

MySQL中的索引可以以一定顺序引用多列，这种索引叫作联合索引。如User表的name和city加联合索引就是(name,city)，而最左前缀原则指的是，如果查询的时候查询条件精确匹配索引的左边连续一列或几列，则此列就可以被用到。如下：

```sql
select * from user where name=xx and city=xx ; // 可以命中索引
select * from user where name=xx ; // 可以命中索引
select * from user where city=xx ; // 无法命中索引
```

这里需要注意的是，查询的时候如果两个条件都用上了，但是顺序不同，如 city= xx and name ＝xx，那么现在的查询引擎会自动优化为匹配联合索引的顺序，这样是能够命中索引的。
由于最左前缀原则，在创建联合索引时，索引字段的顺序需要考虑字段值去重之后的个数，较多的放前面。ORDER BY子句也遵循此规则。

### 创建的索引有没有被使用到?或者说怎么才可以知道这条语句运行很慢的原因?

MySQL提供了explain命令来查看语句的执行计划,MySQL在执行某个语句之前,会将该语句过一遍查询优化器,之后会拿到对语句的分析,也就是执行计划,其中包含了许多信息. 可以通过其中和索引有关的信息来分析是否命中了索引,例如possilbe_key,key,key_len等字段,分别说明了此语句可能会使用的索引,实际使用的索引以及使用的索引长度.

### 那么在哪些情况下会发生针对该列创建了索引但是在查询的时候并没有使用呢?

使用不等于查询,
列参与了数学运算或者函数
在字符串like时左边是通配符.类似于'%aaa'.
当mysql分析全表扫描比使用索引快的时候不使用索引.
当使用联合索引,前面一个条件为范围查询,后面的即使符合最左前缀原则,也无法使用索引.

### 为什么索引结构默认使用B+Tree，而不是Hash，二叉树，红黑树？

* B-tree：因为B树不管叶子节点还是非叶子节点，都会保存数据，这样导致在非叶子节点中能保存的指针数量变少（有些资料也称为扇出），指针少的情况下要保存大量数据，只能增加树的高度，导致IO操作变多，查询性能变低；
* Hash：虽然可以快速定位，但是没有顺序，IO复杂度高。
* 二叉树：树的高度不均匀，不能自平衡，查找效率跟数据有关（树的高度），并且IO代价高。
* 红黑树：树的高度随着数据量增加而增加，IO代价高。

### 怎么查询sql执行计划

查询 sql 之前加上 explain 可查看该条 sql 的执行计划

```
+------+-------------+-------+------+---------------+------+---------+------+------+-------+
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
+------+-------------+-------+------+---------------+------+---------+------+------+-------+
|    1 | SIMPLE      | user  | ALL  | NULL          | NULL | NULL    | NULL |    2 |       |
+------+-------------+-------+------+---------------+------+---------+------+------+-------+
```

我们可以通过分析这个执行计划来知道我们sql的运行情况。现对各列进行解释：

1. id：查询中执行 select 子句或操作表的顺序。
2. select_type：查询中每个 select 子句的类型（简单 到复杂）包括：
* SIMPLE：查询中不包含子查询或者UNION；
* PRIMARY：查询中包含复杂的子部分；
* SUBQUERY：在SELECT或WHERE列表中包含了子查询，该子查询被标记为SUBQUERY；
* DERIVED：衍生，在FROM列表中包含的子查询被标记为DERIVED；
* UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；
* UNION RESULT：从UNION表获取结果的SELECT被标记为UNION RESULT；

3. type：表示 MySQL 在表中找到所需行的方式，又称“访问类型”，包括：
* ALL：Full Table Scan，MySQL将遍历全表以找到匹配的行；
* index：Full Index Scan，index与ALL区别为index类型只遍历索引树；
* range：索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行，常见于between < >等查询；
* ref：非唯一性索引扫描，返回匹配某个单独值的所有行。常见于使用非唯一索引即唯一索引的非唯一前缀进行的查找；
* q_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描；
* onst和system：当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system；
* NULL：MySQL 在优化过程中分解语句，执行时甚至不用访问表或索引。

4. possible_keys：指出 MySQL 能使用哪个索引在表中找到行，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用。

5. key：显示 MySQL 在查询中实际使用的索引，若没有使用索引，显示为 NULL。

6. key_len：表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。

7. ref：表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值。

8. rows：表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值。

9. Extra：其他重要信息 包括：
* Using index：该值表示相应的 select 操作中使用了覆盖索引；
* Using where：MySQL 将用 where 子句来过滤结果集；
* Using temporary：表示 MySQL 需要使用临时表来存储结果集，常见于排序和分组查询；
* Using filesort：MySQL 中无法利用索引完成的排序操作称为“文件排序”。