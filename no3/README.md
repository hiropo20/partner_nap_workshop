
# Lab手順

## 実施環境
* 事前にラボ環境へのInviteを行っておりますので、メールをご確認ください
* 利用するコマンド： git , docker, docker-compose , jq , sudo, curl

## UDF コンポーネントへの接続
Windows Jump HostからCLIの操作を行う場合、以下タブからRDP Clientファイルをダウンロードいただき接続ください
<br><img src="https://user-images.githubusercontent.com/43058573/121031351-3a086900-c7e5-11eb-8a85-f634aa923d4e.png" alt="RDP" width="200"><br>
Docker Hostへの接続は以下メニューを開き利用ください
<br><img src="https://user-images.githubusercontent.com/43058573/121031353-3a086900-c7e5-11eb-90a9-28283cebd5a9.png" alt="Docker Menu" width="200"><br>

## Git clone
ラボで必要なファイルをGitHubから取得
```
cd ~/
git clone https://github.com/laurentpf5/nap-partner-campaign
```
## NGINX Plus lab
### 1. NGINX Plus Container Imageの作成
#### ディレクトリの移動
```
cd ~/nap-partner-campaign/nplus-container
```
#### NGINX Licenseファイルのコピー
```
cp ~/nginx-repo.crt .
cp ~/nginx-repo.key .
```
#### Dockerfileの内容確認
```
less Dockerfile
```
#### NGINX Plus ContainerのBuild
```
docker build --no-cache -t nginxplus .
```
#### 作成したDocker Imageの確認
```
docker images | grep nginx
```
出力結果例
```
$ docker images | grep nginxplus
nginxplus            latest        e36a40c4f511   40 seconds ago   88.1MB
```

### 2. NGINX Container 動作確認
#### ラボ環境の実行
```
docker-compose -f docker-compose-labnginx.yaml up -d 
```
#### コンテナ動作状況の確認
```
docker ps 
```
出力結果例
```
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS          PORTS                                               NAMES
deafd73eac5f   nginxplus:latest     "nginx -g 'daemon of…"   3 minutes ago   Up 57 seconds   80/tcp, 0.0.0.0:8000->8000/tcp, :::8000->8000/tcp   nplus-container_nginxplus_1
b2ba5ea66ff1   ianwijaya/hackazon   "supervisord -n"         3 minutes ago   Up 3 minutes    0.0.0.0:8081->80/tcp, :::8081->80/tcp               nplus-container_app1_1
ecf015fa2d2a   ianwijaya/hackazon   "supervisord -n"         3 minutes ago   Up 3 minutes    0.0.0.0:8082->80/tcp, :::8082->80/tcp               nplus-container_app2_1

```
正しくNGINX Plus Containerが起動しない場合、以下内容を参考に再度docker-composeを実行ください
```

$ docker ps
CONTAINER ID   IMAGE                COMMAND            CREATED          STATUS         PORTS                                   NAMES
b2ba5ea66ff1   ianwijaya/hackazon   "supervisord -n"   11 seconds ago   Up 7 seconds   0.0.0.0:8081->80/tcp, :::8081->80/tcp   nplus-container_app1_1
ecf015fa2d2a   ianwijaya/hackazon   "supervisord -n"   11 seconds ago   Up 7 seconds   0.0.0.0:8082->80/tcp, :::8082->80/tcp   nplus-container_app2_1


$ docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                      PORTS                                   NAM                                                             ES
deafd73eac5f   nginxplus:latest     "nginx -g 'daemon of…"   15 seconds ago   Exited (1) 13 seconds ago                                           nplus-container_nginxplus_1
b2ba5ea66ff1   ianwijaya/hackazon   "supervisord -n"         15 seconds ago   Up 10 seconds               0.0.0.0:8081->80/tcp, :::8081->80/tcp   nplus-container_app1_1
ecf015fa2d2a   ianwijaya/hackazon   "supervisord -n"         15 seconds ago   Up 10 seconds               0.0.0.0:8082->80/tcp, :::8082->80/tcp   nplus-container_app2_1

nginxplus が Exit していることが確認できる

nginxplusのログを確認

$ docker logs $(docker ps -a -f name=nginxplus  -q)
2021/06/08 02:07:43 [emerg] 1#1: host not found in upstream "app1:80" in /etc/nginx/nginx.conf:26
nginx: [emerg] host not found in upstream "app1:80" in /etc/nginx/nginx.conf:26

docker-compsoe を再度実行
$ docker-compose -f docker-compose-labnginx.yaml up -d
nplus-container_app2_1 is up-to-date
Starting nplus-container_nginxplus_1 ...
Starting nplus-container_nginxplus_1 ... done

$ docker ps -a -f name=nginxplus
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                                               NAMES
deafd73eac5f   nginxplus:latest   "nginx -g 'daemon of…"   7 minutes ago   Up 5 minutes   80/tcp, 0.0.0.0:8000->8000/tcp, :::8000->8000/tcp   nplus-container_nginxplus_1

```
#### 疎通の確認
```
curl -s http://localhost:8000/ | head
```
#### ラボ環境の停止
```
docker-compose -f docker-compose-labnginx.yaml down
```

## NGINX App Protect lab
### 1. NGINX Plus + NGINX App Protect Container Imageの作成

#### ディレクトリの移動
```
cd ~/nap-partner-campaign/app-protect-container
```
#### NGINX Licenseファイルのコピー
```
cp ~/nginx-repo.crt .
cp ~/nginx-repo.key .
```
#### Dockerfileの内容確認
```
less Dockerfile
```
#### NGINX Plus + NGINX App Protect ContainerのBuild
```
docker build --no-cache -t app-protect .
```
#### 作成したDocker Imageの確認
```
docker images | grep nginxplus
```


### 2. NGINX Plus + NGINX App Protect 動作確認
#### ラボ環境の実行
```
docker-compose -f docker-compose-labnginx.yaml up -d
```
#### コンテナ動作状況の確認
```
docker ps 
```
#### ELKの設定投入
```
./importkibana.sh 
```
- ※ELKの起動には時間がかかるため、以下のようなエラーとなった場合には少し時間を開けて実行ください   
- ※docker psの結果が正しいにもかかわらず意図した通りELKの設定ができない場合には、一度dockmer-composeを停止の後、再度実行してください   
- ※docker exec <container> bash / ps -aef の結果はりつけておく   

#### NGINX App Protectが正しく動作していることを確認
```
docker logs $(docker ps -f name=approtect -q)
```
※出力結果例※

#### 疎通の確認
```
curl http://localhost/ | head
curl http://localhost/?a=%3Cscript%3E | head
```
※現在ブロックする設定ではないため、WebPageの内容が出力される

#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no3/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）

### 3. NGINX App Protectのログポリシー変更
#### NGINX App Protectコンテナのログ設定ファイルを修正
***修正***
```
docker exec -it <container> bash
grep access_log /etc/nginx/nginx.conf
vi /etc/nginx/custom_log_format.json

変更内容
"request_type": "all"
to
"request_type": "illegal" 
```
#### シェルから抜ける
```
exit
```
#### 設定の読み込み
```
docker exec -it <container> nginx -s reload
```
#### ログの確認
```
docker logs <container>
```
#### いくつかの攻撃リクエストを実行し、その結果を確認する
```
./badtraffic.sh
```
#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no3/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）

### 4. NGINX App Protectのセキュリティポリシー変更
NGINX App Protectコンテナのポリシー設定ファイルを修正
***修正***
```
docker exec -it <container> bash
grep access_policy /etc/nginx/nginx.conf
vi /etc/nginx/policy.json

変更内容
        "enforcementMode": "tranceparent"
to
        "enforcementMode": "blocking"
```
#### シェルから抜ける
```
exit
```
#### 設定の読み込み
```
docker exec -it <container> nginx -s reload
```
#### ログの確認
```
docker logs <container>
```
#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no3/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）



## Kibana操作画面
左側メニューよりDashboardを選択し、ご覧になりたい内容を選択ください
<br><img src="https://user-images.githubusercontent.com/43058573/121031357-3aa0ff80-c7e5-11eb-828d-50d298736837.png" alt="Menu Dashboard" width="150"><br>
<br><img src="https://user-images.githubusercontent.com/43058573/121031356-3aa0ff80-c7e5-11eb-9f7e-9ec2b907592c.png" alt="Dashboard 選択画面" width="450"><br>
<br><img src="https://user-images.githubusercontent.com/43058573/121031361-3b399600-c7e5-11eb-9c33-1c2231de21d1.png" alt="Dashboard Flase Positive" width="500"><br>
<br><img src="https://user-images.githubusercontent.com/43058573/121031341-383ea580-c7e5-11eb-9970-44736c1479ed.png" alt="Dashboard Overview" width="500"><br>
左側メニューよりDiscoverを選択ください
<br><img src="https://user-images.githubusercontent.com/43058573/121031360-3b399600-c7e5-11eb-9fc7-8fa1b704c3ab.png" alt="Menu Discover" width="150"><br>
<br><img src="https://user-images.githubusercontent.com/43058573/121031345-396fd280-c7e5-11eb-9b46-2f8744ec11a1.png" alt="Discover" width="500"><br>
