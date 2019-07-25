
# kafka-stack-docker-compose

This replicates as well as possible real deployment configurations, where you have your zookeeper servers and kafka servers actually all distinct from each other. This solves all the networking hurdles that comes with Docker and docker-compose, and is compatible cross platform.

## Stack version

  - Zookeeper version: 3.4.9
  - Kafka version: 2.2.1 (Confluent 5.2.2)
  - Kafka Schema Registry: Confluent 5.2.2
  - Kafka Rest Proxy: Confluent 5.2.2
  - Kafka Connect: Confluent 5.2.2

# Requirements
  ## Docker

  Please export your environment before starting the stack:
  ```
  export DOCKER_HOST_IP=127.0.0.1
  ```
  (that's the default value and you actually don't need to do a thing)
  
  ## The JDBC driver
  mkdir -p connectors
  curl -k -SL "http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz" | tar -xzf - -C ./connectors --strip-components=1 mysql-connector-java-5.1.47/mysql-connector-java-5.1.47-bin.jar

  mysql-connector-java-5.1.47.jar

 ## Run with:
 ```
 docker-compose -f full-stack.yml up
 docker-compose -f full-stack.yml down
 ```

## Launch a MYSQL database.

docker run -d \
  --name=quickstart-mysql \
  --net=host \
  -e MYSQL_ROOT_PASSWORD=confluent \
  -e MYSQL_USER=confluent \
  -e MYSQL_PASSWORD=confluent \
  -e MYSQL_DATABASE=connect_test \
  mysql

## Next, Create databases and tables. You'll need to exec into the Docker container to create the databases.
  docker exec -it quickstart-mysql bash
  mysql -u confluent -pconfluent


  CREATE DATABASE IF NOT EXISTS connect_test;
  USE connect_test;

  DROP TABLE IF EXISTS test;


  CREATE TABLE IF NOT EXISTS test (
    id serial NOT NULL PRIMARY KEY,
    name varchar(100),
    email varchar(200),
    department varchar(200),
    modified timestamp default CURRENT_TIMESTAMP NOT NULL,
    INDEX `modified_index` (`modified`)
  );

  INSERT INTO test (name, email, department) VALUES ('alice', 'alice@abc.com', 'engineering');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');
  INSERT INTO test (name, email, department) VALUES ('bob', 'bob@abc.com', 'sales');

  CREATE TABLE IF NOT EXISTS test2 (
    id serial NOT NULL PRIMARY KEY,
    name varchar(100),
    email varchar(200),
    department varchar(200),
    modified timestamp default CURRENT_TIMESTAMP NOT NULL,
    INDEX `modified_index` (`modified`)
  );
  exit;



## Create our JDBC Source connector using the Connect REST API.
export CONNECT_HOST=localhost

## Source connector
  curl -X POST \
    -H "Content-Type: application/json" \
    --data '{"name":"quickstart-jdbc-source","config":{"connector.class":"io.confluent.connect.jdbc.JdbcSourceConnector","tasks.max":1,"connection.url":"jdbc:mysql://35.185.0.163:3306/connect_test?user=root&password=root123456","mode":"incrementing","incrementing.column.name":"id","timestamp.column.name":"modified","topic.prefix":"quickstart-jdbc-","table.whitelist":"test","poll.interval.ms":1000}}' \
    http://$CONNECT_HOST:8083/connectors

## Sink connector
  curl -X POST -H "Accept:application/json" -H "Content-Type: application/json" \
    --data '{"name":"quick-jdbc-sink","config":{"connector.class":"io.confluent.connect.jdbc.JdbcSinkConnector","tasks.max":"1","topics":"quickstart-jdbc-test","connection.url":"jdbc:mysql://35.185.0.163:3306/connect_test?user=root&password=root123456","auto.create":"false","insert.mode":"upsert","auto.evolve":"true","delete.enabled":"true","pk.mode":"record_value","pk.fields":"id","table.name.format":"test2"}}' http://$CONNECT_HOST:8083/connectors

## Full stack

 - Single Zookeeper: `$DOCKER_HOST_IP:2181`
 - Single Kafka: `$DOCKER_HOST_IP:9092`
 - Kafka Schema Registry: `$DOCKER_HOST_IP:8081`
 - Kafka Rest Proxy: `$DOCKER_HOST_IP:8082`
 - Kafka Connect: `$DOCKER_HOST_IP:8083`


