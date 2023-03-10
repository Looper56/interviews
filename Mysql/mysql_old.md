# Mysql

### 锁
#### 表级锁：开销小，加锁快，不会出现死锁，锁定粒度大，发生冲突概率最高，并发度最低
#### 行级锁：开销大，加锁慢，会出现死锁，锁定粒度最小，发生冲突的概率最低，并发度也最高
#### 页面锁：开销和加锁时间界于表锁和行锁止键；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

### 悲观锁和乐观锁的区别
#### 悲观锁：假设最坏情况，每次拿数据都会进行上锁，别人想拿数据就会被阻塞，直到拿到锁，共享资源每次只给一个线程使用，传统关系数据库里面都会用到这种锁机制，如上，运用于多写
#### 乐观锁：假设最好情况，默认别人不会修改，所以不会上锁，更新的时候会判断此期间别人有没有去更新这个数据，版本号机制和CAS算法，适用于多读应用类型，运用于多读
##### CAS(compare and swap)算法：无锁算法，实现多线程之间的变量同步

### Mysql中有哪些不同的表格
#### MylSAM2/Heap/Merge/INNODB/MISAM
#### MylSAM2：不支持事务，每次查询都是原子；支持表级锁，每次操作都是对整个表加锁，存储表总行数，有三个文件：索引文件/表结构文件/数据文件
#### INNODB：支持ACID事务，支持事务的四种隔离级别了支持行级锁及外建约束；支持并发；不存储总行数；引擎存储在一个文件空间（共享表空间/大小不受操作系统限制）

### Mysql支持的四种事务隔离级别
#### read uncommitted：读到未提交数据
#### read committed：脏读，不可重复读
#### repeatable：可复读
#### serializable：串行事务

### char和varchar的区别
#### char和varchar类型在存储和检索方面有所不同
#### char列长度固定为创建表时声明的长度，长度取值1-255，当char值被存储时，他们被用空格填充到特定长度，检索char值时需要删除尾随空格

### 主键和候选键有什么区别？
#### 表格的每一行都有主键和唯一标识，一个表只有一个主键
#### 主键也是候选键，候选键可以被指定为主键，并且可以用于任何外键引用

### myisamchk是用来做什么的？
#### 压缩myisam表，减少磁盘或者内存使用

### Mysql如何优化DISTINCT？
#### DISTINCT在所有列上转换为GROUP BY，并与ORDER BY子句结合使用

### 可以使用多少列创建索引
#### 任何标准表最多可以创建16个索引列

### NOW（）和CURENT_DATA（）有什么区别？
#### NOW（）用于显示当前年份，月份，日期，小时，分钟和秒，CURRENT_DATA仅显示当前年份，月份和日期

### 什么是非标准字符串类型？
#### TINYTEXT
#### TEXT
#### MEDIUMTEXT
#### LONGTEXT

### Mysql支持事务吗？
#### 在缺省模式下，Mysql是autocommit模式，所有的数据库更新操作都会即时提交，所以在缺省情况下，mysql是不支持事务的
#### 如果使用的表类型是InnoDB或BDB的话，你的mysql就可以使用事务处理，使用autocommit=0就可以使mysql允许在非autocommit模式，必须使用commit提交你的更改，或者rollback回滚

### Mysql里记录货币用什么字段
#### numeric和decimal类型被mysql实现为同样类型，精度和规模能被指定

### Mysql有关权限的表都有哪几个？
#### Mysql服务器通过权限表来控制用户对数据库的访问，权限表放在Mysql数据库里，由Mysql_install_db脚本初始化，这些权限表分别user,db,table_priv,host

### 列的字符串类型可以是什么？
#### SET/BLOB/ENUM/CHAR/TEXT

### Mysql数据库作发布系统存储，一天五万条以上增量，优化？
#### 设计良好的数据库结构，允许部分数据冗余，尽量避免join查询，提高效率
#### 选择合适的表字段数据类型和存储引擎，适当的添加索引
#### Mysql库主从读写分离
#### 找规律分表，减少单表中的数据量提高查询速度，添加缓存机制，比如memcached，apc等
#### 不经常改动的页面，生成静态页面
#### 书写高效率SQL

### 索引
#### 索引是什么？
##### 索引是一种特殊的文件(InnoDB数据表上的索引表空间)，它们包含着对数据表里所有记录的引用指针
##### 索引是一种数据结构。数据库索引，是数据库管理系统中一个排序的数据结构，以协助快速查询，更新数据库表中数据，索引的实现通常使用B+树，通俗来说，索引就相当于目录，为了方便查询书中内容，而且索引是一个文件，需要占据物理空间
#### 优点：
##### 大大加快数据的检索速度，主要原因
##### 通过使用索引，在查询过程中，优化隐藏器，提高性能
#### 缺点：
##### 时间方面：创建索引和维护索引需要消耗时间，具体地对表中数据进行增加，删除和修改时，索引也要动态维护，会降低增，改，删的执行效率
#### MySQL索引类型？
##### 存储结构划分：Btree索引(B-Tree或B+Tree)，Hash索引，full-index全文索引，R-Tree索引
##### 应用层划分：普通索引(一个索引只包含单个列，可以多个)，唯一索引(索引值必须唯一，允许空)，复合索引(多列值组合搜索)，聚集索引(数据存储方式)，非聚集索引(不是聚集索引，就是非聚集索引)
#### 索引的底层实现
##### Hash索引：基于Hash表实现，只有精确匹配索引所有列的查询才有效，对于每一行数据，存储引擎都会对所有的索引列计算一个Hash码，并且Hash索引将所有的Hash存储在索引中
##### B-Tree索引：B-Tree能加快数据的访问速度，因为存储引擎不再需要进行全表扫描获取数据，数据分布在各个节点
##### B+Tree索引：B-Tree的改进版本，也是数据库索引所采用的存储结构，数据都在叶子节点，并且增加顺序访问指针，每个叶子节点都指向相邻的叶子节点地址，范围查找时只需要查找两个节点，进行遍历即可
#### 为什么索引结构默认使用B+Tree，而不是B-Tree，Hash，二叉树，红黑树？
##### B+Tree的磁盘读写代价更低，B+Tree内部节点并没有指向关键字具体信息查询，因此内部节点相比B-Tree更小，所有同一内部节点关键字放在同一盘块中，关键字数量多，降低IO读写次数
##### Hash：虽然可以快速定位，但是没有顺序，IO复杂度高，只有Memory存储引擎支持Hash索引，适合等值查询(快)，不支持范围查询，不是按照索引值存储的，始终索引所有列的全部内容，不支持匹配查找，重复键降低Hash索引效率
##### 二叉树：树的高度不均匀，不能自平衡，查找效率跟数据有关，并且IO代价高
##### 红黑树：树的高度随着数据量的增加而增加，IO代价高

### mysql索引失效的原因
#### 索引无法存储null值
#### 不合适键值较少的列(重复数据较多)
#### 前导模糊查询不能利用索引：like/or/列类型是字符串，查询则须引号引用/如果mysql认为全表扫描比索引快
#### 复合索引没有全部用到

### mysql查询优化：优化sql语句，创建索引，使用join代替子查询，使用联合union代替手动创建临时表，事务，锁定表

#### 聚簇索引和非聚簇索引
##### 在InnoDb里，B+Tree的叶子节点存储了整行数据的是主键索引，也被称为聚簇索引，找到索引就找到值
##### 索引B+Tree的叶子节点存储了主键值的是非主键索引，非聚簇索引，二级索引
##### 区别：非聚簇索引的叶子节点不存储表中数据

#### 非聚簇索引一定会回表查询吗？
##### 不一定，这涉及查询语句所要求字段是否覆盖全部索引，就不必再进行回表查询，称为“覆盖索引”

#### 联合索引是什么？为什么需要注意索引中的顺序
##### 多个字段建立索引，在联合索引中，如果要命中索引，需要按照建立索引时的字段顺序挨个使用，负责无法命中索引

#### MySQL最左前缀原则？
##### 最左前缀原则就是最左优先，在创建索引时，需要根据业务需求，where子句中使用最频繁的一列放在左边，mysql会一直向右匹配

#### 前缀索引
##### 索引字段很长，即占内存，不利于维护，只把索引前面的公共部分作为一个索引，产生超级加倍效果，order by不支持前缀索引

#### 索引下推
##### MySQl5.6引入索引下推优化，默认开启，可以减少回表次数，在InnoDB中只针对二级索引有效

#### 怎么查看MySQL语句有没有用到索引？
##### EXPLAIN；id：关键字对应唯一id，select_type：关键字查询类型，table：对应表名，type：可判断是否全表扫描，索引扫描，possible_key：可能用到的索引，key：真正使用的索引，filered：查询器预测满足下次查询条件百分比，rows：MySQL查询优化器根据统计信息，估算找到结果需要扫描数据行数，rows越少越好，extra：额外信息

#### 为什么官方建议使用自增长主键作为索引？
##### B+Tree的特点，自增主键是连续的，在插入过程减少页分裂，即使也会分裂较少，减少数据移动，减少分裂和移动频率

### 大数据量表进行添加字段
#### 直接添加：适合十几万数据量，如果很大，加字段会锁表，导致服务崩溃
#### 临时表：新建临时表，新表加字段，复制数据到新表，删除旧表
#### 使用工具pt-online-schema-change
#### 升级MySQL服务器版本，8.0.12之后

### 事务
#### 事务是一系列数据库操作，是数据库应用的基本单位，mysql事务主要处理操作量大，复杂度高的数据
### 特性
#### MySQl只有InnoDB引擎支持事务
#### 原子性：要么全部不执行，要么全部执行
#### 一致性：事务的执行使得数据从一种正确状态转化为另一种正确状态
#### 隔离性：事务正确提交之前，不允许把该事务对数据的任何改变提供给其他事务
#### 持久性：事务提交后，其结果永久保存在数据库中
### 幻读和不可重复读的区别
#### 不可重复读重点是修改：同一事务中，同样条件，第一次和第二次读的数据不一样(其他事务提交了修改)
#### 幻读的重点是新增或者删除：同一事务，同样条件，第一次和第二次记录数不一样(其他事务提交插入/删除)
### 并发事务
#### 更新丢失：当两个或者多个事务选择同一行，然后基于最初选定的值更新该行时，不知其他事务的存在，最后更新覆盖其他事务的更新，在修改时避免其他事务操作
#### 脏读：一个事务对一条数据做修改，在这个事务完成提交前，这条记录数据就处于不一致状态，另一个事务也来读取同一条记录，第二个事务读取脏数据并作出一步处理，就会产生未提交的数据依赖
#### 不可重复读：一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其数据已经发生改变，或某些数据已经被删除
#### 幻读：一个事务按相同的条件查询重新读以前的数据，却发现其他事务插入满足其他查询条件的新数据
### 并发事务的处理
#### 加锁：在读取数据前，对其加锁，阻止其他事务对数据进行修改
#### 提供数据多版本并发控制(MVCC 或 MCC)：多版本数据库，不用加锁，通过一定机制生成一个数据库请求时间点的一致性数据快照，并用这个快照提供一定级别的一致性读取
### 什么是MVCC
#### 多版本并发控制系统，InnoDB和FaIcon存储引擎通过多版本并发控制机制解决幻读问题
### MVCC工作流程
#### InnoDB的MVCC是通过在每行记录后保存两个隐藏列实现，一个保存创建时间，一个保存过期时间，每开始一个新的事务，系统版本号就会自动新增，事务开始时刻的系统版本号作为事务版本👌，用来查询的每行记录版本号进行比较
### REPEATABLE_READ(可重复读)隔离级别下MVCC如何工作
#### select：InnoDB会根据条件检查每一行记录，InnoDB只会查询版本号早于当前事务的版本号，可以确保事务读取的行要么是开始事务之前，要么是事务修改过的；行的版本号要么未定义，要么大于当前事务版本，确保读取之前为删除
#### insert：为新插入的每一行数据保存当前系统版本号作为行版本号
#### delete：为删除的每一行保存当前系统版本号作为行删除标识
#### update：为插入的一行新纪录保存当前系统版本号作为行版本号，同时保存当前系统版本号作为原来行删除的标识保存两个版本号，使大多数操作不用加锁，需要额外的存储空间，需要更多的行检查和额外维护