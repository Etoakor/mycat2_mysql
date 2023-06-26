# mycat2_mysql


systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

cat /etc/selinux/config
vi /etc/selinux/config
SELINUX=disabled
重启机器
getenforce



rpm -qa|grep mysql
rpm -qa | grep mariadb
rpm -e --nodeps  mariadb-libs-5.5.60-1.el7_5.x86_64

rpm -Uvh --force --nodeps *.rpm

tar -vxf   ‘需要解压的mysql包’

rpm -ivh mysql-community-common-8.0.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-8.0.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-icu-data-files-8.0.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.28-1.el7.x86_64.rpm  --nodeps --force

mysqld -V

systemctl  start mysqld   
systemctl  status mysqld
systemctl enable mysqld   
systemctl  restart mysqld

grep -i password /var/log/mysqld.log  查看mysqld初始密码

mysql -uroot -p原始密码   进入mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'dgsgD@2023';     登陆后原始密码过期需要修改新密码

create user  'root'@'%' identified by 'dgsgD@2023';
grant all privileges on *.* to root@'%' with grant option;   远程授权登录



四台机器，双主双从

#主服务器唯一ID 
server-id=1
#启用二进制日志

log-bin=mysql-bin
# 设置不要复制的数据库(可设置多个)binlog-ignore-db=mysql binlog-ignore-db=information_schema
#设置需要复制的数据库 binlog-do-db=需要复制的主数据库名字
#设置logbin格式 binlog_format=STATEMENT
# 在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates
#表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是 1 .. 65535
auto-increment-increment=2
# 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
auto-increment-offset=1

server-id=2
#启用中继日志 
relay-log=mysql-relay



#主服务器唯一ID 
server-id=3
#启用二进制日志
log-bin=mysql-bin
# 设置不要复制的数据库(可设置多个)binlog-ignore-db=mysql binlog-ignore-db=information_schema
#设置需要复制的数据库 binlog-do-db=需要复制的主数据库名字
#设置logbin格式binlog_format=STATEMENT
# 在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates
#表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是 1 .. 65535
auto-increment-increment=2
# 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
auto-increment-offset=2
 
 
 server-id=4
#启用中继日志 
relay-log=mysql-relay
 
 
 #在主机MySQL里执行授权命令
CREATE USER 'slave'@'%' IDENTIFIED BY 'Slave@2023';
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%'; 
#此语句必须执行。否则见下面。
ALTER USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY 'Slave@2023';


show master status;

CHANGE MASTER TO MASTER_HOST='188.102.255.52', MASTER_USER='slave', MASTER_PASSWORD='Slave@2023',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=989;

CHANGE MASTER TO MASTER_HOST='188.102.255.54', MASTER_USER='slave', MASTER_PASSWORD='Slave@2023',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=990;




CHANGE MASTER TO MASTER_HOST='188.102.255.52', MASTER_USER='slave', MASTER_PASSWORD='Slave@2023',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=989;

CHANGE MASTER TO MASTER_HOST='188.102.255.54', MASTER_USER='slave', MASTER_PASSWORD='Slave@2023',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=990;





---mycat2使用

下载对应的 tar 安装包,以及对应的 jar 包
tar(zip)包 : http://dl.mycat.org.cn/2.0/install-template/mycat2-install-template-1.22.zip
jar包 :http://dl.mycat.org.cn/2.0/1.22-release/ (下载最新的jar包) 下载所需的 mycat2 的 fat jar 一般大小为 100mb 的一个 jar 文件 把jar 放进解压的 tar 中的 mycat\lib 文件夹下

上传服务器并修改成最高权限，否则运行启动命令时，会因权限不足而报错

1、在mycat连接的mysql数据库里添加用户

CREATE USER 'mycat'@'%' IDENTIFIED WITH mysql_native_password BY 'Mycat@2023';
GRANT XA_RECOVER_ADMIN ON *.* TO 'root'@'%'; 
GRANT ALL PRIVILEGES ON *.* TO 'mycat'@'%' ; 
flush privileges;

cd mycat/bin 
./mycat start 启动
./mycat stop 停止
./mycat console 前台运行
./mycat restart 重启服务
./mycat pause 暂停
./mycat status 查看启动状态...


此登录方式用于管理维护 Mycat
mysql -umycat -pMycat@2023 -P 9066


此登录方式用于通过 Mycat 查询数据，我们选择这种方式访问 Mycat
mysql -umycat -pMycat@2023 -P 8066


dbeaver登录使用/mycat/conf/users下的用户密码

cp prototypeDs.datasource.json master01.datasource.json
cp prototypeDs.datasource.json master02.datasource.json
cp prototypeDs.datasource.json slave01.datasource.json
cp prototypeDs.datasource.json slave02.datasource.json

修改对应的IP 用户 密码 数据库


cp prototype.cluster.json master-slave.cluster.json

{
    // 集群类型：SINGLE_NODE（单节点）、MASTER_SLAVE（普通主从）、GARELA_CLUSTER（garela cluster/PXC集群）等
    "clusterType":"MASTER_SLAVE",
    "heartbeat":{
        "heartbeatTimeout":1000,
        "maxRetry":3,
        "minSwitchTimeInterval":300,
        "slaveThreshold":0
    },
    "masters":[
        // 主节点数据源名称
        "master01",
        "master02"
    ],
    "replicas":[
        // 从节点数据源名称
        "slave01",
        "slave02",
		    "master02"
    ],
    "maxCon":200,
    // 集群名称。在后面配置物理库（schema）时会用到
    "name":"master-slave",
    //查询负载均衡策略
    "readBalanceType":"BALANCE_ALL_READ",
    // NOT_SWITCH（不进行主从切换）、SWITCH（进行主从切换）
    "switchType":"NOT_SWITCH"
}


vi master_slave.schema.json

{
    "customTables" :{},
    "globalTables":{},
    "normalProcedures":{},
    "normalTables":{},
    "schemaName" : "hbisdt",
    "targetName":"master-slave",
    "shardingTables":{},
    "views":{}
}


vi master_slave2.schema.json

{
    "customTables" :{},
    "globalTables":{},
    "normalProcedures":{},
    "normalTables":{},
    "schemaName" : "test",
    "targetName":"master-slave",
    "shardingTables":{},
    "views":{}
}

