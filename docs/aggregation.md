聚合是 MongoDB 中一个非常实用的功能，类似于 SQL 中的 `where` `group by` 等操作。

可以做分析型、统计型的场景，也可以用来做复杂数据的计算和处理。

## 聚合框架介绍

MongoDB 聚合框架（Aggregation Framework）是一个计算框架，它可以：

- 作用在一个或几个集合上
- 对集合中的数据进行一系列运算
- 将这些数据转化为期望的格式

从效果而言，聚合框架相当于 SQL 查询中的：

- GROUP BY
- LEFT/OUTER/RIGHT/INNER JOIN
- AS 等

## 管道（Pipeline）步骤（Stage）

整个聚合运算过程称为管道（Pipeline），它由多个步骤（Stage）组成。
每个管道：

- 接收一系列文档（原始数据）
- 每个步骤对这些文档进行一系列运算
- 结果文档输出到下一步骤

_原始数据_ --`步骤1`--> _中间结果 1_ --`步骤2`--> _中间结果 2_ --`步骤...`--> _最终结果_

## 聚合运算的基本格式

```javascript
pipline = [$stage1, $stage2, ...$stageN]

db.<COLLECTION>.aggregate(
 pipeline,
 { options }
)
```

定义好每个 `stage`，做什么样的事情，格式为 `json`

### 常见步骤

| 步骤         | 作用                                                      | SQL 等价运算符  |
| ------------ | --------------------------------------------------------- | --------------- |
| $match       | 过滤                                                      | WHERE           |
| $project     | 投影                                                      | AS              |
| $sort        | 排序                                                      | ORDER BY        |
| $group       | 分组                                                      | GROUP BY        |
| $skip        | 结果限制                                                  | SKIP            |
| $limit       | 结果限制                                                  | LIMIT           |
| $lookup      | 左外连接                                                  | LEFT OUTER JOIN |
| $unwind      | 展开数组（较少）(将 Mongo 中的数组，展开变成一行行的数据) | N/A             |
| $graphLookup | 图搜索（较少）                                            | N/A             |
| $facet       | 分面搜索（较少）                                          | N/A             |
| $bucket      | 分面搜索（较少）                                          | N/A             |

#### 常见步骤中的操作符

##### $match

- `$eq`、`$gt` 、`$gte`、`$lt`、`$lte`
- `$and`、`$or`、`$not`、`$in`
- `geoWithin`、`¥intersect`

##### $project

- 选择需要的或排除不需要的字段
- `$map`、`$reduce`、`$filter`
- `$range`
- `$multiply`、`$divide`、`$substract`、`$add`
- `$year`、`$month`、`$dayOfMonth`、`$hour`、`$minute`、`$second`
- ...

##### $group

- `$sum`、`$avg`
- `$push`、`$addToSet`
- `$first`、`$last`、`$max`、`$min`
- ...

### 使用场景

聚合查询可以用于 OLAP 和 OLTP 场景。如：

| OLTP | OLAP                                                                                                               |
| ---- | ------------------------------------------------------------------------------------------------------------------ |
| 计算 | 分析一段时间内的销售额、均值<br>计算一段时间内的净利润<br>分析购买人的年龄分布<br>分析学生成绩分布<br>统计员工绩效 |

#### MQL 与 SQL 对比

> 对比 1:

```sql
SELECT
  FIRST_NAME AS '名',
  LAST_NAME AS '姓',
FROM Users
WHERE GENDER = '男'
SKIP 100
LIMIT 20
```

```javascript
db.users.aggregate([
  { $match: { gender: "男" } },
  { $skip: 100 },
  { $limit: 20 },
  {
    $project: {
      名: "$first_name",
      姓: "$last_name",
    },
  },
]);
```

> 对比 2

```sql
SELECT DEPARTMENT
  COUNT(NULL) AS EMP_QTY
FROM USERS
WHERE GENDER = '女'
GROUP BY DEPARTMENT HAVING
COUNT(*) < 10
```

```javascript
db.users.aggregate([
  { $match: { gender: "女" } },
  {
    $group: {
      _id: "$DEPARTMENT",
      emp_qty: { $sum: 1 },
    },
  },
  { $match: { emp_qty: { $lt: 10 } } },
]);
```

> MQL 特有步骤 $unwind

```javascript
db.students.findOne();
```

结果

```json
{
  "name": "张三",
  "score": [
    { "subject": "语文", "score": 84 },
    { "subject": "数学", "score": 90 },
    { "subject": "外语", "score": 69 }
  ]
}
```

> unwind

```javascript
db.students.aggregate([{ $unwind: "$score" }]);
```

结果

```json
{ "name": "张三", "score": { "subject": "语文", "score": 84 }}
{ "name": "张三", "score": { "subject": "数学", "score": 90 }}
{ "name": "张三", "score": { "subject": "外语", "score": 69 }}
```

> MQL 特有步骤 $bucket

<table>
    <tr>
        <td rowspan="5">price</td>
        <td>0-10</td></td>
        <td>120条</td>
    </tr>
    <tr>
        <td>10-20</td>
        <td>20条</td>
    </tr>
    <tr>
        <td>20-30</td>
        <td>30条</td>
    </tr>
    <tr>
        <td>30-40</td>
        <td>5000条</td>
    </tr>
    <tr>
        <td>40-</td>
        <td>10条</td>
    </tr>
</table>

使用 MQL

```javascript
db.products.aggregate([
  {
    $bucket: {
      groupBy: "$price",
      boundaries: [0, 10, 20, 30, 40],
      default: "other",
      output: { count: { $sum: 1 } },
    },
  },
]);
```

> MQL 特有步骤 $facet

<table>
    <tr>
        <td rowspan="7">数据</td>
    </tr>
    <tr>
        <td rowspan="3">price</td>
        <td>0-10</td>
        <td>20条</td>
    </tr>
    <tr>
        <td>10-20</td>
        <td>20条</td>
    </tr>
    <tr>
        <td>other</td>
        <td>30条</td>
    </tr>
    <tr>
        <td rowspan="3">year</td>
        <td>1890-1901</td>
        <td>80条</td>
    </tr>
    <tr>
        <td>1901-1920</td>
        <td>30条</td>
    </tr>
    <tr>
        <td>other</td>
        <td>40条</td>
    </tr>
</table>

使用 MQL

```javascript
db.prioducts.aggregate([{
  $facet:{
    price:{
      $bucket:{...}
    },
    year:{
      $bucket:{...}
    }
  }
}])
```
