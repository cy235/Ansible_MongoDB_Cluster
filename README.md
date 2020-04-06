# Create a MongoDB sharded cluster

## Prerequisite
1. 3 machines A, B, C installed with CentOS 7, respectively
2. set the static IP for each machine
3. disable the firewall of each machine
set static IP for each machine:
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
replace `BOOTPROTO=”dhcp”` with `BOOTPROTO=”static”`, and add your IP address, for example:</br>
my machine A:
```
IPADDR=192.168.226.130
GATEWAY=192.168.226.2
DNS1=192.168.226.2
NETMASK=255.255.255.0
```
my machine B:
```
IPADDR=192.168.226.131
GATEWAY=192.168.226.2
DNS1=192.168.226.2
NETMASK=255.255.255.0
```
my machine C:
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

##  Install MongoDB
Install MongoDB in 3 machines, respectively by excuting the following command lines: 
```
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.10.tgz
tar zxvf mongodb-linux-x86_64-4.0.10.tgz
mv  mongodb-linux-x86_64-4.0.10/   /usr/local/mongodb
```
## Create necessary folders

```
mkdir /data
cd /data
mkdir /data/confsvr /data/log /data/shard1 /data/shard2 /data/shard3
```

whenever there are unexpected errors, you can killall mongo services and remove and recreate above folders to avoid conflicts when you rerun the mongo.

## Mongod service configuration
create `confsvf.conf` in each machine
```
cd /usr/local/mongodb/bin
mkdir conf
cd conf
vi confsvf.conf 
```
Machine A: <\br>
```
#confsvf.conf 
dbpath=/data/confsvr                             
logpath=/data/log/confsvr.log                 
pidfilepath=/data/confsvr.pid            
directoryperdb=true
logappend=true
replSet=ConfSet                   
bind_ip=192.168.226.130 
port=27400      
fork=true                   
configsvr=true

```
Machine B: <\br>
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
Machine C: <\br>
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

## Mongo shards configuration
create `set1svr.conf`, `set2svr.conf`and `set3svr.conf` under folder /usr/local.mongodb/bin/conf/ in machine A:<\br>
```
#set1svr.conf
dbpath=/data/shard1                            
logpath=/data/log/shard1.log                 
pidfilepath=/data/shard1.pid            
directoryperdb=true
logappend=true
replSet=rs1                 
bind_ip=192.168.226.130 
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
bind_ip=192.168.226.130 
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
bind_ip=192.168.226.130 
port=27103     
fork=true                   
shardsvr=true
```
create `set1svr.conf`, `set2svr.conf`and `set3svr.conf` under folder /usr/local.mongodb/bin/conf/ in machine B:<\br>
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
create `set1svr.conf`, `set2svr.conf`and `set3svr.conf` under folder /usr/local.mongodb/bin/conf/ in machine C:<\br>
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

```
A: 192.168.226.130
B: 192.168.226.131
C: 912.168.226.132
A: mongos   config(primary)27400     setA(primary)     setB (secondary)   setC(arbiter)
B: mongos   config(secondary)27400   setA(arbiter)   setB (primary)     setC(secondary)
C: mongos   config(secondary)27400   setA(secondary)   setB (arbiter)   setC(primary)
```

