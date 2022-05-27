# Elastic stack (ELK) on Docker on WSL2
(refer : https://github.com/deviantony/docker-elk)

------------------------

#### [주의사항!!! docker-compose 파일을 WLS2에서 실행 시 주요 이슈가 발생 부분]

##### 1. docker-compose 실행 path가 windows directory일 경우 volume mount(mysql/kafka/zookeeper) 문제가 발생한다
> 반드시 avoid!! <b>/mnt/c/ </b> 나 <b>/mnt/d/ </b> 에서 실행하지 않는다

##### 2. docker-compose 실행 후 생성되는 kafka/zooker의 권한을 꼭 확인한다
> docker container와 mount되는 path directory가 root권한으로 생성 되는 경우가 많다.
> 꼭 사전에 kafka/logs, zookeeper/data 폴더 생성 후 실행 user 권한으로 변경해준다
```
mkdir -p kafka/logs 
mkdir -p zookeeper/data
chown -R user:user zookeeper kafka
```

##### 3. /etc/hosts를 확인해서 사용할 도메인이 등록되어 있는지 확인한다
```
127.0.0.1 kafka zookeeper
```

##### 4. docker-compose 실행 후 docker container에 mount된 volume 권한을 변경하지 않는다
> docker-compose 재실행 시 권한 문제가 발생한다

![image](https://user-images.githubusercontent.com/30817824/170620369-16000fab-b9e1-47af-b95b-93e1cebf4282.png)


-----------------------

#### 0. change password

for changing password you can modify below files (hdskmms -> new password)
* .env : ELASTIC_PASSWORD='hdskmms' & LOGSTASH_INTERNAL_PASSWORD='hdskmms'
* extensions/curator/config/curator.yml : http_auth: 'elastic:hdskmms'
* extensions/enterprise-search/enterprise-search-compose.yml : ENT_SEARCH_DEFAULT_PASSWORD: 'hdskmms'

#### 1. /etc/hosts setting

```
0.0.0.0 kafka
0.0.0.0 zookeeper
```


#### 2. this command will deploy ELK/akhq/zookeeper/kafka(1broker) on WSL2 Ubuntu


```
docker-compose -f docker-compose.yml up
```

#### 3. when finished, run sample web application generating fake web log

> (refer : https://github.com/kiritbasu/Fake-Apache-Log-Generator)

```
docker-compose -f docker-compose-data-pipeline-application.yml up
```

#### 4. open docker container 
```
docker exec -it <CONTAINER ID> /bin/bash
```

#### 5. set filebeat

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

4) start filebeat 
```
cd /filebeat-7.15.1-linux-x86_64
./filebeat
```

if error occurs check your <b>/etc/hosts</b> file. <br/>
make sure you have below host info
```
0.0.0.0 kafka
```

----------------------------------


#### extra. if kafka/zookeeper sync error occurs 

```
rm -rf kafka zookeeper
```

#### extra. if logstash is unable to resolve DNS 'kafka', try to change below manifest

docker-compose.yml
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
