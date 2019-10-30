
# What is Kafka?

Apache Kafka is a distributed streaming platform used for building real-time data pipelines and
streaming apps. It is horizontally scalable, fault-tolerant, wicked fast, and runs in production in
thousands of companies. Kafka requires a connection to a Zookeeper service.

[https://kafka.apache.org/](https://kafka.apache.org/)

# TL;DR;

## Run the application using Docker Compose

The main folder of this repository contains a functional [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-kafka/blob/master/docker-compose.yml) file. Run the application using it as shown below:

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-kafka/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.
* All Bitnami images available in Docker Hub are signed with [Docker Content Trust (DTC)](https://docs.docker.com/engine/security/trust/content_trust/). You can use `DOCKER_CONTENT_TRUST=1` to verify the integrity of the images.
* Bitnami container images are released daily with the latest distribution packages available.


> This [CVE scan report](https://quay.io/repository/bitnami/kafka?tab=tags) contains a security report with all open CVEs. To get the list of actionable security issues, find the "latest" tag, click the vulnerability report link under the corresponding "Security scan" field and then select the "Only show fixable" filter on the next page.

# How to deploy Apache Kafka in Kubernetes?

Deploying Bitnami applications as Helm Charts is the easiest way to get started with our applications on Kubernetes. Read more about the installation in the [Bitnami Apache Kafka Chart GitHub repository](https://github.com/bitnami/charts/tree/master/bitnami/kafka).

Bitnami containers can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters.

# Why use a non-root container?

Non-root container images add an extra layer of security and are generally recommended for production environments. However, because they run as a non-root user, privileged tasks are typically off-limits. Learn more about non-root containers [in our docs](https://docs.bitnami.com/containers/how-to/work-with-non-root-containers/).

# Supported tags and respective `Dockerfile` links

> NOTE: Debian 8 images have been deprecated in favor of Debian 9 images. Bitnami will not longer publish new Docker images based on Debian 8.

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/).


* [`2-ol-7`, `2.3.1-ol-7-r4` (2/ol-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-kafka/blob/2.3.1-ol-7-r4/2/ol-7/Dockerfile)
* [`2-debian-9`, `2.3.1-debian-9-r3`, `2`, `2.3.1`, `2.3.1-r3`, `latest` (2/debian-9/Dockerfile)](https://github.com/bitnami/bitnami-docker-kafka/blob/2.3.1-debian-9-r3/2/debian-9/Dockerfile)

Subscribe to project updates by watching the [bitnami/kafka GitHub repo](https://github.com/bitnami/bitnami-docker-kafka).

# Get this image

The recommended way to get the Bitnami Kafka Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/kafka).

```bash
docker pull bitnami/kafka:latest
```

To use a specific version, you can pull a versioned tag. You can view the
[list of available versions](https://hub.docker.com/r/bitnami/kafka/tags/)
in the Docker Hub Registry.

```bash
docker pull bitnami/kafka:[TAG]
```

If you wish, you can also build the image yourself.

```bash
docker build -t bitnami/kafka:latest 'https://github.com/bitnami/bitnami-docker-kafka.git#master:2/debian-9'
```

# Persisting your data

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

**Note!**
If you have already started using your database, follow the steps on
[backing up](#backing-up-your-container) and [restoring](#restoring-a-backup) to pull the data from your running container down to your host.

The image exposes a volume at `/bitnami/kafka` for the Kafka data. For persistence you can mount a directory at this location from your host. If the mounted directory is empty, it will be initialized on the first run.

Using Docker Compose:

This requires a minor change to the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-kafka/blob/master/docker-compose.yml) file present in this repository:

```yaml
kafka:
  ...
  volumes:
    - /path/to/kafka-persistence:/bitnami/kafka
  ...
```

# Connecting to other containers

Using [Docker container networking](https://docs.docker.com/engine/userguide/networking/), a Kafka server running inside a container can easily be accessed by your application containers.

Containers attached to the same network can communicate with each other using the container name as the hostname.

## Using the Command Line

In this example, we will create a Kafka client instance that will connect to the server instance that is running on the same docker network as the client.

### Step 1: Create a network

```bash
$ docker network create app-tier --driver bridge
```

### Step 2: Launch the Zookeeper server instance

Use the `--network app-tier` argument to the `docker run` command to attach the Zookeeper container to the `app-tier` network.

```bash
$ docker run -d --name zookeeper-server \
    --network app-tier \
    -e ALLOW_ANONYMOUS_LOGIN=yes \
    bitnami/zookeeper:latest
```

### Step 2: Launch the Kafka server instance

Use the `--network app-tier` argument to the `docker run` command to attach the Kafka container to the `app-tier` network.

```bash
$ docker run -d --name kafka-server \
    --network app-tier \
    -e ALLOW_PLAINTEXT_LISTENER=yes \
    -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper-server:2181 \
    bitnami/kafka:latest
```

### Step 3: Launch your Kafka client instance

Finally we create a new container instance to launch the Kafka client and connect to the server created in the previous step:

```bash
$ docker run -it --rm \
    --network app-tier \
    -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper-server:2181 \
    bitnami/kafka:latest kafka-topics.sh --list  --zookeeper zookeeper-server:2181
```

## Using Docker Compose

When not specified, Docker Compose automatically sets up a new network and attaches all deployed services to that network. However, we will explicitly define a new `bridge` network named `app-tier`. In this example we assume that you want to connect to the Kafka server from your own custom application image which is identified in the following snippet by the service name `myapp`.

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    networks:
      - app-tier
  kafka:
    image: 'bitnami/kafka:latest'
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
```

> **IMPORTANT**:
>
> 1. Please update the `YOUR_APPLICATION_IMAGE` placeholder in the above snippet with your application image
> 2. Configure Kafka and ZooKeeper persistence, and configure them either via environment variables or by [mounting configuration files](#full-configuration).
> 3. In your application container, use the hostname `kafka` to connect to the Kafka server

Launch the containers using:

```bash
$ docker-compose up -d
```

# Configuration

The configuration can easily be setup with the Bitnami Kafka Docker image using the following environment variables:

- `ALLOW_PLAINTEXT_LISTENER`: Allow to use the PLAINTEXT listener. Default: **no**
- `KAFKA_INTER_BROKER_USER`: Kafka inter broker communication user. Default: admin. Default: **admin**
- `KAFKA_INTER_BROKER_PASSWORD`: Kafka inter broker communication password. Default: **bitnami**
- `KAFKA_BROKER_USER`: Kafka client user. Default: **user**
- `KAFKA_BROKER_PASSWORD`: Kafka client user password. Default: **bitnami**
- `KAFKA_ZOOKEEPER_USER`: Kafka Zookeeper user. No defaults
- `KAFKA_ZOOKEEPER_PASSWORD`: Kafka Zookeeper user password. No defaults
- `KAFKA_CERTIFICATE_PASSWORD`: Password for certificates. No defaults.
- `KAFKA_HEAP_OPTS`: Kafka's Java Heap size. Default: **-Xmx1024m -Xms1024m**

Additionally, any environment variable beginning with `KAFKA_CFG_` will be mapped to its corresponding Kafka key. For example, use `KAFKA_CFG_BACKGROUND_THREADS` in order to set `background.threads`.

```bash
docker run --name kafka -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 -e ALLOW_PLAINTEXT_LISTENER=yes bitnami/kafka:latest
```

or by modifying the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-kafka/blob/master/docker-compose.yml) file present in this repository:

```yaml
kafka:
  ...
  environment:
    - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
  ...
```

## Security

The Bitnami Kafka docker image disables the PLAINTEXT listener for security reasons.
You can enable the PLAINTEXT listener by adding the next environment variable, but remember that this
configuration is not recommended for production.

```
ALLOW_PLAINTEXT_LISTENER=yes
```

In order to configure SASL authentication over SSL, you should define the proper listener by
passing the following env vars:

```
KAFKA_CFG_LISTENERS=SASL_SSL://:9092
KAFKA_CFG_ADVERTISED_LISTENERS=SASL_SSL://:9092
```

You **must** also use your own certificates for SSL. You can drop your Java Key Stores files into `/opt/bitnami/kafka/conf/certs`.
If the JKS is password protected (recommended), you will need to provide it to get access to the keystores:

`KAFKA_CERTIFICATE_PASSWORD=myCertificatePassword`

The following script can help you with the creation of the JKS and certificates:

https://raw.githubusercontent.com/confluentinc/confluent-platform-security-tools/master/kafka-generate-ssl.sh

Keep in mind the following notes:

* When prompted to enter a password, use the same one for all.
* Set the Common Name or FQDN values to your Kafka container hostname, e.g. `kafka.example.com`. After entering this value, when prompted "What is your first and last name?", enter this value as well.

The following docker-compose file is an example showing how to mount your JKS certificates protected by the password `certificatePassword123`.
Additionally it is specifying the Kafka container hostname and the credentials for the broker, inter-broker and zookeeper users.

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
     - '2181:2181'
    environment:
      - ZOO_ENABLE_AUTH=yes
      - ZOO_SERVER_USERS=kafka
      - ZOO_SERVER_PASSWORDS=kafka_password
  kafka:
    image: 'bitnami/kafka:latest'
    hostname: kafka.example.com
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_LISTENERS=SASL_SSL://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=SASL_SSL://:9092
      - KAFKA_ZOOKEEPER_USER=kafka
      - KAFKA_ZOOKEEPER_PASSWORD=kafka_password
      - KAFKA_INTER_BROKER_USER=interuser
      - KAFKA_INTER_BROKER_PASSWORD=interpassword
      - KAFKA_BROKER_USER=user
      - KAFKA_BROKER_PASSWORD=password
      - KAFKA_CERTIFICATE_PASSWORD=certificatePassword123
    volumes:
      - './kafka.keystore.jks:/opt/bitnami/kafka/conf/certs/kafka.keystore.jks:ro'
      - './kafka.truststore.jks:/opt/bitnami/kafka/conf/certs/kafka.truststore.jks:ro'
```

### InterBroker communications

By default, communications that happens between brokers are authenticated.
You can provide your own credentials using this environment variables:


- `KAFKA_INTER_BROKER_USER`: Kafka inter broker communication user. Default: **admin**
- `KAFKA_INTER_BROKER_PASSWORD`: Kafka inter broker communication password. Default: **bitnami**

### Kafka client configuration

By default, any Kafka client needs to authenticate before can connect to a broker.
You can provide your own credentials using this environment variables:

- `KAFKA_BROKER_USER`: Kafka client user. Default: **user**
- `KAFKA_BROKER_PASSWORD`: Kafka client user password. Default: **bitnami**

### Kafka ZooKeeper client configuration

In order to authenticate Kafka against a Zookeeper server with SASL authentication you should provide
the next environment variables:

- `KAFKA_ZOOKEEPER_USER`: Kafka Zookeeper user. No defaults.
- `KAFKA_ZOOKEEPER_PASSWORD`: Kafka Zookeeper user password. No defaults.

Below you can see a complete Docker Compose example:


```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
     - '2181:2181'
    environment:
      - ZOO_ENABLE_AUTH=yes
      - ZOO_SERVER_USERS=kafka
      - ZOO_SERVER_PASSWORDS=kafka_password
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_LISTENERS=SASL_SSL://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=SASL_SSL://:9092
      - KAFKA_ZOOKEEPER_USER=kafka
      - KAFKA_ZOOKEEPER_PASSWORD=kafka_password
```

### Connecting services with security enabled

In order to get the required credentials to consume and  produce messages you need to provide the
credentials in the client. If your Kafka client allows it, use the credentials you've provided.

While producing and consuming messages using the `bitnami/kafka` image, you'll need to point to the
`consumer.properties` and/or `producer.properties` file, which contains the needed configuration
to work. You can find this files in the `/opt/bitnami/kafka/conf` directory

Use this to generate messages using a secure setup

```bash
export KAFKA_OPTS="-Djava.security.auth.login.config=/opt/bitnami/kafka/conf/kafka_jaas.conf"
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test --producer.config /opt/bitnami/kafka/conf/producer.properties
```
Use this to consume messages using a secure setup

```bash
export KAFKA_OPTS="-Djava.security.auth.login.config=/opt/bitnami/kafka/conf/kafka_jaas.conf"
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test --consumer.config /opt/bitnami/kafka/conf/consumer.properties
```
If you use other tools to use your Kafka cluster, you'll need to provide the required information.
You can find the required information in the files located at `/opt/bitnami/kafka/conf` directory.

## Setting up a Kafka Cluster

A Kafka cluster can easily be setup with the Bitnami Kafka Docker image using the following environment variables:

 - `KAFKA_CFG_ZOOKEEPER_CONNECT`: Comma separated host:port pairs, each corresponding to a Zookeeper Server.

Create a Docker network to enable visibility to each other via the docker container name

```bash
docker network create app-tier --driver bridge
```

### Step 1: Create the first node for Zookeeper

The first step is to create one Zookeeper instance.

```bash
docker run --name zookeeper \
  --network app-tier \
  -e ALLOW_ANONYMOUS_LOGIN=yes \
  -p 2181:2181 \
  bitnami/zookeeper:latest
```

### Step 2: Create the first node for Kafka

The first step is to create one Kafka instance.

```bash
docker run --name kafka1 \
  --network app-tier \
  -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -e ALLOW_PLAINTEXT_LISTENER=yes \
  -p 9092:9092 \
  bitnami/kafka:latest
```

### Step 2: Create the second node

Next we start a new Kafka container.

```bash
docker run --name kafka2 \
  --network app-tier \
  -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -e ALLOW_PLAINTEXT_LISTENER=yes \
  -p 9092:9092 \
  bitnami/kafka:latest
```

### Step 3: Create the third node

Next we start another new Kafka container.

```bash
docker run --name kafka3 \
  --network app-tier \
  -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -e ALLOW_PLAINTEXT_LISTENER=yes \
  -p 9092:9092 \
  bitnami/kafka:latest
```
You now have a Kafka cluster up and running. You can scale the cluster by adding/removing slaves without incurring any downtime.

With Docker Compose, topic replication can be setup using:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
     - '2181:2181'
    environment
     - ALLOW_ANONYMOUS_LOGIN=yes
  kafka1:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
  kafka2:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
  kafka3:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
```

Then, you can create a replicated topic with:

```bash
root@kafka1:/# /opt/bitnami/kafka/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --topic mytopic --partitions 3 --replication-factor 3
Created topic "mytopic".

root@kafka1:/# /opt/bitnami/kafka/bin/kafka-topics.sh --describe --zookeeper zookeeper:2181 --topic mytopic
Topic:mytopic   PartitionCount:3        ReplicationFactor:3     Configs:
        Topic: mytopic  Partition: 0    Leader: 2       Replicas: 2,3,1 Isr: 2,3,1
        Topic: mytopic  Partition: 1    Leader: 3       Replicas: 3,1,2 Isr: 3,1,2
        Topic: mytopic  Partition: 2    Leader: 1       Replicas: 1,2,3 Isr: 1,2,3
```

## Full configuration
The image looks for configuration in the `conf/` directory of `/opt/bitnami/kafka`.

```
docker run --name kafka -v /path/to/server.properties:/opt/bitnami/kafka/conf/server.properties bitnami/kafka:latest
```
After that, your changes will be taken into account in the server's behaviour.

### Step 1: Run the Kafka image

Run the Kafka image, mounting a directory from your host.

Modify the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-kafka/blob/master/docker-compose.yml) file present in this repository:

```yaml
kafka:
  volumes:
    - /path/to/server.properties:/opt/bitnami/kafka/conf/server.properties
```

### Step 2: Edit the configuration

Edit the configuration on your host using your favorite editor.

```bash
vi /path/to/server.properties
```

### Step 3: Restart Kafka

After changing the configuration, restart your Kafka container for changes to take effect.

```bash
docker restart kafka
```

Or using Docker Compose:

```bash
docker-compose restart kafka
```

# Logging

The Bitnami Kafka Docker image sends the container logs to the `stdout`. To view the logs:

```bash
docker logs kafka
```

Or using Docker Compose:

```bash
docker-compose logs kafka
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

# Maintenance

## Backing up your container

To backup your data, configuration and logs, follow these simple steps:

### Step 1: Stop the currently running container

```bash
docker stop kafka
```

Or using Docker Compose:

```bash
docker-compose stop kafka
```

### Step 2: Run the backup command

We need to mount two volumes in a container we will use to create the backup: a directory on your host to store the backup in, and the volumes from the container we just stopped so we can access the data.

```bash
docker run --rm -v /path/to/kafka-backups:/backups --volumes-from kafka busybox \
  cp -a /bitnami/kafka:latest /backups/latest
```

Or using Docker Compose:

```bash
docker run --rm -v /path/to/kafka-backups:/backups --volumes-from `docker-compose ps -q kafka` busybox \
  cp -a /bitnami/kafka:latest /backups/latest
```

## Restoring a backup

Restoring a backup is as simple as mounting the backup as volumes in the container.

```bash
docker run -v /path/to/kafka-backups/latest:/bitnami/kafka bitnami/kafka:latest
```

You can also modify the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-kafka/blob/master/docker-compose.yml) file present in this repository:

```yaml
kafka:
  volumes:
    - /path/to/kafka-backups/latest:/bitnami/kafka
```

## Upgrade this image

Bitnami provides up-to-date versions of Kafka, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

### Step 1: Get the updated image

```bash
docker pull bitnami/kafka:latest
```

or if you're using Docker Compose, update the value of the image property to
`bitnami/kafka:latest`.

### Step 2: Stop and backup the currently running container

Before continuing, you should backup your container's data, configuration and logs.

Follow the steps on [creating a backup](#backing-up-your-container).

### Step 3: Remove the currently running container

```bash
docker rm -v kafka
```

Or using Docker Compose:


```bash
docker-compose rm -v kafka
```

### Step 4: Run the new image

Re-create your container from the new image, [restoring your backup](#restoring-a-backup) if necessary.

```bash
docker run --name kafka bitnami/kafka:latest
```

Or using Docker Compose:

```bash
docker-compose up kafka
```

# Notable Changes

## 1.1.1-debian-9-r224, 2.2.1-debian-9-r16, 1.1.1-ol-7-r306 and 2.2.1-ol-7-r14

- The following environment variables were beingly wrongly translated into `KAFKA_CFG_` environment variables, and therefore they were being wrongly mapped into Kafka keys:

  - `KAFKA_LOGS_DIRS` -> `KAFKA_CFG_LOG_DIRS`
  - `KAFKA_PORT_NUMBER` -> `KAFKA_CFG_PORT`
  - `KAFKA_ZOOKEEPER_CONNECT_TIMEOUT_MS` -> `KAFKA_CFG_ZOOKEEPER_CONNECTION_TIMEOUT_MS`

- For consistency reasons with previous environment variables, the following `KAFKA_` to `KAFKA_CFG_` environment variable translations are now supported for mapping into Kafka keys:

  - `KAFKA_LOG_DIRS` -> `KAFKA_CFG_LOG_DIRS`
  - `KAFKA_ZOOKEEPER_CONNECTION_TIMEOUT_MS` -> `KAFKA_CFG_ZOOKEEPER_CONNECTION_TIMEOUT_MS`

## 1.1.1-debian-9-r205, 2.2.0-debian-9-r40, 1.1.1-ol-7-r286, and 2.2.0-ol-7-r53

- Configuration changes. Most environment variables now start with `KAFKA_CFG_`, as they are now mapped directly to Kafka keys. Variables changed:
   - `KAFKA_ADVERTISED_LISTENERS` -> `KAFKA_CFG_ADVERTISED_LISTENERS`
   - `KAFKA_BROKER_ID` -> `KAFKA_CFG_BROKER_ID`
   - `KAFKA_DEFAULT_REPLICATION_FACTOR` -> `KAFKA_CFG_DEFAULT_REPLICATION_FACTOR`
   - `KAFKA_DELETE_TOPIC_ENABLE` -> `KAFKA_CFG_DELETE_TOPIC_ENABLE`
   - `KAFKA_INTER_BROKER_LISTENER_NAME` -> `KAFKA_CFG_INTER_BROKER_LISTENER_NAME`
   - `KAFKA_LISTENERS` -> `KAFKA_CFG_LISTENERS`
   - `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP` -> `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP`
   - `KAFKA_LOGS_DIRS` -> `KAFKA_CFG_LOG_DIRS`
   - `KAFKA_LOG_FLUSH_INTERVAL_MESSAGES` -> `KAFKA_CFG_LOG_FLUSH_INTERVAL_MESSAGES`
   - `KAFKA_LOG_FLUSH_INTERVAL_MS` -> `KAFKA_CFG_LOG_FLUSH_INTERVAL_MS`
   - `KAFKA_LOG_MESSAGE_FORMAT_VERSION` -> `KAFKA_CFG_LOG_MESSAGE_FORMAT_VERSION`
   - `KAFKA_LOG_RETENTION_BYTES` -> `KAFKA_CFG_LOG_RETENTION_BYTES`
   - `KAFKA_LOG_RETENTION_CHECK_INTERVALS_MS` -> `KAFKA_CFG_LOG_RETENTION_CHECK_INTERVAL_MS`
   - `KAFKA_LOG_RETENTION_HOURS` -> `KAFKA_CFG_LOG_RETENTION_HOURS`
   - `KAFKA_MAX_MESSAGE_BYTES` -> `KAFKA_CFG_MESSAGE_MAX_BYTES`
   - `KAFKA_NUM_IO_THREADS` -> `KAFKA_CFG_NUM_IO_THREADS`
   - `KAFKA_NUM_NETWORK_THREADS` -> `KAFKA_CFG_NUM_NETWORK_THREADS`
   - `KAFKA_NUM_PARTITIONS` -> `KAFKA_CFG_NUM_PARTITIONS`
   - `KAFKA_NUM_RECOVERY_THREADS_PER_DATA_DIR` -> `KAFKA_CFG_NUM_RECOVERY_THREADS_PER_DATA_DIR`
   - `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` -> `KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR`
   - `KAFKA_PORT` -> `KAFKA_CFG_PORT`
   - `KAFKA_SEGMENT_BYTES` -> `KAFKA_CFG_SEGMENT_BYTES`
   - `KAFKA_SOCKET_RECEIVE_BUFFER_BYTES` -> `KAFKA_CFG_SOCKET_RECEIVE_BUFFER_BYTES`
   - `KAFKA_SOCKET_REQUEST_MAX_BYTES` -> `KAFKA_CFG_SOCKET_REQUEST_MAX_BYTES`
   - `KAFKA_SOCKET_SEND_BUFFER_BYTES` -> `KAFKA_CFG_SOCKET_SEND_BUFFER_BYTES`
   - `KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM` -> `KAFKA_CFG_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM`
   - `KAFKA_TRANSACTION_STATE_LOG_MIN_ISR` -> `KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR`
   - `KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR` -> `KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR`
   - `KAFKA_ZOOKEEPER_CONNECT_TIMEOUT_MS` -> `KAFKA_CFG_ZOOKEEPER_CONNECT_TIMEOUT_MS`
   - `KAFKA_ZOOKEEPER_CONNECT` -> `KAFKA_CFG_ZOOKEEPER_CONNECT`

## 1.1.0-r41

- Configuration is not persisted anymore. It should be mounted as a volume or it will be regenerated each time the container is created.
- Dummy certificates are not used anymore when the SASL_SSL listener is configured. These certificates must be mounted as volumes.

## 0.10.2.1-r3

- The kafka container has been migrated to a non-root container approach. Previously the container run as `root` user and the kafka daemon was started as `kafka` user. From now own, both the container and the kafka daemon run as user `1001`.
  As a consequence, the configuration files are writable by the user running the kafka process.

## 0.10.2.1-r0

- New Bitnami release


# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-kafka/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-kafka/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-kafka/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright (c) 2015-2019 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
