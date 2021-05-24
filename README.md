

* docker
* docker-compose
* mongodb


# Deploy a replica set
## Check default config
```bash
docker-compose-up

docker exec -it mongo-a bash
# enter container
mongo mongodb://root:example@localhost:27017
## mongo shell
db.adminCommand( { getCmdLineOpts: 1  } )
db.adminCommand({getParameter:"*"})
## exit mongo shell
# exit container

```
#### **`getCmdLineOpts output`**
```json
{
    "argv" : [
            "mongod",
            "--auth",
            "--bind_ip_all"
    ],
    "parsed" : {
            "net" : {
                    "bindIp" : "*"
            },
            "security" : {
                    "authorization" : "enabled"
            }
    },
    "ok" : 1
}
```

## Create a config file

```yaml
storage:
  dbPath: /data/db
replication:
  replSetName: "my_rs"
net:
  bindIpAll: true
  bindIp: "*"
security:
  clusterAuthMode: keyFile
  keyFile: /etc/mongo/keyfile
  authorization: enabled
```

### Use the config file
Add the line `command: ["mongod", "--config", "/etc/mongo/mongod.conf"]` in each service in the docker compose file.

### Create keyfile
> A key's length must be between 6 and 1024 characters and may only contain characters in the base64 set. All members of the replica set must share at least one common key.
``` 
openssl rand -base64 756 > config/keyfile
wc -c config/keyfile # check size
chmod 400 config/keyfile
sudo chown 999:999 config/keyfile  # uid of mongodb

```
### Recreate the servers and Iniate replicaset
```bash
docker-compose up --force-recreate --remove-orphans

docker exec -it mongo-a bash
# enter container
mongo mongodb://root:example@mongo-a:27017,mongo-b:27017,mongo-b:27017/?authSource=admin&replicaSet=my_rs

```
db.createUser({
  user: "root" ,
  pwd: "example" ,
  roles: [ { role: "root", db: "admin" } ]
})
```

## mongo shell
rs.initiate(
  {
    _id : my_rs,
    members: [
      { _id : 0, host : "mongo-a:27017" },
      { _id : 1, host : "mongo-b:27017" },
      { _id : 2, host : "mongo-c:27017" }
    ]
  }
)
## end mongo shell
# exit container
#
```


# ReplicaSet


# Backup




# Utils

## Docker

* Clean up the project
```bash
dkc down --remove-orphans --volumes
sudo rm -rf data/mongo-[a,b,c]/*
``` 

* Start project
```bash
dkc up
```


## MongoDB

* [Connection Link](https://docs.mongodb.com/manual/reference/connection-string/) 
* [How mongodb was initialized](https://docs.mongodb.com/manual/reference/command/getCmdLineOpts/)
* [Deploy a Replica Set](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/link)
* [Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options/)