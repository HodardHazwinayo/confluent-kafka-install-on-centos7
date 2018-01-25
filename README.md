# Install Confluent platform on Centos 7

# 1 install directly on centos 7 server
Refer [here](https://docs.confluent.io/current/installation/installing_cp.html) from offical document.
 
## 1.1 Yum Install 

+ **Install the Confluent public key, which is used to sign packages in the YUM repository**

```
$ sudo rpm --import https://packages.confluent.io/rpm/4.0/archive.key
```

+ **Add the repository to your `/etc/yum.repos.d/` directory in a file named `confluent.repo`**

```
$ cat /etc/yum.repos.d/confluent.repo
[Confluent.dist]
name=Confluent repository (dist)
baseurl=https://packages.confluent.io/rpm/4.0/7
gpgcheck=1
gpgkey=https://packages.confluent.io/rpm/4.0/archive.key
enabled=1

[Confluent]
name=Confluent repository
baseurl=https://packages.confluent.io/rpm/4.0
gpgcheck=1
gpgkey=https://packages.confluent.io/rpm/4.0/archive.key
enabled=1
```

+ **Clear the YUM caches**
```
sudo yum clean all
```


+ **Install Confluent Platform**
```
sudo yum install confluent-platform-oss-2.11
```

Note: the version `2.11` here is scala version, not confluent version.

Check installed packages
```
]$ rpm -qa | grep confluent
confluent-kafka-connect-s3-4.0.0-1.noarch
confluent-common-4.0.0-1.noarch
confluent-kafka-connect-hdfs-4.0.0-1.noarch
confluent-kafka-rest-4.0.0-1.noarch
confluent-kafka-connect-elasticsearch-4.0.0-1.noarch
confluent-kafka-2.11-1.0.0-1.noarch
confluent-rest-utils-4.0.0-1.noarch
confluent-kafka-connect-jdbc-4.0.0-1.noarch
confluent-camus-4.0.0-1.noarch
confluent-kafka-connect-storage-common-4.0.0-1.noarch
confluent-cli-4.0.0-1.noarch
confluent-schema-registry-4.0.0-1.noarch
confluent-platform-oss-2.11-4.0.0-1.noarch
```

+ **Start confluent**
```
$ sudo confluent start

Starting zookeeper
zookeeper is [UP]
Starting kafka
kafka is [UP]
Starting schema-registry
schema-registry is [UP]
Starting kafka-rest
kafka-rest is [UP]
Starting connect
connect is [UP]
```

+ **Port Configuration**

| Component                          | Port |
| -----------------------------------|:----:|
| Apache Kafka brokers (plain text)  |9092  | 
| Confluent Control Center           |9021  |
| Kafka Connect REST API             |8083  |
| REST Proxy                         |8082  |
| Schema Registry REST API           |8081  |
| ZooKeeper                          |2181  |


+ **Basic confluent commands**

```
$ sudo confluent help
confluent: A command line interface to manage Confluent services

Usage: confluent <command> [<subcommand>] [<parameters>]

These are the available commands:

    acl         Specify acl for a service.
    config      Configure a connector.
    current     Get the path of the data and logs of the services managed by the current confluent run.
    destroy     Delete the data and logs of the current confluent run.
    list        List available services.
    load        Load a connector.
    log         Read or tail the log of a service.
    start       Start all services or a specific service along with its dependencies
    status      Get the status of all services or the status of a specific service along with its dependencies.
    stop        Stop all services or a specific service along with the services depending on it.
    top         Track resource usage of a service.
    unload      Unload a connector.
```

## 1.2 Test installation

### 1.2.1 create a simple Avro console producer

Create a producer that produce Topic `test`
```
$sudo kafka-avro-console-producer --broker-list localhost:9092 --topic test --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
```

After the producer is started, the process will wait for you to enter messages and your terminal may appear idle. Enter one message per line 
```
{"f1": "value1"}
{"f1": "value2"}
{"f1": "value3"}
```

When you’re done, use `Ctrl+C` to shut down the process.

### 1.2.2 create a consumer listening to topic `test`

```
$  sudo kafka-avro-console-consumer --topic test --zookeeper localhost:2181 --from-beginning
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
{"f1":"value1"}
{"f1":"value2"}
{"f1":"value3"}
```

You can see all messages are there. 

## 2 Install via Docker image
Another way to install confluent kafka is via docker images.


### 2.1 Install Docker Machine

Docker Machine allows you to create VMs on not just your local machine but also on several public cloud providers, private cloud implementations and more. How is it able to manage this? It does this via drivers. When creating a Docker Machine, you have to specific which driver you want to use. For the local setup, it uses Virtual Box and it also provides drivers for AWS, Google Cloud, Azure, and more. You can find more `docker-machine` drivers from [https://docs.docker.com/machine/drivers/](https://docs.docker.com/machine/drivers/)

```
$ curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine &&
$ sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
$ docker-machine -version
docker-machine version 0.13.0, build 9ba6da9
```

### 2.2 Install docker-machine driver 

For test, you need to install virtualBox locally. 
```
$ sudo cd /etc/yum.repos.d/
$ sudo wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
$ sudo yum clean all 
$ sudo yum install VirtualBox-5.2
```

### 2.3 run steps following [this link](https://docs.confluent.io/current/installation/docker/docs/quickstart.html) 

