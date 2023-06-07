# How to deploy _ELK_ on Docker
<a name="readme-top"></a>
<!-- ABOUT -->
## About
An Elastic Stack [Elasticsearch - Logstash - Kibana - Beats] running on Docker with Docker Compose

<!-- INTRODUCTION -->
## Introduction

The **ELK** [Elasticsearch, Logstash, Kibana] stack, also known as the **Elastic Stack**, runs on various setups and operating systems. A simple way to try out, install and test the ELK stack is to run it on Docker.

<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

1. Connect to the Internet for project resources. 
2. Docker is installed and configured.
3. Docker Compose is installed and configured (recommended).
4. Git is installed to clone the ELK Docker repository.
5. Access to a web browser to view the Kibana dashboard.
6. Command-line interface/terminal access with sudo privileges.

### Successful Tested Environment

- Docker Host:
  - macOS Catalina 10.15.7
- Docker Server:
  - Docker Desktop v4.15.0
  - Docker Engine v20.10.21
  - Docker Compose v2.13.0
  - OS/Arch: linux/amd64

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Installation

1. Open Visual Code Console or System Terminal or any Coding Editor
2. (On macOS) Link host `docker.sock` to container `docker.sock`
   ```console
   kanel@Mac-Pro ~ % sudo ln -s "$HOME/.docker/run/docker.sock" /var/run/docker.sock
   Password:
   ln: /var/run/docker.sock: File exists
   ```
3. Clone the **elk** repository from GitHub
   ```console
   kanel@Mac-Pro ~ % git clone https://github.com/kanelorange/elk.git
   ```
4. Change to **elk** directory
   ```console
   kanel@Mac-Pro ~ % cd elk
   ```
5. Run **Docker Compose** command to build and run **ELK Containers**
   ```console
   kanel@Mac-Pro elk % sudo docker-compose up -d
   Password:
   ```
6. Access to **Kibana Dashboard** via a web browser
   ```console
   http://localhost:5601
   ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE -->
## Usage

### Custom _ENVIRONMENT variables_ for personal interest
Define your personal variables in `.env` file. These variables are used in the entire project. They will affect to Elastic Stack version, containers' names, users, passwords, JAVA,... 
Here is an example with ELK Stack version 7.17.7:
  ```
  ELK_VERSION=7.17.7
  ES_JAVA_OPTS="-Xmx256m -Xms256m"
  LS_JAVA_OPTS="-Xmx256m -Xms256m"
  HOSTNAME_ELASTIC="cont-elastic"
  HOSTNAME_LOGSTASH="cont-logstash"
  HOSTNAME_KIBANA="cont-kibana"
  HOSTNAME_METRICBEAT="cont-metric"
  ELASTIC_USERNAME="elastic"
  ELASTIC_PASSWORD="kanelelk"
  ```

### Preparing images with DOCKERFILE
**1. Elasticsearch + Logstash + Kibana**
- Create `Dockerfile` in `elk\elasticsearch_data` directory:
  ```
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM elasticsearch:${ELK_VERSION}
  ```
- Create `Dockerfile` in `elk\logstash_data` directory:
  ```
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM logstash:${ELK_VERSION}
  ```
- Create `Dockerfile` in `elk\kibana_data` directory:
  ```
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM kibana:${ELK_VERSION}
  ```

**2. Metricbeat (monitoring Docker Host)**
- Create `Dockerfile` in `elk\metricbeat_data` directory:
  ```
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM elastic/metricbeat:${ELK_VERSION}
  # Change file permission for docker module monitoring
  USER root

  USER metricbeat
  ```
- Note that `/run/docker.sock` file is created after container is created successfull. You must change to user `root` or grant permission: RUN [ "/bin/sh", "-c", "chmod 775 /run/docker.sock" ]
- Note that this image should grant permission to `docker.sock` file for _docker module_ monitoring.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Build ELK Stack containers with DOCKER COMPOSE
**1. Elasticsearch Container**
  ```yaml
  elasticsearch:
    container_name: cont_elastic
    #-----------------------------------
    # Pull image via docker-compose file
    #image: elasticsearch:${ELK_VERSION}
    #-----------------------------------
    # Build image via Dockerfile
    build:
      context: ./elasticsearch_data
      dockerfile: Dockerfile
      tags:
        - "kanelorange/elasticsearch:${ELK_VERSION}"
      args:
        ELK_VERSION: ${ELK_VERSION}
    #-----------------------------------
    hostname: ${HOSTNAME_ELASTIC}
    networks:
      - localnet
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      # Mount elasticsearch configuration file to container
      - type: bind
        source: ./elasticsearch_data/elasticsearch.yml
        target: /usr/share/elasticsearch/config/elasticsearch.yml
        read_only: true
      # Mount ALL elasticsearch instance data directory to container
      - type: bind
        source: ./elasticsearch_data/es_data
        target: /usr/share/elasticsearch/data
    environment:
      ES_JAVA_OPTS: ${ES_JAVA_OPTS}
      HOSTNAME_ELASTIC: ${HOSTNAME_ELASTIC}
      HOSTNAME_LOGSTASH: ${HOSTNAME_LOGSTASH}
      HOSTNAME_KIBANA: ${HOSTNAME_KIBANA}
      HOSTNAME_METRICBEAT: ${HOSTNAME_METRICBEAT}
      ELASTIC_USERNAME: ${ELASTIC_USERNAME}
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "curl --fail ${ELASTIC_USERNAME}:${ELASTIC_PASSWORD}@${HOSTNAME_ELASTIC}:9200/_cluster/health || exit 1"]
      interval: 10s
      timeout: 30s
      retries: 30
  ``` 
**2. Logstash Container**
  ```yaml
  logstash:
    # Wait until elasticsearch has started
    depends_on:
      elasticsearch:
        condition: service_healthy
    container_name: cont_logstash
    #-----------------------------------
    # Pull image via docker-compose file
    #image: logstash:${ELK_VERSION}
    #-----------------------------------
    # Build image via Dockerfile
    build:
      context: ./logstash_data
      dockerfile: Dockerfile
      tags:
        - "kanelorange/logstash:${ELK_VERSION}"
      args:
        ELK_VERSION: ${ELK_VERSION}
    #-----------------------------------
    hostname: ${HOSTNAME_LOGSTASH}
    networks:
      - localnet
    ports:
      - "5044:5044"
      - "9600:9600"
    volumes:
      - type: bind
        source: ./logstash_data/logstash.yml
        target: /usr/share/logstash/config/logstash.yml
        read_only: true
      - type: bind
        source: ./logstash_data/beats.conf
        target: /usr/share/logstash/beats.conf
        read_only: true
    environment:
      LS_JAVA_OPTS: ${LS_JAVA_OPTS}
      HOSTNAME_ELASTIC: ${HOSTNAME_ELASTIC}
      HOSTNAME_LOGSTASH: ${HOSTNAME_LOGSTASH}
      HOSTNAME_KIBANA: ${HOSTNAME_KIBANA}
      HOSTNAME_METRICBEAT: ${HOSTNAME_METRICBEAT}
      ELASTIC_USERNAME: ${ELASTIC_USERNAME}
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD}
  ```
**3. Kibana Container**
  ```yaml
  kibana:
  # Wait until elasticsearch and logstash has started  
    depends_on:
      elasticsearch:
        condition: service_healthy
      logstash:
        condition: service_started
    container_name: cont_kibana
    #-----------------------------------
    # Pull image via docker-compose file
    #image: kibana:${ELK_VERSION}
    #-----------------------------------
    # Build image via Dockerfile
    build:
      context: ./kibana_data
      dockerfile: Dockerfile
      tags:
        - "kanelorange/kibana:${ELK_VERSION}"
      args:
        ELK_VERSION: ${ELK_VERSION}
    #-----------------------------------
    hostname: ${HOSTNAME_KIBANA}
    networks:
      - localnet
    ports:
      - "5601:5601"
    volumes:
      - type: bind
        source: ./kibana_data/kibana.yml
        target: /usr/share/kibana/config/kibana.yml
        read_only: true
    environment:
      HOSTNAME_ELASTIC: ${HOSTNAME_ELASTIC}
      HOSTNAME_LOGSTASH: ${HOSTNAME_LOGSTASH}
      HOSTNAME_KIBANA: ${HOSTNAME_KIBANA}
      HOSTNAME_METRICBEAT: ${HOSTNAME_METRICBEAT}
      ELASTIC_USERNAME: ${ELASTIC_USERNAME}
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "curl -s ${ELASTIC_USERNAME}:${ELASTIC_PASSWORD}@${HOSTNAME_KIBANA}:5601/api/status?v8format=true | grep -q '\"overall\":{\"level\":\"available\"'"]
      #test: ["CMD-SHELL", "curl --fail ${ELASTIC_USERNAME}:${ELASTIC_PASSWORD}@${HOSTNAME_KIBANA}:5601/api/status | grep -q 'available'"]
      interval: 10s
      timeout: 30s
      retries: 30
  ```
**4. Metricbeat Container**
  ```yaml
  metricbeat:
  # Wait until < E-L-K > has started  
    depends_on:
      elasticsearch:
        condition: service_healthy
      logstash:
        condition: service_started
      kibana:
        condition: service_healthy
    container_name: cont_metricbeat
    #-----------------------------------
    # Pull image via docker-compose file
    #image: elastic/metricbeat:${ELK_VERSION}
    #-----------------------------------
    # Build image via Dockerfile
    build:
      context: ./metricbeat_data
      dockerfile: Dockerfile
      tags:
        - "kanelorange/metricbeat:${ELK_VERSION}"
      args:
        ELK_VERSION: ${ELK_VERSION}
    #-----------------------------------
    hostname: ${HOSTNAME_METRICBEAT}
    networks:
      - localnet
    volumes:
      # Mount metricbeat configuration file to container
      - type: bind
        source: ./metricbeat_data/metricbeat.yml
        target: /usr/share/metricbeat/config/metricbeat.yml
        read_only: true
      # Mount docker-host-monitoring directories to container
      - /:/hostfs:ro
      - /proc:/hostfs/proc:ro
      - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      HOSTNAME_ELASTIC: ${HOSTNAME_ELASTIC}
      HOSTNAME_LOGSTASH: ${HOSTNAME_LOGSTASH}
      HOSTNAME_KIBANA: ${HOSTNAME_KIBANA}
      HOSTNAME_METRICBEAT: ${HOSTNAME_METRICBEAT}
      ELASTIC_USERNAME: ${ELASTIC_USERNAME}
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD}
    #-----------------------------------
    # Run metricbeat setup with mounted configuration file
    command:
      'metricbeat -e -c /usr/share/metricbeat/config/metricbeat.yml'
  ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Run ELK Stack containers with DOCKER COMPOSE
```console
kanel@Mac-Pro elk % sudo docker compose up -d
```
### Stop and remove containers
```console
kanel@Mac-Pro elk % sudo docker compose down
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- TROUBLESHOOTING -->
## Troubleshooting

### How to fix Elasticsearch docker container errors
___Error:___
```ruby
cont_elastic           | "stacktrace": "org.elasticsearch.bootstrap.StartupException: ElasticsearchException
[failed to bind service]; nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];
```
<details>
  <summary><b><i>Error Logs:</i></b></summary>

```ruby
cont_elastic           | {"type": "server", "timestamp": "2023-06-07T03:15:12,005Z", "level": "ERROR", "component": 
"o.e.b.ElasticsearchUncaughtExceptionHandler", "cluster.name": "kanel-elk-docker", "node.name": "ELK Master", "message": 
"uncaught exception in thread [main]", 
cont_elastic           | "stacktrace": ["org.elasticsearch.bootstrap.StartupException: ElasticsearchException[failed to bind service]; 
nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];",
cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:173) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:160) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:77) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:112) ~[elasticsearch-cli-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.cli.Command.main(Command.java:77) ~[elasticsearch-cli-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:125) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:80) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "Caused by: org.elasticsearch.ElasticsearchException: failed to bind service",
cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:1088) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:309) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:434) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:169) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "... 6 more",
cont_elastic           | "Caused by: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/data/nodes",
cont_elastic           | "at sun.nio.fs.UnixException.translateToIOException(UnixException.java:90) ~[?:?]",
cont_elastic           | "at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:106) ~[?:?]",
cont_elastic           | "at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111) ~[?:?]",
cont_elastic           | "at sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:397) ~[?:?]",
cont_elastic           | "at java.nio.file.Files.createDirectory(Files.java:700) ~[?:?]",
cont_elastic           | "at java.nio.file.Files.createAndCheckIsDirectory(Files.java:807) ~[?:?]",
cont_elastic           | "at java.nio.file.Files.createDirectories(Files.java:793) ~[?:?]",
cont_elastic           | "at org.elasticsearch.env.NodeEnvironment.lambda$new$0(NodeEnvironment.java:300) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.env.NodeEnvironment$NodeLock.<init>(NodeEnvironment.java:224) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.env.NodeEnvironment.<init>(NodeEnvironment.java:298) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:429) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:309) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:434) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:169) ~[elasticsearch-7.17.7.jar:7.17.7]",
cont_elastic           | "... 6 more"] }
cont_elastic           | uncaught exception in thread [main]
cont_elastic           | ElasticsearchException[failed to bind service]; nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];
cont_elastic           | Likely root cause: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/data/nodes
cont_elastic           | 	at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:90)
cont_elastic           | 	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:106)
cont_elastic           | 	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
cont_elastic           | 	at java.base/sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:397)
cont_elastic           | 	at java.base/java.nio.file.Files.createDirectory(Files.java:700)
cont_elastic           | 	at java.base/java.nio.file.Files.createAndCheckIsDirectory(Files.java:807)
cont_elastic           | 	at java.base/java.nio.file.Files.createDirectories(Files.java:793)
cont_elastic           | 	at org.elasticsearch.env.NodeEnvironment.lambda$new$0(NodeEnvironment.java:300)
cont_elastic           | 	at org.elasticsearch.env.NodeEnvironment$NodeLock.<init>(NodeEnvironment.java:224)
cont_elastic           | 	at org.elasticsearch.env.NodeEnvironment.<init>(NodeEnvironment.java:298)
cont_elastic           | 	at org.elasticsearch.node.Node.<init>(Node.java:429)
cont_elastic           | 	at org.elasticsearch.node.Node.<init>(Node.java:309)
cont_elastic           | 	at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:234)
cont_elastic           | 	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:234)
cont_elastic           | 	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:434)
cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:169)
cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:160)
cont_elastic           | 	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:77)
cont_elastic           | 	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:112)
cont_elastic           | 	at org.elasticsearch.cli.Command.main(Command.java:77)
cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:125)
cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:80)
cont_elastic           | For complete error details, refer to the log at /usr/share/elasticsearch/logs/kanel-elk-docker.log
cont_elastic exited with code 1
```
</details>

___Solution:___
- Change the permissions of the host directory that mapped to `/usr/share/elasticsearch/data`.
```console
kanel@Mac-Pro elk % sudo chown -R 1000:1000 [directory]
```
- For example in this instance, that directory is `elk/elasticsearch_data/es_data`.
```console
kanel@Mac-Pro elk % sudo chown -R 1000:1000 ./elasticsearch_data/es_data
```
