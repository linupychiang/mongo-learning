## 导入测试数据

测试数据地址：[aggregation-data](https://github.com/tapdata/
geektime-mongodb-course/raw/master/aggregation/dump.tar.gz)

导入方法：

```shell
wget https://github.com/tapdata/geektime-mongodb-course/raw/master/aggregation/dump.tar.gz
tar -xvf dump.tar.gz
mongorestore -h localhost:27017
```

## 聚合实验一：总销量

计算到目前为止，所有订单的总销量

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: null,
      total: {
        $sum: "$total",
      },
    },
  },
]);
```

结果为：

```json
{"_id": null,"total": NumberDecimal("44019609")}
```

## 聚合实验二：订单金额汇总

计算 2019 年第一季度（1 月 1 日至 3 月 31 日）已完成的（completed）订单总金额和订单总数

```javascript
db.orders.aggregate([
  //步骤1：匹配条件
  {
    $match: {
      status: "completed",
      orderDate: {
        $gte: ISODate("2019-01-01T00:00:00.000Z"),
        $lt: ISODate("2019-04-01T00:00:00.000Z"),
      },
    },
  },
  // 步骤2：聚合订单总金额，总运费，总数量
  {
    $group: {
      _id: null,
      total: {
        $sum: "$total",
      },
      shippingFee: {
        $sum: "$shippingFee",
      },
      count: {
        $sum: 1,
      },
    },
  },
  // 计算总金额
  {
    $project: {
      grandTotal: {
        $add: ["$total", "$shippingFee"],  // 引用已有的数据
      },
      count: 1, // 1 包括（project中的使用规范）
      _id: 0, // 0 不包括（project中的使用规范）
    },
  },
]);
```

结果为：

```json
{ "count": 5875, "grandTotal": NumberDecimal("2636376.0") }
```
