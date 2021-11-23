


# Lab手順

## 実施環境
* 事前にラボ環境へのInviteを行っておりますので、メールをご確認ください
* 利用するコマンド： git , docker, docker-compose , jq , sudo, curl
* NGINX Trialライセンスの取得、ラボ実施ユーザのHome Directryへ配置

## UDF コンポーネントへの接続
### Windows Jump HostへのRDP接続
Windows Jump HostからCLIの操作を行う場合、以下タブからRDP Clientファイルをダウンロードいただき接続ください
<br><img src="https://user-images.githubusercontent.com/43058573/121283531-417b5f80-c916-11eb-9ff9-8d0e7e8f1aad.png" alt="RDP" width="200"><br>

Windows Jump Hostへログインいただくと、SSHを実行するバッチファイルがありますので、そちらをダブルクリックしDocker_HOSTへ接続ください
<br><img src="https://user-images.githubusercontent.com/43058573/121170154-c8392980-c88f-11eb-9879-f7bb9c4f13a3.png" alt="ssh"><br>
RDPのUser名、パスワードはDETAILSをクリックし、GeneralのタブのCredentialsの項目を参照ください
<br><img src="https://user-images.githubusercontent.com/43058573/121283534-417b5f80-c916-11eb-88af-9f95c2ced284.png" alt="DETAILS" width="200"><br>
<img src="https://user-images.githubusercontent.com/43058573/121283535-4213f600-c916-11eb-8a89-67362d7a340b.png" alt="Generals" width="300"><br>

### Linux Hostへの接続
Docker Hostへの接続は以下メニューを開き利用ください
<br><img src="https://user-images.githubusercontent.com/43058573/121283528-404a3280-c916-11eb-8a60-bfde13129dfc.png" alt="Docker Menu" width="200"><br>
Docker HOSTへのSSH接続は、Jump Host経由　または、SSH鍵認証を用いて接続可能です。SSH鍵の登録手順は以下を参照ください   
***SSH鍵を登録頂いていない場合、SSHはグレーアウトします***
<br><a href="https://github.com/hiropo20/partner_nap_workshop_secure/blob/main/UDF_SSH_Key.pdf">UDF LAB SSH鍵登録マニュアル</a> (ラボ実施時閲覧可に変更します)<br>

## LAB環境
<br><img src="https://user-images.githubusercontent.com/43058573/122137731-498c4f80-ce80-11eb-84d6-a6e6f97e1a20.png" alt="lab" width="800"><br>
- Windows Jumpo Host経由でログインするか、SSH Clientかどちらかの方法で接続ください
 
## ユーザの確認
Docker Hostにログインし、実行ユーザの確認
```
whoami

出力結果がcentosであることを確認してください。
webshell を利用してrootで操作している場合には、su - centos でユーザを切り替えてください
```

## Docker Composeのインストール
### 1. Install Docker Compose 
Docker composeは複数のコンテナを利用するDockerアプリケーションを定義するツールです。Docker Composeを利用する場合には、YAMLファイルにアプリケーションとして実行したい内容を記述します。その後、コマンド実行時にYAMLファイルを指定することで、指定の通りアプリケーションを起動することが可能です
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
Docker Composeコマンドに適切な権限を付与
```
sudo chmod +x /usr/local/bin/docker-compose
```
Docker Composeコマンドの動作を確認
```
docker-compose --version
```
### 2. ContainerでNGINXを起動

Docker Container ImageをPull
```
docker pull nginx
```
NGINX Containerの起動
```
docker run --name ngx-docker -p 80:80 -d nginx
```
Web Serverの動作確認
```
curl http://localhost:80/
```
以下のコマンドを参考にContainerの動作状況などを確認
docker ps コマンドの結果を確認し、CONTAINER_IDの表示結果を参考に以下を実施
```
docker ps
docker exec -it <<CONTAINER_ID>>  bash
```
上記コマンドと同様の内容を以下コマンドで実行できることを確認
```
docker ps -aqf "name=ngx-docker"
docker exec -it `docker ps -aqf "name=ngx-docker"`  bash
```
Containerの停止
```
docker stop ngx-docker
```
Containerの削除
```
docker rm ngx-docker
```
新しいコンテナの起動
```
docker run --name ngx-docker-new -p 80:80 -d nginx
```
新しいコンテナの停止
```
docker stop ngx-docker-new
```
新しいコンテナの削除
```
docker rm ngx-docker-new
```
### 3.	Docker Composeを利用してアプリケーションを実行
Deployment file:
https://github.com/mcheo-nginx/handson_training/tree/main/basic_docker

GitHubよりファイルを取得
```
cd ~
git clone https://github.com/mcheo-nginx/handson_training.git
cd handson_training/basic_docker/
```
デプロイの実行
```
docker-compose -f docker-compose.yml up -d
```
デプロイ結果の確認
```
curl -sv localhost:8000 | head
```
設定の変更。nginx.conf ファイルの変更。(proxy_passのコメントアウト、“return” directiveの追加)
```
vi ./nginx.conf
```
設定変更内容
```
        location / {
            #proxy_pass http://backend;
            return 200 "Container Lab\n";
        }
```
デプロイしたアプリケーションの再起動
```
docker-compose -f docker-compose.yml restart
```
再度、デプロイ結果の確認
```
curl -sv localhost:8000 | head
```


### 参考: Docker cheatsheet for beginners
Dockerコマンド参考情報。以下の内容やInternetの情報を参考にDockerコマンドを実行してください  

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
