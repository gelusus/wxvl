#  CNNVD关于PHP 资源管理错误漏洞的通报  
原创 CNNVD
                    CNNVD  CNNVD安全动态   2026-10-22 02:31  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/g1thw9GoocfpeKv1eicF4icEx1vUX4LQ1JjlMnGl5z2XiaAQGZdFulYs0vsE3icB8RUiawPqDSb5lvm8G0drb7iaw7sQ/640?wx_fmt=gif&from=appmsg "")  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/g1thw9GoocfpeKv1eicF4icEx1vUX4LQ1Js3VkKswpUtkoDWibZ1YQl1lIdcctfqePCcSPEdc38SnhJGdqGJUFx9w/640?wx_fmt=gif&from=appmsg "")  
  
**点击蓝字 关注我们**  
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/g1thw9GoocfpeKv1eicF4icEx1vUX4LQ1Js3VkKswpUtkoDWibZ1YQl1lIdcctfqePCcSPEdc38SnhJGdqGJUFx9w/640?wx_fmt=gif&from=appmsg "")  
  
  
**漏洞情况**  
  
近日，国家信息安全漏洞库（CNNVD）收到关于PHP 资源管理错误漏洞（CNNVD-2026-75300632、CVE-2026-109618）（CNNVD-2026-41545346、CVE-2026-109619）情况的报送。成功利用漏洞的攻击者，可通过构造恶意 HTTP 请求实现内存泄露，进而远程执行代码。PHP多个版本均受此漏洞影响。目前，PHP官方已发布新版本修复了该漏洞，建议用户及时确认产品版本，尽快采取修补措施。  
  
## 一漏洞介绍  
  
  
PHP是PHP团队开源的一种在服务器端执行的脚本语言。PHP存在资源管理错误漏洞，其中CNNVD-2026-75300632、CVE-2026-109618）源于使用longjmp()处理内存时错误，导致请求内存在分配前处于未初始化状态。（CNNVD-2026-41545346、CVE-2026-109619）源于  
PHP  
在全局状态请求结束后未正确重置，使后续请求在正常初始化之前访问已释放的请求内存。未经认证的攻击者可利用漏洞，通过构造恶意 HTTP 请求实现内存泄露，进而远程执行代码。  
  
## 二危害影响      
  
  
PHP 8.2.35版本之前的8.2.x版本、8.3.36版本之前的8.3.x版本、8.4.27版本之前的8.4.x版本和8.5.12版本之前的8.5.x版本均受此漏洞影响。  
  
## 三修复建议  
  
  
目前，PHP官方已发布新版本修复了该漏洞，建议用户及时确认产品版本，尽快采取修补措施。  
  
官方参考链接：https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79whttps://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
  
   
  
本通报由CNNVD技术支撑单位——杭州安恒信息技术股份有限公司、北京长亭科技有限公司、奇安信网神信息技术（北京）股份有限公司、零熵科技（甘肃）有限公司、南京禾盾信息科技有限公司、北京山石网科信息技术有限公司、浪潮电子信息产业股份有限公司、北京容辉智信科技有限公司、深信服科技股份有限公司、北京国御科技有限公司、擎境（日照）网络信息科技有限责任公司、山东广迪信息科技有限公司、北京自然原数科技有限公司、上海戎磐网络科技有限公司、广州纬安科技有限公司、北京威努特技术有限公司、建信金融科技有限责任有限公司、北京天融信网络安全技术有限公司、超聚变数字技术股份有限公司、青鸾信息技术（青海）有限公司、中国银联股份有限公司等技术支撑单位提供支持。     
  
CNNVD将继续跟踪上述漏洞的相关情况，及时发布相关信息。如有需要，可与CNNVD联系。联系方式: cnnvd@itsec.gov.cn      
  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/g1thw9GoocfpeKv1eicF4icEx1vUX4LQ1JMd8aMOqNkic25xydKvYcCVEsHXvm506icfXiaFep4AfohjraUj3F2jMfg/640?wx_fmt=gif&from=appmsg "")  
  
  
