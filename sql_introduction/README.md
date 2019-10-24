# SQL 查询入门
> 天下难事必作于易，天下大事必作于细

SQL 是结构化查询语言(Structured Query Language)，可以用来操作数据库中的数据。

## 数据表
我们使用数据表 (table) 来存储一个对象的数据，一行记录通常就表示一个具体对象，数据表的列 (column，也称字段) 就表示这个对象的属性。例如广告表的部分结构和数据如下:

**table**: advertisement

| id | channel_id | platform | createdAt | purpose | materialType |
| - | - | - | - | - | - |
| 11111 | 105 | 2 | 2019-10-23 19:33:27 | 2 | 102 |
| 22222 | 601 | 1 | 2019-10-23 19:53:27 | 4 | 102 |

**字段的含义**

- **id**: 内部 id
- **channel_id**: 广告平台(渠道号)
- **platform**: 投放平台, 1=>Android, 2=>iOS, 3=>Android&iOS, 4=>OTT
- **createdAt**: 广告首次出现时间
- **purpose**: 广告推广类型(推广目的), 1=>应用推广, 2=>活动推广, 4=>电商推广, 8=>账号推广
- **materialType**: 素材类型


首先要熟悉表和每个字段的具体含义，才能知道如何查询自己想要的数据。

## 查询
使用语句从数据库查询数据。基本使用方法如下:
```
SELECT 需要的字段 [FROM 数据表 [WHERE 筛选条件] [GROUP BY 聚合字段] [ORDER BY 排序规则] [LIMIT 结果数量]];
```

- 一条语句以分号 `;` 结尾
- 中括号内为可选内容
- `SELECT` 等关键词大小写都可以，甚至你想混着大小写都没问题

### 选择语句
- 使用 `SELECT` 语句选择要查询的内容，通常和 `FROM` 一起使用，从表中选择需要的字段
- 表名和字段名可以加上反引号 \` 包裹起来，也可以不加
- 多个字段使用逗号 `,` 隔开
- 每个选取的字段，可以用 `AS xxx` 来设置别名。使用别名时 `AS` 也可省略
- 使用通配符 `*` 表示选出这张表的所有字段

例如 (-- 开头为注释内容):

```sql
-- 返回所有广告的内部 id
SELECT id FROM advertisement;
-- 返回所有广告的渠道号和推广类型，并分别使用别名
SELECT `channel_id` AS `渠道号` , `purpose` AS `推广目的` FROM `advertisement`;
-- 查询广告表的所有内容
SELECT * FROM advertisement;
-- 查询所有广告包含的不同渠道号 (DISTINCT + 字段)
SELECT DISTINCT channel_id FROM advertisement;
```

### 筛选条件
上面的语句是将表中全部数据都返回，小可爱通常需要按条件进行筛选。
- 使用 `WHERE` 语句可以设置筛选条件
- 可以设置多个筛选条件，使用 `AND`、`OR` 和 括号 `( )` 进行组合
- 筛选条件是一个逻辑表达式，常见的筛选是对字段值进行比较

例如要查今日头条渠道、在 2019 年 7 月新增的全部广告，可以这样写:
```sql
SELECT * FROM advertisement
WHERE channel_id=105  -- 105 表示今日头条
AND createdAt>='2019-07-01' AND createdAt<'2019-08-01';  -- 首次出现时间大于等于 7月1号 且小于 8月1号
```

逻辑表达式中可使用的运算符有以下:
```sql
-- 渠道号等于 105
(省略了前面的语句)... WHERE channel_id = 105
-- 渠道号不等于 105
... WHERE channel_id != 105
-- 广告出现天数大于等于 3
... WHERE duration >= 3

-- 渠道号在 100 和 200 之间
... WHERE channel_id BETWEEN 100 AND 200
-- 等同于上一句
... WHERE channel_id >= 100 AND channel_id <= 200

-- 渠道号是 105, 106, 107 中的任意一个
... WHERE channel_id IN (105, 106, 107)
-- 等同于上一句
... WHERE channel_id=105 OR channel_id=106 OR channel_id=107

-- 表: advertiser
-- 公司名称以 有限公司 结尾，末尾没有百分号
... WHERE searchName LIKE '%有限公司';
-- 公司名称以 广州 开头，头部没有百分号
... WHERE searchName LIKE '广州%';
-- 公司名称包含 科技 二字，前后都有百分号
... WHERE searchName LIKE '%科技%';
```

`LIKE` 用于文本的模式匹配，内容较多，这里仅举一例。更多使用方法参考 [通配符的用法](https://www.runoob.com/sql/sql-wildcards.html)

### 函数
`SELECT` 选取的字段不一定是表的原生字段，也可以是运算表达式。通常使用函数来进行计算。

函数可分为两类: **标量函数** 和 **聚合函数**
- **标量 (Scalar) 函数**: 基于每条记录的输入值，计算返回值。相当于将列的值做了转换
- **聚合 (Aggrerate) 函数**: 从选取的所有记录中，计算出一个返回值
- 函数可以嵌套使用

例子:
```sql
-- 查询全部今日头条广告，返回它们的 首次发现日期 和 推广目的。标量函数，每条记录都会有对应的 create_date
SELECT DATE(createdAt) AS create_date, purpose FROM advertisement WHERE channel_id=105;
-- 查询今日头条的广告总数
SELECT COUNT(*) FROM advertisement WHERE channel_id=105;
-- 查询广告最早出现日期 和 最新出现时间
SELECT DATE(MAX(createdAt)) AS `earliest_date`, MAX(createdAt) AS `newest_time` FROM advertisement;
```

更多函数的使用，参考 [SQL 函数](https://www.runoob.com/sql/sql-function.html)

### 聚合查询
使用 `GROUP BY` 语句，可以将筛选的结果按列的值进行分组，分组之后每组产生一条结果。

- `GROUP BY` 语句通常结合 [聚合函数](#函数) 一起使用，使函数每组计算一个返回值
- 可以设置多层分组字段，会先按第一个字段分组，每组内再按照接下来的字段进行分组

下面是两个聚合查询，以及它们的返回结果示例:

**查询每个渠道的广告数量**
```sql
SELECT channel_id, COUNT(*) AS channel_ad_count FROM advertisement GROUP BY channel_id;
```

| channel_id | channel_ad_count |
| - | - |
| 1101 | 4932980 |
| 1103 | 20125 |

**查询每个渠道下，每个推广类型的广告数量**
```sql
SELECT channel_id, purpose, COUNT(*) AS purpose_ad_count FROM advertisement GROUP BY channel_id, purpose;
```

| channel_id | purpose | purpose_ad_count |
| - | - | - |
| 1101 | 1 | 742325 |
| 1101 | 2 | 707803 |
| 1101 | 8 | 3482852 |
| 1103 | 1 | 3938 |
| 1103 | 2 | 16187 |

### 排序规则
- 可以使用 `ORDER BY` 来指定查询结果的排列顺序，格式为 结果的字段 + 排序属性
- 排序属性包括 升序 `ASC` 和 降序 `DESC`，不指定则默认升序
- 可以指定多个排序规则，按顺序比较: 如果前一个规则相等，则应用下一个规则
- 多个规则使用逗号 `,` 进行分隔
- 不指定排序规则时，一般会按照 id 升序排列(并不准确)

```sql
-- 新广告在前 (按首次出现时间倒序)，如果首次时间一样，则按照 最后出现时间 降序排列
SELECT * FROM advertisement ORDER BY createdAt DESC, updatedAt DESC;
```

### 联表查询
数据并不总是在一个表里，有时需要从有关联的两张表中查询数据，需要用到 `JOIN` 语句。格式为:
```
SELECT 需要的字段 FROM t1
JOIN t2 ON 联表条件
WHERE ...
```

`JOIN` 也有多种，通常推荐使用 `LEFT JOIN` 来进行联表查询。它比较符合逻辑顺序，以左边的表为基础，连接右边的表。

比如 ad_format 表的结构和数据如下:

| id | adid (广告id) | format (广告形式) |
| - | - | - |
| 1 | 11111 | 106 |
| 2 | 22222 | 102 |
| 3 | 22222 | 106 |

查询广告数据的时候，同时要查它的所有广告形式，就可以这样写:
```sql
SELECT advertisement.*, ad_format.format FROM advertisement  -- 返回广告的全部数据，和它的广告形式
LEFT JOIN ad_format ON advertisement.id=ad_format.adid;  -- 联表条件为: 广告表的 id 等于 ad_format 表的 adid，通常选取含义相同的字段
```

两张表可能有相同的字段名，所以使用 `表名.字段名` 来区分。

因为表名比较长，通常会使用别名来简化语句。下面这条与上面功能是相同的(省略了 `AS`):
```sql
SELECT a.*, b.format FROM advertisement a
LEFT JOIN ad_format b ON a.id=b.adid;
```

最终返回的结果数量，需要看右表中有多少条记录符合联表条件。比如按照前面定义的数据
- 因为广告 11111 只有一个对应的 format，所以只有一条结果
- 广告 22222 有两个对应的 format 102 和 106，所以同样的广告记录被返回了两次，分别连接了对应的 format

最终返回结果应该是:

| id | channel_id | platform | createdAt | purpose | materialType | format |
| - | - | - | - | - | - | - |
| 11111 | 105 | 2 | 2019-10-23 19:33:27 | 2 | 102 | 106 |
| 22222 | 601 | 1 | 2019-10-23 19:53:27 | 4 | 102 | 102 |
| 22222 | 601 | 1 | 2019-10-23 19:53:27 | 4 | 102 | 106 |

#### 多重联表
`JOIN` 语句同样可以多重叠加，通常用在关联多个表的情况。使用方式为:
```
SELECT 需要的字段 FROM t1
JOIN t2 ON 联表条件1
JOIN t3 ON 联表条件2
WHERE ...
```

比如 ad_industry_tag 表存储的是广告和标签的对应关系，industry_tag 存储的是标签数据。

要查询某个广告的数据，同时要返回它的标签名称，可以这样查:
```sql
SELECT a.*, c.name FROM advertisement a  -- 查询广告数据，和标签名称
LEFT JOIN ad_industry_tag b ON a.id=b.adid  -- 使用 广告id 来联接 广告标签的对应关系表
LEFT JOIN industry_tag c ON b.tagid=c.id  -- 使用 标签id 来联接 标签数据
WHERE a.id=11111;
```

### 子查询
一次查询的结果，可以被当作一张临时表，用来进行再次查询。使用方式如下:
```
SELECT 第二次需要的字段 FROM
(SELECT 第一次需要的字段 FROM 原表) AS 临时表
WHERE 筛选条件...;
```

### 例子
#### 查询各一级行业的广告数分布
可以从 广告-标签 的对应关系表入手，先查出 所有一级行业标签 和广告的关联关系。
1. 这一步要查的是关联关系表，所以将 ad_industry_tag 作为左表
2. 筛选条件 **一级** 和 **行业标签** 都是 industry_tag 的属性，所以 industry_tag 是右表，联接条件为 industry_tag.tagid=industry_tag.id
3. 这一步需要的字段是 广告的id 和 标签的id，把它们填入 SELECT 的字段列表中

```sql
SELECT a.adid, a.tagid FROM ad_industry_tag a  -- 最后需要的数据一般放到左表
LEFT JOIN industry_tag b ON a.tagid=b.id
WHERE b.category=1 AND b.parent_id=0;  -- category 为 1 表示行业, parent_id 为 0 表示一级标签
```

在广告-标签对应关系表中筛选出了一级行业标签的数据，接下来就可以统计每个一级标签的广告数量:
1. 将上面的查询结果按照 一级行业标签 来分组
2. 使用 `COUNT` 函数来统计每组的广告数量

```sql
SELECT a.tagid, count(a.adid) FROM ad_industry_tag a
LEFT JOIN industry_tag b ON a.tagid=b.id
WHERE b.category=1 AND b.parent_id=0
GROUP BY a.tagid;
```

待补充更多例子
