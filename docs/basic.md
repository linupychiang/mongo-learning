## 基本操作

### insert 插入

> mongo shell

#### insertOne

```javascript
db.fruit.insertOne({ name: "apple" });
```

#### insertMany

```javascript
db.fruit.insertMany({ name: "apple" }, { name: "pear" }, { name: "orange" });
```

### find 查询文档

1. `find` 是 MongoDB 中查询数据的基本指令，相当于 SQL 中的 `select`
2. `find` 返回的是游标

> 示例

#### 单条件

```javascript
db.movie.find({ year: 1975 });
```

#### 多条件 and

```javascript
db.movie.find({ year: 1975, title: "batman" });
```

#### 另一种 and

```javascript
db.movie.find({ $and: [{ year: 1975, title: "batman" }] });
```

#### 多条件 or

```javascript
db.movie.find({$or:[{"year":1975,"title":"batman"}]}})
```

#### 正则查询

```javascript
db.movie.find({ title: /^B/ }); // B开头
```

> 查询条件对照表

| SQL             | MQL                             | 解释                     |
| --------------- | ------------------------------- | ------------------------ |
| a = 1           | {a:1}                           |                          |
| a <> 1          | {a:{$ne:1}}                     | not equal                |
| a > 1           | {a:{$gt:1}}                     | greater than             |
| a >= 1          | {a:{$gte:1}}                    | greater than or equal to |
| a < 1           | {a:{$lt:1}}                     | less than                |
| a <= 1          | {a:{$lte:1}}                    | less than or equal to    |
| a = 1 AND b = 1 | {a:1,b:1} 或 {$and:[{a:1,b:1}]} |                          |
| a = 1 OR b = 1  | {$or:[{a:1,b:1}]}               |                          |
| a is NULL       | {a:{$exists:false}}             |                          |
| a IN (1,2,3)    | {a:{$in:[1,2,3]}}               |                          |

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

```javascript
db.fruit.insertOne({
  name: "apple",
  from: {
    country: "China",
    province: "GuangDong",
  },
});
```

两种查询方式，第一种为查询子文档, 能够正常查到数据

```javascript
db.fruit.find({ "from.county": "China" });
```

第二种为，查询 `from` 字段为 `{"country":"China"}`, 所以查不到数据

```javascript
db.fruit.find({ from: { country: "China" } });
```

### find 搜索数组

find 支持对数组中元素进行搜索，假设文档

```javascript
db.fruit.insert([
  { name: "Apple", color: ["red", "green"] },
  { name: "Mango", color: ["yellow", "green"] },
]);
```

两种查询方式，第一种查询，`color` 为 `red`，返回 1 条数据

```javascript
db.fruit.find({ color: "red" });
```

第二种查询，查询 `color` 为 `red` 或 `yellow`，返回 2 条数据

```javascript
db.fruit.find({ $or: [{ color: "red" }, { color: "yellow" }] });
```

### find 搜索数组中的对象

可以搜索数组中的对象，假设文档

```javascript
db.movies.insertOne({
  title: "Raiders of the Lost Ark",
  filming_locations: [
    { city: "Los Angeles", state: "CA", contry: "USA" },
    { city: "Rome", state: "Lazio", contry: "Italy" },
    { city: "Florence", state: "SC", contry: "USA" },
  ],
});
```

查找城市是 `Rome` 的记录

```javascript
db.movies.find({ "filming_locations.city": "Rome" });
```

#### $elemMatch

> 当在数组中搜索子对象的多个字段，可以使用 `$elemMatch`。表示必须是同一个子对象满足多个条件

比如，要查询 `city` 是 `Rome` ，并且 `contry` 是 `USA` 时

```javascript
db.movies.find({
  "filming_locations.city": "Rome",
  "filming_locations.contry": "USA",
});
```

```javascript
db.movies.find({
  filming_locations: {
    $elemMatch: {
      city: "Rome",
      contry: "USA",
    },
  },
});
```

### 控制 find 返回的字段

当 find 返回内容较大或不需要返回全部字段时，**可以指定只返回指定的字段**

- `_id` 字段默认返回，如不需要返回，需要明确指明不返回
- 在 MongoDB 中称为投影（projection）

假设只需要返回 `title` 字段且不返回 `_id`

```javascript
db.movies.find({ category: "action" }, { _id: 0, title: 1 });
```

### remove 删除文档

- remove 命令需要配合查询条件使用
- 匹配查询条件的文档会被删除
- **指定一个空文档会删除所有文档**

示例命令：

```javascript
// 删除a=1的记录
db.test.remove({ a: 1 });
// 删除a<1的记录
db.test.remove({ a: { $lt: 5 } });
// 删除所有记录
db.test.remove({});
// 报错
db.test.remove;
```

### update 更新文档

update 需要满足以下条件：

- updateOne 无论匹配多少条记录，始终只更新一条
- updateMany 匹配多少条，就更新多少条
- updateOne/updateMany 要求更新条件部分必须具有以下操作符之一，否则报错
    - $set/$unset
    - $push/$pushAll/$pop
    - $pull/$pullAll
    - $addToSet

如下语句，将会报错

```javascript
db.fruit.updateOne({name:"apple"},{from:"China"})
```
#### 操作符解释

| 操作符    | 解释                                     |
| --------- | ---------------------------------------- |
| $push     | 增加一个对象到数组底部                   |
| $pushAll  | 增加多个对象到数组底部                   |
| $pop      | 从数组底部删除一个对象                   |
| $pull     | 如果匹配指定的值，从数组中删除相应的对象 |
| $pullAll  | 如果匹配任意的值，从数据中删除相应的对象 |
| $addToSet | 如果不存在，则增加一个值到数组           |

### drop 删除集合
- 使用 `db.dropDatabase()` 来删除数据库
- 数据库相应文件也会被删除，磁盘空间将被释放

```javascript
db.dropDatabase()
// {"dropped":"test","ok":1}
```
