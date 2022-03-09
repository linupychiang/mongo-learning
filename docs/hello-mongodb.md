
## 1. 安装驱动

```shell
pip install pymongo
```

## 2. 创建连接

MongoDB链接串:

- 格式： `mongodb:数据库服务器主机地址:端口号`
- 例如： `mongodb://127.0.0.1:27017`

初始化数据库连接

```python
from pymongo import MongoClient
uri = 'mongodb://127.0.0.1:27017'
client = MongoClient(uri)
print(client)
```

## 3. 数据库操作

### 插入用户

初始化数据库和集合

```python
db = client['eshop']
user_coll = db['users']
```

插入一条新数据

```python
new_user = {'username':'nina', 'password':'xxxx','email':'123@qq.com'}
result = user_coll.insert_one(new_user)
print(result)
```

> **注意：** 并没有提前创建数据库和表（集合）
