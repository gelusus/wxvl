#  一次中风险漏洞处理复盘：从RSA密钥交换到全面启用TLSv1.3  
原创 弈秋
                    弈秋  一只岸上的鱼   2026-10-26 06:09  
  
# 一次中风险漏洞处理复盘：从RSA密钥交换到全面启用TLSv1.3  
## 缘起  
  
上周收到一份网安的扫描报告，有一个中风险漏洞需要处理，主要内容是：  
```
**目标主机**:支持RSA密钥交换【原理扫描】**详细描述**:目标主机支持 RSA 密钥交换意味着缺乏前向保密，存在历史流量解密风险，是过时且不推荐使用的配置，建议更新为 ECDHE 等临时密钥交换算法，以增强会话安全性和隐私保护。**解决办法**:禁用 RSA key exchange：只支持临时密钥交换算法（如 ECDHE_RSA 或 ECDHE_ECDSA）。强制 TLS 1.2 或 TLS 1.3，并优先选择支持前向保密的密码套件，比如：TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```  
  
记录一下分析处理过程  
## 分析  
  
这台主机上有一个nginx网关，主机在私网，是签名的证书，根据报告提示，应该是nginx启用了TLS 1.1 以及不支持前向保密的算法，解决办法就是修改nginx的配置，删除TLS1.2以下的支持，删除不支持前向保密的密码套件。  
## 基本概念  
  
网上搜了一下涉及的基本概念：  
### 为什么https也不安全了  
  
很多人第一反应是：“我的 HTTPS 不是好好的吗？证书也没过期，为什么突然就不安全了？”  
  
其实，问题不出在证书本身，而出在**握手阶段协商密钥**  
的方式上。在传统的配置中，如果我们允许使用 RSA 进行密钥交换，一旦服务器的私钥在未来某天泄露（无论是被黑客拖库还是内部失误），攻击者就可以利用这把私钥，解密过去所有被截获的加密流量。这就是所谓的"**先记录，后解密**  
"攻击。  
  
为了杜绝这种隐患，现代安全标准要求必须启用**前向保密**  
（Perfect Forward Secrecy, PFS）。而实现 PFS 的关键，就是抛弃静态的 RSA 密钥交换，转而使用**ECDHE**  
（椭圆曲线临时密钥交换）。  
### 什么是前向保密（PFS）  
  
前向保密是一种安全属性，它确保即使服务器的长期私钥（Long-term Private Key）泄露，也无法解密过去的会话数据。  
- ‎**没有 PFS**  
（如 RSA 密钥交换）：会话密钥是用服务器公钥加密传输的。私钥泄露 = 所有历史会话密钥可被还原 = 所有历史流量裸奔。  
  
- ‎**拥有 PFS**  
（如 ECDHE）：会话密钥是通过双方临时生成的参数计算出来的，计算完后临时私钥立即销毁。即使服务器长期私钥泄露，攻击者也无法推算出过去的会话密钥。  
  
### TLS版本的区别  
  
这里只说和本次安全相关的  
- ‎**TLS 1.1**  
：已淘汰，存在已知漏洞，不支持现代加密套件。  
  
- ‎**TLS 1.2**  
：目前主流，支持 RSA 和 ECDHE。 但是不强制ECDHE  
  
- ‎**TLS 1.3**  
：最新标准，**直接废除了 RSA 密钥交换**  
和不支持前向保密的算法。只要启用 TLS 1.3，天然具备前向保密性。  
  
## 解决方案  
  
鉴于本次主机是私网服务器，也只服务于自己的服务器，不考虑兼容性，所以直接只开启TLS 1.3  
  
**注意：**这里只是模拟了rsa 1.2版本环境，其他低版本环境一样  
### 测试demo  
  
这是验证测试的一个demo，基于docker环境打造：  
  
nginx docker:  
```
mkdir -p ./conf ./html ./logs ./conf.d ./ssltouch compose.yml
```  
  
nginx docker环境配置：  
```
services: nginx:   image: docker.1ms.run/nginx:latest   container_name: nginx   restart: always   ports:     - "443:443"   volumes:     - ./conf/nginx.conf:/etc/nginx/nginx.conf     - ./html:/usr/share/nginx/html     - ./logs:/var/log/nginx     - ./conf.d:/etc/nginx/conf.d     - ./ssl:/etc/nginx/ssl
```  
  
自签名证书：  
  
**注意：**模拟测试，只能使用rsa，使用ECDSA 证书会导致无法协商 RSA 密钥交换套件  
```
cd sslopenssl req -x509 -nodes -days 3650 -newkey rsa:2048 \  -keyout server.key \  -out server.crt \  -subj "/C=CN/ST=NanJing/L=NanJing/O=InternalOps/CN=internal.local" \  -addext "subjectAltName=DNS:internal.local,DNS:localhost,IP:127.0.0.1,IP:192.168.1.188"chmod 600 server.keychmod 644 server.crt
```  
  
nginx server关键配置：  
```
server {    listen 443 ssl;    server_name internal.local 192.168.1.188;    ssl_certificate /etc/nginx/ssl/server.crt;    ssl_certificate_key /etc/nginx/ssl/server.key;    ssl_protocols TLSv1.2;    ssl_ciphers AES128-GCM-SHA256:@SECLEVEL=1;    ssl_prefer_server_ciphers on;    location / {        root /usr/share/nginx/html;        index index.html;    }}
```  
  
测试：  
```
openssl s_client -connect 127.0.0.1:443 -tls1_2 -servername internal.local
```  
  
测试rsa 1.2版本结果：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/RJrNBTwulveVAEJiafFC6EX4xF74MWnqiavKichN0DrdghDLf5TkKldKvXNhtrgiaY9wIuyQqRMJNbzk07zBnqfkAIs30ZLTLl2wTy5ftjM8XK8/640?wx_fmt=png&from=appmsg "")  
  
## 改造只支持TLS1.3  
  
nginx 关键配置：  
```
server {    listen 443 ssl;    server_name internal.local 192.168.1.188;    ssl_certificate /etc/nginx/ssl/server.crt;    ssl_certificate_key /etc/nginx/ssl/server.key;	ssl_protocols TLSv1.3;	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;	ssl_conf_command Ciphersuites TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256;	ssl_prefer_server_ciphers on;    location / {        root /usr/share/nginx/html;        index index.html;    }}
```  
  
**注意**  
：当只配置1.3版本的时候，ssl_ciphers配置是不需要的，但是nginx本身会校验配置，所以这个字段必须保留  
```
# 测试1.3版本openssl s_client -connect 127.0.0.1:443 -tls1_3 -servername internal.local# 测试1.2版本openssl s_client -connect 127.0.0.1:443 -tls1_2 -servername internal.local
```  
  
1.3版本支持：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/RJrNBTwulveKlKz4knOsgic2YMqK2sWlX0j0KnAZjblgcrmeIcRico0lchbJ4ibiaAQnPlm0QbzbvH5Quagf2uIIkgfDicIop7hY8mmIsWcao3PM/640?wx_fmt=png&from=appmsg "")  
  
  
1.2版本不支持：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/RJrNBTwulvcCVZC8vF9ian9iay2xEtRm4UWGI4z6J4RRSvBibNral15KSqQ9xbDy1H9jBpl8nFm3BIc4jmgP9hv1zQhgZpGrJDuTck5XxxA5zk/640?wx_fmt=png&from=appmsg "")  
  
## 小结  
1. 精简配置，能不支持就不支持，原来的服务器是默认配置，从1.1开始，现在只给1.3  
  
1. 别太大意，前有log4j,后有php，谁知道哪天来个大问题  
  
  
  
  
