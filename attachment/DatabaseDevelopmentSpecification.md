# 数据库MySQL开发使用规范  

!>  **当前版本**：数据库MySQL开发使用规范 V1.3.0   
 **更新时间**：2018-08-27  

###  一、 表设计  


?> 1. 库名、表名、字段名使用小写字母，“_”分割。  
 
?> 2. 库名、表名、字段名不超过 12 个字符。  
   
?> 3. 库名、表名、字段名见名知意,尽量使用名词儿不是动词。  
   
?> 4. 使用 InnoDB存储引擎,如需使用其他引擎，请说明原因。  
   
?> 5. 表必须有主键。  
   
?> 6. 存储精确浮点数使用 DECIMAL 替代 FLOAT 和 DOUBLE，如果需要频繁计算的数值字段，建议使用BIGINT代替DECIMAL。 【FAQ】  
   
?> 7. 使用 UNSIGNED 存储非负数值。  
   
?> 8. 使用 INT UNSIGNED 存储 IPV4。【FAQ】  
   
?> 9. 整形定义中不添加长度，比如使用 INT，而不是 INT[4]。【FAQ】  
   
?> 10. 使用短数据类型，比如取值范围为 0-80 时，使用 TINYINT UNSIGNED。  
   
?> 11. 不建议使用 ENUM、SET 类型，使用 TINYINT 来代替。  
   
?> 12. 尽可能不使用 TEXT、BLOB 类型。  
   
?> 13. VARCHAR(N)，N 表示的是字符数而不是字节数，比如 VARCHAR(255)，可以最大可存储 255 个汉字。  
   
?> 14. VARCHAR(N)，N 尽可能小，因为 MySQL 一个表中所有的 VARCHAR 字段最大长度是 65535 个字节，进行排序和创建临时表一类的内存操作时，会使用 n 的长度申请内存。  
   
?> 15. VARCHAR(N)，N>5000 时，使用 BLOB 类型。  
   
?> 16. 表字符集选择 UTF8。  
   
?> 17. 使用 VARCAHAR 存储变长字符串。  
   
?> 18. 存储年使用 YEAR 类型。  
   
?> 19. 存储日期使用 DATE 类型。  
   
?> 20. 存储时间（精确到秒）使用 TIMESTAMP 类型，因为 TIMESTAMP 使用 4 字节，DATETIME 使用 8 个字节。【FAQ】  
   
?> 21. 除非有特殊需求，所有字段使用NOT NULL，并设置默认值。  
   
?> 22. 将过大字段拆分到其他表中。  
   
?> 23. 不在数据库中使用 VARBINARY、BLOB 存储图片、文件等。  
   
?> 24.整套数据库中同一字段字段类型要一致。   


###  二、 索引  

   
?> 1. 非唯一索引按照“idx_字段名称_字段名称[_字段名]”进行命名。  
   
?> 2. 唯一索引按照“uniq_字段名称_字段名称[_字段名]”进行命名。  
   
?> 3. 索引名称使用小写。  
   
?> 4. 索引中的字段数不超过5个。  
   
?> 5. 唯一键由3个以下字段组成，并且字段都是整形时，使用唯一键作为主键。  
   
?> 6. 没有唯一键或者唯一键不符合 5 中的条件时，使用自增（或者通过发号器获取）id 作为主键。  
   
?> 7. 唯一键不和主键重复。  
     
?> 8. 索引字段的顺序需要考虑字段值去重之后的个数，个数多的放在前面，同时考虑查询类型，等值放左边，范围查询放右边。  
   
?> 9. ORDER BY，GROUP BY，DISTINCT 的字段需要添加在索引的后面。  
   
?> 10. 单张表的索引数量控制在 5 个以内。  
   
?> 11. 使用EXPLAIN判断 SQL语句是否合理使用索引，尽量避免extra 列出现：Using File Sort， Using Temporary。【FAQ】  
   
?> 12. UPDATE、DELETE 语句需要根据 WHERE 条件添加索引，为防止SQL注入误操作，不用索引不能执行。  
   
?> 13. 不建议使用%前缀模糊查询，例如 LIKE “%ezubao”，此类查询用不到索引，考虑倒序存储。  
   
?> 14. 对长度大于 50 的 VARCHAR 字段建立索引时，使用其他方法。【FAQ】  
   
?> 15. 合理创建联合索引（避免冗余），(a,b,c) 相当于 (a) 、(a,b) 、(a,b,c)。  
   
?> 16. 合理利用覆盖索引。【FAQ】  
   
?> 17. 尽量不要使用 FORCE INDEX。  
   
?> 18. 注意自增主键表存在唯一索引时，会引起自增主键跳跃问题。  
  


###  三、 SQL 语句  

?> 1. 使用 prepared statement，可以提高性能并且避免 SQL 注入。  
  
?> 2. SQL 语句中 IN 包含的值不超过 500。  
   
?> 3. UPDATE、DELETE 语句不使用 LIMIT。 【FAQ】  
   
?> 4. WHERE条件中使用合适的类型，避免 MySQL 进行隐式类型转化。【FAQ】  
   
?> 5. SELECT语句只获取需要的字段。  
   
?> 6. SELECT、INSERT语句显式的指明字段名称，不使用 SELECT *，不使用 INSERT INTO table()。  
   
?> 7. 使用 SELECT column_name1, column_name2 FROM table WHERE [condition]而不是 SELECT column_name1 FROM table WHERE [condition]和 SELECT column_name2 FROM table WHERE [condition]。  
   
?> 8. WHERE 条件中的非等值条件（IN、BETWEEN、<、<=、>、>=）会导致后面的条件是使用不了索引。  
 
?> 9. 避免在 SQL 语句进行数学运算或者函数运算，容易将业务逻辑和 DB 耦合在一起。  
   
?> 10. INSERT 语句使用 batch 提交（INSERT INTO table VALUES(),(),()„„），values 的个数不超过500。  
   
?> 11. 避免使用存储过程、触发器、函数等，容易将业务逻辑和 DB 耦合在一起，并且 MySQL 的存储过程、触发器、函数中存在一定的bug，出现问题极难定位。
   
?> 12. 尽量避免使用JOIN。  
   
?> 13. 使用合理的SQL语句减少与数据库的交互次数。【FAQ】  
   
?> 14. 不使用 ORDER BY RAND()，使用其他方法替换。【FAQ】  
   
?> 15. 使用合理的分页方式以提高分页的效率。【FAQ】  
   
?> 16. 统计表中记录数时使用 COUNT(*)，而不是 COUNT(primary_key)和 COUNT(1)。  
   
?> 17. 禁止在生产系统中执行复杂的和返回大量结果集的操作，尤其是主从集群中，可能会产生长时间的锁，造成主从延时。    


### 四、 散表  


?> 1. 每张表数据量控制在 5000w 以下。  
  
?> 2. 可以结合使用 hash、range、lookup table 进行散表。  
   
?> 3. Hash 散表，表名后缀使用 16 进制，比如 user_ff。  
  
?> 4. 使用时间散表，表名后缀使用日期，比如 user_20110209。     

#### FAQ  

?> 1. 频繁计算字段为什么建议用bigint代替DECIMAL？  
   
是因为CPU并不直接支持这种计算。浮点运算稍微快些，因为CPU本地执行了这些运算。如果需要保存三位小数的准确浮点数，则先将数值乘以1000后存为bigint，取时除以1000   
?> 2. 如何使用 INT UNSIGNED 存储 ip？  
   
使用 INT UNSIGNED 而不是 char(15)来存储 ipv4 地址，通过 MySQL 函数 inet_ntoa 和 inet_aton 来进行转化。Ipv6 地址目前没有转化函数，需要使用 DECIMAL 或者两个 bigINT 来 存储。例如： SELECT INET_ATON('209.207.224.40'); 3520061480 SELECT INET_NTOA(3520061480); 209.207.224.40   
?> 3.	INT[M]，M 值代表什么含义？   
注意数值类型括号后面的数字只是表示宽度而跟存储范围没有关系，比如 INT(3)默认显 示 3 位，空格补齐，超出时正常显示，python、java 客户端等不具备这个功能。   
?> 4.	为什么建议使用 TIMESTAMP来存储时间而不是DATETIME？  
   
DATETIME 和 TIMESTAMP 都是精确到秒，优先选择 TIMESTAMP，因为 TIMESTAMP只有4个字节，而 DATETIME8个字节。同时TIMESTAMP具有自动赋值以及自动更新的特性。   
?> 5. 如何使用 TIMESTAMP 的自动赋值属性？  
   
a) 将当前时间作为 ts 的默认值：ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP。   
b) 当行更新时，更新 ts 的值：ts TIMESTAMP DEFAULT 0 ON UPDATE CURRENT_TIMESTAMP。   
c) 可以将 1 和 2 结合起来：ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP。   
?> 6. 如何对长度大于 50 的 VARCHAR 字段建立索引？  
   
下面的表增加一列 url_crc32，然后对 url_crc32 建立索引，减少索引字段的长度，提高效率。  
``` 
CREATE TABLE url( 
…… 
url VARCHAR(255) NOT NULL DEFAULT 0, 
url_crc32 INT UNSIGNED NOT NULL DEFAULT 0, 
…… 
index idx_url(url_crc32) ) 
使用前缀索引 
CREATE TABLE url( 
…… 
url VARCHAR(255) NOT NULL DEFAULT 0, 
…… 
index idx_url(10) )
```   
?> 7. update和delete不能使用limit,分批更新分批删除咋么做？  
   
分批更新：  
``` 
update t1 
set t1.name=’Liu’ 
from t1 
inner join （select id from t1 order by time limit 100000,10）t2 
on t1.id = t2.tid 
分批删除： 
delete from t1 
from t1 
inner join (select id from t1 order by time limit 100000,10) t2 
on t1.id = t2.tid 
```  
分批更新删除对锁持有的时间更短，对线上业务影响更小。   
?> 8. 为什么需要避免 MySQL 进行隐式类型转化？  
   
因为 MySQL 进行隐士类型转化之后，可能会将索引字段类型转化成=号右边值的类型， 导致使用不到索引，原因和避免在索引字段中使用函数是类似的。   
?> 9. 为什么避免使用复杂的SQL?  
      
拒绝使用复杂的 SQL，将大的 SQL 拆分成多条简单 SQL 分步执行。   
原因：   
简单的 SQL 容 易使用到 MySQL 的 query cache；   
减少锁表时间特别是 MyISAM；   
可以使用多核 cpu。   
?> 10. 为什么不建议使用 SELECT *?  
   
增加很多不必要的消耗（cpu、io、内存、网络带宽）；   
增加了使用覆盖索引的可能性；   
当表结构发生改变时，前端也需要更新。   
?> 11. InnoDB 存储引擎为什么避免使用 COUNT(*)?   
  
InnoDB表避免使用COUNT(*)操作，计数统计实时要求较强可以使用memcache或者redis， 非实时统计可以使用单独统计表，定时更新。   
?> 12. MySQL 中如何进行分页？  
   
假如有类似下面分页语句：   
SELECT * FROM table ORDER BY TIME DESC LIMIT 10000,10;   
这种分页方式会导致大量的 io，因为 MySQL 使用的是提前读取策略。   
推荐分页方式：  
``` 
1、 SELECT 
* 
FROM table 
WHERE TIME<last_TIME 
ORDER BY TIME 
DESC LIMIT 10 
2、 SELECT
* 
FROM table 
inner JOIN(SELECT 
id 
FROM table 
ORDER BY TIME 
LIMIT 10000,10) as t 
USING(id) 
```  
?> 13. 为什么 MySQL 的性能依赖于索引？  
   
MySQL 的查询速度依赖良好的索引设计，因此索引对于高性能至关重要。合理的索引会 加快查询速度（包括 upDATE 和 DELETE 的速度，MySQL 会将包含该行的 page 加载到内存中， 然后进行 upDATE 或者 DELETE 操作），不合理的索引会降低速度。 MySQL 索引查找类似于新华字典的拼音和部首查找，当拼音和部首索引不存在时，只能 通过一页一页的翻页来查找。当 MySQL 查询不能使用索引时，MySQL会进行全表扫描，会消耗大量的 IO。   
?> 14. 为什么一张表中不能存在过多的索引？  
   
InnoDB 的 secondary index 使用 b+tree 来存储，因此在 upDATE、DELETE、INSERT 的时候 需要对 b+tree 进行调整，过多的索引会减慢更新的速度。   
?> 15. 什么是覆盖索引？  
   
InnoDB 存储引擎中，secondary index（非主键索引）中没有直接存储行地址，存储主键值。如果用户需要查询 secondary index 中所不包含的数据列时，需要先通过 secondary index 查找到主键值，然后再通过主键查询到其他数据列，因此需要查询两次。 覆盖索引的概念就是查询可以通过在一个索引中完成，覆盖索引效率会比较高，主键查询是天然的覆盖索引。 合理的创建索引以及合理的使用查询语句，当使用到覆盖索引时可以获得性能提升。 比如 SELECT email,uid FROM user_email WHERE uid=xx，如果 uid 不是主键，适当时候可以将索引添加为 index(uid,email)，以获得性能提升。   
?> 16. EXPLAIN 语句 EXPLAIN 语句（在 MySQL 客户端中执行）可以获得 MySQL 如何执行 SELECT 语句的信息。  
   
通过对 SELECT 语句执行 EXPLAIN，可以知晓 MySQL 执行该 SELECT 语句时是否使用了索引、 全表扫描、临时表、排序等信息。尽量避免 MySQL 进行全表扫描、使用临时表、排序等。 详见官方文档。  

   
 

 