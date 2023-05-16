<a name="readme-top"></a>
# How to deploy _ELK_ on Docker
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

<p align="right">(<a href="#readme-top">back to top</a>)</p>

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
   ```http
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

<p align="right">(<a href="#readme-top">back to top</a>)</p>

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
  
<p align="right">(<a href="#readme-top">back to top</a>)</p>

**2. Metricbeat (monitoring Docker Host)**
- Create `Dockerfile` in `elk\metricbeat_data` directory:
  ```
  ARG ELK_VERSION
  # Get image from Docker Hub
  FROM elastic/metricbeat:${ELK_VERSION}
  # Change file permission for docker module monitoring
  USER root
  RUN [ "/bin/sh", "-c", "chmod 775 /run/docker.sock" ]
  USER metricbeat
  ```
- Note that this image should grant permission to `docker.sock` file for _docker module_ monitoring.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Build and run ELK Stack containers with DOCKER COMPOSE
