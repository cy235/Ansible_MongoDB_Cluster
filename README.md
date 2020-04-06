# Create a MongoDB sharded cluster

## Prerequisite
1. 3 machines installed with CentOS 7
2. set the static IP for each machine
3. disable the firewall of each machine
set static IP for each machine:
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
replace `BOOTPROTO=”dhcp”` with `BOOTPROTO=”static”`, and add your IP address, for example:
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

whenever there are unexpected errors, you can remove and recreate above folders to avoid conflicts.

## Create configuration files
