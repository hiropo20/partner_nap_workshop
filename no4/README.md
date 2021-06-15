


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

## ユーザの確認
実行ユーザの確認
```
whoami

出力結果がcentosであることを確認してください。
webshell を利用してrootで操作している場合には、su - centos でユーザを切り替えてください
```
## Git clone

ラボで必要なファイルをGitHubから取得
```
cd ~/
git clone https://github.com/laurentpf5/nap-partner-campaign.git
```
## NGINX App Protect Lab Log management
### 1. NGINX Plus + NGINX App Protect Container Imageの作成 (前回のおさらい)

#### ディレクトリの移動
```
cd ~/nap-partner-campaign/app-protect-container
```
#### NGINX Licenseファイルのコピー
```
cp ~/nginx-repo.crt .
cp ~/nginx-repo.key .
```
#### NGINX Plus + NGINX App Protect ContainerのBuild
```
docker build --no-cache -t app-protect .
```
#### 作成したDocker Imageの確認
```
docker images | grep app-protect
```

### 2. NGINX Plus + NGINX App Protect 動作確認 (前回のおさらい)
#### ラボ環境の実行
```
docker-compose -f docker-compose-lab-appprotect.yaml up -d
```
#### コンテナ動作状況の確認
```
docker ps 
```
出力結果例
```
$ docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS                                                                                                                                                           NAMES
63fd3abc1f0e   sebp/elk:793         "/usr/local/bin/star…"   3 minutes ago   Up 2 minutes   0.0.0.0:5144->5144/tcp, :::5144->5144/tcp, 0.0.0.0:5601->5601/tcp, :::5601->5601/tcp, 5044/tcp, 9300/tcp, 9600/tcp, 0.0.0.0:9200->9200/tcp, :::9200->9200/tcp   app-protect-container_elasticsearch_1
3777958f5c2c   ianwijaya/hackazon   "supervisord -n"         3 minutes ago   Up 2 minutes   0.0.0.0:8082->80/tcp, :::8082->80/tcp                                                                                                                           app-protect-container_app2_1
b6e1cbef4bcb   app-protect:latest   "sh /root/entrypoint…"   3 minutes ago   Up 2 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp                                                                                                                               app-protect-container_approtect_1
d94a3eff6df4   ianwijaya/hackazon   "supervisord -n"         3 minutes ago   Up 2 minutes   0.0.0.0:8081->80/tcp, :::8081->80/tcp                                                                                                                           app-protect-container_app1_1
```

#### ELKの起動確認
※ELK起動までに少し時間がかかります。１～２分お待ち下さい
以下の内容を参考に正常動作を確認してください
elastic , logstash , kibanaのUIDでプロセスが動作していることを確認し、再度実行してください
```
docker exec -it  $(docker ps -a -f name=elastic  -q) ps -aef
```
出力結果例
```
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 02:28 ?        00:00:00 /bin/bash /usr/local/bin/start.s
root        14     1  0 02:28 ?        00:00:00 /usr/sbin/cron
elastic+   194     1 17 02:28 ?        00:00:40 /opt/elasticsearch/jdk/bin/java
elastic+   225   194  0 02:28 ?        00:00:00 /opt/elasticsearch/modules/x-pac
logstash   394     1 39 02:29 ?        00:01:19 /usr/bin/java -Xms1g -Xmx1g -XX:
kibana     405     1 24 02:29 ?        00:00:48 /opt/kibana/bin/../node/bin/node
root       407     1  0 02:29 ?        00:00:00 tail -f /var/log/elasticsearch/e
root       559     0  0 02:32 pts/0    00:00:00 ps -aef
```
ログの結果を確認
```
docker logs $(docker ps -a -f name=elastic  -q) | grep running
```
出力結果例
```
[2021-06-09T04:13:26,731][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
{"type":"log","@timestamp":"2021-06-09T04:14:34Z","tags":["info","http","server","Kibana"],"pid":390,"message":"http server running at http://0.0.0.0:5601"}
{"type":"log","@timestamp":"2021-06-09T04:14:34Z","tags":["listening","info"],"pid":390,"message":"Server running at http://0.0.0.0:5601"}
```
#### ELKの設定投入
```
./importkibana.sh 
```
正しく投入された場合の出力
```
正しく設定が投入できた場合、JSONの結果が表示される
$ ./importkibana.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 43277  100  1419  100 41858  15306   440k --:--:-- --:--:-- --:--:--  444k
{
  "objects": [
    {
      "id": "eb626140-73a6-11ea-9cfb-3598e28db774",
      "type": "visualization",
      "error": {
        "statusCode": 409,
        "error": "Conflict",
        "message": "Saved object [visualization/eb626140-73a6-11ea-9cfb-3598e28db774] conflict"
      }
    },
※省略※
    {
      "id": "140fbf30-363e-11ea-983a-f74b5d6c2f97",
      "type": "dashboard",
      "error": {
        "statusCode": 409,
        "error": "Conflict",
        "message": "Saved object [dashboard/140fbf30-363e-11ea-983a-f74b5d6c2f97] conflict"
      }
    }
  ]
}

```
ELKが起動しておらず、設定投入がエラーとなった場合以下のような出力になる場合があります
その場合は前の項目を参考に、ELKの起動を確認してください
```
スクリプト実行の結果、Connectionが失敗する
$ ./importkibana.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0 41858    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (56) Recv failure: Connection reset by peer
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0 57179    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (56) Recv failure: Connection reset by peer


一定時間結果して状況が改善しない場合、再度docker-composeを実行してください
docker-compose -f docker-compose-lab-appprotect.yaml down
docker-compose -f docker-compose-lab-appprotect.yaml up -d
``` 

#### NGINX App Protectが正しく動作していることを確認
```
docker logs $(docker ps -f name=approtect -q)
```

#### 疎通の確認
```
curl -s http://localhost/ | head
curl -s http://localhost/?a=%3Cscript%3E | head
```
※現在ブロックする設定ではないため、WebPageの内容が出力される


### 3. NGINX App Protectのログポリシー変更 (前回のおさらい)
#### NGINX App Protectコンテナのログ設定ファイルを修正

```
docker exec -it $(docker ps -f name=approtect -q) bash
grep app_protect_security_log /etc/nginx/nginx.conf

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
docker exec -it $(docker ps -f name=approtect -q) nginx -s reload
```
#### ログの確認
```
docker logs $(docker ps -f name=approtect -q)
```
#### いくつかの攻撃リクエストを実行し、その結果を確認する
```
cat << EOF > badtraffic.sh
#!/bin/bash
curl -s -H "1.2.3.4" http://localhost -o /dev/null
curl -s http://localhost/%09 -o /dev/null
curl -s http://localhost/index.bak -o /dev/null
curl -s http://localhost?a=%3Cscript%3E -o /dev/null
curl -s http://localhost -o /dev/null
curl -s http://localhost/\<script\> -o /dev/null
curl -s -H "Content-Length: -26" http://localhost/ -o /dev/null
curl -s http://localhost/index.php -o /dev/null
curl -s http://localhost/test.exe -o /dev/null
curl -s http://localhost/index.html -o /dev/null
curl -s http://localhost/basic/index.php -o /dev/null
EOF

./badtraffic.sh
```
#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no4/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）


### 4. Security Log 
セキュリティログの設定内容、パラメータについては以下をご覧ください
[Available Security Log Attributes](https://docs.nginx.com/nginx-app-protect/configuration/#available-security-log-attributes)

#### NGINX App Protectコンテナのログ設定ファイルを修正

```
docker exec -it $(docker ps -f name=approtect -q) bash
cp /etc/nginx/custom_log_format.json /etc/nginx/custom_log_format.json-bk
vi /etc/nginx/custom_log_format.json

以下内容に変更
{
    "filter": {
        "request_type": "illegal"
    },
    "content": {
        "format": "user-defined",
        "format_string": "NAP support ID: %support_id% - NAP Violation: %violations% - NAP outcome: %outcome% - NAP reason: %outcome_reason% - NAP policy name: %policy_name% - NAP Rating: %violation_rating% - NAP version: %version% NGINX request: %request% NGINX status: %request_status%", 
        "max_request_size": "any",
        "max_message_size": "10k"
    }
}


vi /etc/nginx/nginx.conf

以下の内容の通り変更（以下変更の他、宛先ファイルなど自由に指定可能)
 app_protect_security_log "/etc/nginx/custom_log_format.json" syslog:server=elasticsearch:5144;
to
 app_protect_security_log "/etc/nginx/custom_log_format.json" stderr;


```
#### シェルから抜ける
```
exit
```
#### 設定の読み込み
```
docker exec -it $(docker ps -f name=approtect -q) nginx -s reload
```

#### 疎通の確認
```
curl -s http://localhost/ | head
curl -s http://localhost/?a=%3Cscript%3E | head
```
※現在ブロックする設定ではないため、WebPageの内容が出力される

#### ログの確認
```
docker logs $(docker ps -f name=approtect -q)
```
定義したフォーマットでログがstderrに出力され、docker logsで表示できることを確認


## NGINX App Protect Lab Policy management
### 1. Policy の変更1

#### NGINX App Protectの設定ファイルを修正

```
docker exec -it $(docker ps -f name=approtect -q) bash
grep policy_file /etc/nginx/nginx.conf
vi /etc/nginx/labpolicy.json

変更内容
  "enforcementMode": "transparent"
to
  "enforcementMode": "blocking"
```
ログの宛先、ログフォーマットの設定を元の状態に戻す
```
vi /etc/nginx/nginx.conf

変更内容
 app_protect_security_log "/etc/nginx/custom_log_format.json" stderr;
to
 app_protect_security_log "/etc/nginx/custom_log_format.json" syslog:server=elasticsearch:5144;

cp /etc/nginx/custom_log_format.json-bk /etc/nginx/custom_log_format.json 

※ 設定するログファイルの設定は以下の通り

{
    "filter": {
        "request_type": "illegal"
    },
    "content": {
        "format": "default",
        "max_request_size": "any",
        "max_message_size": "10k"
    }
}


```


#### シェルから抜ける
```
exit
```
#### 設定の読み込み
```
docker exec -it $(docker ps -f name=approtect -q) nginx -s reload
```
#### ログの確認
```
docker logs $(docker ps -f name=approtect -q)
```

#### 攻撃リクエストを実行し、その結果を確認する
badtrafficの実行
```
./badtraffic.sh
```
curlコマンドを実行し、出力を確認
```
curl -s http://localhost/?a=%3Cscript%3E | head

以下のようなBlock Pageが表示される。Support ID(例：787019502751076693)を控えておく

<html><head><title>Request Rejected</title></head><body>The requested URL was rejected. Please consult with your administrator.<br><br>Your support ID is: 787019502751076693<br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

```
#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no4/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）
Discoverで攻撃の結果を確認する

`outcome: REJECTED` をフィルタの条件として入力し結果を確認

<img src="" alt="rejected" width="400">

先程確認したSupport IDを参考に`support_id:"  **SUPPORT ID**  "` (Support IDをダブルクォーテーション「"」で括る点に注意) と入力し結果を確認
<img src="" alt="supportid" width="400">


### 2. Policy の変更2
#### NGINX App Protectの設定ファイルを修正

```
docker exec -it $(docker ps -f name=approtect -q) bash
grep policy_file /etc/nginx/nginx.conf
vi /etc/nginx/labpolicy.json

変更内容
{
    "policy": {
        "name": "policy-blockjpeg",
        "template": {
            "name": "POLICY_TEMPLATE_NGINX_BASE"
        },
        "applicationLanguage": "utf-8",
        "enforcementMode": "blocking",
        "blocking-settings": {
            "violations": [
                {
                    "name": "VIOL_FILETYPE",
                    "alarm": true,
                    "block": true
                }
            ]
        },
        "filetypes": [
            {
                "name": "jpg",
                "type": "wildcard",
                "allowed": false
            }
        ]
    }


```


#### シェルから抜ける
```
exit
```
#### 設定の読み込み
```
docker exec -it $(docker ps -f name=approtect -q) nginx -s reload
```
#### ログの確認
```
docker logs $(docker ps -f name=approtect -q)
```

#### 攻撃リクエストを実行し、その結果を確認する
curlコマンドを実行し、出力を確認
```
curl -s http://localhost/dummy.jpg | head

以下のようなBlock Pageが表示される。Support ID(例：787019502751076693)を控えておく

<html><head><title>Request Rejected</title></head><body>The requested URL was rejected. Please consult with your administrator.<br><br>Your support ID is: 787019502751076693<br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

```
#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no4/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）
Discoverで攻撃の結果を確認する

`outcome: REJECTED` をフィルタの条件として入力し結果を確認

<img src="" alt="rejected" width="400">

先程確認したSupport IDを参考に`support_id:"  **SUPPORT ID**  "` (Support IDをダブルクォーテーション「"」で括る点に注意) と入力し結果を確認
<img src="" alt="supportid" width="400">


### 3. Policy の変更3
#### NGINX App Protectの設定ファイルを修正

```
docker exec -it $(docker ps -f name=approtect -q) bash
grep policy_file /etc/nginx/nginx.conf
vi /etc/nginx/labpolicy.json

変更内容
{
    "name": "policy-dataguard",
    "template": {
        "name": "POLICY_TEMPLATE_NGINX_BASE"
    },
    "applicationLanguage": "utf-8",
    "enforcementMode": "blocking",
    "blocking-settings": {
        "violations": [
            {
                "name": "VIOL_DATA_GUARD",
                "alarm": true,
                "block": false
            }
        ]
    },
    "data-guard": {
        "enabled": false,
        "maskData": true,
        "creditCardNumbers": true,
        "usSocialSecurityNumbers": true,
        "enforcementMode": "ignore-urls-in-list",
        "enforcementUrls": [],
        "lastCcnDigitsToExpose": 4,
        "lastSsnDigitsToExpose": 4
    }
}

```


#### シェルから抜ける
```
exit
```
#### 設定の読み込み
```
docker exec -it $(docker ps -f name=approtect -q) nginx -s reload
```
#### ログの確認
```
docker logs $(docker ps -f name=approtect -q)
```

#### 攻撃リクエストを実行し、その結果を確認する
curlコマンドを実行し、出力を確認
```
curl -s "http://localhost/?security_id=5364-0756-2298-8054?x=1&y=2" | head
```
※現在ブロックする設定ではないため、WebPageの内容が出力される

#### Kibanaを開き、結果を確認（操作メニューは[こちら](https://github.com/hiropo20/partner_nap_workshop/blob/main/no4/README.md#kibana%E6%93%8D%E4%BD%9C%E7%94%BB%E9%9D%A2)を参照）
Discoverで攻撃の結果を確認する

`request: "*security_id*"` をフィルタの条件として入力し結果を確認

<img src="" alt="rejected" width="400">



## コンテナ内のSignature Database 
### Sinagureがアップデートされたコンテナの作成

```
docker build -f Dockerfile-attack-signatures -t app-protect-signature .
```

コンテナが生成できたことを確認
```
docker images | grep app-protect

app-protect-signature が追加されていることを確認

app-protect-signature   latest     411b37584ef5   3 minutes ago   636MB
app-protect             latest     2ab5a8cac274   3 hours ago     622MB
```

コンテナを起動するための内容を確認

```
$ cat docker-compose-nap-signature.yaml
version: '3'
services:
    approtect-nap-signature:
        image: app-protect-signature:latest
        volumes:
            - ./custom_log_format.json:/etc/nginx/custom_log_format.json
            - ./labpolicy.json:/etc/nginx/labpolicy.json
            - ./nginx.conf-sig:/etc/nginx/nginx.conf
            - ./nginx-repo.crt:/etc/ssl/nginx/nginx-repo.crt
            - ./nginx-repo.key:/etc/ssl/nginx/nginx-repo.key
        ports:
            - "8001:80"

```
実行

```
$ docker-compose -f docker-compose-nap-signature.yaml up -d
WARNING: Found orphan containers (app-protect-container_approtect_1, app-protect-container_app1_1, app-protect-container_elasticsearch_1, app-protect-container_app2_1, app-protect-container_approtect-nap-convertedpolicy_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Creating app-protect-container_approtect-nap-signature_1 ... done

```
以下の通り、正しく起動していることを確認
```
$ docker ps | grep approtect-nap-signature
d37d57824c51   app-protect-signature:latest   "sh /entrypoint.sh"      9 seconds ago       Up 7 seconds       0.0.0.0:8001->80/tcp, :::8001->80/tcp    app-protect-container_approtect-nap-signature_1

```
### 違いの比較
```
利用していたAppProtectコンテナ
$  docker exec -it app-protect-container_approtect_1 bash
[root@20739ad1411a /]# yum list | grep app-protect
app-protect.x86_64                        24+3.512.0-1.el7.ngx           @nginx-plus
app-protect-compiler.x86_64               6.64.2-1.el7.ngx               @nginx-plus
app-protect-engine.x86_64                 6.64.2-1.el7.ngx.el7.centos    @nginx-plus
app-protect-plugin.x86_64                 3.512.0-1.el7.ngx              @nginx-plus


Signature Update済みコンテナ
[centos@ip-10-1-1-5 ~]$  docker exec -it app-protect-container_approtect_1 bash
# yum list | grep app-protect
app-protect.x86_64                        24+3.512.0-1.el7.ngx           @nginx-plus
app-protect-attack-signatures.x86_64      2021.06.11-1.el7.ngx           @app-protect-security-updates
app-protect-compiler.x86_64               6.64.2-1.el7.ngx               @nginx-plus
app-protect-engine.x86_64                 6.64.2-1.el7.ngx.el7.centos    @nginx-plus
app-protect-plugin.x86_64                 3.512.0-1.el7.ngx              @nginx-plus
app-protect-threat-campaigns.x86_64       2021.06.14-1.el7.ngx           @app-protect-security-updates
```

```
利用していたAppProtectはコンテナ作成時のSignature Updateを実施していない
$ docker logs app-protect-container_approtect_1 2>&1 | grep attack_signatures_package
2021/06/15 12:48:46 [notice] 13#13: APP_PROTECT { "event": "configuration_load_success", "software_version": "3.512.0", "user_signatures_packages":[],"attack_signatures_package":{"revision_datetime":"2019-07-16T12:21:31Z"},"completed_successfully":true,"threat_campaigns_package":{}}
2021/06/15 12:49:16 [notice] 13#13: APP_PROTECT { "event": "configuration_load_success", "software_version": "3.512.0", "user_signatures_packages":[],"attack_signatures_package":{"revision_datetime":"2019-07-16T12:21:31Z"},"completed_successfully":true,"threat_campaigns_package":{}}

今回作成したコンテナはSignature Updateを行っている
$ docker logs app-protect-container_approtect-nap-signature_1  2>&1 | grep attack_signatures_package
2021/06/15 14:54:10 [notice] 14#14: APP_PROTECT { "event": "configuration_load_success", "software_version": "3.512.0", "user_signatures_packages":[],"attack_signatures_package":{"revision_datetime":"2021-06-11T14:07:02Z","version":"2021.06.11"},"completed_successfully":true,"threat_campaigns_package":{"revision_datetime":"2021-06-14T13:14:59Z","version":"2021.06.14"}}
```
DockerFileのポイントも比較

## BIG-IP AWAF Security Policyのコンバート
### BIG-IP AWAF上でPolicyの確認
BIG-IPにログイン

以下GitHubのSecurity Policy(XML)をローカルにダウンロード

手順に従ってBIG-IPでインポート

### NGINX App ProtectのPolicyにコンバート
ポリシーコンバートを行うコンテナの作成
```
docker build -f Dockerfile-Converter -t policy-converter .
```
正しくできている
```
$ docker images | grep policy-converter
policy-converter        latest     50dc92c3742f   40 seconds ago      526MB
```
以下の通りファイルを作成
```
$ cat docker-compose-nap-convertedpolicy.yaml
version: '3'
services:
    approtect-nap-convertedpolicy:
        image: app-protect:latest
        volumes:
            - ./custom_log_format.json:/etc/nginx/custom_log_format.json
            - ./convertedpolicy.json:/etc/nginx/labpolicy.json
            - ./nginx.conf:/etc/nginx/nginx.conf
            - ./nginx-repo.crt:/etc/ssl/nginx/nginx-repo.crt
            - ./nginx-repo.key:/etc/ssl/nginx/nginx-repo.key
        ports:
            - "8002:80"
```

BIG-IPポリシーファイルの配置場所
```
mkdir  /var/tmp/convert
```
XMLを配置
```
$ cp policy.xml /var/tmp/convert
$ ls /var/tmp/convert
policy.xml
```
コンバートの実行
```
docker run -v /var/tmp/convert:/var/tmp/convert policy-converter:latest /opt/app_protect/bin/convert-policy -i /var/tmp/convert/policy.xml -o /var/tmp/convert/policy.json | jq

以下の様にNAPのSecurity Policyコンバート際の警告、出力結果の情報を確認してください
{
  "warnings": [
    "Default header '*-bin' cannot be deleted.",
    "Traffic Learning, Policy Building, and staging are unsupported",
    "Element '/plain-text-profiles' is unsupported.",
    "/signature-settings/signatureStaging must be 'false' (was 'true').",
    "/csrf-urls/enforcementAction value 'verify-csrf-token' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_BLOCKING_CONDITION' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_BRUTE_FORCE' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_CONVICTION' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_CSRF_EXPIRED' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_GEOLOCATION' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_HOSTNAME_MISMATCH' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_MALICIOUS_DEVICE' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_MALICIOUS_IP' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_REDIRECT' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_SESSION_AWARENESS' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_WEBSOCKET_BINARY_MESSAGE_LENGTH' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_WEBSOCKET_EXTENSION' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_WEBSOCKET_FRAMES_PER_MESSAGE_COUNT' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_WEBSOCKET_FRAME_LENGTH' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_WEBSOCKET_TEXT_NULL_VALUE' is unsupported.",
    "/blocking-settings/violations/name value 'VIOL_XML_SCHEMA' is unsupported.",
    "/blocking-settings/http-protocols/description value 'Several Content-Length headers' is unsupported.",
    "/blocking-settings/http-protocols/description value 'No Host header in HTTP/1.1 request' is unsupported.",
    "/blocking-settings/http-protocols/description value 'CRLF characters before request start' is unsupported.",
    "/blocking-settings/http-protocols/description value 'Content length should be a positive number' is unsupported.",
    "/blocking-settings/http-protocols/description value 'Bad host header value' is unsupported.",
    "/general/enableEventCorrelation must be 'false' (was 'true').",
    "/urls/performStaging value true is unsupported",
    "Element '/websocket-urls' is unsupported.",
    "/protocolIndependent must be 'true' (was 'false').",
    "Element '/redirection-protection' is unsupported.",
    "Element '/gwt-profiles' is unsupported.",
    "/signature-sets/learn value true is unsupported"
  ],
  "file_size": 23967,
  "filename": "/var/tmp/convert/policy.json",
  "completed_successfully": true
}

```
コンバートされたポリシー情報を確認
```
$ grep Demo_NGINX_Policy /var/tmp/convert/policy.json
      "fullPath" : "/Common/Demo_NGINX_Policy",
      "name" : "Demo_NGINX_Policy",

```
コンバートされたポリシーをコピー
```
cp /var/tmp/convert/policy.json convertedpolicy.json
```

コンバート済みポリシーをNAPで参照し、起動
```
$ docker-compose -f docker-compose-nap-convertedpolicy.yaml up -d
WARNING: Found orphan containers (app-protect-container_app2_1, app-protect-container_elasticsearch_1, app-protect-container_app1_1, app-protect-container_approtect_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Recreating app-protect-container_approtect-nap-convertedpolicy_1 ... done
```
コンバート済みポリシーをインポートしたNAPが起動しているか確認
```
$ docker ps | grep approtect-nap-convertedpolicy
84b8084c2e62   app-protect:latest             "sh /root/entrypoint…"  8 minutes ago       Up 8 minutes       0.0.0.0:8002->80/tcp, :::8002->80/tcp    app-protect-container_approtect-nap-convertedpolicy_1    
```
正しくポリシーが読み込まれていることがわかる
```
$ docker logs app-protect-container_approtect-nap-convertedpolicy_1  2>&1 | grep policy
2021/06/15 14:45:22 [notice] 13#13: APP_PROTECT policy '/Common/Demo_NGINX_Policy' from: /etc/nginx/labpolicy.json compiled successfully
```





## Kibana操作画面
### Kibanaへの接続
LapTop PCからご覧頂く場合、以下メニューより「elk」をクリックしてください
<br><img src="https://user-images.githubusercontent.com/43058573/121283528-404a3280-c916-11eb-8a60-bfde13129dfc.png" alt="Docker Menu" width="200"><br>

Jump Hostからご覧頂く場合、以下URLに接続してください
```
http://10.1.1.5:5601/app/home#/
```

### Dashboard
左側メニューよりDashboardを選択し、ご覧になりたい内容を選択ください
<br><img src="https://user-images.githubusercontent.com/43058573/121031357-3aa0ff80-c7e5-11eb-828d-50d298736837.png" alt="Menu Dashboard" width="150"><br>
レポート選択画面
<br><img src="https://user-images.githubusercontent.com/43058573/121031356-3aa0ff80-c7e5-11eb-9f7e-9ec2b907592c.png" alt="Dashboard 選択画面" width="450"><br>
False Positive
<br><img src="https://user-images.githubusercontent.com/43058573/121031361-3b399600-c7e5-11eb-9c33-1c2231de21d1.png" alt="Dashboard Flase Positive" width="500"><br>
Overview
<br><img src="https://user-images.githubusercontent.com/43058573/121031341-383ea580-c7e5-11eb-9970-44736c1479ed.png" alt="Dashboard Overview" width="500"><br>
### Discover
左側メニューよりDiscoverを選択ください
<br><img src="https://user-images.githubusercontent.com/43058573/121031360-3b399600-c7e5-11eb-9fc7-8fa1b704c3ab.png" alt="Menu Discover" width="150"><br>
Discover
<br><img src="https://user-images.githubusercontent.com/43058573/121031345-396fd280-c7e5-11eb-9b46-2f8744ec11a1.png" alt="Discover" width="500"><br>
