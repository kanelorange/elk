<a name="readme-top"></a>
# elk on Docker
An Elastic Stack [Elasticsearch - Logstash - Kibana - Beats] running on Docker with Docker Compose

<!-- INTRODUCTION -->
## Introduction

The ELK [Elasticsearch, Logstash, Kibana] stack, also known as the Elastic Stack, runs on various setups and operating systems. A simple way to try out, install and test the ELK stack is to run it on Docker.

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
2. Clone the **elk** repository from GitHub
   ```console
   admin@Mac-Pro ~ % git clone https://github.com/kanelorange/elk.git
   ```
3. Change to **elk** directory
   ```console
   admin@Mac-Pro ~ % cd elk
   ```
4. Run **Docker Compose** command to build and run **ELK Containers**
   ```console
   admin@Mac-Pro elk % docker-compose up -d
   ```
5. Access to **Kibana Dashboard** via a web browser
   ```http
   http://localhost:5601
   ```   

<p align="right">(<a href="#readme-top">back to top</a>)</p>
