#  Nginx文件上传绕过之文件扩展名解析漏洞  
原创 晨星安全团队
                    晨星安全团队  晨星安全团队   2026-11-03 08:47  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/btLkCxjz5dLiahbEBkic8hMU78WfeeQ3QsBmFXQZ1y8JsQGPEeKI0ELqhk6edQUoy4STjSo1Cy9WYPoaPGfySUdhqpskiaknibpRJntXLbknialw/640?wx_fmt=png "")  
  
Nginx文件上传绕过之  
  
文件扩展名解析漏洞  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/DvLO5GxGnazyBIAvakaCexo0nSYNyFv4IbWsROD1dHtj6aibm0ec5przPe3z2ND6ycYHQHarZmhya4oRaswCkZ5ATlelMVd7nbg90ibFsbzos/640?wx_fmt=png "")  
  
  
Autumn.  
  
  
  
  
该漏洞存在的原因是 Nginx 在配置中对文件扩展名的处理方式  
  
当请求一个具有多个扩展名的文件（如 file.jpg.php）时， Nginx 可能会根据最后一个扩展名来处理，即使该文件最初是作为图片上传的  
  
这种配置错误通常出现在以下 Nginx 配置中  
  
  
  
当请求   
file.jpg/.php  
 时，Nginx 会将其作为 PHP 文件处理并发送给 PHP-FPM 解析，尽管它实际上是一个图片文件  
  
这种行为可以被利用来执行作为图片文件上传的恶意代码  
  
访问  
http://your-ip/uploadfiles/nginx.png  
 和  
http://your-ip/uploadfiles/nginx.png/.php  
即可  
  
查看效果  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZCjMVth9QjgibwnK0B3DaSLLtUy4zSFT0IdNq0NbuekygGFGh5vdoIDnJucPbvXhIKh9mibVBibl7YjhnzicWSAR5mT2h2QJVA18I3aXEuSibIvM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZCjMVth9QjgNf18Du4HfLqRg2iaklqcttiaLFic5W5C1nRKm6icySRRPMtw24ziaxUXhxoaRTNQ1DXbWIIlg6EXicorcJjZYoWOaWdpD5KTiaqicNlU/640?wx_fmt=png&from=appmsg "")  
  
访问  
**http://your-ip/index.php**  
可以测试上传功能，上传代码不存在漏洞，但利用解析漏洞即可 GetShell  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZCjMVth9QjjoHztqYkJUxcFrWuicUNCCQjwOruIiaodlOGab6VpL7Q3rLTs319Ev24YxSRaIkuQKE6FLldUibTVjIg7y1JSzYJic7r0R2KhZHeM/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/ZCjMVth9QjglXqodIRgwXlfprB7XYaE2ibJvGsBgmJnLn3aajKs3fFN4q0icZLKr0KNfnuBibiaibfVweZppQ41wLTTMAzaRzY7ogU1E7gPnrZgY/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/ZCjMVth9QjiaXozoKkDTgrfq9ibuH2IwDghpAXpvcIolW14uuAUjGHIwHm6lyIxjajXYSiah41TqI9YRwr8pd3mG0lsyAcuuw8pv3GNjlLbnM0/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/bVMOGKMcXo0myO73CK2xJRPrGeH0icUJkarial09M3Swb54mzHU0W0I9WnibTOiayApEeap8CXErPdyziaT4UFhGyJg/640?wx_fmt=png "")  
  
  
**修复方法**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/btLkCxjz5dITq3icaKaooxeNgq64POe3jHib0vQpg8KnbDbBkvghCDkYiahU7m25vFuPnHsDiaq5AC7WVPnGgTI3dauaJoPdJib3e2cDlU3qEicUI/640?wx_fmt=png "")  
  
  
1. 修改 php.ini，设置   
cgi.fix_pathinfo = 0  
 ，然后重启 php-cgi（会影响到 pathinfo 伪静态应用）  
  
1. 在Nginx的配置文件添加如下内容后重启   
if ( $fastcgi_script_name ~ \..*\/.*php ) {return 403;}   
（会影响 /5.0/test.php 的形式）  
  
1. 使用 OSS 存储桶（目前最好的方法）  
  
-END-  
  
