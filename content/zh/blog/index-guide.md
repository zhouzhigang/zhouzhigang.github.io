---
title: "索引设计指南"
linkTitle: "索引设计指南"
date: 2016-04-17
description: >
  索引的设计、使用及调优；三星索引、多个查询设计索引
---

### 索引须知

1. 【强制】索引是基于应用的**查询路径**来进行选择的，对每个索引的必要性都应该经过仔细分析，要有建立的依据，严禁盲目推测或为特定要求而随意创建索引。

 <font color="#FF4500">说明：</font>

    - 索引的目的主要是为了提高查询速度，因此需要基于查询进行分析，而不是基于数据库设计的`Schema`或`E-R`图。
    - 谓词表达式是索引设计的主入手点，建议对实际查询信息（如相关查询的`SQL`、读写频率、重要度、选择性等）全面分析后来构建相应的索引。

2. 【推荐】尽量在开发阶段设计索引，而不要等到后期再添加索引。

 <font color="#FF4500">说明：</font>

    + 在上线之后才添加索引，会导致事后要花费DBA和研发更多的时间去弥补这个错误。
    + 事后优化和重构是很一种重要的方式，数据更真实，但代价也更高，更适合小规模优化与重构；

3. 【参考】推荐对不同类型的表（表大小以及读写比例）使用不同的准则
    + 数据量较少的小型表：全表扫描与索引查询几乎无区别，不强求，但推荐使用索引；
    + 数据量大、很少写、读多（每次只访问很少数据）的核心表（如会员、商品表）：可以为满足各种查询需求创建大量索引；
    + 数据量大、读写频繁(不可忽视数据插入的代价)的业务表：需要根据收集业务相关的所有查询类型，并寻找能够满足所有读取要求的比较理想的索引组合；
    + 数据量大、写多（只增不改）、读少的流水表（如日志表）：创建尽可能少的有效索引；

4. 【推荐】了解一些基础的原理有利于更好理解并设计索引; 比如`SQL`执行流程、索引的结构、随机`IO`和顺序`IO`成本等。

### 索引设计原则

1. 【强制】索引建立原则
    + 将所有查询列出来进行比较并设计，而不是单个设计，避免顾此失彼;
    + 如果单一的列就可以确保大部分查询都有较好的性能，则可基于该列建立单一索引；
    + 单列索引无法满足时，尽量对索引进行扩展（组合索引），而不是新增单列索引；

 <font color="#FF4500">说明：</font>

    - 基于单个的SQL建立的索引可能是局部最优，但不一定全局最优，可能会造成过多的索引; 我们需要用尽可能少的索引满足尽可能多的需求。
    - 选择度高的列单独创建索引，可以提高索引的弹性与灵活性；如果可以满足需求，那就应该选择单一索引。
    - 对于多个单列索引，虽然MySQL引入了索引合并(`Index Merge`)功能，但这种情况更多时候说明了表的索引建的很糟糕。
        - `AND`意味着可以使用组合索引，`OR`耗费大量CPU和内存资源在缓存、排序和合并上，且优化器不会把这些成本计算到查询成本中。

2. 【强制】索引列选择原则
    + `WHERE`查询条件中频繁出现的列，包括等值和范围查询(不忽视选择性低的列以及不稳定的列)
    + 区分度/选择性高的列
    + 次要的候选列
        + `ORDER BY`或`GROUP BY`后的列(索引扫描排序)
        + `SELECT`后的列(索引覆盖)

 <font color="#FF4500">说明：</font>

    - **常见误区**：不能在选择性低上的列（如性别、状态等）上建索引。
        - 准确的说法应该是不适合在选择性低上的列**单独**建索引，但是可以对选择性较低的列建立的组合索引，发挥多列的综合效应（很多选择性低的列可能会被频繁查询）。
        - 对于选择性低的列，可以`IN(list)`（如在SQL语句中增加`AND sex IN('f', 'm')`）来让MYSQL选择该索引；
    - **常见误区**：不能在不稳定的列（如订单状态）上建索引。
        - 以前一般只有根页常驻内存，现在非叶子页基本是保留在内存中的，即便是更新导致迁移到不同的叶子页，也仅是两次随机读取；
        - 何况很多索引为组合索引，当不稳定的列处于组合索引的尾列时，更新不会导致迁移到新叶子页。
        - 此外，在当前磁盘条件下，只有当列的更新频率大于`10次/s`时，不稳定列才可能造成磁盘驱动负载过高。

 <font color="#FF4500">示例：</font>

    - 订单的状态在每次查询中都会用到(除了查询全部订单状态)，但是选择性很低(只有少数几个状态)，依然可以考虑为订单状态建立索引。

3. 【参考】组合索引顺序决定的原则
    1. 频度: 优先考虑使用最频繁的列作为最左候选列；
    2. `"="`次数: 经常使用的列不仅一列时，比较`"="`查询列的使用频率(第一颗星 - 等值谓词)；
    3. 选择性: 两个列其他条件相同时，将选择性较高的列放在前面；
    4. 排序: 若两列选择性差不多的话，将更符合排序需求的放在前面；(第二颗星 - 排序)
    5. 范围查询: 尽可能将需要做范围查询的列放在索引的最后(如果先导列能够充分缩减范围查询的话，则可不加)；
    6. 覆盖: 考虑能否利用索引覆盖来避免查表(第三颗星 - 包含所有查询列);

 <font color="#FF4500">说明：</font>

    + 当查询条件不包含索引的最左列时无法利用索引，因此，组合索引的顺序及其先导列的选择及其重要；
    + 优先考虑"="而非选择性
        - 如果只存在“=”比较的情况下，选择性的高低对列的组合顺序没什么影响；
        - 涉及范围（非等值）比较时，先等值，再范围比单纯范围查询（后续“=”不起作用）效率高（无论范围查询的列选择性高或者低）。
        - 选择性高的列放在前面是由前提条件的：均为等值查找，且不考虑排序和分组。
    + 索引的记录是按某种顺序排序的，所以有时候可以使用索引可以减少排序`IO`。`EXPLAIN`出来的`type`列的值为`index`。
        - 若不能覆盖所有列，每扫描一条索引记录都需回表查询一次对应行，随机`IO`很慢。
        - 只有当索引的列顺序和`ORDER BY`的顺序完全一致，并且所有列排序方向均一致，才能利用索引来排序。
        - 同时也需要满足最左前缀的要求(前导列为等值常量比较时因自动补全而无需满足)。
    + 最左前缀匹配原则默认查找到直至第一个范围查询，故范围查询的列尽量放在最后。
        - 如果存在多个范围查询的列的话，仍需根据使用频率确定列的顺序；如果某查询非常重要的话，可以考虑创建新的索引；
        - `MYSQL 5.6`之后新增了`ICP`(Index Condition Pushdown)特性支持多个范围查询过滤(依然要先考虑设计的合理性)；
        - 半宽索引(最大化索引过滤): 一个索引包含了WHERE中所有列(仅必要时才访问表)；
    + 理想的索引是一个三星索引，但现实情况可能很复杂，需要进行权衡。
        - 无法得到第一颗星: `WHERE`条件中包含`OR`表达式导致结果无法聚在一起;
        - 无法得到第二颗星: WHERE`条件中包含范围谓词(`>`, `<`, `IN`, `OR`, `BETWEEN`, `LIKE`); 或者`GROUP BY`和`ORDER BY`使用不同的字段;
        - 无法得到第三颗星: 查询的列超过最大的列限制16; 或者查询中包含超过1000字节的大字段(如`BLOB`, `TEXT`, `VARCHAR`等);
        - 宽索引(只需访问索引): 索引中包含了`SELECT`语句中所涉及的所有列的索引，避免表访问；

 <font color="#FF4500">示例：</font>


    单个查询索引设计示例
    SELECT * FROM t WHERE a = 10 AND b = 20; -- (a,b)与(b,a)效果相同; 联合优于两个单个的索引
    SELECT * FROM t WHERE a = 10 OR b = 20; -- 两个单个的索引可能优于联合; 需结合其他查询来判断是否需要联合及顺序，因为(a, b)包含(a)，且优化器可能只选择其中一个索引
    SELECT * FROM t WHERE a = 10 ORDER BY b; -- 可以利用(a,b)索引排序(常量填充); (b)虽然可以利用索引排序，但效果比全表扫描更差(更多随机`IO`)
    SELECT * FROM t WHERE a > 10 ORDER BY b; -- 不可以利用(a,b)索引排序(不满足最左前缀); a选择性高的话仍优先考虑a做前导列(排序不应该被优先考虑)
    SELECT * FROM t WHERE a IN (10, 11) ORDER BY b; -- 不可以利用(a,b)索引排序(不满足最左前缀); (a, b)有序，b只是局部有序，非全局
    SELECT * FROM t WHERE a > 10 AND b = 20 ORDER BY a; -- 可以利用(b,a)过滤字段及索引排序(常量填充);
    SELECT MAX(b) FROM t GROUP BY a; -- 可利用(a,b)的索引排序获取b的max或min值
    SELECT a, b FROM t WHERE a > 10; -- 可以利用(a,b)索引覆盖，无需查表
    SELECT c, d FROM t WHERE a = 10 AND b = 20 ORDER BY c; -- (a, b, c, d)满足三星索引
    SELECT c, d FROM t WHERE a > 10 AND b = 20 ORDER BY c; -- (b, a, c, d)选择过滤，牺牲第二颗星; (b, c, a, d)选择排序，牺牲第一颗星


    多个查询条件放在一起综合考虑示例(单条语句建索引相对简单,但现实中需要尽量满足各种查询语句的性能)
    a(=)，b(BETWEEN)，c(=)
    b(=)，c(BETWEEN)，d(>)
    a(=)，b(=)，c(LIKE)
    a(=)，b(=)，d(>)
    1. 频度分析(实际各查询可能有不同的权重，此处用出现次数简化): a(3次)，b(4次)，c(3次)，d(2次);
    (b,a)比(a,b)略好; 不过如果a是类似于sex/status的列，则可利用in技巧在第二个查询中加入a，那么a也是4次;
    2. "="次数分析: 在a添加in查询后，相当于a有4次"="，b有3次"=",此时(a,b)更好;
    3. 选择性分析: 一般情况下前两步都可确定先导列; 不过如果第一个查询的 b(BETWEEN) 改成 b(IN)，"="次数相同，则需要比较a, b 的选择性，选择性高的放在最左;
    4. 排序分析: 若a, b 选择性差不多的话，看两者那个更符合期望的排序需求;
    5. 范围查询列分析: 若(a, b)可充分缩减查询范围的话，可不用考虑 c 和 d; 否则继续比较 c 和 d 的顺序(类似前几步)，得出 (a, b, c, d);
    此索引满足前三个查询，第四个查询满足度相对低; 如果第四个查询被频繁使用，且非常重要，可新增 (a, b, d)的索引;

#### 索引设计细则

1. 【强制】表的主键必须有索引，且主键不宜过长

 <font color="#FF4500">说明：</font>
    + 避免使用`UUID`等作为主键
    + 对聚簇索引如`InnoDB`来说主键最好是单调增

2. 【强制】业务上具有唯一特性的字段，即使是组合字段，也必须建成唯一索引(主键是特殊的唯一索引，无需重复创建)。

 <font color="#FF4500">说明：</font>
    + 别嫌唯一索引影响了insert速度，这个速度损耗可以忽略；提高查找速度是明显的；
    + 另外，即使在应用层做了非常完善的校验和控制，只要没有唯一索引，根据墨菲定律，必然有脏数据产生。

3. 【推荐】索引字段的数据越小越好。如果对字符串(如`varchar`，`text`字段)建索引，推荐使用前缀索引(指定索引的长度)；

 <font color="#FF4500">说明：</font>

    + 选择足够长的前缀以保证较高的选择性，同时又不能太长；
    + 可以通过计算完整列的选择性，使前缀的选择性(`COUNT(DISTINCT LEFT(col， len))/COUNT(*)`)接近于完整列的选择性(`COUNT(DISTINCT col)/COUNT(*)`)来计算合适的前缀索引长度`len`
    + 前缀索引可以使索引更小、更快；缺点是无法做使用前缀索引做`ORDER BY`、`GROUP BY`以及覆盖扫描；

4. 【推荐】如果对字符串建前缀索引选择性不高时(如`URL`前几位都相同)，还可以考虑伪`Hash`索引

 <font color="#FF4500">说明：</font>

    + 不要使用`SHA1()`和`MD5`作为哈希函数，因为计算出来的值非常长、浪费空间，比较时也会更慢（其目的是最大限度消除冲突）
    + 推荐使用`CRC32()`做伪`Hash`索引(冲突可以接受，同时提供更好的性能)；如果出现大量冲突，可以考虑自己实现一个简单的64位哈希函数（需返回整数，而不是字符串），比如返回`MD5`的一部分
    + 伪哈希索引需要新增一列存储`Hash`值，缺点是需要维护（可以使用触发器实现）

5. 【推荐】在连接字段上建索引: 为主键建一个索引，并为每个外键创建一个由此外键作为前导列的索引。

 <font color="#FF4500">说明：</font>

    + 连接谓词大部分都是基于`主键=外键`这一条件，若存在外键作为前导列的索引并且结果集不是特别大，嵌套循环可能是最快的连接方式。

### 索引使用原则

1. 【强制】索引字段使用时不能进行函数运算，且进行查询时传入的参数类型需保持一致(避免隐式转换)，否则索引无法被使用。

 <font color="#FF4500">反例：</font>

    + `SELECT * FROM t WHERE a+1 = 5;` -- 错误: 为表达式的一部分，非独立列，无法使用索引
    + `SELECT * FROM t WHERE date_format(gmt_create, '%Y-%m-%d') = '2016-05-08'` -- 错误: 使用了函数，无法使用索引
    + c 为 char，`SELECT * FROM t WHERE c = 5;` -- 错误: 传入了整形，类型不一致

2. 【强制】数据库的`LIKE`尽量限制使用，页面搜索严禁左模糊或者全模糊，如需要，请走搜索引擎。

 <font color="#FF4500">说明：</font>

    + 索引文件是从左向右建的，如果左边的值未确定，那么索引就失去了作用。虽然`ICP`可以进行一些优化，支持多个范围或全模糊的过滤，但并非最佳用法。

3. 【推荐】对低选择性的前导列使用`IN()`优化; 如果可能，用`IN`范围枚举代替部分`BETWEEN`。

 <font color="#FF4500">说明：</font>

    + **注意**该技巧不能滥用，因为每增加一个`IN()`条件，优化所做的组合将以指数形式增加；

 <font color="#FF4500">示例：</font>

    + 优化前: `SELECT * FROM t WHERE b=5;` -- 无法利用索引(a,b)
    + 优化后: `SELECT * FROM t WHERE a IN ('f', 'm') AND b=5;`
    + 优化前: `SELECT * FROM t WHERE a BETWEEN 2 AND 4 AND b=5;` -- 无法利用索引(a,b)
    + 优化后: `SELECT * FROM t WHERE a IN (2,3,4) AND b=5;`

4. 【参考】前导列含`IN`查询不可以利用索引排序时，是否可以转换成`UNION`合并排序结果

 <font color="#FF4500">示例：</font>

    + 优化前: `SELECT * FROM t WHERE a IN (1,2) ORDER BY b LIMIT 5;` -- 无法利用索引(a,b)避免排序
    + 优化后: `(SELECT * FROM t WHERE a=1 ORDER BY b LIMIT 5) UNION ALL (SELECT * FROM t WHERE a=2 ORDER BY b LIMIT 5) ORDER BY b LIMIT 5;`

5. 【推荐】利用索引覆盖和延迟关联(`deferred join`)优化查询或分页语句

 <font color="#FF4500">说明：</font>

    + 优化效果取决于`WHERE`条件中匹配返回的行数，只有当过滤后只返回很少结果集效果才非常明显。

 <font color="#FF4500">示例：</font>
    + 优化前: `SELECT * FROM t WHERE a = '0' AND b LIKE '%test%' AND c IS NOT NULL;` -- 涉及 LIKE 和 NOT NULL，无法利用索引
    + 优化后: `SELECT * FROM t JOIN (SELECT id FROM t WHERE a = '0' AND b like '%test%') AS t1 ON t1.id = t.id;` -- 利用(a,b,c)索引覆盖，延迟对表的访问
    + 优化前: `SELECT * FROM t WHERE a = '0' ORDER BY b LIMIT 10000, 10;` -- 回表次数 10000+10
    + 优化后: `SELECT * FROM (SELECT id FROM t WHERE a = '0' ORDER BY b LIMIT 10000, 10) t1 JOIN t WHERE t1.id = t.id;` -- 回表次数 10

6. 【强制】多表关联查询时，优先使用`WHERE`条件，再做表关联，并且需要保证被关联的字段需要有索引。需要`JOIN`的字段，数据类型保持绝对一致。

### 索引调优原则

1. 【推荐】收集查询日志，找出对应的慢查询，有针对性的优化。
    + `SET GLOBAL slow_query_log = ON;` -- 开启慢查询日志
    + `SET GLOBAL long_query_time = 0;` -- 所有的查询
    + `SET GLOBAL log_slow_verbosity = 'full';` -- 收集更多的信息

2. 【推荐】清除无用的重复、冗余或者未使用索引
    + 重复索引: 如在主键上添加唯一索引和普通索引，相当于有三个重复的索引
    + 冗余索引: 如(a, b)已经包含了(a); (a, id)和(a)也是等价的(对`InnoDB`来说主键列已经包含在二级索引了)

 <font color="#FF4500">说明：</font>
    + 注意: 删除索引要非常小心，建议使用`pt-upgrade`工具来检查计划中的索引变更
    + `pt-duplicate-key-checker`工具可检测冗余索引
    + `SET GLOBAL userstat = ON;` -- 打开索引使用统计查找未使用的索引
    + `pt-index-usage`工具可查看索引的使用报告

3. 【推荐】`EXPLAIN`中`type`列(决定如何查找表中的行)：至少要达到`range`级别，要求是`ref`或`eq_ref`级别。

 <font color="#FF4500">说明：</font>

    + `NULL` 能在优化阶段分解查询语句，执行阶段甚至不用访问表或索引
    + `consts`/`system` 在优化阶段即可读取到数据(如将某行的主键放入`WHERE中`选取该行的主键)。
    + `eq_ref` 使用主键或者唯一索引查找，最多只返回一条符合条件的记录
    + `ref` 非唯一索引(或唯一索引的非唯一前缀)的等值查找，可能返回多个符合条件的行。
    + `range` 对索引进范围检索(`WHERE`条件中含`BETWEEN`或`>`等)。
    + `index` 与全表扫描类似，只是按索引次序进行而不是行; 主要优点是避免了排序，缺点可能有大量随机IO(若`Extra`中含`Using Index`则表明使用了覆盖索引，无需读表)。
    + `ALL`全表扫描。

 <font color="#FF4500">反例：</font>`explain`表的结果，`type=index`(`Extra`中不含`Using Index`)，自我满足了。注意：`index`级别比较`range`还低，会产生大量随机IO。
