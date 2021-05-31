# Lab手順

## 実施環境
* 事前にラボ環境へのInviteを行っておりますので、メールをご確認ください
* 利用するコマンド： git , docker, sudo, curl

## Docker Composeのインストール
### 1. Install Docker Compose 
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
Set execution rights for Docker Compose
```
$ sudo chmod +x /usr/local/bin/docker-compose
```
Test Docker Compose 
```
$ docker-compose --version
```
### 2. Run NGINX as a Container

Pull the container image
```
$ docker pull nginx
```
Run nginx as a container
```
$ docker run --name ngx-docker -p 80:80 -d nginx
```
Test nginx webserver
```
$ curl http://localhost:80/
```
Access the configuration inside the container
Get the ContaineID
```
$ docker exec -it CONTAINER_ID bash
$ docker ps -aqf “name=ngx-docker"
$ docker exec -it `docker ps -aqf "name=ngx-docker"`  bash
```
Stop nginx container
```
$ docker stop ngx-docker
```
Delete nginx container
```
$ docker rm ngx-docker
```
Stop nginx new container
```
$ docker stop ngx-docker-new
```
Delete nginx new container
```
$ docker rm ngx-docker-new
```
### 3.	Deploy application with Docker Compose
Deployment file
https://github.com/mcheo-nginx/handson_training/tree/main/basic_docker
```
$ cd ~
$ git clone https://github.com/mcheo-nginx/handson_training.git
$ cd handson_training/basic_docker/
```
run the deployment
```
$ docker-compose -f docker-compose.yml up -d
```
Test the deployment
```
$ curl -sv localhost:8000 | head
```
Edit Configuration of the deployment
Edit the nginx.conf file. (comment out proxy_pass and add “return” directive)
```
        location / {
            #proxy_pass http://backend;
            return 200 "Container Lab";
        }
```

restart the deployment
```
$ docker-compose -f docker-compose.yml restart
```
Test the deployment
```
$ curl -sv localhost:8000 | head
```


### ref: Docker cheatsheet for beginners
If you want to learn Docker commands, but you don't know where to start here is a nice list of cli commands that I use to manage containers, images and many more using Docker from terminal. 

Docker machine commands  
* Create new: docker-machine create MACHINE  
* List all: docker-machine ls  
* Show env: docker-machine env default  
* Use: eval "$(docker-machine env default)"
* Unset: docker-machine env -u
* Unset: eval $(docker-machine env -u)

Docker image commands
* Download: docker pull IMAGE[:TAG]
* Build from local Dockerfile: docker build -t TAG .
* Build with user and tag: docker build -t USER/IMAGE:TAG .
* List: docker image ls or docker images
* List all: docker image ls -a or docker images -a
* Remove (image or tag): docker image rm IMAGE or docker rmi IMAGE
* Remove all dangling (nameless): docker image prune
* Remove all unused: docker image prune -a
* Remove all: docker rmi $(docker images -aq)
* Tag: docker tag IMAGE TAG
* Save to file:docker save IMAGE > FILE
* Load from file: docker load -i FILE

Docker container commands
* Run from image: docker run IMAGE
* Run with name: docker run --name NAME IMAGE
* Map a port: docker run -p HOST:CONTAINER IMAGE
* Map all ports: docker run -P IMAGE
* Start in background: docker run -d IMAGE
* Set hostname: docker run --hostname NAME IMAGE
* Set domain: docker run --add-host HOSTNAME:IP IMAGE
* Map local directory: docker run -v HOST:TARGET IMAGE
* Change entrypoint: docker run -it --entrypoint NAME IMAGE
* List running: docker ps or docker container ls
* List all: docker ps -a or docker container ls -a
* Stop: docker stop ID or docker container stop ID
* Start: docker start ID
* Stop all: docker stop $(docker ps -aq)
* Kill (force stop): docker kill ID or docker container kill ID
* Remove: docker rm ID or docker container rm ID
* Remove running: docker rm -f ID
* Remove all stopped: docker container prune
* Remove all: docker rm $(docker ps -aq)
* Rename: docker rename OLD NEW
* Create image from container: docker commit ID
* Show modified files: docker diff ID
* Show mapped ports: docker port ID
* Copy from container: docker cp ID:SOURCE TARGET
* Copy to container docker cp TARGET ID:SOURCE
* Show logs: docker logs ID
* Show processes: docker top ID
* Start shell: docker exec -it ID bash

Other useful Docker commands
* Log in: docker login
* Run compose file: docker-compose
* Get info about image: docker inspect IMAGE
* Show stats of running containers: docker stats
* Show version: docker version
