# How to Deploy Elastic Stack _(ELK)_ on Docker Engine
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

1. Docker Host is connected to the Internet for downloading project components. 
2. Docker Engine and Docker Compose are installed and configured (recommended).
3. Git is installed to clone the ELK Docker repository (optional).
4. Learn more about Elastic Stack at https://www.elastic.co/elastic-stack/.
5. Access to a web browser to view the Kibana dashboard.
6. Command-line interface/terminal access with sudo privileges.

### Successful Tested Environment
___Chosen variables:___ These variables can be edited in `.env` file
- ELK Stack demo version: `7.17.7`
- Username: `elastic`
- Password: `kanelelk`

___Best practice:___ CentOS 7 Server  
Learn how to install Docker Engine on CentOS at https://docs.docker.com/engine/install/centos/.

___Other environment:___ MacOS X  
Learn how to install Docker Desktop on MacOS at https://docs.docker.com/desktop/install/mac-install/.  
- Docker Host:
  - macOS Catalina 10.15.7
- Docker Server:
  - Docker Desktop v4.15.0
  - Docker Engine v20.10.21
  - Docker Compose v2.13.0
  - OS/Arch: linux/amd64
> **Warning**  
> Some metricsets or modules in Metricbeat may not work on MacOS, because they are created to monitor Linux systems. The special filesystems /proc and /sys are only available if the host system is running Linux. Attempts to bind-mount these filesystems will fail on MacOS (or Windows).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Installation - TL;DR

1. Open Visual Code Console or System Terminal or any Coding Editor
2. Sign in to GitHub
   ```console
   (Syntax:)
   https://<username>:<password>@github.com/<username>/<repository.git>
   
   (Add your user name:)
   kanel@Mac-Pro ~ % git config --global user.name "your_username"
   
   (Add your email address:)
   kanel@Mac-Pro ~ % git config --global user.email "your_email_address@example.com"
   
   (Check the configuration:)
   kanel@Mac-Pro ~ % git config --global --list
   ```
3. Clone the `elk` repository from GitHub
   ```console
   kanel@Mac-Pro ~ % git clone https://github.com/kanelorange/elk.git
   ```
4. Change to `elk` directory, modify the `.env` file and enter values for environment variables
   ```console
   kanel@Mac-Pro ~ % cd elk
   ```
5. Run **Docker Compose** command to build up and start the **ELK Containers**
   ```console
   kanel@Mac-Pro elk % sudo docker-compose up -d
   Password:
   ```
6. When the deployment has started, open a browser and access to **Kibana Dashboard**, where you can load sample data and interact with your cluster  
   ___From Docker Host:___
   ```console
   http://localhost:5601
   ```
   ___From other clients:___ access to Docker Host via its IP address or hostname 
   ```console
   http://docker-hostname:5601
   ```

### Stop and remove the deployment

1. To stop the cluster, run `docker-compose down`. The data in the Docker volumes is preserved and loaded when you restart the cluster with `docker-compose up`.
   ```console
   kanel@Mac-Pro elk % sudo docker-compose down
   ```
2. To delete the network, containers, and volumes when you stop the cluster, specify the `-v` option:
   ```console
   kanel@Mac-Pro elk % sudo docker-compose down -v
   ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE -->
## Usage

### Custom _ENVIRONMENT variables_ for personal interest
Define your personal variables in `.env` file. These variables are used in the entire project. They will make differences to Elastic Stack version, containers' names, users, passwords, JAVA, etc. 

Here is an example with ELK Stack version `7.17.7`:
  ```dockerfile
  ELK_VERSION="7.17.7"
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
  ```dockerfile
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM elasticsearch:${ELK_VERSION}
  ```
- Create `Dockerfile` in `elk\logstash_data` directory:
  ```dockerfile
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM logstash:${ELK_VERSION}
  ```
- Create `Dockerfile` in `elk\kibana_data` directory:
  ```dockerfile
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM kibana:${ELK_VERSION}
  ```

**2. Metricbeat (monitoring Docker Host)**
- Create `Dockerfile` in `elk\metricbeat_data` directory:
  ```dockerfile
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

>### How to fix Elasticsearch docker container errors
>___Error:___
>```ruby
>cont_elastic           | "stacktrace": "org.elasticsearch.bootstrap.StartupException: ElasticsearchException
>[failed to bind service]; nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];
>```
><details>
>  <summary><b><i>Error Logs:</i></b></summary>
>
>```ruby
>cont_elastic           | {"type": "server", "timestamp": "2023-06-07T03:15:12,005Z", "level": "ERROR", "component": 
>"o.e.b.ElasticsearchUncaughtExceptionHandler", "cluster.name": "kanel-elk-docker", "node.name": "ELK Master", "message": 
>"uncaught exception in thread [main]", 
>cont_elastic           | "stacktrace": ["org.elasticsearch.bootstrap.StartupException: ElasticsearchException[failed to bind service]; 
>nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];",
>cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:173) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:160) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:77) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:112) ~[elasticsearch-cli-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.cli.Command.main(Command.java:77) ~[elasticsearch-cli-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:125) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:80) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "Caused by: org.elasticsearch.ElasticsearchException: failed to bind service",
>cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:1088) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:309) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:434) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:169) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "... 6 more",
>cont_elastic           | "Caused by: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/data/nodes",
>cont_elastic           | "at sun.nio.fs.UnixException.translateToIOException(UnixException.java:90) ~[?:?]",
>cont_elastic           | "at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:106) ~[?:?]",
>cont_elastic           | "at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111) ~[?:?]",
>cont_elastic           | "at sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:397) ~[?:?]",
>cont_elastic           | "at java.nio.file.Files.createDirectory(Files.java:700) ~[?:?]",
>cont_elastic           | "at java.nio.file.Files.createAndCheckIsDirectory(Files.java:807) ~[?:?]",
>cont_elastic           | "at java.nio.file.Files.createDirectories(Files.java:793) ~[?:?]",
>cont_elastic           | "at org.elasticsearch.env.NodeEnvironment.lambda$new$0(NodeEnvironment.java:300) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.env.NodeEnvironment$NodeLock.<init>(NodeEnvironment.java:224) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.env.NodeEnvironment.<init>(NodeEnvironment.java:298) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:429) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.node.Node.<init>(Node.java:309) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:234) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:434) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:169) ~[elasticsearch-7.17.7.jar:7.17.7]",
>cont_elastic           | "... 6 more"] }
>cont_elastic           | uncaught exception in thread [main]
>cont_elastic           | ElasticsearchException[failed to bind service]; nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];
>cont_elastic           | Likely root cause: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/data/nodes
>cont_elastic           | 	at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:90)
>cont_elastic           | 	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:106)
>cont_elastic           | 	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
>cont_elastic           | 	at java.base/sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:397)
>cont_elastic           | 	at java.base/java.nio.file.Files.createDirectory(Files.java:700)
>cont_elastic           | 	at java.base/java.nio.file.Files.createAndCheckIsDirectory(Files.java:807)
>cont_elastic           | 	at java.base/java.nio.file.Files.createDirectories(Files.java:793)
>cont_elastic           | 	at org.elasticsearch.env.NodeEnvironment.lambda$new$0(NodeEnvironment.java:300)
>cont_elastic           | 	at org.elasticsearch.env.NodeEnvironment$NodeLock.<init>(NodeEnvironment.java:224)
>cont_elastic           | 	at org.elasticsearch.env.NodeEnvironment.<init>(NodeEnvironment.java:298)
>cont_elastic           | 	at org.elasticsearch.node.Node.<init>(Node.java:429)
>cont_elastic           | 	at org.elasticsearch.node.Node.<init>(Node.java:309)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:234)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:234)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:434)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:169)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:160)
>cont_elastic           | 	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:77)
>cont_elastic           | 	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:112)
>cont_elastic           | 	at org.elasticsearch.cli.Command.main(Command.java:77)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:125)
>cont_elastic           | 	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:80)
>cont_elastic           | For complete error details, refer to the log at /usr/share/elasticsearch/logs/kanel-elk-docker.log
>cont_elastic exited with code 1
>```
></details>
>
>___Solution:___
>- Change the permissions of the host directory that mapped to `/usr/share/elasticsearch/data`.
>   ```console
>   kanel@Mac-Pro elk % sudo chown -R 1000:1000 [directory]
>   ```
>- For example in this instance, that directory is `elk/elasticsearch_data/es_data`.
>   ```console
>   kanel@Mac-Pro elk % sudo chown -R 1000:1000 ./elasticsearch_data/es_data
>   ```

>### How to fix Metricbeat docker containers failed to start
>___Error:___
>```ruby  
>dependency failed to start: container cont_kibana is unhealthy
>```
>___Solution:___
>- Start Metricbeat containers or compose Elastic Stack again.
>   ```console
>   kanel@Mac-Pro elk % sudo docker start cont_metricbeat_stack cont_metricbeat_host
>   (or/and)
>   kanel@Mac-Pro elk % sudo docker-compose up -d
>   ```

>### How to check Docker Engine is running
>___Error:___
>```console
>kanel@Mac-Pro elk % docker ps
>Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
>```
>___Solution:___
>- Run Docker Engine or start Docker Desktop
>   ```console
>   kanel@Mac-Pro elk % sudo systemctl start docker
>   Password:
>   ```
  
>### How to read `docker.sock` on MacOS
>___Solution:___
>- Link host `docker.sock` to container `docker.sock`
>   ```console
>   kanel@Mac-Pro ~ % sudo ln -s "$HOME/.docker/run/docker.sock" /var/run/docker.sock
>   Password:
>   ln: /var/run/docker.sock: File exists
>   ```

>### How to fix Beats' logs `Non-zero metrics in the last 30s`
>___Error:___
>Apparently logs are transferred from Beats to Elasticsearch. However the Beats logs continuously show this message:
>```ruby
>INFO    [monitoring]    log/log.go:123  Non-zero metrics in the last 30s ...
>```
>___Solution:___
>- It is a periodical metrics report by the Beats. If you would like to disable it or to see it less frequently you can change it in the configuration file:
>   ```yaml
>   # If enabled, filebeat periodically logs its internal metrics that have changed
>   # in the last period. For each metric that changed, the delta from the value at
>   # the beginning of the period is logged. Also, the total values for
>   # all non-zero internal metrics are logged on shutdown. The default is true.
>   logging.metrics.enabled: false
>   
>   # The period after which to log the internal metrics. The default is 30s.
>   #logging.metrics.period: 30s
>   ```  

>### Beats create a “Standalone Cluster” in Kibana Monitoring
>___Elastic.co Explaination:___
>```console
># In Beats version 7.2, we allowed Beats to ship their monitoring data directly to
># a Monitoring Elasticsearch cluster (which might be separate from your Production
># Elasticsearch cluster, where you send the log data being harvested by Beats).
># The new monitoring.* settings introduced in 7.2 let you set the Monitoring cluster
># for Beats.
>
># Now, if your output is elasticsearch, then we know which Elasticsearch cluster your
># Beat is sending its logs data to. In that case the Monitoring UI can correctly associate
># the Beat instance with that Elasticsearch cluster.
>
># However, if the output is something other than elasticsearch, we cannot know if the
># logs data is ending up in an Elasticsearch cluster. So the Monitoring UI shows the
># Beat instance in a "Standalone Cluster".
>```
>___Solution:___
>   ```console
>   As a temporary workaround, you can go back to using the deprecated xpack.monitoring.*
>   settings (instead of the new monitoring.* ones). However, please note that the
>   xpack.monitoring.* settings will eventually go away in a future major version.
>   ``` 

>### How to fix `bootstrap check failure: memory locking requested...`
>___Error:___
>```ruby
>cont_elastic           | ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
>cont_elastic           | bootstrap check failure [1] of [1]: memory locking requested for elasticsearch process but memory is not locked
>```
><details>
>  <summary><b><i>Error Logs:</i></b></summary>
>
>```ruby
>cont_elastic           | ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
>cont_elastic           | bootstrap check failure [1] of [1]: memory locking requested for elasticsearch process but memory is not locked
>cont_elastic           | ERROR: Elasticsearch did not exit normally - check the logs at /usr/share/elasticsearch/logs/kanel-elk-docker.log
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T07:03:14,230Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "stopping ..." }
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T07:03:14,243Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "stopped" }
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T07:03:14,244Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "closing ..." }
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T07:03:14,270Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "closed" }
>cont_elastic           | uncaught exception in thread [process reaper (pid 502)]
>cont_elastic           | java.security.AccessControlException: access denied ("java.lang.RuntimePermission" "modifyThread")
>cont_elastic           | 	at java.base/java.security.AccessControlContext.checkPermission(AccessControlContext.java:485)
>cont_elastic           | 	at java.base/java.security.AccessController.checkPermission(AccessController.java:1068)
>cont_elastic           | 	at java.base/java.lang.SecurityManager.checkPermission(SecurityManager.java:411)
>cont_elastic           | 	at org.elasticsearch.secure_sm.SecureSM.checkThreadAccess(SecureSM.java:160)
>cont_elastic           | 	at org.elasticsearch.secure_sm.SecureSM.checkAccess(SecureSM.java:120)
>cont_elastic           | 	at java.base/java.lang.Thread.checkAccess(Thread.java:2360)
>cont_elastic           | 	at java.base/java.lang.Thread.setDaemon(Thread.java:2308)
>cont_elastic           | 	at java.base/java.lang.ProcessHandleImpl.lambda$static$0(ProcessHandleImpl.java:103)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.<init>(ThreadPoolExecutor.java:637)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor.addWorker(ThreadPoolExecutor.java:928)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor.processWorkerExit(ThreadPoolExecutor.java:1021)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1158)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
>cont_elastic           | 	at java.base/java.lang.Thread.run(Thread.java:1589)
>cont_elastic           | 	at java.base/jdk.internal.misc.InnocuousThread.run(InnocuousThread.java:186)
>cont_elastic exited with code 78
>```
></details>
>
>___Elastic.co Explaination:___
>```js
># Most operating systems try to use as much memory as possible for file system caches and
># eagerly swap out unused application memory. This can result in parts of the JVM heap or
># even its executable pages being swapped out to disk.
>
># Swapping is very bad for performance, for node stability, and should be avoided at all costs.
># It can cause garbage collections to last for minutes instead of milliseconds and can cause
># nodes to respond slowly or even to disconnect from the cluster. In a resilient distributed
># system, it’s more effective to let the operating system kill the node.
>
># There are three approaches to disabling swapping. The preferred option is to completely
># disable swap. If this is not an option, whether or not to prefer minimizing swappiness
># versus memory locking is dependent on your environment.
>
># Disable all swap files:
>
># Usually Elasticsearch is the only service running on a box, and its memory usage is controlled
># by the JVM options. There should be no need to have swap enabled.
>
># On Linux systems, you can disable swap temporarily by running: `sudo swapoff -a`
>
># This doesn’t require a restart of Elasticsearch.
># To disable it permanently, you will need to edit the `/etc/fstab` file and comment out any lines
># that contain the word `swap`.
>```
>___Solution:___
>- Disable swapping:
>   ```java
>   Swapping needs to be disabled for performance and node stability.
>   For information about ways to do this, see [Elasticsearch important memory configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html).
>   ```  
>- Define the memlock:
>   ```js
>   If you opt for the `bootstrap.memory_lock: true` approach, you also need to define the
>   `memlock: true` ulimit in the Docker Daemon, or explicitly set for the container as shown
>   in the docker-compose file. When using docker run, you can specify:
>
>   -e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1
>   ```    

  
  
>### How to fix `bootstrap check failure: Transport SSL must be enabled...`
>___Error:___
>```ruby
>cont_elastic           | ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
>cont_elastic           | bootstrap check failure [1] of [1]: Transport SSL must be enabled if security is enabled on a [basic] license. Please set [xpack.security.transport.ssl.enabled] to [true] or disable security by setting [xpack.security.enabled] to [false]
>```
><details>
>  <summary><b><i>Error Logs:</i></b></summary>
>  
>```ruby  
>cont_elastic           | ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
>cont_elastic           | bootstrap check failure [1] of [1]: Transport SSL must be enabled if security is enabled on a [basic] license. Please set [xpack.security.transport.ssl.enabled] to [true] or disable security by setting [xpack.security.enabled] to [false]
>cont_elastic           | ERROR: Elasticsearch did not exit normally - check the logs at /usr/share/elasticsearch/logs/kanel-elk-docker.log
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T08:06:15,521Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "stopping ..." }
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T08:06:15,539Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "stopped" }
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T08:06:15,539Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "closing ..." }
>cont_elastic           | {"type": "server", "timestamp": "2023-06-14T08:06:15,555Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "kanel-elk-docker", "node.name": "ELK-Master", "message": "closed" }
>cont_elastic           | uncaught exception in thread [process reaper (pid 502)]
>cont_elastic           | java.security.AccessControlException: access denied ("java.lang.RuntimePermission" "modifyThread")
>cont_elastic           | 	at java.base/java.security.AccessControlContext.checkPermission(AccessControlContext.java:485)
>cont_elastic           | 	at java.base/java.security.AccessController.checkPermission(AccessController.java:1068)
>cont_elastic           | 	at java.base/java.lang.SecurityManager.checkPermission(SecurityManager.java:411)
>cont_elastic           | 	at org.elasticsearch.secure_sm.SecureSM.checkThreadAccess(SecureSM.java:160)
>cont_elastic           | 	at org.elasticsearch.secure_sm.SecureSM.checkAccess(SecureSM.java:120)
>cont_elastic           | 	at java.base/java.lang.Thread.checkAccess(Thread.java:2360)
>cont_elastic           | 	at java.base/java.lang.Thread.setDaemon(Thread.java:2308)
>cont_elastic           | 	at java.base/java.lang.ProcessHandleImpl.lambda$static$0(ProcessHandleImpl.java:103)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.<init>(ThreadPoolExecutor.java:637)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor.addWorker(ThreadPoolExecutor.java:928)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor.processWorkerExit(ThreadPoolExecutor.java:1021)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1158)
>cont_elastic           | 	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
>cont_elastic           | 	at java.base/java.lang.Thread.run(Thread.java:1589)
>cont_elastic           | 	at java.base/jdk.internal.misc.InnocuousThread.run(InnocuousThread.java:186)
>cont_elastic exited with code 78
>```  
></details>  
>  
>___Solution:___
>- Set the configuration in `elasticsearch.yml` file:
>   ```yaml
>   xpack.security.transport.ssl.enabled: true
>   ```
  
<!-- REFERENCES -->
## References  
[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)  
[Install Kibana with Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)
<p align="right">(<a href="#readme-top">back to top</a>)</p>
