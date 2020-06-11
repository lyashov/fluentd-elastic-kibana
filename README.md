
## Fluentd with elasticsearch and kibana 

### Docker-compose.yml master
```
version: '3'
services:
  mdm-ok-test:
   build: mdm-ok-test
   image: 31soft/mdm-ok-test:latest
   container_name: mdm-ok-test
   expose:
    - "38081"
   ports:
    - '80:80'
    - '8080:8080'
    - '9990:9990' 
   volumes:
     - "/var/tmp:/logs"
   restart: always  
   links:
     - fluentd
   logging:
     driver: "fluentd"
     options:
       fluentd-address: localhost:24224
       tag: httpd.access    
  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    links:
      - "elasticsearch"
    ports:
      - "24224:24224"
      - "24224:24224/udp"
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    container_name: elasticsearch
    environment:
      - "discovery.type=single-node"
    expose:
      - "9200"
    ports:
      - "9200:9200"

  kibana:
    image: kibana:7.2.0
    links:
      - "elasticsearch"
    ports:
      - "5601:5601"
    
```

### Docker-compose.yml slave
```
version: '3'
services:
  mdm-ok-test:
   build: mdm-ok-test
   image: 31soft/mdm-ok-test:latest
   container_name: mdm-ok-test
   expose:
    - "38081"
   ports:
    - '80:80'
    - '8080:8080'
    - '9990:9990' 
   volumes:
     - "/var/tmp:/logs"
   restart: always  
   logging:
     driver: "fluentd"
     options:
       fluentd-address: 194.87.238.201:24224
       fluentd-async-connect: 'true'
       tag: httpd.access
```

### fluent.conf
```
<source>
  @type forward
  port 24224
</source>
<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
```
### Dockerfile 
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=mdm-ok-test.jar
COPY ${JAR_FILE} app.jar
#ENTRYPOINT ["java", "-jar", "-Dhttp.proxyHost=10.0.16.240", "-Dhttps.proxyHost=10.0.16.240", "/app.jar"]
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom", "-jar","/app.jar"]
```

### Command
`docker-compose up`

### Site Info
http://localhost:5601

