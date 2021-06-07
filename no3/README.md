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
### 2. NGINX Container Lab
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

### 2. NGINX Container Lab
ラボ環境の実行
```
docker-compose -f docker-compose-lab-appprotect.yaml up -d
```
コンテナ動作状況の確認
```
docker ps 
```
※出力結果例※

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


