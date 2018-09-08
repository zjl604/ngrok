# ngrok 服务端部署
操作系统：centos7.2
1. go 环境配置
wget https://golang.org/doc/install?download=go1.9.2.linux-amd64.tar.gz
tar -C /usr/local/ -zxvf go1.9.2.linux-amd64.tar.gz
vim /etc/profile
#go lang
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin

2. ngrok server
 cd /usr/local/src & git clone https://github.com/inconshreveable/ngrok.git
 openssl genrsa -out rootCA.key 2048  
 openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=zjl.asia" -days 5000 -out rootCA.pem
 openssl genrsa -out device.key 2048
 openssl req -new -key device.key -subj "/CN=zjl.asia" -out device.csr
 openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
 yes|cp rootCA.pem assets/client/tls/ngrokroot.crt
 yes|cp device.crt assets/server/tls/snakeoil.crt
 yes|cp device.key assets/server/tls/snakeoil.key
 GOOS=linux GOARCH=amd64 make release-server
 
 ```
 [root@bz-op-ngrok ngrok]# ls
assets  contrib       device.crt  device.key  LICENSE   nohup.out  README.md   rootCA.pem  src
bin     CONTRIBUTORS  device.csr  docs        Makefile  pkg        rootCA.key  rootCA.srl
```

cd ./bin
./bin/ngrokd -tlsKey="assets/server/tls/snakeoil.key" -tlsCrt="assets/server/tls/snakeoil.crt" -domain="uboff.com"  -httpAd
dr=":80" -httpsAddr=":8081" -tunnelAddr=":4443"

nohup ./bin/ngrokd -tlsKey="assets/server/tls/snakeoil.key" -tlsCrt="assets/server/tls/snakeoil.crt" -domain="zjl.asia"  -h
ttpAddr=":80" -httpsAddr=":443" -tunnelAddr=":10000" &

ln -s /usr/local/src/ngrok/bin/ngrokd /usr/bin/ngrokd

编译客户端：
GOOS=linux GOARCH=amd64 make release-client

nohup ngrokd -tlsKey="/usr/local/src/ngrok/assets/server/tls/214349979280240.key" -tlsCrt="/usr/local/src/ngrok/assets/serv
er/tls/214349979280240.crt" -domain="startdt.net" -httpAddr=":80" -httpsAddr=":443" -tunnelAddr=":10000" &

# ngrok 客户端部署
1.上传编译好的 bin 文件  
2.配置文件
```
server_addr: "startdt.net:10000"
trust_host_root_certs: true

tunnels:
    ssh:
        remote_port: 10086
        proto:
            tcp: "192.168.0.220:2222"
    api:
        subdomain: localserver
        proto:
            https: "192.168.0.220:8001"
 ```

