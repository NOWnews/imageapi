# GO伺服器安裝

## **centos7 update**
```
yum check-update
yum update
```
## **安裝GVM**
```
yum group install "Development Tools"
```
```
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```
```
source /root/.gvm/scripts/gvm
```
先安裝golang1.4版為基底build go新版的底層
```
gvm install go1.4 -B
```

使用1.4版為底層

```
gvm use go1.4
```

```
export GOROOT_BOOTSTRAP=$GOROOT
```

再安裝golang新版

```
gvm install go1.8.3
```
```
gvm use go1.8.3 --default
```
## **更改TCP/IP 最大連線數**
```
vim /etc/security/limits.conf
```

最底下加入改變tcp/ip連線數與openfiles數量 修改完後reboot重啟

```
 * soft nofile 65536
 * hard nofile 65536
 * soft noproc 65536
 * hard noproc 65536
 
```


## **在.bashrc內增加GOPATH路徑**

```
[[ -s "/root/.gvm/scripts/gvm" ]] && source "/root/.gvm/scripts/gvm"
export GOPATH="/root/golang"
```

重新加載 .bashrc

```
source .bashrc
```

確認GOPATH是不是在指定的路徑下

```
echo $GOPATH
```

創建資料夾擺放原始碼


### 創建glang資料夾
```
mkdir golang

cd golang/


mkdir src

cd src/
```
### clone原始碼
```
git clone https://github.com/NOWnews/imageapi.git
```
### 啟動程式擺放的位置
```
cd imageapi/cmd/imageproxy/
```
### 安裝第三方套件
```
go get willnorris.com/go/imageproxy/third_party/http
```
### go build 執行檔
```
go build main.go
```

### 執行程式並將服務掛在8080port下 指定cache資料夾位置 設定白名單 並在背景執行
```
./main -addr 0.0.0.0:8080 -cache /tmp/imagelab -whitelist img.nownews.com,s.nownews.com,rssimg.nownews.com,e.now
news.com,legacy.nownews.com &
```
### 查看log是否有進來
```
tail -f /tmp/main.INFO
```
### 查看錯誤log 
```
tail -f /tmp/main.ERROR
```
### 查看process有沒有啟用
```
ps -ef | grep main
```
### 幹掉進程
```
kill -9 PID
```

# 安裝supervisor管理進程

### centos上安裝支援python安裝工具

```
yum install python-setuptools
```

### 安裝supervisor

```
easy_install supervisor
```

### 將config指向 要用的config

```
/usr/bin/supervisord -c /etc/supervisord.conf
```

### 修改supervisord config
```
vim supervisord.conf 
```

### 修改supervisord.conf 內容調整

```
[program:golang-http-server]
command=/root/golang/src/imageapi/cmd/imageproxy/main -addr 0.0.0.0:8877 -cache /tmp/imagelab -whitelist img.nownews.com,s.nownews.com,rssimg.nownews.com,e.nownews.com,legacy.nownews.com
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/var/log/imagelab.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/var/log/imagelab_err.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
```

### 啟用supervisor
```
supervisorctl
```

### 查看supervisor進程有沒有跑起來
```
ps -ef | grep supervisord
```

# 固定清理圖片cache
### shellscript

```
#/bin/bash
cmd=`find /tmp/imagelab -type f -mtime +2 -exec rm {} \;`
$cmd
```

### 排程修改
```
crontab -e 
```

### 排程修改內容
```
0 1 * * * /usr/local/golang_service/clean_image_cache.sh
```

                                                          

