1.1  mongodb准备初始化副本集语句
```

#模板如下:
 rs.initiate({
    _id: "name",
    members: [
        { _id : 0, host : "ip:端口" },
        { _id : 1, host : "ip:端口" },
        { _id : 2, host : "ip:端口" }
    ]
});
#############################################################
# 修改 _id  ip 端口 3个参数
#   _id : "mongos" 的 mongos 和 docker-compose 中  --replSet mongos  一致
# 这里我设置为 mongos
####################################################
  rs.initiate({
    _id: "mongos",
    members: [
        { _id : 0, host : "10.65.18.115:27017" },
        { _id : 1, host : "10.65.18.116:27017" },
        { _id : 2, host : "10.65.18.117:27017" }
    ]
});
```
 
1.2 操作 （只需要在任意一台节点上初始化）


```
root@44ec45bd63e1:/# mongo -u root -p 密码
MongoDB shell version v4.4.5
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("0a61f5a7-f9b1-4fcb-802f-d8bc9565c957") }
MongoDB server version: 4.4.5

> rs.initiate({
...     _id: "mongos",
...     members: [
...         { _id : 0, host : "10.64.18.118:27017" },
...         { _id : 1, host : "10.64.18.119:27017" },
...         { _id : 2, host : "10.64.18.120:27017" }
...     ]
... });

{ "ok" : 1 }


#执行初始化配置命令，显示{ "ok" : 1 }就代表已经成功了，等待一会儿就会选举出PRIMARY节点，可以用rs.status()查看当前副本集状态。
1.3 增加节点
# 在新的服务器上部署并启动容器，部署的时候需要将`mongodb.key`和`.env`文件一起复制过去，端口根据实际情况修改映射
  `docker-compsoe.yaml`文件
# 启动后  在mongedb节点运行

rs.add("10.15.0.83:27017")
1.4 创建数据库
#假设用户：graylog  密码：xxxxxxxx
#----语句1
rs0:PRIMARY> use graylog     
switched to db graylog
#----语句2
rs0:PRIMARY> db.createUser( {
... user: "graylog",
... pwd: "xxxxxxxxx",
... roles: [ { role: "readWrite", db: "graylog" } ]
... });      
#----语句3
rs0:PRIMARY> db.grantRolesToUser( "graylog" , [ { role: "dbAdmin", db: "graylog" } ])
#----语句4
rs0:PRIMARY> show users
#----语句5  测试登录
rs0:PRIMARY> db.auth("graylog","xxxxxxxx")
```
