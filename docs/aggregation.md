聚合是MongoDB中一个非常实用的功能，类似于SQL中的 `where` `group by` 等操作。

可以做分析型、统计型的场景，也可以用来做复杂数据的计算和处理。

## 聚合框架介绍

MongoDB聚合框架（Aggregation Framework）是一个计算框架，它可以：

- 作用在一个或几个集合上
- 对集合中的数据进行一系列运算
- 将这些数据转化为期望的格式

从效果而言，聚合框架相当于SQL查询中的：

- GROUP BY
- LEFT/OUTER/RIGHT/INNER JOIN
- AS等

## 管道（Pipeline）步骤（Stage）

整个聚合运算过程称为管道（Pipeline），它由多个步骤（Stage）组成。
每个管道：

- 接收一系列文档（原始数据）
- 每个步骤对这些文档进行一系列运算
- 结果文档输出到下一步骤

*原始数据* --`步骤1`--> *中间结果1* --`步骤2`--> *中间结果2* --`步骤...`--> *最终结果*

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

| 步骤         | 作用     | SQL等价运算符   |
| ------------ | -------- | --------------- |
| $match       | 过滤     | WHERE           |
| $project     | 投影     | AS              |
| $sort        | 排序     | ORDER BY        |
| $group       | 分组     | GROUP BY        |
| $skip/$limit | 结果限制 | SKIP/LIMIT      |
| $lookup      | 左外连接 | LEFT OUTER JOIN |
