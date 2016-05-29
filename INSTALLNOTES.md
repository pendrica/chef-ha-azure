# Chef Server HA 2.0 "Quick" install on Azure (unofficial notes)

### Chef HA 2.0 versions tested:
- chef-backend 0.8.7
- chef-server-core 12.6.0
- Ubuntu 14.04.4-LTS


## 1. Create Cluster on first node

### download chef-backend

```
curl -L https://chef.bintray.com/path/to/chef-backend.deb -o chef-backend.deb
```

### install backend on first node

```
sudo dpkg -i chef-backend.deb
```

### update /etc/chef-backend/chef-backend.rb with IP of the first node

```
/sbin/ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ printf "publish_address\t\x27%s\x27", $1}' | sudo tee /etc/chef-backend/chef-backend.rb
```

### create cluster

```
sudo chef-backend-ctl create-cluster
```

* Read license agreement and agree by typing 'yes'
* Accept NTP warning and continue by typing 'proceed' 
  ```
  CBWARN001: The NTP daemon is not running on this system.

           Time drift between Chef Backend nodes can affect the stability of
           the cluster.  We strongly recommend providing some means of ensuring
           consistent system time across all Chef Backend nodes.

           Please follow the package installation procedures for your distribution
           to install the NTP daemon and ensure that it runs by default.
  ```
* Services will start

### copy config to other backend nodes

```
sudo scp /etc/chef-backend/chef-backend-secrets.json azure@chef-ha-be-srv-02:/home/azure
sudo scp /etc/chef-backend/chef-backend-secrets.json azure@chef-ha-be-srv-03:/home/azure
```

## 2. Install and configure backend nodes 2 and 3

SSH to chef-ha-be-srv-02 and chef-ha-be-srv-03 in turn and run the following **(DO NOT DO THIS IN PARALLEL)**

### download and install chef-backend

```
curl -L https://chef.bintray.com/path/to/chef-backend.deb -o chef-backend.deb
```

### install backend on node

```
sudo dpkg -i chef-backend.deb
```

### join the cluster from the node

```
sudo chef-backend-ctl join-cluster chef-ha-be-srv-01 -s /home/azure/chef-backend-secrets.json
```

* Read license agreement and agree by typing 'yes'
* Select IP address to publish (should be 10.51.1.x if you kept the defaults)
* Accept NTP warning and continue by typing 'proceed' 
  ```
  CBWARN001: The NTP daemon is not running on this system.

           Time drift between Chef Backend nodes can affect the stability of
           the cluster.  We strongly recommend providing some means of ensuring
           consistent system time across all Chef Backend nodes.

           Please follow the package installation procedures for your distribution
           to install the NTP daemon and ensure that it runs by default.
  ```
* Services will start

If successful you should be able to run:

```
sudo chef-backend-ctl status
``` 

and see results such as:

```
azure@chef-ha-be-srv-01:~$ sudo chef-backend-ctl
Service        Local Status        Time in State  Distributed Node Status
leaderl        running (pid 8990)  0d 0h 19m 55s  leader: 1; waiting: 0; follower: 2; total: 3
etcd           running (pid 8949)  0d 0h 20m 3s   health: green; healthy nodes: 3/3
postgresql     running (pid 9038)  0d 0h 19m 55s  leader: 1; offline: 0; syncing: 0; synced: 2
elasticsearch  running (pid 8913)  0d 0h 20m 5s   state: green; nodes online: 3/3
```

## 3. Generate Chef frontend server config

On chef-ha-be-srv-01

### Generate the server config for the first frontend node

Note: you'll need your intended FQDN for this, in my example: pendrica-demo01-chef-ha.westeurope.cloudapp.azure.com  

```
sudo chef-backend-ctl gen-server-config pendrica-demo01-chef-ha.westeurope.cloudapp.azure.com -f /home/azure/chef-server.rb
```

### SCP config file to the local machine

On chef-ha-fe-srv-01 (because BE machines cannot SCP/SSH to FE machines by design...)

```
scp azure@chef-ha-be-srv-01:/home/azure/chef-server.rb /home/azure/chef-server.rb
```

### download frontend software

```
curl -L https://chef.bintray.com/path/to/chef-backend.deb -o chef-server-core.deb
```

### install frontend software

```
sudo dpkg -i chef-server-core.deb
```

### copy chef-server.rb to /etc/opscode

```
sudo cp /home/azure/chef-server.rb /etc/opscode/chef-server.rb
```

---
##DIVERSION
### fix non-samenet connections in postgresql (NB: This workaround may not be required)

On chef-ha-be-srv-01, chef-ha-be-srv-02, chef-ha-be-srv-03,

```
echo "host       all         all    10.51.0.0/24    md5" >> /var/opt/chef-backend/postgresql/9.5/data/pg_hba.conf
```

### restart postgresql (NB: This workaround may not be required)

```
sudo chef-backend-ctl restart postgresql
```
---

### configure frontend node

```
sudo chef-server-ctl reconfigure
```

## 4. Add more frontend nodes

TBC
