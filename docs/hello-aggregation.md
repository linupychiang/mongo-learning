## 导入测试数据

测试数据地址：[aggregation-data](https://github.com/tapdata/
geektime-mongodb-course/raw/master/aggregation/dump.tar.gz)

导入方法：

```shell
wget https://github.com/tapdata/geektime-mongodb-course/raw/master/aggregation/dump.tar.gz
tar -xvf dump.tar.gz
mongorestore -h localhost:27017
```

## 聚合实验：总销量

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
