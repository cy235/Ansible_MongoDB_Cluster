# Create a MongoDB Sharded Cluster

## 1. Overview of MongoDB Sharded Cluster
This repo shows how to create a MongoDB sharded cluster with Ansible. All the steps and configuration details are straightforward and easy to understand.    

This figure show the process how end users access the target data in MongoDB servers.
![image](https://github.com/cy235/MongoDB_Cluster/blob/master/MongoDB1.png)
First, the end user put a request to the Mongos servers, then the Mongo servers obtain the sharding configuration from the configuration clusters according to the user's request, and finally the user will access the target data.
## 2. Configuration of MongoDB Sharded Cluster
The following figure show the specific configuration details for MongoDB clusters with 3 nodes/machines,
![image](https://github.com/cy235/MongoDB_Cluster/blob/master/MongoDB2.png)
 where a complete dataset is partutioned into 3 sub sets (e.g. red, cyan and yellow sets in the figure), the replica of subsets are distributed in different machines. No arbiter set is allowed in configuration cluster, while both arbiter and secondary sets are allowed in
replica sets.
## 3. Prerequisite
1. 3 machines A, B, C installed with CentOS, respectively
2. set the static IP for each machine
3. disable the firewall of each machine

set static IP for each machine by modifying the following file:
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
replace `BOOTPROTO="dhcp"` with `BOOTPROTO="static"`, and add your IP address, for example:</br>
machine A:
```
IPADDR=192.168.226.129
GATEWAY=192.168.226.2
DNS1=192.168.226.2
NETMASK=255.255.255.0
```
machine B:
```
IPADDR=192.168.226.131
GATEWAY=192.168.226.2
DNS1=192.168.226.2
NETMASK=255.255.255.0
```
machine C:
```
IPADDR=192.168.226.132
GATEWAY=192.168.226.2
DNS1=192.168.226.2
NETMASK=255.255.255.0
```
disable the firewall by excuting following command in each machine:
```
systemctl disable firewalld
```

## 4. Install MongoDB
Install MongoDB in 3 machines, respectively by excuting the following command lines: 
```
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.10.tgz
tar zxvf mongodb-linux-x86_64-4.0.10.tgz
mv  mongodb-linux-x86_64-4.0.10/   /usr/local/mongodb
```
## 5. Create necessary folders

```
mkdir /data
cd /data
mkdir /data/confsvr /data/log /data/shard1 /data/shard2 /data/shard3
```

whenever there are unexpected errors, you can kill all mongo services, then remove and recreate above folders to avoid conflicts when you rerun the mongo.

You can also creat the above clean folder `/data/` in a control machine, then synchronize this folder to each machines with Ansible:
```
[root@master ansible]# ansible-playbook clean_data_folder.yaml 
```

## 6. Mongod service configuration
create `confsvf.conf` in each machine
```
cd /usr/local/mongodb/bin
mkdir conf
cd conf
vi confsvf.conf 
```
Machine A: </br>
```
#confsvf.conf 
dbpath=/data/confsvr                             
logpath=/data/log/confsvr.log                 
pidfilepath=/data/confsvr.pid            
directoryperdb=true
logappend=true
replSet=ConfSet                   
bind_ip=192.168.226.129
port=27400      
fork=true                   
configsvr=true
```
Machine B: </br>
```
#confsvf.conf 
dbpath=/data/confsvr                             
logpath=/data/log/confsvr.log                 
pidfilepath=/data/confsvr.pid            
directoryperdb=true
logappend=true
replSet=ConfSet                   
bind_ip=192.168.226.131
port=27400      
fork=true                   
configsvr=true
```
Machine C: </br>
```
#confsvf.conf 
dbpath=/data/confsvr                             
logpath=/data/log/confsvr.log                 
pidfilepath=/data/confsvr.pid            
directoryperdb=true
logappend=true
replSet=ConfSet                   
bind_ip=192.168.226.132 
port=27400      
fork=true                   
configsvr=true
```
Then, excuting following commands in 3 machines, respectively
```
cd /usr/local/mongodb/bin 
./mongod -f conf/confsvr.conf
```

## 7. Mongo shards configuration
create `set1svr.conf`, `set2svr.conf`and `set3svr.conf` under folder /usr/local.mongodb/bin/conf/ in machine A:</br>
```
#set1svr.conf
dbpath=/data/shard1                            
logpath=/data/log/shard1.log                 
pidfilepath=/data/shard1.pid            
directoryperdb=true
logappend=true
replSet=rs1                 
bind_ip=192.168.226.129 
port=27101     
fork=true                   
shardsvr=true
```
```
#set2svr.conf
dbpath=/data/shard2                            
logpath=/data/log/shard2.log                 
pidfilepath=/data/shard2.pid            
directoryperdb=true
logappend=true
replSet=rs2                 
bind_ip=192.168.226.129 
port=27102     
fork=true                   
shardsvr=true
```
```
#set3svr.conf
dbpath=/data/shard3                            
logpath=/data/log/shard3.log                 
pidfilepath=/data/shard3.pid            
directoryperdb=true
logappend=true
replSet=rs3                 
bind_ip=192.168.226.129 
port=27103     
fork=true                   
shardsvr=true
```
create `set1svr.conf`, `set2svr.conf`and `set3svr.conf` under folder /usr/local.mongodb/bin/conf/ in machine B:</br>
```
#set1svr.conf
dbpath=/data/shard1                            
logpath=/data/log/shard1.log                 
pidfilepath=/data/shard1.pid            
directoryperdb=true
logappend=true
replSet=rs1                 
bind_ip=192.168.226.131 
port=27101     
fork=true                   
shardsvr=true
```
```
#set2svr.conf
dbpath=/data/shard2                            
logpath=/data/log/shard2.log                 
pidfilepath=/data/shard2.pid            
directoryperdb=true
logappend=true
replSet=rs2                 
bind_ip=192.168.226.131 
port=27102     
fork=true                   
shardsvr=true
```
```
#set3svr.conf
dbpath=/data/shard3                            
logpath=/data/log/shard3.log                 
pidfilepath=/data/shard3.pid            
directoryperdb=true
logappend=true
replSet=rs3                 
bind_ip=192.168.226.131 
port=27103     
fork=true                   
shardsvr=true
```
create `set1svr.conf`, `set2svr.conf`and `set3svr.conf` under folder /usr/local.mongodb/bin/conf/ in machine C:</br>
```
#set1svr.conf
dbpath=/data/shard1                            
logpath=/data/log/shard1.log                 
pidfilepath=/data/shard1.pid            
directoryperdb=true
logappend=true
replSet=rs1                 
bind_ip=192.168.226.132
port=27101     
fork=true                   
shardsvr=true
```
```
#set2svr.conf
dbpath=/data/shard2                            
logpath=/data/log/shard2.log                 
pidfilepath=/data/shard2.pid            
directoryperdb=true
logappend=true
replSet=rs2                 
bind_ip=192.168.226.132 
port=27102     
fork=true                   
shardsvr=true
```
```
#set3svr.conf
dbpath=/data/shard3                            
logpath=/data/log/shard3.log                 
pidfilepath=/data/shard3.pid            
directoryperdb=true
logappend=true
replSet=rs3                 
bind_ip=192.168.226.132 
port=27103     
fork=true                   
shardsvr=true
```
You can also do step 6 and step 7 with Ansible:
```
[root@master ansible]# ansible-playbook mongo_config.yaml
```

## 8. Start mongod Services for Each Machine
Then, run the following in 3 different machines
./mongod -f conf/set1svr.conf
./mongod -f conf/set2svr.conf
./mongod -f conf/set3svr.conf

You can also do it with Ansible:
```
[root@master ansible]# ansible-playbook mongo_run.yaml 
```

## 9. Build Connections for Shards in Each Machine
In the next, we connect the nodes according to the following table:
```
A: 192.168.226.129
B: 192.168.226.131
C: 912.168.226.132
A: mongos   config(primary)27400     setA(primary)     setB (secondary)   setC(arbiter)
B: mongos   config(secondary)27400   setA(arbiter)     setB (primary)     setC(secondary)
C: mongos   config(secondary)27400   setA(secondary)   setB (arbiter)     setC(primary)
```
In the machine A: </br>
```
./mongo 192.168.226.129:27101

rs1:PRIMARY> rs.initiate()
rs.add("192.168.226.132:27101")
rs.add({host:"192.168.226.131:27101",arbiterOnly:true})
```
In the machine B: </br>
```
./mongo 192.168.226.131:27102
rs1:PRIMARY> rs.initiate()
rs.add("192.168.226.129:27102")
rs.add({host:"192.168.226.132:27102",arbiterOnly:true})
```
In the machine C:</br>
```
./mongo 192.168.226.132:27103
rs1:PRIMARY> rs.initiate()
rs.add("192.168.226.131:27103")
rs.add({host:"192.168.226.129:27103",arbiterOnly:true})
```
You can also do it with Ansible:
```
[root@master ansible]# ansible-playbook mongo_connect.yaml 
```

## 10. Mongos service configuration

select a machine, e.g., machine A, and create a file mongossvr.conf under the folder /usr/local/mongodb/bin/conf
```
#mongossvr.conf
logpath=/data/log/mongossvr.log                 
pidfilepath=/data/mongossvr.pid            
logappend=true
bind_ip=192.168.226.129 # the IP of the selected machine
port=20000   
fork=true                   
configdb=ConfSet/192.168.226.129:27400, 192.168.226.131:27400, 192.168.226.132:27400
```

start mongos:
```
nohup ./mongos -f conf/mongossvr.conf &
```
connect mongos:
```
cd /usr/local/mongodb/bin
./mongo 192.168.226.129:20000
mongos> use admin  # this step is essential
db.runCommand({addshard: 'rs1/192.168.226.129:27101, 192.168.226.131:27101, 192.168.226.132:27101'})
db.runCommand({addshard: 'rs2/192.168.226.129:27102, 192.168.226.131:27102, 192.168.226.132:27102'})
db.runCommand({addshard: 'rs3/192.168.226.129:27103, 192.168.226.131:27103, 192.168.226.132:27103'})
```

