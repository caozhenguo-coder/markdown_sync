# **PostgreSQL主从流复制与手动主备切换架构** #

使用PostgreSQL 11.3 创建两个节点：node1 和 node2； 配置主从流复制，然后做手动切换（failover）。为了配置过程简单，两个节点在同一台物理机器上。

首先建立主从同步流复制，起初，node1作为主节点（Primary），node2作为从节点（Standby）。


接下来，模拟主节点(node1)无法工作，把从节点(node2)手动切换成主节点。然后把原来的主节点(node1)作为从节点加入系统。 此时，node2是主 (Primary), node1是从(Standby)。

最后，模拟主节点(node2)无法工作，把从节点(node1)手动切换成主节点。然后把node2作为从节点加入系统。 此时，node1是主 (Primary), node2是从(Standby)。

创建node1和node2，配置主从流复制

上级目录是:

/Users/tom

两个子目录，testdb113,testdb113b, 分别属于两个节点, node1, node2:

testdb113  --> node1
testdb113b --> node2

创建主节点(Primary) node1

mkdir testdb113
initdb -D ./testdb113
pg_ctl -D ./testdb113 start

node1: 创建数据库账号，用于流复制,创建账号 'replicauser'，该账号存在于node1和node2 。

CREATE ROLE replicauser WITH REPLICATION LOGIN ENCRYPTED PASSWORD 'abcdtest';

node1: 创建归档目录

mkdir ./testdb113/archive

node1: 在配置文件中(postgresql.conf)设定参数

#for the primary
wal_level = replica
synchronous_commit = remote_apply
archive_mode = on
archive_command = 'cp %p /Users/tom/testdb113/archive/%f'
max_wal_senders =  10
wal_keep_segments = 10
synchronous_standby_names = 'pgslave001'

#for the standby
hot_standby = on

node1: 编辑文件pg_hba.conf，设置合法地址:

add following to then end of pg_hba.conf
host     replication    replicauser          127.0.0.1/32            md5
host     replication    replicauser          ::1/128                 md5\

node1: 预先编辑文件:recovery.done

注意: 该文件后来才会用到。当node1成为从节点时候，需要把recovery.done重命名为recovery.conf

standby_mode = 'on'
recovery_target_timeline = 'latest'
primary_conninfo = 'host=localhost port=6432 user=replicauser password=abcdtest application_name=pgslave001'
restore_command = 'cp /Users/tom/testdb113b/archive/%f %p'
trigger_file = '/tmp/postgresql.trigger.5432'

创建和配置从节点(node2)

mkdir testdb113b
pg_basebackup  -D ./testdb113b
chmod -R 700 ./testdb113b

node2: 编辑文件postgresql.conf，设定参数,这个文件来自node1(由于使用了pg_basebackup)，只需要更改其中的个别参数。

#for the primary
port = 6432
wal_level = replica
synchronous_commit = remote_apply
archive_mode = on
archive_command = 'cp %p /Users/tom/testdb113b/archive/%f'
max_wal_senders =  2
wal_keep_segments = 10
synchronous_standby_names = 'pgslave001'
#for the standby
hot_standby = on

node2: 建立归档目录,确保归档目录存在

mkdir ./testdb113b/archive

node2: 检查/编辑文件pg_hba.conf,这个文件来自node1，不需要编辑

node2: 创建/编辑文件recovery.conf

PG启动时，如果文件recovery.conf存在，则工作在recovery模式，当作从节点。如果当前节点升级成主节点，文件recovery.conf将会被重命名为recovery.done。

standby_mode = 'on'
recovery_target_timeline = 'latest'
primary_conninfo = 'host=localhost port=5432 user=replicauser password=abcdtest application_name=pgslave001'
restore_command = 'cp /Users/tom/testdb113/archive/%f %p'
trigger_file = '/tmp/postgresql.trigger.6432'

简单测试

重新启动主节点(node1):

pg_ctl -D ./testdb113 restart

启动从节点(node2)

pg_ctl -D ./testdb113b start

在node1上做数据更改,node1支持数据的写和读

psql -d postgres
postgres=# create table test ( id int, name varchar(100));
CREATE TABLE
postgres=# insert into test values(1,'1');
INSERT 0 1

节点node2读数据,node2是从节点(Standby)，只能读，不能写。

psql -d postgres -p 6432
postgres=# select * from test;
 id | name
----+------
  1 | 1
(1 row)

postgres=# insert into test values(1,'1');
ERROR:  cannot execute INSERT in a read-only transaction
postgres=#

模拟主节点node1不能工作时，把从节点node2提升到主节点

停止主节node1,node1停止后，只有node2工作，仅仅支持数据读。

pg_ctl -D ./testdb113 stop

尝试连接到节点node1时失败

因为node1停止了。

Ruis-MacBook-Air:tom$ psql -d postgres
psql: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?

尝试连接到node2，成功；插入数据到node2，失败

node2在工作，但是只读

Ruis-MacBook-Air:~ tom$ psql -d postgres -p 6432
psql (11.3)
Type "help" for help.

postgres=# insert into test values(1,'1');
ERROR:  cannot execute INSERT in a read-only transaction
postgres=#

node2: 把node2提升到主

提升后，node2成为了主(Primary)，既能读，也能写。

touch /tmp/postgresql.trigger.6432

node2: 插入数据到node2，阻塞等待

node2是主节点(Primary)，能够成功插入数据。但是由于设置了同步流复制，node2必须等待某个从节点把数据更改应用到从节点，然后返回相应的LSN到主节点。

postgres=# insert into test values(2,'2');

node1: 创建文件recovery.conf

使用之前建立的文件recovery.done作为模版，快速创建recovery.conf。

mv recovery.done recovery.conf

node1: 启动node1，作为从节点(Standby)

启动后，系统中又有两个节点：node2是主，node1是从。

pg_ctl -D ./testdb113 start

插入数据到node2(见3.3)成功返回

由于节点node1作为从节点加入系统，应用主节点的数据更改，然后返回相应WAL的LSN到主节点，使主节点等到了回应，从而事务处理得以继续。

postgres=# insert into test values(2,'2');
INSERT 0 1
postgres=#

分别在node1和node2读取数据

由于两个节点都只是数据读，而且流复制正常工作，所有读取到相同的结果

ostgres=# select * from test;
 id | name
----+------
  1 | 1
  2 | 2
(2 rows)

模拟node2无法工作时，提升node1成为主(Primary)

停止node2

pg_ctl -D ./testdb113b stop

尝试访问node2，失败

因为node2已经停止了.

Ruis-MacBook-Air:~ tom$ psql -d postgres -p 6432
psql: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?

node1: 连接到node1，成功；插入数据到node1，失败

node1只读

Ruis-MacBook-Air:~ tom$ psql -d postgres
psql (11.3)
Type "help" for help.

postgres=# insert into test values(3,'3');
ERROR:  cannot execute INSERT in a read-only transaction
postgres=#

node1: 提升node1，成为主(Primary)

touch /tmp/postgresql.trigger.5432

node1: 插入数据到node1，阻塞等待
node1成功插入数据，但是要等待从节点的 remote-apply.

postgres=# insert into test values(3,'3');

node2: 创建文件recovery.conf

mv recovery.done recovery.conf

node2: 启动node2，作为从节点(Standby)

pg_ctl -D ./testdb113b start

node1: 插入数据(见4.3)成功返回。

postgres=# insert into test values(3,'3');
INSERT 0 1

分别在node1和node2读取数据

由于两个节点都只是数据读，而且流复制正常工作，所有读取到相同的结果

postgres=# select * from test;
 id | name
----+------
  1 | 1
  2 | 2
  3 | 3
(3 rows)

自动化解决方案

当主节点不能工作时，需要把一个从节点提升成为主节点。 如果是手工方式提升，可以使用trigger文件，也可以使用命令'pg_ctl promote' 。

目前也有很多种自动化监控和切换的方案，在网上都能搜索到。或者，可以自己动手写一个自动化监控和切换的方案。