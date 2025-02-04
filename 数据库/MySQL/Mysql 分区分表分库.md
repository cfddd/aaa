> [搞懂MySQL分区 - GrimMjx - 博客园 (cnblogs.com)](https://www.cnblogs.com/GrimMjx/p/10526821.html)
> [腾讯后台开发暑期实习一面｜讲解｜0312_牛客网 (nowcoder.com)](https://www.nowcoder.com/discuss/596865907796787200)
> [MySQL的分区/分库/分表总结 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/342814592#:~:text=%E5%88%86%E5%8C%BA%EF%BC%9A%E6%8A%8A%E4%B8%80%E5%BC%A0%E8%A1%A8%E7%9A%84%E6%95%B0%E6%8D%AE%E5%88%86%E6%88%90%E5%A4%9A%E4%B8%AA%E5%8C%BA%E5%9D%97%EF%BC%8C%E5%9C%A8%E9%80%BB%E8%BE%91%E4%B8%8A%E7%9C%8B%E6%9C%80%E7%BB%88%E5%8F%AA%E6%98%AF%E4%B8%80%E5%BC%A0%E8%A1%A8%EF%BC%8C%E4%BD%86%E5%BA%95%E5%B1%82%E6%98%AF%E7%94%B1%E5%A4%9A%E4%B8%AA%E7%89%A9%E7%90%86%E5%8C%BA%E5%9D%97%E7%BB%84%E6%88%90%E7%9A%84%20%E5%88%86%E8%A1%A8%EF%BC%9A%E6%8A%8A%E4%B8%80%E5%BC%A0%E8%A1%A8%E6%8C%89%E4%B8%80%E5%AE%9A%E7%9A%84%E8%A7%84%E5%88%99%E5%88%86%E8%A7%A3%E6%88%90%E5%A4%9A%E4%B8%AA%E5%85%B7%E6%9C%89%E7%8B%AC%E7%AB%8B%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4%E7%9A%84%E5%AE%9E%E4%BD%93%E8%A1%A8%E3%80%82,%E7%B3%BB%E7%BB%9F%E8%AF%BB%E5%86%99%E6%97%B6%E9%9C%80%E8%A6%81%E6%A0%B9%E6%8D%AE%E5%AE%9A%E4%B9%89%E5%A5%BD%E7%9A%84%E8%A7%84%E5%88%99%E5%BE%97%E5%88%B0%E5%AF%B9%E5%BA%94%E7%9A%84%E5%AD%97%E8%A1%A8%E6%98%8E%EF%BC%8C%E7%84%B6%E5%90%8E%E6%93%8D%E4%BD%9C%E5%AE%83%E3%80%82%20%E5%88%86%E5%BA%93%EF%BC%9A%E6%8A%8A%E4%B8%80%E4%B8%AA%E5%BA%93%E6%8B%86%E6%88%90%E5%A4%9A%E4%B8%AA%E5%BA%93%EF%BC%8C%E7%AA%81%E7%A0%B4%E5%BA%93%E7%BA%A7%E5%88%AB%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9CI%2FO%E7%93%B6%E9%A2%88%E3%80%82)

## 概念

1. 分区：**把一张表的数据分成多个区块，在逻辑上看最终只是一张表，但底层是由多个物理区块组成的** 
2. 分表：**把一张表按一定的规则分解成多个具有独立存储空间的实体表。系统读写时需要根据定义好的规则得到对应的字表明，然后操作它。**
3. 分库：**把一个库拆成多个库，突破库级别的数据库操作I/O瓶颈。**
### InnoDB逻辑存储结构
![](https://img2018.cnblogs.com/blog/1465200/201903/1465200-20190313211422354-1043302132.png)
> [MySQL原理(一)：逻辑存储结构 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/628357230)
- **段**：
	段不对应表空间中某一个连续的物理区域，而是一个逻辑上的概念，由若干个零散的页面（碎片区）和一系列完整的区组成。
	
	一个表空间里面会有很多个段组成。常见的段有数据段、索引段、回滚段等。
- **区**：
	区是段的组成部分，它是一个连续的磁盘空间，用于存储数据页。在 InnoDB 存储引擎中，一个区通常包含多个数据页，这些页可以存储表数据、索引数据或其他数据。
- **页**：
	页是 MySQL 中数据存储的最小单位，它代表了一个固定大小的数据块，通常为 16KB。每个页可以存储一定数量的数据行、索引键和其他数据。在 InnoDB 存储引擎中，数据页用于存储表数据、索引数据和事务日志等。


## 分区
- **概念：** 分区是将一个大表在物理上分割成多个较小的、更易于管理的片段，但从逻辑上来看，它们仍然被视为同一个表。
- **类比理解：** 你可以想象一个大书架，上面摆满了书，很难管理。如果我们将这些书按照类别（比如小说、历史、科学等）分到不同的小书架上，每个小书架就更容易管理了。这就是分区的基本思想。
- **示例：** 假设有一个销售表，记录了多年的销售数据。我们可以按照年份进行分区，这样查询某一年的数据时，只需要扫描该年份的分区即可。
- **实现**：在分区策略中，尽管表在物理存储上被分成了多个部分（每个部分有自己的.ibd文件），但在逻辑上它们仍然被视为一个整体，并且只与一个.frm文件相关联（**1个.frm文件对应多个.ibd文件**）

> .frm 文件用于存储表的结构定义，包括列名、数据类型等
> .ibd 文件用于存储表的数据和索引
### 分区类型
1. **RANGE分区**：基于一个给定区间边界，得到若干个连续区间范围，按照分区键的落点，把数据分配到不同的分区；
2. LIST分区：类似RANGE分区，区别在于LIST分区是基于枚举出的值列表分区，RANGE是基于给定连续区间范围分区；
3. HASH分区：基于用户自定义的表达式的返回值，对其根据分区数来取模，从而进行记录在分区间的分配的模式。这个用户自定义的表达式，就是MySQL希望用户填入的哈希函数。
4. KEY分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且使用MySQL 服务器提供的自身的哈希函数。

### RANGE 类型
```sql
-- 插入下面的数据，使用 RANGE 分区
create database `m_test_db`;
CREATE TABLE `m_test_db`.`Order` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `partition_key` INT NOT NULL,
  `amt` DECIMAL(5) NULL,
  PRIMARY KEY (`id`, `partition_key`)) PARTITION BY RANGE(partition_key) PARTITIONS 5( PARTITION part0 VALUES LESS THAN (201901),  PARTITION part1 VALUES LESS THAN (201902),  PARTITION part2 VALUES LESS THAN (201903),  PARTITION part3 VALUES LESS THAN (201904),  PARTITION part4 VALUES LESS THAN (201905)) ;

INSERT INTO `m_test_db`.`Order` (`id`, `partition_key`, `amt`) VALUES ('1', '201901', '1000');
INSERT INTO `m_test_db`.`Order` (`id`, `partition_key`, `amt`) VALUES ('2', '201902', '800');
INSERT INTO `m_test_db`.`Order` (`id`, `partition_key`, `amt`) VALUES ('3', '201903', '1200');

-- 查看这张表所有的分区
SELECT * FROM information_schema.PARTITIONS WHERE TABLE_SCHEMA = 'm_test_db' AND TABLE_NAME = 'Order';
-- 查看这一个查询用到的分区
SELECT * FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'Order' 
AND TABLE_SCHEMA = 'm_test_db' 
AND PARTITION_DESCRIPTION = '201902';

-- PARTITION_ORDINAL_POSITION 2
```

### LIST分区

　　LIST分区和RANGE分区很相似，只是分区列的值是离散的，不是连续的。LIST分区使用VALUES IN，因为每个分区的值是离散的，因此只能定义值。

### HASH分区

　　说到哈希，那么目的很明显了，将数据均匀的分布到预先定义的各个分区中，保证每个分区的数量大致相同。

### KEY分区

　　KEY分区和HASH分区相似，不同之处在于HASH分区使用用户定义的函数进行分区，KEY分区使用数据库提供的函数进行分区。


## 分表
- **概念：** 分表是将一个大表拆分成多个具有相同结构的小表，每个小表存储不同的数据子集。
- **类比理解：** 这有点类似于将一个大班级分成多个小班，每个小班管理起来更加容易。
- **示例：** 假设有一个用户表，用户数量巨大。我们可以按照用户ID的范围进行分表，比如用户ID 1-1000在表1，用户ID 1001-2000在表2，以此类推。
- **实现**：在分表策略中，每个分表都有自己独立的.frm和.ibd文件，这些分表在逻辑和物理存储上都是完全独立的。（**1个.frm文件对应1个.ibd文件**）

### 水平分表
水平分表和分区很像，或者说分区就是水平分表的数据库实现版本，它们分的都是行记录，就像用一把刀，水平的将一个表切成多张表一样。

针对数据量巨大的单张表（比如订单表），我们按照某种规则，切分到多张表里面去。
### 垂直分表
水平分表分的是行记录，而垂直分表，分的是列字段，它就像用一把刀，垂直的将一个表切成多张表一样。

将不常用的，**或者数据量大的字段**拆分到“扩展表”上。这样避免查询时，数据量太大造成的“跨页”问题。

## 分库
一台机器的性能是有限制的。

**将一个库分成多个库，并在多个服务器上部署，就可以突破单服务器的性能瓶颈**，这是分库必要性的最主要原因。

分库之后也可以进行分表，两种方法结合
### 分库的类型
分库同样分为水平分库和垂直分库，与水平分表和垂直分表的规则类似。
## 分库分表的问题
1. **事务问题**
	- **解释**：由于数据存储到了不同的库上，无法依赖数据库的事务。
	- **解决方法**：利用分布式事务，协调不同库之间的数据原子性，一致性。
2. **跨库跨表的join问题**
	- **解释**：在不同库、表的数据被切割，原本简单的join复杂性大大增加，难以定位，查询次数也有可能增加。
	- **解决方法**：尽力避免跨库join，或者查询代码整合好
3. **因分库分表带来的额外性能开销**

### 怎么判断该要 分区 还是 分库分表？
对于一般业务的数据量、访问量和增长量，使用分区是绰绰有余的，**分区是介于分库分表和单表之间的简单过渡方案**

很多情况下，面对单表存储性能达到上限的情况，大家都会无脑选择进行分库分表，但是这显然是不对的，因为没有考虑一个问题：是单表B+树的数据结构达到了承载上限，还是单机性能达到了上限

多数情况下，数据并没有完全利用好单机的性能，盲目地扩容机器只会增加业务结构的复杂性 和 开发的工作量

下面给出一些评判要素：
- **数据量和增长趋势**：实际数据量决定架构
- **维护成本和复杂性**：分区较为简单；分库分表较为复杂
- **数据库性能和扩展性**：数据库的性能指标，如 CPU 利用率、内存占用、磁盘 I/O 等，当其中一个成为短板，就会产生类似木桶效应的后果，拖慢数据库的吞吐量

综合以上要素考虑，就能得出大致选择那种方案了
## 小结
分区一般都是放在单机里的，方便归档。

分区的类型有 RANGE、LIST、HASH、KEY分区，用的比较多的是**时间范围RANGE分区**

分表是指拆分一张表，变成多个表。方法有水平和垂直分表。

分库逻辑上和分表类似，是指拆分一个完整的库，分布式部署到多机上去

分库分表需要代码实现，分区则是mysql内部实现。分库分表和分区并不冲突，可以结合使用。

分库分表会带来 **事务、跨库跨表join问题和额外的性能开销**。
