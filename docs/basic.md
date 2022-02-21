## 基本操作
### insert 插入
> mongo shell

```
db.fruit.inserOne({name: "apple"})
db.fruit.inserMany(
	{name: "apple"},
	{name: "pear"},
	{name: "orange"},
)
```