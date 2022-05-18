# Elastic stack (ELK) on Docker on WSL2
(refer : https://github.com/deviantony/docker-elk)

------------------------

### 0. change password

> for changing password you can modify below files <b>(hdskmms -> new password)</b>
* <b>.env :</b> ELASTIC_PASSWORD='hdskmms' 
* <b>.env :</b> LOGSTASH_INTERNAL_PASSWORD='hdskmms'
* <b>extensions/curator/config/curator.yml :</b> http_auth: 'elastic:hdskmms'
* <b>extensions/enterprise-search/enterprise-search-compose.yml :</b> ENT_SEARCH_DEFAULT_PASSWORD: 'hdskmms'

  
### 1. /etc/hosts setting

```
0.0.0.0 kafka
0.0.0.0 zookeeper
```


### 2. this command will deploy ELK/akhq/zookeeper/kafka(1broker) on WSL2 Ubuntu


```
docker-compose -f docker-compose.yml up
```

### 3. when finished, run sample web application generating fake web log

> (refer : https://github.com/kiritbasu/Fake-Apache-Log-Generator)

```
docker-compose -f docker-compose-data-pipeline-application.yml up
```

### 4. open docker container 
```
docker exec -it <CONTAINER ID> /bin/bash
```

### 5. set filebeat

1) fake logs are stored in /log_generator/access_log_YYYYMMDD-XXXXXX.log
2) open filebeat.yml file in "/filebeat-7.15.1-linux-x86_64"
3) set log paths and output (output.elasticsearch -> output.kafka)

```
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /log_generator/access*.log


output.kafka:
  # Array of hosts to connect to.
  hosts: ["kafka:9092"]
  topic: "webapp-logs"
  compression: gzip
```


-------------------------------------------


#### extra. if kafka/zookeeper sync error occurs 

```
rm -rf kafka zookeeper
```

#### extra. if logstash is unable to resolve DNS 'kafka', try to change below manifest

> <b>docker-compose.yml</b>
```
      #내부 IP/DNS/PORT
      KAFKA_LISTENERS: PLAINTEXT://kafka:9092
      #외부 IP/DNS/PORT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092          
```

#### IP:Port
```
#AKHQ
http://localhost:8080/

#Elastic
http://localhost:5601
ID : elastic
PWD : hdskmms
```
