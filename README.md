# PiggyMetrics MongoDB

## 在本地运行MongoDB

在本地以容器方式运行MongoDB:

```bash
# create mongodb data directory in host machine
# replace as your path for container volume
cd /Users/william/work/data
mkdir -p mongodb-data
chmod -R 777 mongodb-data

# run bitnami mongodb container
# replace as your path for container volume
podman run -d --name mymongodb -p 27017:27017 \
  -v /Users/william/work/data/mongodb-data:/bitnami/mongodb \
  -e MONGODB_USERNAME=user \
  -e MONGODB_PASSWORD=password \
  -e MONGODB_ROOT_PASSWORD=password123 \
  -e MONGODB_DATABASE=authdb \
  bitnami/mongodb:latest
```

说明：
- `podman run`命令中的`-v`参数指定了MongoDB数据目录在宿主机的位置，这样可以保证MongoDB容器重启后数据不会丢失。
- `podman run`命令中的`-e`参数指定了MongoDB的用户名、密码、数据库等信息，这样可以在MongoDB容器启动时自动创建数据库和用户。
- `podman run`命令中的`bitnami/mongodb:latest`指定了使用bitnami/mongodb镜像，这个镜像是一个MongoDB的官方镜像，但是在官方镜像的基础上做了一些定制化的修改，比如支持自定义用户名、密码、数据库等信息，这样可以在MongoDB容器启动时自动创建数据库和用户。


参考文档：
- https://hub.docker.com/r/bitnami/mongodb

## 在MongoDB容器中验证

进入MongoDB容器：
```bash
podman exec -it mymongodb /bin/bash
```

在MongoDB容器中验证：
```bash
# login by root
mongosh -u root -p password123
show dbs

# login by user
mongosh -u user -p password --authenticationDatabase authdb
use authdb
show collections
```

## 在宿主机中验证

在VSCode中安装插件`MongoDB for VSCode`，可以在VSCode中直接连接到MongoDB。

添加MongoDB连接：
```bash
mongodb://user:password@localhost:27017/authdb
```

验证可以从MongoDB容器外远程连接MongoDB。

## 创建其他MongoDB database

前面在MongoDB启动时已经创建了一个数据库`authdb`，这个数据库用于存储认证信息。

下面说明如何创建一个新的MongoDB database。

先用root用户登录MongoDB：

```bash
podman exec -it mymongodb /bin/bash
mongosh -u root -p password123
```

创建新的数据库，以accountdb为例：
```bash
# create a new database if not exisits
use accountdb

# create a user/password and grant roles to the user
db.createUser({
  user: "user",
  pwd: "password",
  roles: [ { role: "readWrite", db: "accountdb" } ]
})

# exit
exit
```

验证用户可以访问新创建的数据库：
```bash
mongosh -u user -p password --authenticationDatabase accountdb
use accountdb
show collections
```

在VS Code的MongoDB插件中添加新数据库的连接：

```bash
mongodb://user:password@localhost:27017/accountdb
```

并将连接名为accountdb

