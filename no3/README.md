# Lab手順

## 実施環境
* 事前にラボ環境へのInviteを行っておりますので、メールをご確認ください
* 利用するコマンド： git , docker, docker-compose , jq , sudo, curl

## Git clone
ラボで必要なファイルをGitHubから取得
```
cd ~/
git clone https://github.com/laurentpf5/nap-partner-campaign
```
## NGINX Plus lab
### 1. NGINX Plus Container Imageの作成

```
cd ~/nap-partner-campaign/nplus-container
```
NGINX Licenseファイルのコピー
```
cp ~/nginx-repo.crt .
cp ~/nginx-repo.key .
```
Dockerfileの内容確認
```
less Dockerfile
```
NGINX Plus ContainerのBuild
```
docker build --no-cache -t nginxplus .
```
作成したDocker Imageの確認
```
docker images | grep nginx
```
※出力結果例※
### 2. NGINX Container 動作確認
ラボ環境の実行
```
docker-compose -f docker-compose-labnginx.yaml up -d 
```
コンテナ動作状況の確認
```
docker ps 
```
疎通の確認
```
curl http://localhost:8000/ | head
```
ラボ環境の停止
```
docker-compose -f docker-compose-labnginx.yaml down
```

## NGINX App Protect lab
### 1. NGINX Plus + NGINX App Protect Container Imageの作成

Docker Container ImageをPull
```
cd ~/nap-partner-campaign/app-protect-container
```
NGINX Licenseファイルのコピー
```
cp ~/nginx-repo.crt .
cp ~/nginx-repo.key .
```
Dockerfileの内容確認
```
less Dockerfile
```
NGINX Plus + NGINX App Protect ContainerのBuild
```
docker build --no-cache -t app-protect .
```
作成したDocker Imageの確認
```
docker images | grep nginx
```
※出力結果例※

### 2. NGINX Plus + NGINX App Protect 動作確認
ラボ環境の実行
```
docker-compose -f docker-compose-lab-appprotect.yaml up -d
```
コンテナ動作状況の確認
```
docker ps 
```
※出力結果例※

ELKの設定投入
```
./importkibana.sh 
```
※ELKの起動には時間がかかるため、以下のようなエラーとなった場合には少し時間を開けて実行ください
※docker psの結果が正しいにもかかわらず意図した通りELKの設定ができない場合には、一度dockmer-composeを停止の後、再度実行してください

※docker exec <container> bash / ps -aef の結果はりつけておく

NGINX App Protectが正しく動作していることを確認
```
docker logs $(docker ps -f name=approtect -q)
```
※出力結果例※

疎通の確認
```
curl http://localhost/ | head
curl http://localhost/?a=%3Cscript%3E | head
```
※現在ブロックする設定ではないため、WebPageの内容が出力される

Kibanaを開き、結果を確認
1. Dashboard
    1. Overview
    1. False Positive
1. Discover

### 2. NGINX App Protectのログポリシー変更
NGINX App Protectコンテナのログ設定ファイルを修正
***修正***
```
docker exec -it <container> bash
grep access_log /etc/nginx/nginx.conf
vi /etc/nginx/custom_log_format.json
変更内容
```
"request_type": "all"
to
"request_type": "illegal" 
```
シェルから抜ける
```
exit
```
設定の読み込み
```
docker exec -it <container> nginx -s reload
```
ログの確認
```
docker logs <container>
```
いくつかの攻撃リクエストを実行し、その結果を確認する
```
./badtraffic.sh
```
Kibanaを開き、結果を確認

### 3. NGINX App Protectのセキュリティポリシー変更
NGINX App Protectコンテナのログ設定ファイルを修正
***修正***
```
docker exec -it <container> bash
grep access_policy /etc/nginx/nginx.conf
vi /etc/nginx/policy.json
変更内容
```
        "enforcementMode": "tranceparent"
to
        "enforcementMode": "blocking"
```
シェルから抜ける
```
exit
```
設定の読み込み
```
docker exec -it <container> nginx -s reload
```
ログの確認
```
docker logs <container>
```
Kibanaを開き、結果を確認
