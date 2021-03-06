[TOC]



在mongodb中有一个admin数据库，牵涉到服务器配置层面的操作，需要先切换到admin数据库，use admin命令相当于进入超级用户管理模式。

mongo的用户是以数据库为单位来建立的，每个数据库有自己的管理员。

我们在设置用户时，需要先在admin数据库下建立管理员，这个管理员登陆后，相当于超级管理员，然后可以切换到其他库，添加普通用户。

注意: mongodb服务器启动时，默认是不需要认证的，要让用户生效，需要启动服务器时指定–auth选项。添加用户后，我们再次退出并登陆，认证才有效。

# 1. 创建管理员

```shell
# (1)创建用户管理员(在管理身份验证数据库)。
use admin
db.createUser(
  {
    user: "admin",
    pwd: "xxx",
    roles: [{role: "userAdminAnyDatabase", db: "admin"}]
  }
)

# (2)重新启动MongoDB实例与访问控制。
mongod --dbpath /data/mongodb/database --logpath /data/mongodb/log/mongodb.log --port 27017 --fork --auth

# (3)连接和用户管理员进行身份验证。
mongo --port 27017 -u "krislin" -p "xxx" --authenticationDatabase "admin"
```

# 2. 创建自定义用户

```shell
# (1) 给其他数据库test配置一个读写权限的用户
use test
db.createUser(
  {
    user: "myTester",
    pwd: "xyz123",
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)

# (2)mytest连接和验证。
mongo --port 27017 -u "myTester" -p "xyz123" --authenticationDatabase "test"
# 或登录后执行命令验证
use test
db.auth("myTester","xyz123")
```

# 3. 查看已经创建的用户

```shell
# 查看创建的用户需要在管理员数据库上执行
use admin
db.system.users.find()

# 查看当前数据库用户
show users
```

# 4. 删除用户

```shell
# 删除一个用户
use dbname
db.system.users.remove({user:"username"})

# 删除管理员
use admin
db.system.users.remove({user:"admin"})
```