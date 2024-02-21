# SavC-MiniDockerProject

[SYS265 GitHub Wiki](https://github.com/savannahc502/SavC-TechJournal-SYS265/wiki)

## Overview of the Project

**Goal:** Docker is a great way to run applications independently of your OS, plus the Docker Hub stores thousands of images to pull from using the dimple `docker run` command. However, Docker Compose facilitates simultaneous use of two or more Docker containers, allowing for interaction between them. The configuration is specified in a YAML file that can be customized to one's use case. This mini-project aims to practice creating the files necessary to use Docker Compose. 

**Project Services:** Deploying Chronograf, InfluxDB, and Telegraf to create a network monitoring stack of services. 
* InfluxDB: Time-series datapase
* Chronograf: Web-based interface that visulaizes data stored in InfluxDB. Depends on InfluxDB to run (or a similar database)
* Telegraf: Agent to collect, process, and send system metrics.

In my project setup, telegraf would be used to collect system metrics of the nodes on a given network. That data would then be sent to InfluxDB to be organized, and Chronograf displays it in a visualize dashboard format. All three are essential and dependent on one another in order to have a proper network monitoring system (though similar services could replace some, like Grafana and Prometheus). 

**Resources:** 
* [What is Docker Compose? How to Use it with an Example](https://www.freecodecamp.org/news/what-is-docker-compose-how-to-use-it/)
* [GitHub: docker-compose-telegraf-influxdb-grafana](https://github.com/fcuiller/docker-compose-telegraf-influxdb-grafana/blob/master/docker-compose.yml)
* [How do I create a folder in a GitHub repository?](https://stackoverflow.com/questions/12258399/how-do-i-create-a-folder-in-a-github-repository)
* [Introduction to Docker Compose](https://www.baeldung.com/ops/docker-compose)
* [Diego P.'s Project for SYS-265](https://github.com/dpzrz/Docker-Mini-Project-SYS265)
* [DockerHub: telegraf](https://hub.docker.com/_/telegraf)

***
## File Structure

![image](https://github.com/savannahc502/SavC-MiniDockerProject/assets/113316987/3e006a78-14cf-47e8-8839-dc8db1f13faa)

### Git for Pulling GitHub Repositories
Git is a powerful tool for pulling code from GitHub repositories. 
* `sudo apt install git` > command for install may differ between operating systems. This project is being tested on an Ubuntu 20.04 Cloud Server
* `sudo git clone https://github.com/savannahc502/SavC-MiniDockerProject.git` > this will copy the repository to the current working directory

***
## The YAML File
Docker Compose uses the YAML file in order to run your specified services and the configurations. In this section I will go through what the configurations mean in my [docker-compose.yml file](https://github.com/savannahc502/SavC-MiniDockerProject/blob/main/docker-compose.yml). 

![image](https://github.com/savannahc502/SavC-MiniDockerProject/assets/113316987/110e931f-d840-4252-a1b5-d45cc50f8667)
* Image specifies that the DockerHub image to be pulled, in this case the most recent version on InfluxDB. It is best practice to pull specific images rather than the latest version for consistency.
* Container name is simply what you want the image to be called
* Ports map the port 8086 on the host to the port 8086 in the container. This is an important step to have proper communication between nodes and the container
* Volumes: mounts the volume named 'influxdb-storage' to the path 'var/lib/influxdb' in the container. The concepts of volumes are explored later in this file.
* Environment variables: Specifiying user login information so that it is configured upon deployment 

***
![image](https://github.com/savannahc502/SavC-MiniDockerProject/assets/113316987/581a40ee-9c62-4729-9345-c21e9ca5724d)
* Latest image for named the conatiner chronograf
* Maps ports 8888 on the host to 8888 in the container
* Mounts a volume named chronograf-storage to /var/lib/chronograf in the container.
* Depends on: Ensures that the Chronograf service starts only after the InfluxDB service has started.
* INFLUXDB_URL: Sets the InfluxDB URL for communication to 'http://influxdb:8086'

***
![image](https://github.com/savannahc502/SavC-MiniDockerProject/assets/113316987/e165f921-b771-46e9-ab15-47c59569cd99)
* Similar settings as the other services
* Volumes are declared at the end of the file in order to define tha named volumes used by the services. I am creating two named volumes: influxdb-storage and chronograf-storage. These volumes will be available for use by any service in the Docker Compose file. Services can then reference these volumes in their respective configurations.

***
## What about those other files?
The files chronograf.yml, influxdb.yml, and telegraf.conf provide information about the DockerHub images being pulled for the specified services in the YAML file. These files are nicknamed the "docker files" and can be used to define the base image, set up the environment, install dependencies, and more intructions on how it should be built. Common instructions are `FROM`, `COPY`, `RUN`, and `CMD`. For Cronograf and Influxdb, these are very simple files:

![image](https://github.com/savannahc502/SavC-MiniDockerProject/assets/113316987/d0e5c113-7b0f-4cac-9c8f-1bd35cd512e7)
* Simply telling the YAML file to pull the defualt service images

The [telegraf](https://github.com/savannahc502/SavC-MiniDockerProject/blob/main/telegraf/telegraf.conf) service required some more configurations, which I based heavily on the repository linked in the file: 

![image](https://github.com/savannahc502/SavC-MiniDockerProject/assets/113316987/68508b38-8e9c-49d4-a1a9-14793830a276)
* The outputs.influxdb section specifiy that metrics collected should be sent to the influxdb service
* The inputs specify settings for collecting metric data
* The outputs.file section configures Telegraf to write the processed metrics to a file (in this case, it's writing to standard output).

Such configurations for Chronograf and InfluxDB are in the YAML file (specifically the environment variables that are setup already). 

***
## What are Volumes? 
Docker volumes are used in order to persist data between the host, containers, and different containers. This makes sure that there is a way to store and manage container data outside of the contanier, ensuring that the data remains when the container is stopped or removed. Even if the container is removed, the data remains on the host machine. Storing data in volumes makes it easier to back up and restore application data, and it can be manipulated on the host (since the data is stored on the host file system). 

For further research/refresher: https://spacelift.io/blog/docker-volumes 

***
## Important Commands
* `sudo docker-compose up -d`
* `sudo docker ps`
* `sudo docker ps -all`
* `sudo docker stop` or `sudo docker stop container_name`
* `sudo docker-compose down` > shut down containers (do this before any configuration file edits) 
* `docker compose build --no-cache` > build containers from scratch
