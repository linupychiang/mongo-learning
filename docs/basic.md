## 基本操作

### insert 插入

> mongo shell

#### insertOne

```shell
db.fruit.inserOne({name: "apple"})
```

#### insertMany

```shell
db.fruit.inserMany(
    {name: "apple"},
    {name: "pear"},
    {name: "orange"},
)
```

### find 查询文档

1. `find` 是MongoDB中查询数据的基本指令，相当于SQL中的 `select`
2. `find` 返回的是游标

> 示例

#### 单条件

```shell
db.movie.find({"year":1975})
```

#### 多条件and

```shell
db.movie.find({"year":1975,"title":"batman"})
```

#### 另一种and

```shell
db.movie.find({$and:[{"year":1975,"title":"batman"}]})
```

#### 多条件or

```shell
db.movie.find({$or:[{"year":1975,"title":"batman"}]}})
```

#### 正则查询

```shell
db.movie.find({"title":/^B/})  // B开头
```

> 查询条件对照表

|SQL|MQL|解释|
|--|--|--|
|a = 1|{a:1}||
|a <> 1|{a:{$ne:1}}|not equal|
|a > 1|{a:{$gt:1}}|greater than|
|a >= 1|{a:{$gte:1}}|greater than or equal to|
|a < 1|{a:{$lt:1}}|less than|
|a <= 1|{a:{$lte:1}}|less than or equal to|
|a = 1 AND b = 1|{a:1,b:1} 或 {$and:[{a:1,b:1}]}||
|a = 1 OR b = 1|{$or:[{a:1,b:1}]}||
|a is NULL|{a:{$exists:false}}||
|a IN (1,2,3)|{a:{$in:[1,2,3]}}||

> 查询逻辑运算符

- $lt: 存在并小于
- $lte: 存在并小于等于
- $gt: 存在并大于
- $gte: 存在并大于等于
- $ne: 不存在或存在但不等于
- $in: 存在并在指定数组中
- $nin: 不存在或不在指定数组中
- $or: 匹配两个或多个条件
- $and: 匹配全部条件

### find 搜索子文档

支持使用 `field.sub_field` 形式查询子文档，假设文档

```shell
db.fruit.insertOne({
    name:"apple",
    from:{
        country:"China",
        province:"GuangDong"
    }
})
```

两种查询方式，第一种为查询子文档, 能够正常查到数据

```shell
db.fruit.find({"from.county":"China"})
```

第二种为，查询 `from` 字段为 `{"country":"China"}`, 所以查不到数据

```shell
db.fruit.find({"from":{"country":"China"}})
```

### find 搜索数组

find 支持对数组中元素进行搜索，假设文档

```shell
db.fruit.insert([
    {name:"Apple", color:["red","green"]},
    {name:"Mango", color:["yellow","green"]},
])
```

两种查询方式，第一种查询，`color` 为 `red`，返回1条数据

```shell
db.fruit.find({color:"red"})
```

第二种查询，查询 `color` 为 `red` 或 `yellow`，返回2条数据

```shell
db.fruit.find({$or:[{color:"red"},{color:"yellow"}]})
```

### find 搜索数组中的对象

#### $eleMatch
